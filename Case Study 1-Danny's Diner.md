# Case Study #1 - Danny's Diner

![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/59b0c939-2af0-4915-ae31-5473a1801735)



## Introduction
In the beginning of 2021, Danny pursued his love for Japanese cuisine by launching "Danny's Diner," a delightful restaurant specializing in sushi, curry, and ramen. Despite gathering essential data during its initial months, the restaurant faced challenges in utilizing this information effectively to drive informed business decisions. Seeking assistance, Danny's Diner aims to harness its data to ensure the continued success and growth of the restaurant


## Problem Statement
Danny wants to gain valuable insights from the customers visiting patterns, spending habits, and favorite menu items. By establishing a deeper connection with his customers, he can provide a more personalized experience for his loyal patrons.


He plans to use these insights to make informed decisions about expanding the existing customer loyalty program. Additionally, Danny seeks assistance in generating basic datasets for his team to inspect the data conveniently, without requiring SQL expertise.


Due to privacy concerns, he has shared a sample of his overall customer data, hoping it will be sufficient for you to create fully functional SQL queries to address his questions.


Danny has shared with you 3 key datasets for this case study:
* sales
* menu
* members

## Entity Relationship Diagram


![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/42064e3e-d47b-4146-9f85-f38237008b53)



## Case Study Queries and Solutions
1. What is the total amount each customer spent at the restaurant?
   ```sql
   SELECT sales.customer_id,SUM(menu.price) AS total_spent FROM sales
   JOIN menu ON sales.product_id = menu.product_id
   GROUP BY sales.customer_id
   ORDER BY sales.customer_i;
   ```
Answer:

![1-1](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/983c3565-bb27-46f0-88cf-c829a0404ffe)




2. How many days has each customer visited the restaurant?
```sql
   SELECT customer_id,COUNT(DISTINCT order_date) AS no_of_days_visisted
   FROM sales
   GROUP BY customer_id
```
Answer:

![1-2](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/9df11e1a-546c-4e08-ae75-a81fd2430c62)




3. What was the first item from the menu purchased by each customer?
```sql
   WITH first_item_purchased AS
   (SELECT sales.customer_id,menu.product_name,DENSE_RANK() OVER(PARTITION BY       customer_id ORDER BY order_date) AS denserank 
   FROM sales JOIN menu ON menu.product_id=sales.product_id)
   SELECT customer_id,product_name FROM first_item_purchased 
   WHERE denserank=1
```
Answer:

![1-3](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/12bc5acf-91d0-4db3-8802-02836c93aeee)



4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
   SELECT menu.product_name,COUNT(sales.product_id) AS most_purchased FROM sales
   JOIN menu ON sales.product_id = menu.product_id
   GROUP BY menu.product_name
   ORDER BY most_purchased DESC
   LIMIT 1
```
Answer:

![1-4](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/471b2231-3ce1-4f16-acd3-f7906ede3a2d)



5. Which item was the most popular for each customer?
```sql
   WITH most_popular_item AS
   (SELECT sales.customer_id,menu.product_name,COUNT(sales.product_id) AS count_of_order,
      DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(sales.product_id) DESC) AS drnk 
   FROM sales
   JOIN menu ON sales.product_id = menu.product_id
   GROUP BY sales.customer_id,menu.product_name)
   SELECT customer_id,product_name,count_of_order FROM most_popular_item
   WHERE drnk = 1
```
Answer:

![1-5](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/d65c6769-477d-470d-855c-990e22f718f0)




6. Which item was purchased first by the customer after they became a member?
```sql
   WITH cte AS
   (SELECT s.customer_id,s.product_id,s.order_date,m.join_date,
	   DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date) AS drank 
   FROM sales s
   JOIN members m ON s.customer_id=m.customer_id
   WHERE s.order_date > m.join_date)
   SELECT customer_id,product_name,order_date FROM cte
   JOIN menu ON cte.product_id = menu.product_id
   WHERE drank = 1
   ORDER BY customer_Id
``` 
Answer:

![1-6](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/a4781fb7-dabb-443f-a8e0-5be8292c6587)




7. Which item was purchased just before the customer became a member?
```sql
   WITH cte AS
   (SELECT s.customer_id,s.product_id,s.order_date,m.join_date,
	   DENSE_RANK() OVER(PARTITION BY s.customer_id ORDER BY s.order_date DESC) AS drank 
   FROM sales s
   JOIN members m ON s.customer_id=m.customer_id
   WHERE s.order_date < m.join_date)
   SELECT customer_id,product_name,order_date FROM cte
   JOIN menu ON cte.product_id = menu.product_id
   WHERE drank = 1
   ORDER  BY customer_id
```
Answer:

![1-7](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/72db60a8-0e32-49b5-b2ec-259eeaeaf7b5)




8. What is the total items and amount spent for each member before they became a member?
```sql
   SELECT s.customer_id,SUM(price) FROM sales s 
   JOIN menu ON s.product_id=menu.product_id
   JOIN members m ON m.customer_id=s.customer_id 
   WHERE s.order_date < m.join_date
   GROUP BY s.customer_id
   ORDER BY s.customer_id
```
Answer:

![1-8](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/8dc104d1-0eb6-4c19-9cea-010f6dbd5769)




9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
   WITH cte AS
      (SELECT s.customer_id,CASE WHEN menu.product_name='sushi' THEN price*20 ELSE price* 10 END AS points FROM sales s
      JOIN menu ON s.product_id = menu.product_id) 
   SELECT customer_id,SUM(points) FROM cte
   GROUP BY customer_id
   ORDER BY customer_id
```
Answer:

![1-9](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/246b9841-69a7-4469-b476-3c0f6660d7a4)



10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
   WITH cte AS 
   (SELECT s.customer_id,
   CASE WHEN m.product_name = 'sushi' OR s.order_date BETWEEN mb.join_date - 1 AND mb.join_date + 6 THEN m.price * 20
      ELSE m.price * 10
      END AS points
   FROM members mb
   LEFT JOIN sales s ON s.customer_id = mb.customer_id
   LEFT JOIN menu m ON s.product_id = m.product_id
   WHERE s.order_date <= '2021-01-31'
   )
   SELECT customer_id, SUM(points) AS points
   FROM cte
   GROUP BY customer_id
```
Answer:

![1-10](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/b359eb65-3e9a-4a1e-a6ac-51e7a2bef336)




## Bonus Queries and Solutions
1. Join All The Things
```sql
   WITH customer_member AS
   (SELECT s.customer_id,s.order_date,m.product_name,m.price,CASE WHEN mb.join_date <= s.order_date THEN 'Y' ELSE 'N' END AS member_or_not
   FROM sales s 
   LEFT JOIN menu m ON s.product_id=m.product_id
   LEFT JOIN members mb ON s.customer_id=mb.customer_id)
   SELECT customer_id,order_date,product_name,price,member_or_not FROM customer_member
   ORDER BY customer_id,member_or_not,order_date
```
Answer:

![1b1](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/bfff3e84-5a22-4d00-baa9-e789a2fa857d)



2. Rank All The Things
```sql
   WITH customer_rank AS
   (SELECT s.customer_id,s.order_date,m.product_name,m.price,CASE WHEN mb.join_date <= s.order_date THEN 'Y' ELSE 'N' END AS member_or_not
   FROM sales s 
   LEFT JOIN menu m ON s.product_id=m.product_id
   LEFT JOIN members mb ON s.customer_id=mb.customer_id
   ORDER BY customer_id,member_or_not,order_date)
   SELECT *,CASE WHEN member_or_not = 'N' THEN NULL 
   ELSE DENSE_RANK() OVER(PARTITION BY customer_id,member_or_not ORDER BY order_date) END AS ranking
   FROM customer_rank
```
Answer:

![1b2](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/eeb0154b-eb1a-47dd-baf4-ac6095b9bb21)






