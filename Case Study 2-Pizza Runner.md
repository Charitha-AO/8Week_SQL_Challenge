# Case Study #2 - Pizza Runner

![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/cd0fee35-99d6-4334-8ef6-64b14d0a1f5f)



## Introduction
Danny caught his eye on “80s Retro Styling and Pizza Is The Future!” and launched a Pizza Runner, a Pizza delivery system.

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

Let's Explore the Schema for Insights

## Entity Relationship Diagram

![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/59db7e16-f32c-4f72-b68e-5963c35a96c8)



## Data Cleaning and Data Tranformation

**Table customer_orders BEFORE DATA CLEANING**

![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/f51eefa0-dfad-4f99-9f2a-88c3967ea6fc)


**Transforming exclusions Column**

* Change Null Values
```sql
UPDATE customer_orders
SET exclusions = CASE WHEN exclusions = 'null' 
     THEN NULL
     WHEN exclusions = ''
     THEN NULL
     END
WHERE exclusions = 'null'
OR exclusions = ''
```
**Transforming extras Column** 

* Change Null Values
```sql
UPDATE customer_orders
SET extras = CASE WHEN extras = 'null'  
    THEN NULL
    WHEN extras = ''
       THEN NULL
    END
WHERE extras = 'null'
OR extras = ''
```

**Table customer_orders AFTER DATA CLEANING**

![custorders](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/75e78797-1588-4963-9ec9-c3825c8a7ed7)



**Table runner_orders BEFORE DATA CLEANING**

![image](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/514b81ce-fc1e-4c68-adc8-e964cdd035e1)


**Transforming pickup_time Column**

* Change Null Values
```sql
UPDATE runner_orders
SET pickup_time = CASE WHEN pickup_time = 'null' 
    THEN NULL
    END
WHERE pickup_time = 'null'
```

**Transforming distance Column**

* Change Null Values
```sql
UPDATE runner_orders
SET distance = CASE WHEN distance = 'null' 
    THEN NULL
    END
WHERE distance = 'null'
```
* Remove ‘km’ at the end of rows in the distance column
```sql
UPDATE runner_orders
SET distance = REPLACE(distance, 'km', '')
```
* Trim the distance column
```sql
UPDATE runner_orders
SET distance = TRIM(distance)
```

**Transforming Duration Column**

* Change Null Values
```sql
UPDATE runner_orders
SET duration = CASE WHEN duration = 'null' 
    THEN NULL
    END
WHERE duration = 'null'
```
* Remove minutes/mins/minute added at the end of the duration column
```sql
UPDATE runner_orders
SET duration = SUBSTRING(duration, 1, 2)
```
* Trim the duration column
```sql
UPDATE runner_orders
SET duration = TRIM(duration)
```

**Transforming cancellation Column**

* Change Null Values
```sql
UPDATE runner_orders
SET cancellation = CASE WHEN cancellation = 'null'
    THEN NULL
    WHEN cancellation = '' 
    THEN NULL
    END
WHERE cancellation = 'null'
OR cancellation = ''
```

**Tranforming the data types of columns**
```sql
ALTER TABLE runner_orders_temp
	ALTER COLUMN pickup_time TYPE timestamp without time zone
	USING pickup_time::timestamp,
	ALTER COLUMN distance TYPE NUMERIC
	USING distance::numeric,
	ALTER COLUMN duration TYPE INT
	USING duration::integer 
```

**Table runner_orders AFTER DATA CLEANING**

![runnerorders](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/d8b5c497-aa38-4b17-bd50-0c5f090c2123)


## Case Study Queries and Solutions
* A Pizza Metrics

1. How many pizzas were ordered?
```sql
SELECT COUNT(order_id) AS total_pizza_orders
FROM customer_orders
```
Answer :

![2a1](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/31e5e950-0e62-434f-8672-df8b6f212d8d)


2. How many unique customer orders were made?
```sql
SELECT COUNT(DISTINCT order_id) AS unique_customer_orders
FROM customer_orders
```
Answer :

![2a2](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/d1d463bc-21ce-4a1d-b133-0192d2509a93)


3. How many successful orders were delivered by each runner?
```sql
SELECT runner_id,COUNT(DISTINCT order_id) AS successful_deliveries
FROM runner_orders
WHERE cancellation IS NULL 
GROUP BY runner_id
```
Answer :

![2a3](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/f3c25deb-d466-4f85-a2dc-9fbb4ffdf09d)


4. How many of each type of pizza was delivered?
```sql
SELECT p.pizza_id,p.pizza_name,COUNT(c.pizza_id) as pizza_delivered_count
FROM customer_orders c
LEFT JOIN pizza_names p ON c.pizza_id = p.pizza_id
LEFT JOIN runner_orders r ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY p.pizza_id,p.pizza_name
ORDER BY pizza_delivered_count DESC
```
Answer :

![2a4](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/0669af2c-1847-4129-a4cb-11475f86b161)


5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT  c.customer_id,
	SUM(CASE WHEN c.pizza_id = 1 THEN 1 ELSE 0 END) Meatlovers,
	SUM(CASE WHEN c.pizza_id = 2 THEN 1 ELSE 0 END) Vegetarian
FROM pizza_names p
LEFT JOIN customer_orders c ON p.pizza_id = c.pizza_id
GROUP BY c.customer_id
ORDER BY c.customer_id
```
Answer :

![2a5](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/2277b6bc-ff7c-43f2-b41f-9cce67a0014f)


6. What was the maximum number of pizzas delivered in a single order?
```sql
WITH cte as(SELECT c.order_id,COUNT(c.pizza_id) AS order_count
			FROM customer_orders c
			JOIN runner_orders r ON c.order_id = r.order_id
            GROUP BY c.order_id)
SELECT max(order_count) AS max_pizza_deivered FROM cte
```
Answer :

![2a6](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/115a3402-9384-4d2e-9934-66e6795fdc8a)


7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT c.customer_id,SUM(CASE WHEN c.exclusions IS NOT NULL OR c.extras IS NOT NULL THEN 1
		ELSE 0 END) at_least_1_change,
	SUM(CASE WHEN c.exclusions IS NULL AND c.extras IS NULL THEN 1
		ELSE 0 END) no_change
FROM customer_orders c
LEFT JOIN runner_orders r ON r.order_id = c.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
ORDER  BY c.customer_id
```
Answer :

![2a7](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/46bdc46b-bf7c-4555-b945-dfee8c589abc)


8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT SUM(CASE WHEN c.exclusions != '' AND c.extras != '' THEN 1
		ELSE 0 END) exclusions_extras_pizza_count
FROM customer_orders c
LEFT JOIN runner_orders r ON r.order_id = c.order_id
WHERE r.cancellation IS NULL
```
Answer :

![2a8](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/0d5242e0-d663-49e0-8ece-77b4286cc78c)


9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT EXTRACT(hours FROM order_time) AS hours,COUNT(order_id) AS pizza_ordered_by_hours
FROM customer_orders
GROUP BY hours
ORDER BY hours

```
Answer :

![2a9](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/f1fef9e4-0333-451a-8e75-8a150e103378)

10. What was the volume of orders for each day of the week?
```sql
SELECT to_char(order_time, 'DAY') AS day,COUNT(order_id) AS pizza_ordered_by_day
FROM customer_orders
GROUP BY day
```
Answer :

![2a10](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/90abedea-3bed-4e16-859d-2e91cc252f65)


* B Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT to_char(registration_date::date, 'ww')::int AS weeks,COUNT(runner_id)AS runner_week
FROM runners
GROUP BY weeks
ORDER BY weeks
```
Answer:

![2b1](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/feddde66-eafc-491a-bc94-9ab00ad5e9c5)

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH runners_pickup_time AS (
  SELECT r.runner_id,c.order_id, c.order_time, r.pickup_time, 
  EXTRACT(minutes from r.pickup_time - c.order_time) AS pickup_minutes
  FROM customer_orders AS c
  JOIN runner_orders AS r
  ON c.order_id = r.order_id
  WHERE r.cancellation IS NULL
  GROUP BY r.runner_id, c.order_id, c.order_time, r.pickup_time
)
SELECT runner_id,ROUND(AVG(pickup_minutes)) AS average_time
FROM runners_pickup_time
GROUP BY runner_id
ORDER BY runner_id
```
Answer:

![2b2](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/4de54b7a-3336-4cd0-9906-060f75ca3b17)


3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH cte AS
(SELECT c.order_id, COUNT(c.order_id) AS count_order, c.order_time, r.pickup_time,
 EXTRACT(minutes FROM pickup_time - order_time) AS pickup_minutes 
FROM customer_orders c 
JOIN runner_orders r 
ON c.order_id = r.order_id
WHERE cancellation IS NULL
GROUP BY c.order_id, c.order_time, r.pickup_time)
SELECT cte.count_order,ROUND(AVG(pickup_minutes)) AS avg_prep_time
FROM cte 
WHERE pickup_minutes >= 1 
GROUP BY 1
ORDER BY 1
```
Answer:

![2b3](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/317e217f-6263-407f-b686-606de3b7a9c5)

4. What was the average distance travelled for each customer?
```sql
SELECT c.customer_id, ROUND(AVG(distance)) AS average_distance
FROM customer_orders c 
JOIN runner_orders r ON c.order_id=r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
ORDER BY c.customer_id
```

Answer:

![2b4](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/541725b7-08f4-4e04-98fd-ad570685ec47)


5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT MAX(duration) - MIN(duration) AS diff_delivery_time
FROM runner_orders
```

Answer:

![2b5](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/6001df22-2445-44f1-8ba5-19d7d7072720)

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT DISTINCT r.runner_id,c.order_id,r.distance,r.duration,
round((r.distance/r.duration*60),2) AS speed
FROM customer_orders c
LEFT JOIN runner_orders r ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.order_id, r.runner_id, r.distance, r.duration
ORDER BY r.runner_id
```

Answer:

![2b6](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/9824c511-2a43-4b4b-80aa-657d8a69444f)

7. What is the successful delivery percentage for each runner?
```sql
SELECT r.runner_id, ROUND(100 * SUM(CASE WHEN r.cancellation IS NULL THEN 1 ELSE 0 END)/COUNT(*)) percentage
FROM runner_orders r
GROUP BY r.runner_id
ORDER BY r.runner_id
```
Answer:

![2b7](https://github.com/Charitha-AO/8Week_SQL_Challenge/assets/86000133/bf6b8b51-5059-4363-abec-f70583d07ec9)
