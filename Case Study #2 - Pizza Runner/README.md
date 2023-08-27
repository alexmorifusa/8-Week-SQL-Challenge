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
FROM pizza_runner.customer_orders
INNER JOIN pizza_runner.pizza_names
ON cutomer_orders.pizza_id = pizza_names.pizza_id
GROUP BY customer_id, pizza_name
ORDER BY customer_id
```
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

### Q8) How many pizzas were delivered that had both exclusions and extras?

### Q9) What was the total volume of pizzas ordered for each hour of the day?
### Q10) What was the volume of orders for each day of the week?
