
# [Case Study #2 - Pizza Runner](https://8weeksqlchallenge.com/case-study-2/)

## Data Cleaning

### a. Cleaning `customer_orders`

#### Query:

```sql
DROP TABLE IF EXISTS tmp_customer_orders;
CREATE TEMP TABLE tmp_customer_orders AS
SELECT
  order_id,
  customer_id,
  pizza_id,
  CASE WHEN exclusions = 'null' THEN NULL ELSE exclusions END,
  CASE WHEN extras = 'null' THEN NULL ELSE extras END,
  order_time
FROM customer_orders;
```

### b. Cleaning `runner_orders`

#### Query:

```sql
DROP TABLE IF EXISTS tmp_runner_orders;
CREATE TEMP TABLE tmp_runner_orders AS
SELECT
  order_id,
  runner_id,
  CAST(CASE WHEN pickup_time = 'null' THEN NULL ELSE pickup_time END AS TIMESTAMP),
  CAST (
    CASE
	  WHEN distance = 'null' THEN NULL
	  WHEN distance LIKE '%km' THEN TRIM(distance, 'km') ELSE distance
    END AS FLOAT),
  CAST(
	CASE
	  WHEN duration = 'null' THEN NULL
	  WHEN duration LIKE '%mins' THEN TRIM(duration, 'mins')
	  WHEN duration LIKE '%minute' THEN TRIM(duration, 'minute')
	  WHEN duration LIKE '%minutes' THEN TRIM(duration, 'minutes') ELSE duration
    END AS FLOAT),
  CASE WHEN cancellation = 'null' or cancellation = '' THEN NULL ELSE cancellation END
FROM runner_orders;
```

---

## Case Study Questions

## a. Pizza Metrics

### 1. How many pizzas were ordered?

#### Query:

```sql
SELECT COUNT(*)AS total_pizzas_ordered
FROM customer_orders;
```

#### Output:

|   total_pizzas_ordered |
|-----------------------:|
|                     14 |

### 2. How many unique customer orders were made?

#### Query:

```sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM customer_orders;
```

#### Output:

|   unique_orders |
|----------------:|
|              10 |

### 3. How many successful orders were delivered by each runner?

#### Query:

```sql
SELECT
  ro.runner_id,
  COUNT(DISTINCT ro.order_id) AS successful_orders
FROM tmp_customer_orders co
INNER JOIN tmp_runner_orders ro
  ON co.order_id = ro.order_id AND ro.distance IS NOT NULL
GROUP BY ro.runner_id;
```

#### Output:

|   runner_id |   successful_orders |
|------------:|--------------------:|
|           1 |                   4 |
|           2 |                   3 |
|           3 |                   1 |

### 4. How many of each type of pizza was delivered?

#### Query:

```sql
SELECT
  pn.pizza_name,
  COUNT(*) AS total_delivered
FROM tmp_customer_orders co
INNER JOIN tmp_runner_orders ro
  ON co.order_id = ro.order_id AND ro.distance IS NOT NULL
INNER JOIN pizza_names pn
  ON co.pizza_id = pn.pizza_id
GROUP BY pn.pizza_name;
```

#### Output:

| pizza_name   |   total_delivered |
|:-------------|------------------:|
| Meatlovers   |                 9 |
| Vegetarian   |                 3 |


### 5. How many Vegetarian and Meatlovers were ordered by each customer?

#### Query:

```sql
SELECT
  co.customer_id,
  pn.pizza_name,
  COUNT(*)
FROM tmp_customer_orders co
INNER JOIN pizza_names pn
  ON co.pizza_id = pn.pizza_id
GROUP BY co.customer_id, pn.pizza_name
ORDER BY co.customer_id;
```

#### Output:

|   customer_id | pizza_name   |   count |
|--------------:|:-------------|--------:|
|           101 | Meatlovers   |       2 |
|           101 | Vegetarian   |       1 |
|           102 | Meatlovers   |       2 |
|           102 | Vegetarian   |       1 |
|           103 | Meatlovers   |       3 |
|           103 | Vegetarian   |       1 |
|           104 | Meatlovers   |       3 |
|           105 | Vegetarian   |       1 |


### 6. What was the maximum number of pizzas delivered in a single order?

#### Query:

```sql
WITH pizzas_delivered_by_order AS (
  SELECT
	co.order_id,
	COUNT(*) AS total_pizzas_delivered
  FROM tmp_customer_orders co
  INNER JOIN tmp_runner_orders ro
	ON co.order_id = ro.order_id AND ro.distance IS NOT NULL
  GROUP BY co.order_id
),
ranked_pizzas_delievered AS (
	SELECT *, RANK() OVER(ORDER BY total_pizzas_delivered DESC)
	FROM pizzas_delivered_by_order
)
SELECT
  order_id,
  total_pizzas_delivered
FROM ranked_pizzas_delievered
WHERE rank = 1;
```

#### Output:

|   order_id |   total_pizzas_delivered |
|-----------:|-------------------------:|
|          4 |                        3 |
