# Danny's Diner SQL Case Study

## Overview
Danny's Diner is a fictional restaurant where customers order different menu items. This SQL case study analyzes customer spending habits, menu popularity, and membership benefits using SQL queries. The dataset consists of three tables: `sales`, `menu`, and `members`.

## Database Schema

### 1. `sales` Table
This table records customer orders, including the customer ID, order date, and product ID.

| Column      | Data Type |
|------------|----------|
| customer_id | VARCHAR(1) |
| order_date  | DATE |
| product_id  | INTEGER |

### 2. `menu` Table
This table contains menu items and their prices.

| Column      | Data Type |
|------------|----------|
| product_id  | INTEGER |
| product_name | VARCHAR(5) |
| price       | INTEGER |

### 3. `members` Table
This table tracks customer membership start dates.

| Column      | Data Type |
|------------|----------|
| customer_id | VARCHAR(1) |
| join_date   | DATE |

## Case Study Questions & Solutions

### 1. Total Amount Spent by Each Customer
Calculates the total money spent by each customer.
```sql
SELECT s.customer_id, SUM(m.price) AS total_amount_spent
FROM sales AS s
JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

### 2. Number of Days Each Customer Visited
Counts distinct order dates for each customer.
```sql
SELECT customer_id, COUNT(DISTINCT order_date) AS visit_days
FROM sales
GROUP BY customer_id;
```

### 3. First Item Purchased by Each Customer
Finds the first menu item each customer purchased.
```sql
SELECT customer_id, order_date, first_item
FROM (
  SELECT s.customer_id, s.order_date, m.product_name AS first_item,
         ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS row_num
  FROM sales AS s
  JOIN menu AS m ON s.product_id = m.product_id
) subquery
WHERE row_num = 1;
```

### 4. Most Purchased Menu Item
Finds the most ordered menu item and its count.
```sql
SELECT TOP 1 m.product_name, COUNT(*) AS purchase_count
FROM sales AS s
JOIN menu AS m ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY purchase_count DESC;
```

### 5. Most Popular Item for Each Customer
Finds the most frequently ordered item per customer.
```sql
SELECT customer_id, product_name, total_orders
FROM (
    SELECT s.customer_id, m.product_name, COUNT(*) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY COUNT(*) DESC) AS rn
    FROM sales AS s
    JOIN menu AS m ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
) AS t
WHERE rn = 1;
```

### 6. First Item Purchased After Becoming a Member
Finds the first item a customer purchased after joining the membership.
```sql
SELECT t.customer_id, t.first_purchase_date, m.product_name
FROM (
    SELECT s.customer_id, MIN(s.order_date) AS first_purchase_date
    FROM sales AS s
    JOIN members AS mm ON mm.customer_id = s.customer_id
    WHERE s.order_date > mm.join_date
    GROUP BY s.customer_id
) AS t
JOIN sales AS s ON t.customer_id = s.customer_id AND t.first_purchase_date = s.order_date
JOIN menu AS m ON s.product_id = m.product_id;
```

### 7. Item Purchased Just Before Membership
Finds the last item a customer bought before joining the membership.
```sql
SELECT customer_id, product_id, last_order, product_name
FROM (
    SELECT s.customer_id, s.product_id, MAX(s.order_date) AS last_order, m.product_name,
           ROW_NUMBER() OVER (PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS rn
    FROM sales AS s
    JOIN menu AS m ON s.product_id = m.product_id
    JOIN members AS mm ON s.customer_id = mm.customer_id
    WHERE s.order_date < mm.join_date
    GROUP BY s.customer_id, s.product_id, m.product_name
) subquery
WHERE rn = 1;
```

### 8. Total Items and Amount Spent Before Membership
Calculates the total items purchased and amount spent before becoming a member.
```sql
SELECT s.customer_id, COUNT(s.product_id) AS total_items, SUM(m.price) AS total_amount_spent
FROM sales AS s
JOIN members AS mm ON s.customer_id = mm.customer_id
JOIN menu AS m ON s.product_id = m.product_id
WHERE s.order_date < mm.join_date
GROUP BY s.customer_id;
```

### 9. Customer Points Calculation
Calculates points where every $1 spent earns 10 points, and sushi gives 2x points.
```sql
SELECT s.customer_id,
       SUM(CASE WHEN m.product_name = 'sushi' THEN 2 * m.price ELSE m.price END) * 10 AS total_points
FROM sales AS s
JOIN menu AS m ON s.product_id = m.product_id
GROUP BY s.customer_id;
```

### 10. Points Earned in the First Week After Membership
Calculates total points earned, doubling points for the first week of membership.
```sql
SELECT s.customer_id,
       SUM(CASE WHEN order_date <= DATEADD(DAY, 6, mm.join_date) THEN m.price * 2 ELSE m.price END) * 10 AS total_points
FROM sales AS s
JOIN members AS mm ON s.customer_id = mm.customer_id
JOIN menu AS m ON s.product_id = m.product_id
WHERE YEAR(order_date) = 2021 AND MONTH(order_date) = 1
GROUP BY s.customer_id;
```

## How to Use
1. **Create the Database:** Run `CREATE DATABASE dannys_diner;`
2. **Create Tables:** Execute the `CREATE TABLE` statements.
3. **Insert Data:** Use the `INSERT INTO` queries.
4. **Run Queries:** Execute the case study questions to analyze the data.

## Conclusion
This case study helps understand SQL querying techniques such as joins, aggregations, ranking functions, and window functions to analyze restaurant data effectively. 🚀

