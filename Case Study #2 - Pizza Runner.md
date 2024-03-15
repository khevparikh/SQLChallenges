### Problem Statement: 

Danny has launched Pizza Runner, a pizza delivery service aiming to combine 80s Retro Styling with pizza and an Uber-like delivery model. He started by recruiting runners to deliver 
pizzas and developed a mobile app for customer orders. The problem statement involves analyzing the datasets to gain insights into Pizza Runner's operations, such as order fulfillment rates, popular pizza types, runner performance, and 
customer preferences. Cleaning and preprocessing of data may be necessary before conducting analysis or building models. 

<b> The data consists of six datasets</b>:

1) <b> Runners Table:</b> This dataset contains the registration dates for each new runner.
2) <b> Customer Orders Table:</b> Each row in this table represents an individual pizza order made by a customer. It includes the pizza ID, exclusions (ingredients to be removed), 
and extras (ingredients to be added). Customers can order multiple pizzas with varying exclusions and extras, even if the pizza type is the same. The exclusions and extras columns 
need cleaning before analysis.
3) <b> Runner Orders Table:</b> Orders are assigned to runners, but not all orders are completed and may be canceled. This table includes timestamps for pickup, distance, and duration 
traveled by the runner. There are known data issues in this table.
4) <b> Pizza Names Table:</b> Pizza Runner offers two types of pizzas: Meat Lovers and Vegetarian.
5) <b> Pizza Recipes Table:</b> Each pizza type has a standard set of toppings defined by the pizza ID.
6) <b> Pizza Toppings Table:</b> This table lists all topping names with their corresponding IDs.

The entity relationship diagram below provides insights into the information contained within each table, including their columns, data types, and the relationships between them. 
It illustrates the tables that are linked and identifies the common keys used for these connections.

<img src=repo_images/entity_diagram_2.png width="1000" height="500">

---

**Query #1: How many pizzas were ordered?**
```sql
SELECT COUNT(pizza_id) as num_pizzas_ordered
FROM pizza_runner.customer_orders;
```

| num_pizzas_ordered |
| ------------------ |
| 14                 |

---

**Query #2: How many unique customer orders were made?**
```sql
SELECT COUNT(DISTINCT order_id) as num_orders_placed
FROM pizza_runner.customer_orders;
```

| num_orders_placed |
| ----------------- |
| 10                |

---

**Query #3: How many successful orders were delivered by each runner?**
```sql
SELECT COUNT(order_id) as successful_orders 
FROM pizza_runner.runner_orders
WHERE pickup_time != 'null';
```
Note: Some rows contain null values, which are represented as strings rather than the built-in null function.

| successful_orders |
| ----------------- |
| 8                 |

---

**Query #4: How many of each type of pizza was delivered?**
```sql
SELECT	p.pizza_name
       , COUNT(c.pizza_id) AS num_delivered 
FROM pizza_runner.runner_orders r
LEFT JOIN pizza_runner.customer_orders c
       ON r.order_id = c.order_id
LEFT JOIN pizza_runner.pizza_names p
       ON c.pizza_id = p.pizza_id 
WHERE
       pickup_time != 'null'
GROUP BY
       p.pizza_name;
```

| pizza_name | num_delivered    |
| ---------- | ---------------- |
| Meatlovers | 9                |
| Vegetarian | 3                |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/mdd7LYzxFXSFYHXAbgzQiy/0)
