#  Case Study #1 - Danny's Diner :stew:

<img src="https://github.com/awasthishubhika/8-Week-SQL-Challenge/assets/83754715/1a08c033-54b4-4d7f-bb9e-d05863756a13" width="500" height="500" />

### Case Overview
* Danny, a lover of Japanese cuisine, opened a restaurant named Danny's Diner in 2021, focusing on sushi, curry, and ramen.
* In order to stay afloat he seeks assistance in order to understand his customers better
* Purpose of this analysis is to expand the existing loyalty program

#### Details of Data
* Danny has provided us with a sample of the data which has three tables namely,
    - `sales`
    - `menu`
    - `members`
* ERD or Entity Relationship Diagram is as below:

<img src="https://github.com/awasthishubhika/8-Week-SQL-Challenge/assets/83754715/a66f169f-d43a-4a57-9ce7-798c7bdca1f9" width="600" height="300" />

### Metadata

Table name: `sales` 📊
| Column Name | Description | 
|---|---|
|customer_id|unique identification number of customer|
|order_date|date of order|
|product_id|menu item ordered|



Table name: `menu` 📝
| Column Name | Description | 
|---|---|
|product_id|unique identification number of product|
|product_name|dish on the menu|
|price|price of the menu item|

Table name: `members` :star:
| Column Name | Description | 
|---|---|
|customer_id|unique identification number of customer|
|join_date|date of joining loyalty program|

### Solutions to Case Questions
🧰 PostgreSQL and MySQL

1. **What is the total amount each customer spent at the restaurant?**
```sql
SELECT sales.customer_id,
       SUM(price) AS total_sales
FROM sales AS sales
INNER JOIN menu AS menu ON sales.product_id = menu.product_id
GROUP BY sales.customer_id;
```
*Query Result:*
| customer_id | total_sales |
|---|---|
|A|76|
|B|74|
|C|36|

---

2. **How many days has each customer visited the restaurant?**
```sql
SELECT customer_id,
       COUNT(DISTINCT order_date) AS visits
FROM sales
GROUP BY customer_id;
```
*Query Result:*
| customer_id | visits |
|---|---|
|A|4|
|B|6|
|C|2|
---

3. **What was the first item from the menu purchased by each customer?**
```sql
SELECT *
FROM
  (SELECT *,
          RANK() OVER(PARTITION BY customer_id
                      ORDER BY order_date) AS ranking
   FROM sales) AS ranked_table
WHERE ranking=1;
```
*Query Result:*
| customer_id | order_date | product_id | ranking |
|---|---|---|---|
|A|2021-01-01|1|1|
|A|2021-01-01|2|1|
|B|2021-01-01|2|1|
|C|2021-01-01|3|1|
|C|2021-01-01|3|1|
---

4. **What is the most purchased item on the menu and how many times was it purchased by all customers?**
```sql
SELECT product_id,
       COUNT(order_date) AS frequency
FROM sales
GROUP BY product_id
ORDER BY frequency DESC
LIMIT 1;
```
*Query Result:*
| product_id | frequency |
|---|---|
|3|8|
---

5. **Which item was the most popular for each customer?**
```sql
WITH cte AS
  (SELECT customer_id,
          product_id,
          RANK() OVER(PARTITION BY customer_id
                      ORDER BY COUNT(order_date) DESC) AS ranking
   FROM sales
   GROUP BY customer_id,
            product_id)
SELECT *
FROM cte
WHERE ranking = 1;
```
*Query Result:*
| costumer_id | product_id | ranking|
|--|--|--|
|A|3|1|
|B|2|1|
|B|1|1|
|B|3|1|
|C|3|1|
---

6. **Which item was purchased first by the customer after they became a member?**
```sql
WITH cte AS
  (SELECT s.customer_id,
          m.join_date,
          s.order_date,
          s.product_id,
          RANK() OVER(PARTITION BY s.customer_id
                      ORDER BY s.order_date) AS ranking
   FROM members AS m
   INNER JOIN sales AS s ON m.customer_id = s.customer_id
   WHERE s.order_date > m.join_date)
SELECT *
FROM cte
WHERE ranking = 1;
```
*Query Result:*
| costumer_id | join_date | order_date | product_id | ranking|
|--|--|--|--|--|
|A|2021-01-07|2021-01-10|3|1|
|B|2021-01-09|2021-01-11|1|1|
 
---

7. **Which item was purchased just before the customer became a member?**
```sql
WITH cte AS
  (SELECT s.customer_id,
          m.join_date,
          s.order_date,
          s.product_id,
          RANK() OVER(PARTITION BY s.customer_id
                      ORDER BY s.order_date DESC) AS ranking
   FROM members AS m
   INNER JOIN sales AS s ON m.customer_id = s.customer_id
   WHERE s.order_date < m.join_date)
SELECT *
FROM cte
WHERE ranking = 1;
```
*Query Result:*
| costumer_id | join_date | order_date | product_id | ranking|
|--|--|--|--|--|
|A|2021-01-07|2021-01-01|1|1|
|A|2021-01-07|2021-01-01|2|1|
|B|2021-01-09|2021-01-04|1|1|
---

8. **What is the total items and amount spent for each member before they became a member?**
```sql
WITH cte AS
  (SELECT s.customer_id,
          s.order_date,
          m.product_id,
          m.product_name,
          m.price,
          mem.join_date
   FROM sales AS s
   INNER JOIN menu AS m ON s.product_id = m.product_id
   INNER JOIN members AS mem ON mem.customer_id = s.customer_id)
SELECT customer_id,
       SUM(price) AS total_order_value
FROM cte
WHERE join_date > order_date
GROUP BY customer_id;
```
*Query Result:*
| customer_id | total_order_value |
|--|--|
|A|25|
|B|40|

---

9. **If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?**
```sql
WITH cte AS
  (SELECT s.customer_id,
          CASE
              WHEN s.product_id=1 THEN SUM(m.price*20)
              ELSE SUM(m.price*10)
          END AS points
   FROM sales AS s
   INNER JOIN menu AS m ON s.product_id = m.product_id
   GROUP BY s.customer_id,
            s.product_id)
SELECT customer_id,
       SUM(points) as total_points
FROM cte
GROUP BY customer_id;
```
*Query Result:*
| customer_id | total_points |
|--|--|
|A|860|
|B|940|
|C|360|
---

10. **In the first week after a customer joins the program (including their join date) they earn 2x points
    on all items, not just sushi - how many points do customer A and B have at the end of January?**
```sql
WITH cte AS
  (SELECT s.customer_id,
          CASE
              WHEN s.order_date BETWEEN mem.join_date AND DATE_ADD(mem.join_date, INTERVAL 7 DAY) THEN SUM(m.price * 20)
              ELSE SUM(price * 10)
          END AS points
   FROM sales AS s
   INNER JOIN menu AS m ON s.product_id = m.product_id
   INNER JOIN members AS mem ON mem.customer_id = s.customer_id
   GROUP BY s.customer_id,
            s.order_date,
            mem.join_date)
SELECT customer_id,
       SUM(points) AS total_points
FROM cte
GROUP BY customer_id;
```
*Query Result:*
| customer_id | total_points |
|--|--|
|A|1270|
|B|960|
## FIN
