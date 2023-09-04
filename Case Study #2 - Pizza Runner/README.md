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

## B. Runner and Customer Experience
### Q1) How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT
EXTRACT(week from r.registration_date + INTERVAL '1 week') AS week,
COUNT(runner_id) AS signup
FROM pizza_runner.runners AS r
GROUP BY week
ORDER BY week ASC
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/a10894a5-7695-4bd2-be49-b4dfdf61d6d1">

### Q2) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
SELECT
AVG(DATE_PART('minute', r.pickup_time::timestamp - c.order_time)) AS avg_time
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
ON c.order_id = r.order_id
WHERE r.distance IS NOT NULL 
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/22d6c39f-877e-46e7-bbf6-ecb03d3a61eb">

### Q3) Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH prep_time AS 
(
  SELECT
  c.order_id,
  c.order_time,
  r.pickup_time,
  COUNT(c.order_id) AS pizza_order,
  DATE_PART('Minute', r.pickup_time::TIMESTAMP - c.order_time::TIMESTAMP) AS avg_minutes
  FROM pizza_runner.customer_orders AS c
  JOIN pizza_runner.runner_orders AS r
  ON c.order_id = r.order_id
  WHERE r.distance IS NOT NULL
  GROUP BY c.order_id, c.order_time, r.pickup_time
)

SELECT 
pizza_order,
AVG(avg_minutes) AS avg_time
FROM prep_time
GROUP BY pizza_order
ORDER BY pizza_order ASC
```
Unlike other forms of SQL, PostgreSQL does not have DATEDIFF, so I had to implement a DATE_PART function to find the difference between the pickup_time and order_time. I also had to cast both to TIMESTAMP so the calculation will go through.
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/73280370-e19e-4fa0-acc6-ed7c79b682a8">

### Q4) What was the average distance travelled for each customer?
```sql
SELECT
c.customer_id,
AVG(TRIM('km' FROM r.distance)::numeric) AS avg_distance
FROM pizza_runner.customer_orders AS c
JOIN pizza_runner.runner_orders AS r
ON c.order_id = r.order_id
WHERE distance IS NOT NULL
GROUP BY customer_id
ORDER BY customer_id
```
I had to remove the units of the distance because I needed numerics to use the AVG function. I used the TRIM() function to remove km and casted the results as numerics since the TRIM function returns characters. 
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/1b3cb7f7-4f60-49e0-afb6-1e1cd3cb45c1">

### Q5) What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT
MAX(duration::numeric) - MIN(duration::numeric) AS differnce 
FROM pizza_runner.runner_orders
WHERE duration IS NOT NULL
```
I mannually removed the units from the duration column so I could treat the column as numeric values to preform calculations. 
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/1a5bba17-3fc5-4350-b10d-10d3ace1fbbb">

### Q6) What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT 
r.runner_id, c.customer_id, c.order_id, 
COUNT(c.order_id) AS pizza_count, r.distance, R
OUND((TRIM('km' FROM r.distance)::numeric/r.duration::numeric * 60), 2) AS avg_speed
FROM pizza_runner.runner_orders AS r
JOIN pizza_runner.customer_orders AS c
  ON r.order_id = c.order_id
WHERE distance IS NOT NULL
GROUP BY r.runner_id, c.customer_id, c.order_id, r.distance, r.duration
ORDER BY c.order_id;
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/24c57324-fa1a-4d1e-a654-9d56a1c39118">

* Runner 1 had an average speed of 37.5, 44.44, 40.20, and 60.
* Runner 2 had an average speed of 35.1, 60, and 93.60.
* Runner 3 had an average speed of 40.
* All are in km per hour.

Runner 2 sky rockets between average speeds by around 30 every trip. There may be corrupt or misinputted data since there is such a huge fluctuation of average speed. 

### Q7)
 What is the successful delivery percentage for each runner?
```sql
SELECT 
r.runner_id, 
SUM(
  CASE WHEN r.distance IS NULL THEN 0
  ELSE 1 END) * 100 / COUNT(*) AS success_percentage
FROM pizza_runner.runner_orders AS r
GROUP BY r.runner_id
ORDER BY r.runner_id
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/7ab0749b-32a1-45d8-9ed9-9bb9b396dd9c">

## C. Ingredient Optimisation
### Q1) What are the standard ingredients for each pizza?
```sql
SELECT
pizza_name,
STRING_AGG(topping_name, ', ') AS toppings
FROM pizza_runner.pizza_toppings AS t,
pizza_runner.pizza_recipes AS r
JOIN pizza_runner.pizza_names AS n 
ON r.pizza_id = n.pizza_id
WHERE
  t.topping_id IN
(
  SELECT
  UNNEST(STRING_TO_ARRAY(r.toppings, ',')::int[])
)
GROUP BY pizza_name
```
UNNEST(STRING_TO_ARRAY()) functions to separate the topping IDs in pizza_recipes to an array of integers. THan convert those integers back to the names of pizza toppings using STRING_AGG().
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/316074e2-8461-497d-be24-61395777a995">

### Q2) What was the most commonly added extra?
```sql
WITH toppings AS 
(
SELECT
extras, number_of_pizzas
FROM
  (
    WITH extras_table AS (
      SELECT
      order_id, UNNEST(STRING_TO_ARRAY(extras, ',')::int[]) AS topping_id
      FROM
        pizza_runner.customer_orders AS c
      WHERE
        extras IS NOT NULL
    )
SELECT
topping_name AS extras,
COUNT(topping_name) AS number_of_pizzas
FROM
extras_table AS et
JOIN pizza_runner.pizza_toppings AS t
ON et.topping_id = t.topping_id
GROUP BY topping_name
  ) AS t
ORDER BY number_of_pizzas DESC
LIMIT 1
```
I made a new extras_table to store an array of topping ids for extras from the customer_orders table. 
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/acaa7b06-01cc-4d0a-bace-60a2d1a79dcd">

### Q3) What was the most common exclusion?
```sql
SELECT
exclusions, number_of_pizzas
FROM
  (
    WITH ex_table AS (
      SELECT
      order_id, UNNEST(STRING_TO_ARRAY(exclusions, ',')::int[]) AS topping_id
      FROM
        pizza_runner.customer_orders AS c
      WHERE
        exclusions IS NOT NULL
    )
SELECT
topping_name AS exclusions,
COUNT(topping_name) AS number_of_pizzas
FROM
ex_table AS et
JOIN pizza_runner.pizza_toppings AS t
ON et.topping_id = t.topping_id
GROUP BY topping_name
  ) AS t
ORDER BY number_of_pizzas DESC
LIMIT 1
```
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/aba71460-9646-4351-98e4-d1c963d4e1de">

### Q4) Generate an order item for each record in the customers_orders table in the format of one of the following:
Meat Lovers
Meat Lovers - Exclude Beef
Meat Lovers - Extra Bacon
Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

### Q5) Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

### Q6) What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
