# Slowly Changing Dimensions (SCD) Implementation

This project contains the implementation of Slowly Changing Dimensions (SCD) of types 1, 2, and 3 in SQL. These types are used in data warehousing to manage and store historical data over time. I used *ORACLE SQL DEVELOPER*

## Overview of SCD Types

- **SCD Type 1**: Overwrite the existing data with new values. No history is preserved.
- **SCD Type 2**: Create new records to preserve full historical data, with the addition of start and end date columns to indicate the period of validity.
- **SCD Type 3**: Store limited historical data, typically keeping only the previous value alongside the current value.

##### TO DISPLAY THE OUTPUT OF DBMS_OUTPUT.PUT_LINE('CAN BE ANY STRING')
```SQL
SET SERVEROUTPUT ON;
```
## SCD Type-1

### Product Stage Table (`PRODUCT_STG`)

This table holds information about products in the staging area. It includes product details such as `PRODUCT_ID`, `PRODUCT_NAME`, and `PRICE`.

### Table Structure

| Column Name   | Data Type | Description                          |
|---------------|-----------|--------------------------------------|
| `PRODUCT_ID`  | `INT`     | Unique identifier for each product   |
| `PRODUCT_NAME`| `VARCHAR` | Name of the product                  |
| `PRICE`       | `INT`     | Price of the product in currency     |

## Sample Data

| PRODUCT_ID | PRODUCT_NAME | PRICE |
|------------|--------------|-------|
| 1          | Mouse        | 800   |
| 2          | Keyboard     | 1300  |

```SQL
CREATE TABLE PRODUCT_STG
(
PRODUCT_ID INT PRIMARY KEY,
PRODUCT_NAME VARCHAR(55) NOT NULL,
PRICE INT NOT NULL );

INSERT INTO PRODUCT_STG VALUES
(1,'Mouse',800);
INSERT INTO PRODUCT_STG VALUES
(2,'Keyboard',1300);
--INSERT INTO product_stg VALUES (4,'CPU_3',2300);
--INSERT INTO product_stg VALUES (2,'Keyboard_N',1150);
```
### Product Target Table (`PRODUCT_TARGET`)

This table holds information about products in the target database. It includes product details such as `PRODUCT_ID`, `PRODUCT_NAME`, `PRICE`, and `LAST_UPDATED` date.

### Table Structure

| Column Name   | Data Type     | Description                                    |
|---------------|---------------|------------------------------------------------|
| `PRODUCT_ID`  | `INT`         | Unique identifier for each product             |
| `PRODUCT_NAME`| `VARCHAR(55)` | Name of the product                            |
| `PRICE`       | `INT`         | Price of the product in currency               |
| `LAST_UPDATED`| `DATE`        | Date when the product information was last updated |

## Sample Data

| PRODUCT_ID | PRODUCT_NAME | PRICE | LAST_UPDATED  |
|------------|--------------|-------|---------------|
| 1          | Mouse        | 1000  | 2024-01-15    |
| 3          | Monitor      | 6700  | 2024-01-21    |

```SQL
CREATE TABLE PRODUCT_TARGET
(
PRODUCT_ID INT NOT NULL,
PRODUCT_NAME VARCHAR(55) NOT NULL,
PRICE INT NOT NULL,
LAST_UPDATED DATE );

INSERT INTO PRODUCT_TARGET VALUES
(1,'Mouse',1000,TO_DATE('2024-01-15', 'yyyy-mm-dd'));
INSERT INTO PRODUCT_TARGET VALUES
(3,'Monitor',6700,TO_DATE('2024-01-21', 'yyyy-mm-dd'));


-- SELECT * FROM PRODUCT_STG;
-- SELECT * FROM PRODUCT_TARGET;
-- TRUNCATE TABLE PRODUCT_STG;
-- TRUNCATE TABLE PRODUCT_TARGET;
```
```SQL
CREATE OR REPLACE PROCEDURE SCD_ONE
IS
  TODAYS_DATE DATE := SYSDATE;
BEGIN
  MERGE INTO PRODUCT_TARGET TARGET
  USING PRODUCT_STG SOURCE
  ON (TARGET.PRODUCT_ID = SOURCE.PRODUCT_ID)
  WHEN MATCHED THEN
    UPDATE SET TARGET.PRICE = SOURCE.PRICE,
            TARGET.PRODUCT_NAME = SOURCE.PRODUCT_NAME,
               TARGET.LAST_UPDATED = TODAYS_DATE
    WHERE TARGET.PRICE <> SOURCE.PRICE OR TARGET.PRODUCT_NAME <> SOURCE.PRODUCT_NAME
  WHEN NOT MATCHED THEN
    INSERT (TARGET.PRODUCT_ID, TARGET.PRODUCT_NAME, TARGET.PRICE, TARGET.LAST_UPDATED)
    VALUES (SOURCE.PRODUCT_ID, SOURCE.PRODUCT_NAME, SOURCE.PRICE, TODAYS_DATE);
  DELETE FROM PRODUCT_STG; -- Be cautious with TRUNCATE, it's irreversible
  DBMS_OUTPUT.PUT_LINE('Successfully implemented SCD 1');
END;
```

```SQL
-- Execute the procedure SCD_ONE
EXEC SCD_ONE;
```

## SCD Type-2

### Employee Staging Table (`EMP_STG`)

This table holds information about employees in the staging area. It includes details such as `EMP_ID`, `EMP_NAME`, `EMP_LOCATION`, `EMP_SALARY`, and the `UPDATEDON` date.

### Table Structure

| Column Name   | Data Type     | Description                                    |
|---------------|---------------|------------------------------------------------|
| `EMP_ID`      | `INT`         | Unique identifier for each employee            |
| `EMP_NAME`    | `VARCHAR(100)`| Name of the employee                           |
| `EMP_LOCATION`| `VARCHAR(50)` | Location of the employee                       |
| `EMP_SALARY`  | `INT`         | Salary of the employee                         |
| `UPDATEDON`   | `DATE`        | Date when the employee's data was last updated |

### Sample Data

| EMP_ID | EMP_NAME       | EMP_LOCATION | EMP_SALARY | UPDATEDON   |
|--------|----------------|--------------|------------|-------------|
| 1001   | Rahul Sharma   | Bangalore    | 45000      | 2023-05-18  |
| 1003   | Shilpa Gaur    | Pune         | 37000      | 2023-06-01  |
| 1004   | Shradha Gunjan | Pune         | 33000      | 2023-02-23  |

```SQL
CREATE TABLE EMP_STG
(EMP_ID INT PRIMARY KEY,
EMP_NAME VARCHAR(100) NOT NULL,
EMP_LOCATION VARCHAR(50) NOT NULL,
EMP_SALARY INT,
UPDATEDON DATE NOT NULL);
INSERT INTO EMP_STG
    VALUES (1001,'Rahul Sharma','Bangalore',45000,TO_DATE('2023-05-18','YYYY-MM-DD'));
INSERT INTO EMP_STG
    VALUES (1003,'Shilpa Gaur','Pune',37000,TO_DATE('2023-06-01','YYYY-MM-DD'));
INSERT INTO EMP_STG
    VALUES (1004,'Shradha Gunjan','Pune',33000,TO_DATE('2023-02-23','YYYY-MM-DD'));
```

### Sequence Purpose:

- The sequence `EMP_TARGET_SEQ` is primarily used for generating unique `DIM_ID` values in the `EMP_TARGET` table.
- When a new record is inserted into the `EMP_TARGET` table, the `DIM_ID` column is populated automatically by the sequence to ensure uniqueness.

```SQL
-- CREATING A SEQUENCE TO USE IN EMP_TARGET TABLE WITH HELP OF TRIGGER
CREATE SEQUENCE EMP_TARGET_SEQ
  START WITH 1
  INCREMENT BY 1
  NOCACHE;
```

### Employee Target Table (`EMP_TARGET`)

This table holds information about employees in the target database, including historical records for tracking employment details over time. The table contains the employee's basic information, employment period, and active status.

### Table Structure

| Column Name  | Data Type     | Description                                                    |
|--------------|---------------|----------------------------------------------------------------|
| `DIM_ID`     | `INT`         | Unique identifier for each dimension record (Primary Key)      |
| `EMP_ID`     | `INT`         | Unique identifier for the employee                             |
| `EMP_NAME`   | `VARCHAR(100)`| Name of the employee                                           |
| `EMP_LOCATION`| `VARCHAR(50)`| Location of the employee                                       |
| `EMP_SALARY` | `INT`         | Salary of the employee                                         |
| `STARTDATE`  | `DATE`        | Start date when the employee record becomes valid              |
| `ENDDATE`    | `DATE`        | End date when the employee record is no longer valid (Optional)|
| `ACTIVE_FLAG`| `INT`         | Flag to indicate if the record is active (1 = Active, 0 = Inactive) |

**Note**: The `ENDDATE` column defaults to `'9999-12-31'`, which indicates the employee record is currently active.

### Sample Data

| DIM_ID | EMP_ID | EMP_NAME       | EMP_LOCATION | EMP_SALARY | STARTDATE   | ENDDATE     | ACTIVE_FLAG |
|--------|--------|----------------|--------------|------------|-------------|-------------|-------------|
| 1      | 1001   | Rahul Sharma   | Bangalore    | 45000      | 2023-05-18  | 9999-12-31  | 1           |
| 2      | 1003   | Shilpa Gaur    | Pune         | 37000      | 2023-06-01  | 9999-12-31  | 1           |
| 3      | 1004   | Shradha Gunjan | Pune         | 33000      | 2023-02-23  | 9999-12-31  | 1           |

### Notes:
- The `STARTDATE` is the date when the employee's record starts being valid.
- The `ENDDATE` is set to `'9999-12-31'` by default, which is used to indicate that the employee record is currently active. If the employee is no longer active, the `ENDDATE` will be updated with the date the employee left the company.
- The `ACTIVE_FLAG` indicates whether the employee is currently active (`1` for active, `0` for inactive).
```SQL
CREATE TABLE EMP_TARGET
(
DIM_ID INT PRIMARY KEY,
EMP_ID INT NOT NULL,
EMP_NAME VARCHAR(100) NOT NULL,
EMP_LOCATION VARCHAR(50) NOT NULL,
EMP_SALARY INT,
STARTDATE DATE NOT NULL,
ENDDATE DATE, -- DEFAULT TO_DATE('9999-12-31','YYYY-MM-DD')
ACTIVE_FLAG INT NOT NULL CHECK(ACTIVE_FLAG IN (0,1)));
```

### Trigger: `TRG_EMP_TARGET_INSERT`

The `TRG_EMP_TARGET_INSERT` trigger is designed to automatically populate the `DIM_ID` column in the `EMP_TARGET` table when a new record is inserted. It ensures that the `DIM_ID` is generated using the `EMP_TARGET_SEQ` sequence.

### Trigger Purpose:

- The trigger fires **before** each insert operation on the `EMP_TARGET` table.
- It automatically generates the `DIM_ID` using the `EMP_TARGET_SEQ.NEXTVAL` sequence and assigns it to the `DIM_ID` column.
- This removes the need to manually specify the `DIM_ID` value when inserting new records into the table.

### Trigger Logic

When a new record is inserted into the `EMP_TARGET` table, the following actions occur:
1. The trigger fires before the insert operation.
2. It generates a unique value for the `DIM_ID` column by selecting the next value from the `EMP_TARGET_SEQ` sequence.
3. The generated value is assigned to the `DIM_ID` column in the new row.

```SQL
-- Create a Trigger to Automatically Populate the dim_id
CREATE OR REPLACE TRIGGER TRG_EMP_TARGET_INSERT
BEFORE INSERT ON EMP_TARGET
FOR EACH ROW
BEGIN
  SELECT EMP_TARGET_SEQ.NEXTVAL INTO :NEW.DIM_ID FROM DUAL;
END;
```
```SQL
INSERT INTO EMP_TARGET(EMP_ID, EMP_NAME, EMP_LOCATION, EMP_SALARY, STARTDATE, ENDDATE, ACTIVE_FLAG)
    VALUES (1001,'Rahul Sharma','Pune',29000,TO_DATE('2021-08-22','YYYY-MM-DD'),NULL,1);
INSERT INTO EMP_TARGET(EMP_ID, EMP_NAME, EMP_LOCATION, EMP_SALARY, STARTDATE, ENDDATE, ACTIVE_FLAG)
    VALUES (1002,'Kunal Nath','Pune',32000,TO_DATE('2020-09-10','YYYY-MM-DD'),NULL,1);
INSERT INTO EMP_TARGET(EMP_ID, EMP_NAME, EMP_LOCATION, EMP_SALARY, STARTDATE, ENDDATE, ACTIVE_FLAG)
    VALUES(1003,'Shilpa Gaur','Gurgaon',25900,TO_DATE('2022-01-27','YYYY-MM-DD'),NULL,1);


-- SELECT * FROM EMP_STG;
-- SELECT * FROM EMP_TARGET;
-- TRUNCATE TABLE EMP_STG;
-- TRUNCATE TABLE EMP_TARGET;
```
```SQL
CREATE OR REPLACE PROCEDURE SCD_TWO
AS
BEGIN
INSERT INTO EMP_TARGET(EMP_ID, EMP_NAME, EMP_LOCATION, EMP_SALARY, STARTDATE, ENDDATE, ACTIVE_FLAG)
SELECT SOURCE.EMP_ID, SOURCE.EMP_NAME, SOURCE.EMP_LOCATION, SOURCE.EMP_SALARY, SOURCE.UPDATEDON, NULL, 1
FROM EMP_STG SOURCE JOIN EMP_TARGET TARGET ON SOURCE.EMP_ID=TARGET.EMP_ID
WHERE SOURCE.EMP_LOCATION <> TARGET.EMP_LOCATION OR SOURCE.EMP_SALARY<>TARGET.EMP_SALARY;
MERGE INTO EMP_TARGET TGT
USING EMP_STG SRC
ON (SRC.EMP_ID=TGT.EMP_ID)
WHEN MATCHED
THEN UPDATE SET TGT.ENDDATE=(SRC.UPDATEDON)-1, TGT.ACTIVE_FLAG=0
WHERE TGT.EMP_LOCATION <> SRC.EMP_LOCATION OR TGT.EMP_SALARY<>SRC.EMP_SALARY
WHEN NOT MATCHED
THEN INSERT (EMP_ID, EMP_NAME, EMP_LOCATION, EMP_SALARY, STARTDATE, ENDDATE, ACTIVE_FLAG)
VALUES(SRC.EMP_ID, SRC.EMP_NAME, SRC.EMP_LOCATION, SRC.EMP_SALARY, SRC.UPDATEDON, NULL, 1);
DELETE FROM EMP_STG;
DBMS_OUTPUT.PUT_LINE('Successfully implemented SCD 2');
END;
```

```SQL
-- Execute the procedure SCD_TWO
EXEC SCD_TWO;
```

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## SCD Type-3

### SHIPPING_STG Table

This table contains information about products in the shipping staging area, including product details, price, manufacturer address, and the date the record was last updated.

### Table Structure

| Column Name     | Data Type        | Description                                            |
|-----------------|------------------|--------------------------------------------------------|
| `PROD_ID`       | `INT`            | Unique identifier for each product (Primary Key)       |
| `PROD_NAME`     | `VARCHAR(100)`    | Name of the product                                    |
| `PROD_PRICE`    | `DECIMAL(10,2)`   | Price of the product                                   |
| `MANUF_ADDRESS` | `VARCHAR(50)`     | Manufacturer’s address of the product                  |
| `UPDATEDON`     | `DATE`            | Date when the product record was last updated          |

### Sample Data

| PROD_ID | PROD_NAME     | PROD_PRICE | MANUF_ADDRESS | UPDATEDON   |
|---------|---------------|------------|----------------|-------------|
| 201     | LED TV        | 17000.25   | Bangalore      | 2023-09-23  |
| 204     | Refrigerator  | 13999.00   | Kolkata        | 2024-01-21  |
| 205     | Split AC      | 23000.75   | Mumbai         | 2023-11-29  |
```sql
CREATE TABLE SHIPPING_STG
(
PROD_ID INT PRIMARY KEY,
PROD_NAME VARCHAR(100) NOT NULL,
PROD_PRICE DECIMAL(10,2) NOT NULL,
MANUF_ADDRESS VARCHAR(50) NOT NULL,
UPDATEDON DATE NOT NULL );

INSERT INTO SHIPPING_STG
    VALUES(201,'LED TV',17000.25,'Bangalore',TO_DATE('2023-09-23', 'yyyy-mm-dd'));
INSERT INTO SHIPPING_STG
    VALUES(204,'Refrigerator',13999.00,'Kolkata',TO_DATE('2024-01-21', 'yyyy-mm-dd'));
INSERT INTO SHIPPING_STG
    VALUES(205,'Split AC',23000.75,'Mumbai',TO_DATE('2023-11-29', 'yyyy-mm-dd'));
-- INSERT INTO SHIPPING_STG VALUES(201,'LCD TV', 17000.25,'Bangalore',TO_DATE('2024-10-23', 'yyyy-mm-dd'));
-- INSERT INTO SHIPPING_STG VALUES(204,'REFRIGERATOR',17000.25,'PUNE',TO_DATE('2024-09-23', 'yyyy-mm-dd'));
```

### SHIPPING_TARGET Table

The `SHIPPING_TARGET` table stores information about products, including their current and previous prices, manufacturer addresses, and the last updated date for each product.

### Table Structure

| Column Name      | Data Type        | Description                                              |
|------------------|------------------|----------------------------------------------------------|
| `PROD_ID`        | `INT`            | Unique identifier for each product (Primary Key)         |
| `PROD_NAME`      | `VARCHAR(100)`    | Name of the product                                      |
| `MANUF_ADDRESS`  | `VARCHAR(50)`     | Manufacturer’s address of the product                    |
| `CURRENTPRICE`   | `DECIMAL(10,2)`   | Current price of the product                             |
| `PREVIOUSPRICE`  | `DECIMAL(10,2)`   | Previous price of the product (nullable)                 |
| `LASTUPDATED`    | `DATE`            | Date when the product record was last updated            |

### Sample Data

| PROD_ID | PROD_NAME        | MANUF_ADDRESS | CURRENTPRICE | PREVIOUSPRICE | LASTUPDATED  |
|---------|------------------|----------------|--------------|---------------|--------------|
| 201     | LED TV           | Bangalore      | 19000.55     | NULL          | 2023-07-12   |
| 202     | Washing Machine  | Chennai        | 12000.75     | NULL          | 2024-03-19   |
| 203     | Water Geyser     | Delhi          | 6000.00      | NULL          | 2023-10-22   |
| 204     | Refrigerator     | Kolkata        | 17000.00     | NULL          | 2024-03-14   |

```sql
CREATE TABLE SHIPPING_TARGET
(
PROD_ID INT PRIMARY KEY,
PROD_NAME VARCHAR(100) NOT NULL,
MANUF_ADDRESS VARCHAR(50) NOT NULL,
CURRENTPRICE DECIMAL(10,2) NOT NULL,
PREVIOUSPRICE DECIMAL(10,2),
LASTUPDATED DATE NOT NULL );

INSERT INTO SHIPPING_TARGET
    VALUES(201,'LED TV','Bangalore',19000.55,NULL,TO_DATE('2023-07-12', 'yyyy-mm-dd'));
INSERT INTO SHIPPING_TARGET
    VALUES(202,'Washing Machine','Chennai',12000.75,NULL,TO_DATE('2024-03-19', 'yyyy-mm-dd'));
INSERT INTO SHIPPING_TARGET
    VALUES(203,'Water Geyser','Delhi',6000.00,NULL,TO_DATE('2023-10-22', 'yyyy-mm-dd'));
INSERT INTO SHIPPING_TARGET
    VALUES(204,'Refrigerator','Kolkata',17000.00,NULL,TO_DATE('2024-03-14', 'yyyy-mm-dd'));


-- SELECT * FROM SHIPPING_STG;
-- SELECT * FROM SHIPPING_TARGET;
-- TRUNCATE TABLE SHIPPING_STG;
-- TRUNCATE TABLE SHIPPING_TARGET;
```
```sql
CREATE OR REPLACE PROCEDURE SCD_THREE
AS
BEGIN
MERGE INTO SHIPPING_TARGET TARGET
USING SHIPPING_STG SOURCE
ON (SOURCE.PROD_ID = TARGET.PROD_ID)
WHEN MATCHED THEN UPDATE
    SET TARGET.MANUF_ADDRESS=SOURCE.MANUF_ADDRESS,
        TARGET.PREVIOUSPRICE=TARGET.CURRENTPRICE,
        TARGET.CURRENTPRICE=SOURCE.PROD_PRICE,
        TARGET.LASTUPDATED=SOURCE.UPDATEDON
WHERE TARGET.CURRENTPRICE<>SOURCE.PROD_PRICE
        OR TARGET.MANUF_ADDRESS<>SOURCE.MANUF_ADDRESS
WHEN NOT MATCHED
THEN INSERT(PROD_ID, PROD_NAME, MANUF_ADDRESS, CURRENTPRICE, PREVIOUSPRICE, LASTUPDATED)
VALUES(SOURCE.PROD_ID, SOURCE.PROD_NAME, SOURCE.MANUF_ADDRESS, SOURCE.PROD_PRICE, NULL, SOURCE.UPDATEDON);
DELETE FROM SHIPPING_STG;
DBMS_OUTPUT.PUT_LINE('SUCCESSFULLY IMPLEMENTED SCD TYPE 3');
END;
```
```sql
-- Execute the procedure SCD_THREE
EXEC SCD_THREE;
```
