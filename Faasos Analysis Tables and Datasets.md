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
```
### Table: `customer_orders`

| Column Name          | Data Type      | Description                                    |
|----------------------|----------------|------------------------------------------------|
| `order_id`           | `integer`      | Unique identifier for the customer order      |
| `customer_id`        | `integer`      | Unique identifier for the customer            |
| `roll_id`            | `integer`      | Unique identifier for the roll ordered        |
| `not_include_items`  | `varchar(4)`    | Comma-separated list of items not included    |
| `extra_items_included`| `varchar(4)`   | Comma-separated list of extra items included  |
| `order_date`         | `datetime`     | Date and time when the order was placed       |

### Insert Data:

```sql
INSERT INTO customer_orders(order_id, customer_id, roll_id, not_include_items, extra_items_included, order_date)
VALUES 
    (1, 101, 1, '', '', '01-01-2021 18:05:02'),
    (2, 101, 1, '', '', '01-01-2021 19:00:52'),
    (3, 102, 1, '', '', '01-02-2021 23:51:23'),
    (3, 102, 2, '', 'NaN', '01-02-2021 23:51:23'),
    (4, 103, 1, '4', '', '01-04-2021 13:23:46'),
    (4, 103, 1, '4', '', '01-04-2021 13:23:46'),
    (4, 103, 2, '4', '', '01-04-2021 13:23:46'),
    (5, 104, 1, null, '1', '01-08-2021 21:00:29'),
    (6, 101, 2, null, null, '01-08-2021 21:03:13'),
    (7, 105, 2, null, '1', '01-08-2021 21:20:29'),
    (8, 102, 1, null, null, '01-09-2021 23:54:33'),
    (9, 103, 1, '4', '1,5', '01-10-2021 11:22:59'),
    (10, 104, 1, null, null, '01-11-2021 18:34:49'),
    (10, 104, 1, '2,6', '1,4', '01-11-2021 18:34:49');
