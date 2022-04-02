
## [Case Study #1 - Danny's Diner](https://8weeksqlchallenge.com/case-study-1/)

### 1. What is the total amount each customer spent at the restaurant?

#### Query:

```sql
SELECT
  s.customer_id,
  SUM(m.price) AS total_amount_spent
FROM sales s
INNER JOIN menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_amount_spent DESC;
```

#### Output:

| customer_id   |   total_amount_spent |
|:--------------|---------------------:|
| A             |                   76 |
| B             |                   74 |
| C             |                   36 |

---

### 2. How many days has each customer visited the restaurant?

#### Query:

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS total_days
FROM sales
GROUP BY customer_id
ORDER BY total_days DESC;
```

#### Output:

| customer_id   |   total_days |
|:--------------|-------------:|
| B             |            6 |
| A             |            4 |
| C             |            2 |

---

### 3. What was the first item from the menu purchased by each customer?

#### Query:

```sql
WITH ranked_orders AS (
	SELECT
	  customer_id,
	  order_date,
	  product_name,
	  RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
	FROM sales s
	INNER JOIN menu m
	  ON s.product_id = m.product_id
)
SELECT
  customer_id,
  product_name
FROM ranked_orders
WHERE rank = 1
GROUP BY customer_id, product_name;
```

#### Output:

| customer_id   | product_name   |
|:--------------|:---------------|
| A             | curry          |
| A             | sushi          |
| B             | curry          |
| C             | ramen          |

---

### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

#### Query:

```sql
WITH items_sold_count AS (
	SELECT
	  m.product_name,
	  COUNT(*) AS sold_count
	FROM sales s
	INNER JOIN menu m
	  ON s.product_id = m.product_id
	GROUP BY product_name
),
ranked_items_sold_count AS (
	SELECT
	  product_name,
	  sold_count,
	  RANK() OVER (ORDER BY sold_count DESC) AS rank
	FROM items_sold_count
)
SELECT
  product_name,
  sold_count
FROM ranked_items_sold_count
WHERE rank = 1;
```

#### Output:

| product_name   |   sold_count |
|:---------------|-------------:|
| ramen          |            8 |

---

### 5. Which item was the most popular for each customer?

#### Query:

```sql
WITH customer_item_count AS (
	SELECT customer_id, product_name, COUNT(*)
	FROM sales s
	INNER JOIN menu m
	  ON s.product_id = m.product_id
	GROUP BY customer_id, product_name
),
rank_customer_item_count AS (
	SELECT *, RANK() OVER(PARTITION BY customer_id ORDER BY count DESC)
	FROM customer_item_count
)
SELECT
  customer_id,
  product_name
FROM rank_customer_item_count
WHERE rank = 1;
```

#### Output:

| customer_id   | product_name   |
|:--------------|:---------------|
| A             | ramen          |
| B             | sushi          |
| B             | curry          |
| B             | ramen          |
| C             | ramen          |

---

### 6. Which item was purchased first by the customer after they became a member?

#### Query:

```sql
WITH ranked_after_membership AS (
	SELECT
	  s.customer_id,
	  s.product_id,
	  m2.product_name,
	  s.order_date,
	  m1.join_date,
	  RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
	FROM sales s
	INNER JOIN members m1
	  ON s.customer_id = m1.customer_id
	INNER JOIN menu m2
	  ON s.product_id = m2.product_id
	WHERE s.order_date >= m1.join_date
)
SELECT
  customer_id,
  product_name
FROM ranked_after_membership
WHERE rank = 1;
```

#### Output:

| customer_id   | product_name   |
|:--------------|:---------------|
| A             | curry          |
| B             | sushi          |

---

### 7. Which item was purchased just before the customer became a member?

#### Query:

```sql
WITH ranked_before_membership AS (
	SELECT
	  s.customer_id,
	  s.product_id,
	  m2.product_name,
	  s.order_date,
	  m1.join_date,
	  RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rank
	FROM sales s
	INNER JOIN members m1
	  ON s.customer_id = m1.customer_id
	INNER JOIN menu m2
	  ON s.product_id = m2.product_id
	WHERE s.order_date < m1.join_date
)
SELECT
  customer_id,
  product_name
FROM ranked_before_membership
WHERE rank = 1;
```

#### Output:

| customer_id   | product_name   |
|:--------------|:---------------|
| A             | sushi          |
| A             | curry          |
| B             | sushi          |

---

### 8. What is the total items and amount spent for each member before they became a member?

#### Query:

```sql
SELECT
  s.customer_id,
  COUNT(DISTINCT s.product_id) AS total_items,
  SUM(price) AS amount_spent
FROM sales s
INNER JOIN members m1
  ON s.customer_id = m1.customer_id
INNER JOIN menu m2
  ON s.product_id = m2.product_id
WHERE s.order_date < m1.join_date
GROUP BY s.customer_id;
```

#### Output:

| customer_id   |   total_items |   amount_spent |
|:--------------|--------------:|---------------:|
| A             |             2 |             25 |
| B             |             2 |             40 |

---

### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

#### Query:

```sql
SELECT
  s.customer_id,
  SUM(CASE WHEN m.product_name = 'sushi' THEN price * 20 ELSE price * 10 END) AS total_points
FROM sales s
INNER JOIN menu m
  ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY total_points DESC;
```

#### Output:

| customer_id   |   total_points |
|:--------------|---------------:|
| B             |            940 |
| A             |            860 |
| C             |            360 |
