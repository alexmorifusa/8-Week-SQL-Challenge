# Case Study #2 - Pizza Runner
![image](https://github.com/alexmorifusa/SQL/assets/137368881/1e66f6d9-e37b-407c-b148-64b4a2e47d31)

# Questions:
## A. Pizza Metrics
### Q1) How many pizzas were ordered?
```sql
SELECT
COUNT(customer_orders.order_id) AS total_orders
FROM pizza_runner.customer_order
```
![imgage](https://github.com/alexmorifusa/SQL/assets/137368881/167b39c6-19b8-4122-a5cd-5a8ada8e20d5)

### Q2) How many unique customer orders were made?
```sql
SELECT
COUNT(DISTINCT order_id) AS unique_orders
FROM pizza_runner.customer_orders
```
![imgage](https://github.com/alexmorifusa/SQL/assets/137368881/25118e3e-dd2a-409d-b79e-9732bd9e9191)

### Q3) How many successful orders were delivered by each runner?
```sql
SELECT
runner_id, COUNT(order_id) AS successful_orders
FROM pizza_runner.runner_orders
WHERE distance != 'null'
GROUP BY runner_id
ORDER BY runner_id ASC
```
![image](https://github.com/alexmorifusa/SQL/assets/137368881/c737d411-c9e9-4986-9bfb-cba168a17e16)

### Q4) How many of each type of pizza was delivered?
```sql
SELECT
pizza_name, COUNT(customer_orders.pizza_id) AS delivered_orders
FROM pizza_runner.customer_orders
INNER JOIN pizza_runner.pizza_names
ON customer_orders.pizza_id = pizza_names.pizza_id
INNER JOIN pizza_runner.runner_orders
ON customer_orders.order_id = runner_orders.order_id
WHERE distance != 'null'
GROUP BY pizza_name
```
![image](https://github.com/alexmorifusa/SQL/assets/137368881/c902219d-c957-4d8b-bebf-e2d5e2352e67)

### Q5) How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT
customer_id, pizza_name, COUNT(pizza_name) AS num_orders
FROM pizza_runner.customer_orders AS c
INNER JOIN pizza_runner.pizza_names AS p
ON c.pizza_id = p.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id
```
Without aliasing, I kept on running into an error of "missing FROM-clause entry for table". Aliasing the table names helped get ride of those errors and run my code.
![image](https://github.com/alexmorifusa/SQL/assets/137368881/abe34f0c-fb0f-4adc-8403-3f9e20e194e9)

### Q6) What was the maximum number of pizzas delivered in a single order?
```sql
SELECT
customer_id, order_id, COUNT(order_id) AS num_pizzas
FROM pizza_runner.customer_orders
GROUP BY customer_id, order_id
ORDER BY num_pizzas DESC
LIMIT 1
```
![image](https://github.com/alexmorifusa/SQL/assets/137368881/4b4348ba-9e11-4707-9181-d1e41bfd5ea1)

### Q7) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
  c.customer_id,
  SUM(
    CASE WHEN c.exclusions IS NOT NULL OR c.extras IS NOT NULL THEN 1
    ELSE 0
    END) AS change_made,
  SUM(
    CASE WHEN c.exclusions IS NULL AND c.extras IS NULL THEN 1 
    ELSE 0
    END) AS no_change
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
I had to clean the exclusions and extras column in customer_orders so all null or '' values were set as NULL so I could accurately count whether a customer asked for a change or not. This was the same for the runner_orders table so I can accurately check if the order was cancelled or not. 
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/eadcdaa4-02ba-4e45-b0c4-36611e834427">

### Q8) How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT 
  c.customer_id,
  SUM(
    CASE WHEN c.exclusions IS NOT NULL AND c.extras IS NOT NULL THEN 1
    ELSE 0
    END) AS both_changes
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL
GROUP BY c.customer_id
ORDER BY c.customer_id;
```
<img  src="https://github.com/alexmorifusa/SQL/assets/137368881/0e2e8f12-ae2b-408f-8df8-bfe70ec4b4f0">

### Q9) What was the total volume of pizzas ordered for each hour of the day?
USING the EXTRACT() function I can return a specific part of the day (the hour in this case).
```sql
SELECT
EXTRACT(hour FROM c.order_time) AS hour,
COUNT(order_id) AS pizza_orders
FROM pizza_runner.customer_orders AS c
GROUP BY hour
ORDER BY hour
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/a7b78eef-7778-424a-a632-7189cfbf699d">

### Q10) What was the volume of orders for each day of the week?
```sql
SELECT
TO_CHAR(order_time + INTERVAL '2 day', 'DAY') AS day_of_week,
  COUNT(order_id) AS total_pizzas_ordered
FROM pizza_runner.customer_orders
GROUP BY day_of_week
ORDER BY day_of_week
```
I used TO_CHAR to extract the day of the week as a name from a date column. I also had to add 2 days to order_time to get the accurate day of the week since 01/01/2020 was on a Friday.
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/054aa8a9-b53c-46e5-a89b-a06a0378828e">

Reflection:
I ran into a couple troubles since some functions and values were treated differently in PostgreSQL. I also should have cleaned and organized my data at the start. 
