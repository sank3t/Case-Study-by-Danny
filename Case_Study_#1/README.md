
## Case Study #1 - Danny's Diner

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

### 2. How many days has each customer visited the restaurant?

```sql
SELECT
  customer_id,
  COUNT(DISTINCT order_date) AS total_days
FROM sales
GROUP BY customer_id
ORDER BY total_days DESC;
```
