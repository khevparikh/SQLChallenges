
### Problem Statement: 

Danny wants to leverage customer data to gain insights into their visiting patterns, spending habits, and favorite menu items. This understanding will enable him to enhance customer experience and possibly expand his loyalty program. Danny has provided three main datasets: sales, menu, and members. These datasets will be used to analyze customer behavior and preferences, ultimately informing decisions regarding customer engagement and loyalty initiatives.

The entity relationship diagram below provides insights into the information contained within each table, including their columns, data types, and the relationships between them. It illustrates the tables that are linked and identifies the common keys used for these connections.

<img src=entity_diagram.jpg width="600" height="500">

---

**Query #1: What is the total amount each customer spent at the restaurant?** 

```sql
SELECT s.customer_id
       , SUM(m.price) AS total_spent
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
       ON s.product_id = m.product_id
GROUP BY s.customer_id
ORDER BY s.customer_id;
```

| customer_id | total_spent |
| ----------- | ----------- |
| A           | 76          |
| B           | 74          |
| C           | 36          |

---

**Query #2: How many days has each customer visited the restaurant?**

```sql
SELECT customer_id
       , COUNT(DISTINCT order_date) AS num_of_visits
FROM dannys_diner.sales
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | num_of_visits |
| ----------- | ------------- |
| A           | 4             |
| B           | 6             |
| C           | 2             |

---

**Query #3: What was the first item from the menu purchased by each customer?**

```sql
WITH items_order_CTE AS (
    SELECT s.customer_id
           , s.order_date
           , m.product_id
           , m.product_name
           , ROW_NUMBER() OVER(PARTITION BY customer_id ORDER BY order_date) AS items_order
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
           ON s.product_id = m.product_id
)

SELECT customer_id
       , product_name
FROM items_order_CTE
WHERE items_order = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | Curry        |
| B           | Curry        |
| C           | Ramen        |

---

**Query #4: What is the most purchased item on the menu and how many times was it purchased by all customers?**

```sql
SELECT	m.product_name
        , COUNT(s.product_id) AS num_purchased
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m 
       ON s.product_id = m.product_id
GROUP BY m.product_name
ORDER BY num_purchased DESC
LIMIT 1;

```
| product_name | num_purchased |
| ------------ | ------------- |
| ramen        | 8             |

---

**Query #5: Which item was the most popular for each customer?**

```sql
WITH most_popular_item_per_customer AS (
    SELECT  s.customer_id
            , m.product_name
            , RANK() OVER(PARTITION BY s.customer_id ORDER BY COUNT(s.product_id)) AS popularity_rank
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m 
           ON s.product_id = m.product_id
    GROUP BY s.customer_id, m.product_name
)

SELECT	customer_id
        , product_name
FROM most_popular_item_per_customer 
WHERE popularity_rank = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | ramen        |
| B           | curry        |
| B           | sushi        |
| C           | ramen        |

---

**Query #6: Which item was purchased first by the customer after they became a member?**

``` sql 
WITH items_bought_after_membership AS (
    SELECT	s.*, m.join_date, ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS purchase_sequence
    FROM dannys_diner.sales s
    INNER JOIN dannys_diner.members m
            ON s.customer_id = m.customer_id
    WHERE s.order_date >= m.join_date
) 
    
SELECT * 
FROM items_bought_after_membership
LEFT JOIN dannys_diner.menu m 
       ON items_bought_after_membership.product_id = m.product_id
WHERE purchase_sequence = 1;

```
| customer_id | order_date               | product_id | join_date                | purchase_sequence | product_id | product_name | price |
| ----------- | ------------------------ | ---------- | ------------------------ | ----------------- | ---------- | ------------ | ----- |
| B           | 2021-01-11T00:00:00.000Z | 1          | 2021-01-09T00:00:00.000Z | 1                 | 1          | sushi        | 10    |
| A           | 2021-01-07T00:00:00.000Z | 2          | 2021-01-07T00:00:00.000Z | 1                 | 2          | curry        | 15    |

<b>Refining to return the customer name and product name only:</b>
```sql

SELECT	customer_id
        , m.product_name
FROM items_bought_after_membership
LEFT JOIN dannys_diner.menu m 
       ON items_bought_after_membership.product_id = m.product_id
WHERE purchase_sequence = 1;
```
| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | curry        |

---

**Query #7: Which item was purchased just before the customer became a member?**

``` sql
WITH items_bought_before_membership AS (
  SELECT    s.*
            , m.join_date
            , ROW_NUMBER() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS purchase_sequence
  FROM dannys_diner.sales s
  INNER JOIN dannys_diner.members m 
          ON s.customer_id = m.customer_id
  WHERE s.order_date < m.join_date 
) 

SELECT	customer_id
        , m.product_name
FROM items_bought_before_membership
LEFT JOIN dannys_diner.menu m 
       ON items_bought_before_membership.product_id = m.product_id
WHERE purchase_sequence = 1;
```

| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | sushi        |

---

**Query #8: What is the total items and amount spent for each member before they became a member?**

``` sql
SELECT  s.customer_id
        , COUNT(s.product_id) AS total_items
        , SUM(mu.price) AS total_spent
FROM dannys_diner.sales s
INNER JOIN dannys_diner.members m
        ON s.customer_id = m.customer_id
LEFT JOIN dannys_diner.menu mu
       ON s.product_id = mu.product_id        
WHERE s.order_date < m.join_date
GROUP BY s.customer_id
ORDER BY s.customer_id; 
```

| customer_id | total_items | total_spent |
| ----------- | ----------- | ----------- |
| A           | 2           | 25          |
| B           | 3           | 40          |

---

**Query #9: If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**

``` sql
WITH points_calculation AS (
    SELECT	s.customer_id
            , s.product_id
            , m.product_name
            , m.price
            , CASE 
                WHEN product_name = 'sushi' THEN price*10*2
                ELSE price*10
                END AS points
    FROM dannys_diner.sales s
    LEFT JOIN dannys_diner.menu m
           ON s.product_id = m.product_id
) 
    
SELECT  customer_id
        , SUM(points) AS total_points
FROM points_calculation
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total_points |
| ----------- | ------------ |
| A           | 860          |
| B           | 940          |
| C           | 360          |

---

**Query #10: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?**

``` sql
WITH points_calculation AS (
SELECT	s.customer_id
        , s.order_date
        , s.product_id
        , m.product_name
        , m.price
        , CASE 
            WHEN (s.order_date BETWEEN me.join_date AND me.join_date + 7)
                 OR m.product_name = 'sushi' THEN m.price * 10 * 2
            ELSE m.price * 10 
        END AS points
FROM dannys_diner.sales s
LEFT JOIN dannys_diner.menu m
       ON s.product_id = m.product_id
LEFT JOIN dannys_diner.members me
       ON s.customer_id = me.customer_id
)
 
SELECT customer_id
        , SUM(points) AS total_points
FROM points_calculation
GROUP BY customer_id
ORDER BY customer_id;
```

| customer_id | total_points |
| ----------- | ------------ |
| A           | 1370         |
| B           | 1060         |
| C           | 360          |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/46HdTg76sGXC8Bx7ytiBpm/0)
