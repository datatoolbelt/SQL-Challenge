#  Case Study #2 - Pizza Runner 🍕

<img src="https://github.com/datatoolbelt/8-Week-SQL-Challenge/assets/161499632/2e68c725-a992-4c21-8760-cfa216edb304" width="500" height="500" />

### Case Overview
* Danny is *Uberizing* his pizza delivery
* His big idea is recruiting "runners" on their run through an app to deliver pizza

#### Details of Data
* We have been provided the 5 data tables namely,
    - `runners`
    - `customer_orders`
    - `runner_orders`
    - `pizza_names`
    - `pizza_recipes`
    - `pizza_toppings`
* ERD or Entity Relationship Diagram is as below:

<img src="https://github.com/datatoolbelt/8-Week-SQL-Challenge/assets/161499632/d21d2817-084f-4a54-a230-b799b945b810" width="600" height="300" />

### Metadata

Table name: `runners` 🏃 - clean
| Column Name | Description | 
|---|---|
|runner_id|unique identification of each runner|
|registration_date|date of registration|

Table name: `customer_orders` 📝 - to be cleaned before use
| Column Name | Description | 
|---|---|
|order_id|order number|
|customer_id|customer identification number|
|pizza_id|which pizza was ordered|
|exclusions|ingredient_id values which should be removed from the pizza|
|extras|ingredient_id values which need to be added to the pizza|
|order_time|exact timestamp of each order|

Table name: `runner_orders` 🛍️ - to be cleaned before use
| Column Name | Description | 
|---|---|
|order_id|order number|
|runner_id|unique identification of each runner|
|pickup_time|runner pick's up from restaurant|
|distance|how far is the delivery (in km)|
|duration|time to deliver(minutes)|
|cancellation|cancelled by|

Table name: `pizza_names` 🍕 - clean
| Column Name | Description | 
|---|---|
|pizza_id|unique identification of the pizza|
|pizza_name|name of the pizza|

Table name: `pizza_recipes` 📔 - ok to use
| Column Name | Description | 
|---|---|
|pizza_id|unique identification of the pizza|
|toppings|topping identification numbers|

Table name: `pizza_toppings` 🔥 - clean
| Column Name | Description | 
|---|---|
|topping_id|unique identification of toppings|
|topping_name|name of topping|

### Solutions to Case Questions
🧰 SQL Workbench
There are 25+ queries below, divided into 5 different files to keep it readable.
1. Data Cleaning
2. Pizza Metrics
3. Runner and Customer Experience
4. Ingredient Optimisation
5. Pricing and Ratings

## 1. Data Cleaning
Before moving into querying the data, it's extremely important that we clean the tables in order to get accurate results. As per our note two tables have inconsistencies `customer_orders` and `runner_orders`.
Let's take one table at a time and clean:

Issues in `customer_orders`:
- inconsitent data in extras and exclusions columns

```sql
-- get unique values in `extras`
SELECT DISTINCT extras
FROM customer_orders;

-- replace '','null' and NULL with '0'
UPDATE customer_orders
SET extras = '0'
WHERE extras IN ('', 'null') OR extras IS NULL;

-- get unique values in `exclusions`
SELECT DISTINCT exclusions
FROM customer_orders;

-- replace '', 'null' with '0'
UPDATE customer_orders
SET exclusions = '0'
WHERE exclusions IN ('', 'null') OR exclusions IS NULL;
```

Issues in `runner_orders`:
- inconsitent data in 4 columns

```sql
-- 1. `pickup_time`
SELECT DISTINCT pickup_time
FROM runner_orders;

-- replace 'null' with NULL
UPDATE runner_orders
SET pickup_time = NULL
WHERE pickup_time = 'null';

-- 2. `cancellation`
-- fill all uncancelled orders as 'Completed'
UPDATE runner_orders
SET cancellation = 'No'
WHERE cancellation IS NULL OR cancellation IN ('','null');

-- 3. `distance` & `duration`
-- add two new columns, distance_km and duration_min
ALTER TABLE runner_orders
ADD COLUMN distance_km DECIMAL(10,2) AFTER distance,
ADD COLUMN duration_min INTEGER AFTER duration;

-- `distance`: replace null with '00'
UPDATE runner_orders
SET distance = '0'
WHERE distance = 'null';

-- `distance`: remove 'km' and convert into integer
UPDATE runner_orders
SET distance_km = cast(
						trim(replace(distance, 'km', '')) 
                        AS DECIMAL(10,2));

-- `duration`: replace null with '00'
UPDATE runner_orders
SET duration = '0'
WHERE duration = 'null';

-- `duration`: remove characters and convert into integer
UPDATE runner_orders 
SET duration_min =	CAST(trim(left(duration, 2)) AS DECIMAL);

-- drop `distance` and `duration` columns
ALTER TABLE runner_orders
DROP COLUMN distance,
DROP COLUMN duration;
```

As the data is now clean enough to work. I will move into the first set of case questions 

## Next Step here
[1. Pizza Metrics](https://github.com/datatoolbelt/8-Week-SQL-Challenge/blob/8540424042c50e5e7e365402c4f3d7bcc2875ce0/Case%20Study%20%232%20-%20Pizza%20Runner%20/1.%20Pizza%20Metrics.md)
