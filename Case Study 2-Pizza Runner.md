# Case Study #2 - Pizza Runner





## Introduction
Danny caught his eye on “80s Retro Styling and Pizza Is The Future!” and launched a Pizza Runner, a Pizza delivery system.

Danny started by recruiting “runners” to deliver fresh pizza from Pizza Runner Headquarters (otherwise known as Danny’s house) and also maxed out his credit card to pay freelance developers to build a mobile app to accept orders from customers.

Because Danny had a few years of experience as a data scientist - he was very aware that data collection was going to be critical for his business’ growth.

Let's Explore the Schema for Insights

## Entity Relationship Diagram

## Data Cleaning and Data Tranformation

**Table customer_orders BEFORE DATA CLEANING**




* Transforming exclusions Column
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
* Transforming extras Column
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







**Table runner_orders BEFORE DATA CLEANING**




**Transforming pickup_time Column**

* Change Null Values
```sql
UPDATE runner_orders
SET pickup_time = CASE WHEN pickup_time = 'null' 
    THEN NULL
    END
WHERE pickup_time = 'null'
```

**Transforming Distance Column**

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


