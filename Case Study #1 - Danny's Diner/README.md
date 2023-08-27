# Case Study #1 - Danny's Diner
<p align="center">
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/9533bb2f-0dea-4ebc-a966-51dbb54843dd">

# Task At Hand:
Danny has started a restaurant that sells his 3 favorite foods: sushi, curry, and ramen. Our job is to use the data given below to answer question about his customers and their visting patterns, how much money they've spent and also which items are their favorite.

![image](https://github.com/alexmorifusa/SQL/assets/137368881/bae701c9-c31e-4bc5-8ccc-fc4f1d1d118e)

# Data:
### Table #1: Sales
![image](https://github.com/alexmorifusa/SQL/assets/137368881/157ec620-58c7-4223-b7a6-62c8e35bd7d1)
### Table #2: Menu
![image](https://github.com/alexmorifusa/SQL/assets/137368881/980580ca-428b-4eef-b8cb-e08d4cf7c829)
### Table #3: Members
![image](https://github.com/alexmorifusa/SQL/assets/137368881/3552cbee-97c9-459d-87ac-510e82155209)

### Interactive SQL
All data for your SQL code can be found here https://www.db-fiddle.com/f/2rM8RAnq7h5LLDTzZiRWcd/138.

# Questions:
### 1) What is the total amount each customer spent at the restaurant?
``` sql
SELECT
sales.customer_id, SUM(menu.price) AS total
FROM
dannys_diner.sales
INNER JOIN dannys_diner.menu ON
sales.product_id = menu.product_id
GROUP BY
sales.customer_id
ORDER BY
customer_id ASC
```
### Thought Process:
Calculate the total amount (so SUM()) of what each customer spent. 
By inner joining the menu and sales table, I was able to figure out how much each customer spent by grouping by their id. 
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/7271caa2-f607-4a67-a6fe-087f8aa4389d)

### 2) How many days has each customer visited the restaurant?
``` sql
SELECT
customer_id,
COUNT(DISTINCT order_date) AS Visits
FROM
dannys_diner.sales
GRUP BY
customer_id
ORDER BY
customer_id ASC
```
### Thought Process: 
By counting the distinct order dates and grouping by customer id, I produced a table that counts how many times each customer visited the restaurant.
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/744e72d9-768a-46ec-8548-beda77e1402a)

### 3) What was the first item from the menu purchased by each customer?
``` sql
WITH first_item AS(
  SELECT
    sales.customer_id, menu.product_name,
    sales.order_date,
  DENSE_RANK() OVER(
    PARTITION BY sales.customer_id
    ORDER BY sales.order_date) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
  ON sales.product_id = menu.product_id
)
SELECT customer_id, product_name
FROM
  first_item
WHERE rank = 1
```
### Thought Process:
Using the WITH clause, I created a temporary table called first_item that inner joins the sales table and menu table. Then, I used the DENSE_RANK() function to rank each sale (starting with 1,2,3...) which enables me to find the first item ordered by each customer. To separate by customer I used the OVER(PARTITION BY...) function so each customer has their own set of ranks. Without PARTITION BY, the table will only return the first item bought by any customer, not by each customer. 
### Result:
<img src="https://github.com/alexmorifusa/SQL/assets/137368881/2645a720-2bfb-4bbc-b316-53109f07ea54">

### 4) What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT
menu.product_name,
COUNT(sales.product_id) AS purchased
FROM
dannys_diner.sales
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY
menu.product_name
ORDER BY
purcahsed DESC
```
^ Produces all purchased items from most purchased to least so I added a LIMIT to just the most.
```sql
LIMIT 1
```
### Thought Process:
This question does not require me to separate by each customer which makes my life a little easier. I used the COUNT() function to count all products by their id and grouped each product by their name. To find the most purchased item, I ordered the table the purchased items in DESC order and added LIMIT 1 to only return a table with the most purchased item. 
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/31f3921f-a7d8-4a51-8d58-190f0241ee86)

### 5) Which item was the most popular for each customer?
```sql
WITH most_ordered AS(
  SELECT
    sales.customer_id, menu.product_name,
    COUNT(menu.product_id) AS order_count,
    DENSE_RANK() OVER(
      PARTITION BY sales.customer_id
      ORDER BY COUNT(sales.customer_id) DESC) AS rank
  FROM dannys_diner.menu
  INNER JOIN dannys_diner.sales
  ON sales.product_id = menu.product_id
  GROUP BY
  menu.product_name, sales.customer_id
)
SELECT
  customer_id, product_name, order_count
FROM
  most_ordered
WHERE rank = 1
```
### Thought Process:
Back with the WITH clause I created a temporary table to count each product that each customer purchased and ranked them using the DENSE_RANK() function to determine the most popular item for each customer. Instead of LIMIT 1, I added a WHERE rank = 1 to only return the most popular item for each customer. 
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/08f871c0-94dd-48a9-a946-78f228bcbc54)

### 6) Which item was purchased first by the customer after they became a member?
```sql
WITH member_purchase AS(
  SELECT
    members.customer_id, sales.product_id,
    DENSE_RANK() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
  ON members.customer_id = sales.customer_id
  AND sales.order_date > members.join_date
)
SELECT
customer_id, product_name
FROM members_purchase
INNER JOIN dannys_diner.menu
ON member_purchase.product_id = menu.product_id
WHERE rank = 1
ORDER BY
customer_id ASC
```
### Thought Process:
To focus on only the orders after each customer became a member I implemented sales.order_date > members.join_date after inner joining the members and sales table. Beside that, I followed the same DENSE_RANK() and OVER(PARTITION BY…) format as #5 but instead of focusing on the products, I focused and ranked by the purchase date.
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/d90171c3-b01f-41c5-9574-c445c12db38c)

### 7) Which item was purchased just before the customer became a member?
```sql
WITH before_member AS(
  SELECT
    members.customer_id, sales.product_id,
    ROW_NUMBER() OVER(
      PARTITION BY members.customer_id
      ORDER BY sales.order_date DESC) AS rank
  FROM dannys_diner.members
  INNER JOIN dannys_diner.sales
  ON members.customer_id = sales.customer_id
  AND sales.order_date < members.join_date
)
SELECT
customer_id, product_name
FROM before_member
INNER JOIN dannys_diner.menu
ON member_purchase.product_id = menu.product_id
WHERE rank = 1
ORDER BY
customer_id ASC
```
### Thought Process:
This question is similar to the question before, but they ask for the order before each customer becomes a member. I changed the first ORDER BY statement to DESC by sales.order_date but I kept on getting 2 final orders for customer A. Since I only needed their one final order, I had to switch the DENSE_RANK() function to ROW_NUMBER() function which simply returns the row in any order starting with 1. No row can have the same value unlike rank where there can be multiple.
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/b068b6d0-b487-4228-9727-55c596ff8cd1)

### 8) What is the total items and amount spent for each member before they became a member?
```sql
SELECT
sales.customer_id, COUNT(sales.product_id) AS total_items,
SUM(menu.price) AS total_cost
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
AND sales.order_date < members.join_date
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id
```
### Thought Process:
I used 2 inner joins for this question. One inner join is the exact same as the previous one where I filtered out everything after each customer became a member. The second inner join joins the sales and menu table to make the calculations for total items and amount spent possible. To calculate the total items I used the COUNT function and to calculate the amount spent I used the SUM function. 
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/f112641f-d341-46bf-8176-594b46810bb0)

### 9) If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
WITH point_calc AS(
  SELECT
  menu.product_id,
  CASE
    WHEN product_id = 1 THEN price * 20
    ELSE price * 10 END AS points
  FROM dannys_diner.menu
)
SELECT
sales.customer_id, SUM(point_calc.points) AS total_points
FROM dannys_diner.sales
INNER JOIN point_calc
ON sales.product_id = point_calc.product_id
GROUP BY sales.customer_id
ORDER BY sales.customer_id
```
### Thought Process:
I created a point calculator as seen in the WITH statement where they find the specific case to have double points when a customer buys sushi. END signifies the end of the case and we can alias the calculations AS points. From there I simply selected and sorted the points for each customer.
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/e86a99ec-e689-4150-a6ee-c0fee07dffa2)

### 10) In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
SELECT
sales.customer_id,
SUM(CASE
    WHEN menu.product_id = 1 THEN menu.price * 20
    WHEN sales.order_date between members.join_date AND join_date + 6 THEN
    20 * menu.price ELSE 10 * menu.price END) AS total_points
FROM dannys_diner.sales
INNER JOIN dannys_diner.members
ON sales.customer_id = members.customer_id
INNER JOIN dannys_diner.menu
ON sales.product_id = menu.product_id
AND sales.order_date <= '2021-01-31'
GROUP BY sales.customer_id
ORDER BY sales.customer_id
```
### Thought Process:
Here we build on our previous program by implementing extra calculations for when customers join the program. Customers get double points for a week after they joined the program. So after calculating double points for sushi like before, I added another WHEN clause where I check if sales.order_date is between the join_date and a week after that where they would earn double points. I also added an AND statement to check if the order_dates were in the month of January by filtering in only the dates on or before ‘2021-01-31’ by checking if the dates was before or on the last day of January. 
### Result:
![image](https://github.com/alexmorifusa/SQL/assets/137368881/6cf9f1a9-6a94-49a4-b8f1-1fb2f880e059)






