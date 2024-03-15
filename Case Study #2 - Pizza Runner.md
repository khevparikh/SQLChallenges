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
