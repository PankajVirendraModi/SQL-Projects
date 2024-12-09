```sql

### Table: `driver`

| Column Name | Data Type | Description                        |
|-------------|-----------|------------------------------------|
| `driver_id` | `integer` | Unique identifier for the driver  |
| `reg_date`  | `date`    | Registration date of the driver   |

### Insert Data:

```sql
INSERT INTO driver(driver_id, reg_date) 
VALUES 
    (1, '01-01-2021'),
    (2, '01-03-2021'),
    (3, '01-08-2021'),
    (4, '01-15-2021');
```

### Table: `ingredients`

| Column Name     | Data Type  | Description                          |
|-----------------|------------|--------------------------------------|
| `ingredients_id`| `integer`  | Unique identifier for the ingredient |
| `ingredients_name` | `varchar(60)` | Name of the ingredient         |

### Insert Data:

```sql
INSERT INTO ingredients(ingredients_id, ingredients_name) 
VALUES 
    (1, 'BBQ Chicken'),
    (2, 'Chilli Sauce'),
    (3, 'Chicken'),
    (4, 'Cheese'),
    (5, 'Kebab'),
    (6, 'Mushrooms'),
    (7, 'Onions'),
    (8, 'Egg'),
    (9, 'Peppers'),
    (10, 'Schezwan Sauce'),
    (11, 'Tomatoes'),
    (12, 'Tomato Sauce');
```

### Table: `rolls`

| Column Name  | Data Type  | Description                         |
|--------------|------------|-------------------------------------|
| `roll_id`    | `integer`  | Unique identifier for the roll     |
| `roll_name`  | `varchar(30)` | Name of the roll                |

### Insert Data:

```sql
INSERT INTO rolls(roll_id, roll_name) 
VALUES 
    (1, 'Non Veg Roll'),
    (2, 'Veg Roll');
```

### Table: `rolls_recipes`

| Column Name  | Data Type  | Description                        |
|--------------|------------|------------------------------------|
| `roll_id`    | `integer`  | Unique identifier for the roll    |
| `ingredients`| `varchar(24)` | Comma-separated list of ingredient IDs for the roll's recipe |

### Insert Data:

```sql
INSERT INTO rolls_recipes(roll_id, ingredients) 
VALUES 
    (1, '1,2,3,4,5,6,8,10'),
    (2, '4,6,7,9,11,12');
```

### Table: `driver_order`

| Column Name    | Data Type    | Description                          |
|----------------|--------------|--------------------------------------|
| `order_id`     | `integer`    | Unique identifier for the order     |
| `driver_id`    | `integer`    | Unique identifier for the driver    |
| `pickup_time`  | `datetime`   | Time when the order was picked up   |
| `distance`     | `varchar(7)` | Distance covered during the order   |
| `duration`     | `varchar(10)`| Duration of the order ride          |
| `cancellation` | `varchar(23)`| Cancellation status of the order    |

### Insert Data:

```sql
INSERT INTO driver_order(order_id, driver_id, pickup_time, distance, duration, cancellation) 
VALUES 
    (1, 1, '01-01-2021 18:15:34', '20km', '32 minutes', ''),
    (2, 1, '01-01-2021 19:10:54', '20km', '27 minutes', ''),
    (3, 1, '01-03-2021 00:12:37', '13.4km', '20 mins', 'NaN'),
    (4, 2, '01-04-2021 13:53:03', '23.4', '40', 'NaN'),
    (5, 3, '01-08-2021 21:10:57', '10', '15', 'NaN'),
    (6, 3, null, null, null, 'Cancellation'),
    (7, 2, '01-08-2021 21:30:45', '25km', '25mins', null),
    (8, 2, '01-10-2021 00:15:02', '23.4 km', '15 minute', null),
    (9, 2, null, null, null, 'Customer Cancellation'),
    (10, 1, '01-11-2021 18:50:20', '10km', '10minutes', null);

select * from customer_orders;

-- A. roll metrics
-- 1. How many rolls were ordered?
select count(roll_id) total_rolls_ordered from customer_orders;

-- 2. How many unique customer orders were made?
select count(distinct customer_id) unique_customers from customer_orders;

-- 3. How many successful orders were delivered by each driver?
select driver_id, count(distinct order_id) successful_orders
from driver_order
where cancellation not in ('Cancellation','Customer Cancellation')
group by driver_id;

-- 4. How many each type of roll was delivered?
select roll_id, count(roll_id) rolls_count from customer_orders where order_id in(
select order_id from (
select *,
	case when cancellation in ('Cancellation','Customer Cancellation') then 'c'
	else 'nc' end as order_cancellation_details
from driver_order) a
where order_cancellation_details='nc')
group by roll_id;

-- 5. How many veg and non-veg rolls were ordered by each customer?
select co.customer_id, rolls.roll_name, co.roll_id, count(co.roll_id) rolls_count
from customer_orders co join rolls on co.roll_id=rolls.roll_id
group by co.customer_id, rolls.roll_name, co.roll_id;

-- 6. What was the maximum number of rolls delivered in a single order?
select *, rank() over(order by cnt desc) rnk from (
	select order_id, count(roll_id) cnt from (
		select * from customer_orders
		where order_id in (
			select order_id from (
				select *,
				case when cancellation in ('Cancellation','Customer Cancellation') then 'c'
				else 'nc' end as order_cancellation_details
				from driver_order) a
			where order_cancellation_details='nc')) b
	group by order_id)c
order by rnk
offset 0 rows fetch next 1 row only;



-- temperory customer order table
with temp_customer_orders as (
SELECT order_id, customer_id, roll_id,
    CASE 
        WHEN TRY_CAST(not_include_items AS INT) IS NULL THEN 0
        ELSE not_include_items
    END AS not_include_items,
	CASE 
        WHEN TRY_CAST(extra_items_included AS INT) IS NULL THEN 0
        ELSE extra_items_included
    END AS extra_items_included, order_date
FROM customer_orders)
select * from temp_customer_orders;

-- temperory driver order table
with temp_drive_order as  (
select order_id, driver_id, pickup_time, distance, duration,
	case when cancellation in ('Cancellation','Customer Cancellation') then 0
	else 1 end as cancellation from driver_order
)
select * from temp_drive_order;

-- 7. for each customer, how many delivered rolls had at least change and how many had no changes?
with temp_customer_orders as (
SELECT order_id, customer_id, roll_id,
    CASE 
        WHEN not_include_items IS NULL or not_include_items=' ' THEN '0'
        ELSE not_include_items
    END AS not_include_items,
	CASE 
        WHEN extra_items_included IS NULL or extra_items_included=' ' or extra_items_included='NaN' THEN '0'
        ELSE extra_items_included
    END AS extra_items_included, order_date
FROM customer_orders),
temp_drive_order as  (
select order_id, driver_id, pickup_time, distance, duration,
	case when cancellation in ('Cancellation','Customer Cancellation') then 0
	else 1 end as cancellation from driver_order
)
select customer_id, change_no_change, count(customer_id) atleast_one_change from (
select *, case when not_include_items='0' and extra_items_included='0' then 'no change' else 'change' end change_no_change
from temp_customer_orders where order_id in (
select order_id from temp_drive_order where cancellation<>0)) a
group by customer_id, change_no_change;

-- 8. How many rolls were delivered that had both exclusions and extras?
with temp_customer_orders as (
SELECT order_id, customer_id, roll_id,
    CASE 
        WHEN not_include_items IS NULL or not_include_items=' ' THEN '0'
        ELSE not_include_items
    END AS not_include_items,
	CASE 
        WHEN extra_items_included IS NULL or extra_items_included=' ' or extra_items_included='NaN' THEN '0'
        ELSE extra_items_included
    END AS extra_items_included, order_date
FROM customer_orders),
temp_drive_order as  (
select order_id, driver_id, pickup_time, distance, duration,
	case when cancellation in ('Cancellation','Customer Cancellation') then 0
	else 1 end as cancellation from driver_order
)
select either_both_or_not, count(either_both_or_not) count_both_excluded_included from (
select *, case when not_include_items<>'0' and extra_items_included<>'0' then 'both_included_excluded' else 'either_included_or_excluded' end either_both_or_not
from temp_customer_orders where order_id in (
select order_id from temp_drive_order where cancellation<>0)) a
group by either_both_or_not;

-- 9. what was the total number of rolls ordered for each hour of the day?
select hours_bucket, count(hours_bucket) as total_rolls_ordered from (
select *, concat(datepart(hour, order_date),'-',(datepart(hour, order_date)+1)) hours_bucket
from customer_orders) a
group by hours_bucket;

-- 10. What is the total number of orders for each day of the week?

select day_of_week, count(distinct order_id) number_of_orders from (
select *, datename(weekday, order_date) day_of_week from customer_orders) a
group by day_of_week;


select * from customer_orders;
select * from driver_order;
select * from ingredients;
select * from driver;
select * from rolls;
select * from rolls_recipes;

-- B. Driver and Customer Experience
-- 1. What was the average time in minutes it took for each driver to arrive at the fasoos HQ to pickup the order?
select driver_id, sum(total_minutes)/count(order_id) avg_time_taken_by_drivers from (
SELECT do.driver_id, co.order_id, ROW_NUMBER() OVER (PARTITION BY co.order_id ORDER BY DATEDIFF(MINUTE, co.order_date, do.pickup_time)) AS rnk,
	DATEDIFF(MINUTE, co.order_date, do.pickup_time) AS total_minutes 
FROM customer_orders co 
INNER JOIN driver_order do ON co.order_id = do.order_id 
WHERE do.pickup_time IS NOT NULL) as a where rnk =1
group by driver_id;

-- 2. Is there any relationship between the number of rolls and how long the order takes to prepare?
select order_id, count(roll_id) count_of_rolls, sum(total_minutes)/count(roll_id) total_time_taken from (
SELECT co.roll_id, co.customer_id, do.driver_id, co.order_id, ROW_NUMBER() OVER (PARTITION BY co.order_id ORDER BY DATEDIFF(MINUTE, co.order_date, do.pickup_time)) AS rnk,
	DATEDIFF(MINUTE, co.order_date, do.pickup_time) AS total_minutes 
FROM customer_orders co 
INNER JOIN driver_order do ON co.order_id = do.order_id 
WHERE do.pickup_time IS NOT NULL) as a
group by order_id;


-- 3. what was the average distance travelled for each customer?
select customer_id, cast(sum(distance)/count(order_id) as decimal(10,2)) avg_distance from (
select * from (
SELECT do.driver_id, co.order_id, co.customer_id,
cast(trim(replace(lower(do.distance),'km','')) as decimal(10,2)) distance,
ROW_NUMBER() OVER (PARTITION BY co.order_id ORDER BY DATEDIFF(MINUTE, co.order_date, do.pickup_time)) AS rnk
FROM customer_orders co
INNER JOIN driver_order do ON co.order_id = do.order_id 
WHERE do.pickup_time IS NOT NULL) as a where rnk =1) c
group by customer_id
;

-- 4. what was the difference between the longest and shortest delivery times for all orders?
-- select charindex('m', duration) from driver_order;

select (max(cleaned_duration)-min(cleaned_duration)) as [difference] from(
select order_id, driver_id, duration,
cast(case when duration like '%min%' then trim(left(duration, charindex('m', duration)-1)) else duration end as integer) as cleaned_duration
from driver_order where duration is not null) a;


-- 5.what was the average speed for each driver for each delivery and do you notice any trend for these values?
-- speed = distance/time
select a.order_id, a.driver_id, (a.distance/a.cleaned_duration) speed, b.cnt from(
select order_id, driver_id, cast(trim(replace(lower(distance),'km','')) as decimal(10,2)) distance,
	cast(case when duration like '%min%' then trim(left(duration, charindex('m', duration)-1)) else duration end as integer) as cleaned_duration
from driver_order where duration is not null) a inner join
(select order_id, count(roll_id) cnt from customer_orders group by order_id) b on a.order_id=b.order_id;

-- 6. what is the successful delivery percentage for each driver?

select driver_id, concat((successful_delivery*100)/total_orders_takes , '%') successful_delivery_percentage from(
select driver_id, sum(cancellation_percentage) successful_delivery, count(driver_id) total_orders_takes from(
select driver_id, case when cancellation like '%cancel%' then 0 else 1 end as cancellation_percentage from driver_order) a
group by driver_id) b;
```
