**B. Runner and Customer Experience**

**1.How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)**
````SQL
    SELECT
    	DATE_PART('week',registration_date) AS week,
        COUNT(runner_id) AS runners_signed_up
    FROM pizza_runner.runners
    GROUP BY week;
````
| week | runners_signed_up |
| ---- | ----------------- |
| 53   | 2                 |
| 1    | 1                 |
| 2    | 1                 |

---
**2.What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?**
````SQL
    SELECT AVG(time_to_pickup) AS avg_time_to_pickup
    FROM
        (SELECT
         	c.order_id,
         	ABS(DATE_PART('minute',c.order_time-r.pickup_time)) AS 					time_to_pickup
         FROM customer_orders_temp AS c
         JOIN runner_orders_temp AS r
         	ON c.order_id=r.order_id
        	WHERE r.distance!=0) AS pickup_table;
````
| avg_time_to_pickup |
| ------------------ |
| 18.25              |

---
**3.Is there any relationship between the number of pizzas and how long the order takes to prepare?**
````SQL
    SELECT
    	number_pizzas,
    	AVG(prepare_time) AS time_to_prepare
    FROM
    	(SELECT
         	c.order_id,
         	COUNT(c.order_id) AS number_pizzas,
         	ABS(DATE_PART('minute',r.pickup_time-c.order_time)) AS 					prepare_time
         FROM customer_orders_temp AS c
         JOIN runner_orders_temp AS r
         	ON c.order_id=r.order_id
         WHERE r.pickup_time IS NOT NULL
         GROUP BY c.order_id, c.order_time, r.pickup_time ) AS prepare_time_table
    GROUP BY number_pizzas
    ORDER BY number_pizzas;
````
| number_pizzas | time_to_prepare |
| ------------- | --------------- |
| 1             | 12              |
| 2             | 18              |
| 3             | 29              |

---
**4.What was the average distance travelled for each customer?**
````SQL
    SELECT
    	c.customer_id,
        AVG(distance) AS avg_distance
    FROM customer_orders_temp AS c
    JOIN runner_orders_temp AS r
    	ON c.order_id=r.order_id
    WHERE distance!=0
    GROUP BY c.customer_id
    ORDER BY c.customer_id;
````
| customer_id | avg_distance       |
| ----------- | ------------------ |
| 101         | 20                 |
| 102         | 16.733333333333334 |
| 103         | 23.399999999999995 |
| 104         | 10                 |
| 105         | 25                 |

---
**5.What was the difference between the longest and shortest delivery times for all orders?**
````SQL
    SELECT
    	MAX(prepare_time)-MIN(prepare_time) AS delivery_times
    FROM
    	(SELECT
         	c.order_id,
         	ABS(DATE_PART('minute',r.pickup_time-c.order_time)) AS 					prepare_time
         FROM customer_orders_temp AS c
         JOIN runner_orders_temp AS r
         	ON c.order_id=r.order_id
         WHERE r.pickup_time IS NOT NULL
         GROUP BY c.order_id, c.order_time, r.pickup_time ) AS prepare_time_table;
````
| delivery_times |
| -------------- |
| 19             |

---
**6.What was the average speed for each runner for each delivery and do you notice any trend for these values?**
````SQL
    SELECT
    	r.runner_id,
        c.customer_id,
        r.order_id,
        COUNT(pizza_id) AS pizza_count,
        r.distance,
        r.duration/60 AS duration_hour,
        r.distance/r.duration*60 AS avg_speed
    FROM customer_orders_temp AS c
    JOIN runner_orders_temp AS r
      ON c.order_id=r.order_id
    WHERE r.duration!=0 OR r.duration!=0
    GROUP BY r.runner_id,c.customer_id,r.order_id,r.distance,r.duration
    ORDER BY r.runner_id, avg_speed;
````
| runner_id | customer_id | order_id | pizza_count | distance | duration_hour | avg_speed          |
| --------- | ----------- | -------- | ----------- | -------- | ------------- | ------------------ |
| 1         | 101         | 1        | 1           | 20       | 0             | 37.5               |
| 1         | 102         | 3        | 2           | 13.4     | 0             | 40.2               |
| 1         | 101         | 2        | 1           | 20       | 0             | 44.44444444444444  |
| 1         | 104         | 10       | 2           | 10       | 0             | 60                 |
| 2         | 103         | 4        | 3           | 23.4     | 0             | 35.099999999999994 |
| 2         | 105         | 7        | 1           | 25       | 0             | 60                 |
| 2         | 102         | 8        | 1           | 23.4     | 0             | 93.6               |
| 3         | 104         | 5        | 1           | 10       | 0             | 40                 |

---
**7.What is the successful delivery percentage for each runner?**
````SQL
    SELECT
    	runner_id,
    	ROUND(100*SUM(CASE WHEN distance=0 THEN 0 ELSE 1 END)/COUNT(order_id),2)AS perc_successful
    FROM runner_orders_temp
    GROUP BY runner_id
    ORDER BY runner_id;
````
| runner_id | perc_successful |
| --------- | --------------- |
| 1         | 100.00          |
| 2         | 75.00           |
| 3         | 50.00           |

---

[View on DB Fiddle](https://www.db-fiddle.com/f/7VcQKQwsS3CTkGRFG7vu98/65)
