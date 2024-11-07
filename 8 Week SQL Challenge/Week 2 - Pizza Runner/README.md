## Pizza Runner
### Data Cleaning Part

1) Update customer_orders

```sql
SELECT 
	order_id,
    customer_id,
    pizza_id,
    CASE 
		WHEN exclusions = 'null' OR exclusions = '' THEN NULL
        ELSE exclusions END AS exclusions,
	CASE 
		WHEN extras = 'null' OR extras = '' THEN NULL
        ELSE extras END AS extras,
	order_time
FROM customer_orders;

UPDATE customer_orders
SET exclusions = CASE 
		WHEN exclusions = 'null' OR exclusions = '' THEN NULL
        ELSE exclusions END;

UPDATE customer_orders
SET extras = CASE 
		WHEN extras = 'null' OR extras = '' THEN NULL
        ELSE extras END;
```

2) Update runner_orders

```sql
UPDATE runner_orders
SET 
duration = CASE
			WHEN duration = 'null' THEN NULL
            WHEN duration LIKE '%mins' THEN REPLACE (duration, 'mins', '')
            WHEN duration LIKE '% mins' THEN REPLACE (duration, ' mins', '')
			WHEN duration LIKE '%minute' THEN REPLACE (duration, 'minute', '')
            ELSE duration
            END,
            
cancellation = CASE
				WHEN cancellation = 'null' OR cancellation = '' THEN NULL
                ELSE cancellation
                END;
```

3) Update pickup_time

```sql
ALTER TABLE runner_orders
ALTER COLUMN pickup_time TYPE TIMESTAMP USING pickup_time::timestamp;

UPDATE runner_orders
SET pickup_time = TO_TIMESTAMP(pickup_time, 'YYYY-MM-DD HH24:MI:SS');
```

4) Update distance

```sql
select 
	co.customer_id,
	round(avg(replace(distance, 'km', '')::numeric(3, 1)), 1) as avg_distance
from runner_orders as ro
inner join customer_orders as co on co.order_id = ro.order_id
where distance<>'null'
group by co.customer_id
order by co.customer_id;

ALTER TABLE runner_orders
ALTER COLUMN distance TYPE FLOAT USING distance::float;

-- duration
ALTER TABLE runner_orders
ALTER COLUMN duration TYPE INTEGER USING duration::int;
```

5) Update runner_orders

```sql
UPDATE pizza_runner.runner_orders
SET
  distance = CASE
               WHEN distance LIKE 'null' THEN ' '
               WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
               ELSE distance 
             END,
  duration = CASE
               WHEN duration LIKE 'null' THEN ' '
               WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
               WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
               WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
               ELSE duration
             END,
  cancellation = CASE
                   WHEN cancellation IS NULL OR cancellation = 'null' THEN ' '
                   ELSE cancellation
                 END;
```

PART A. PIZZA METRICS

1) How many pizzas were ordered?

```sql
SELECT 
	COUNT(pizza_id) AS ordered_pizza_count
FROM customer_orders;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A1.PNG)

2) How many unique customer orders were made?

```sql
SELECT 
	COUNT(DISTINCT order_id) AS unique_order_count
FROM customer_orders;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A2.PNG)

3) How many successful orders were delivered by each runner?

```sql
SELECT 
	runner_id,
    COUNT(order_id)
FROM runner_orders
WHERE cancellation IS NOT NULL
GROUP BY runner_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A3.PNG)

4) How many of each type of pizza was delivered?

```sql
SELECT 
	pizza_names.pizza_name AS pizza_type,
    COUNT(pizza_names.pizza_id)
FROM runner_orders
	LEFT JOIN customer_orders ON runner_orders.order_id = customer_orders.order_id
	LEFT JOIN pizza_names ON customer_orders.pizza_id = pizza_names.pizza_id
WHERE cancellation IS NOT NULL
GROUP BY pizza_names.pizza_name;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A4.PNG)

5) How many Vegetarian and Meatlovers were ordered by each customer?

```sql
SELECT 
	customer_id,
    COUNT(CASE WHEN pizza_names.pizza_id = 1 THEN pizza_names.pizza_id ELSE NULL END) AS Meatlovers,
    COUNT(CASE WHEN pizza_names.pizza_id = 2 THEN pizza_names.pizza_id ELSE NULL END) AS Vegetarian
FROM customer_orders
	LEFT JOIN pizza_names ON customer_orders.pizza_id = pizza_names.pizza_id
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A6.PNG)

6) What was the maximum number of pizzas delivered in a single order?

```sql
SELECT 
	order_id,
    COUNT(pizza_id)
FROM customer_orders
GROUP BY order_id
ORDER BY COUNT(pizza_id) DESC
LIMIT 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A6.PNG)

7) For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

```sql
SELECT 
	customer_id,
    COUNT(CASE WHEN exclusions IS NULL AND extras IS NULL THEN pizza_id ELSE NULL END) AS no_change,
    COUNT(CASE WHEN exclusions IS NOT NULL OR extras IS NOT NULL THEN pizza_id ELSE NULL END) AS changes_made
FROM customer_orders
	LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NOT NULL
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A7.PNG)

8) How many pizzas were delivered that had both exclusions and extras?

```sql
SELECT 
	customer_id,
    COUNT(CASE WHEN exclusions IS NOT NULL AND extras IS NOT NULL THEN pizza_id ELSE NULL END) AS both_changes_made
FROM customer_orders
	LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NOT NULL
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A8.PNG)

9) What was the total volume of pizzas ordered for each hour of the day?

```sql
SELECT 
	EXTRACT(HOUR FROM order_time) AS order_hour,
    COUNT(pizza_id)
FROM customer_orders
GROUP BY EXTRACT(HOUR FROM order_time)
ORDER BY order_hour;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A9.PNG)

10) What was the volume of orders for each day of the week?

```sql
SELECT 
	TO_CHAR(order_time, 'Day') AS order_day,
    COUNT(pizza_id)
FROM customer_orders
GROUP BY order_day
ORDER BY order_day; 
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/A10.PNG)


PART B. RUNNER AND CUSTOMER EXPERIENCE

1) How many runners signed up for each 1 week period?

```sql
SELECT
	DATE_TRUNC('week', registration_date) AS week,
    COUNT(runner_id)
FROM runners
GROUP BY week;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B1.PNG)

2) What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

```sql
SELECT
    runner_id,
    ROUND(AVG(EXTRACT(MINUTE FROM (pickup_time - order_time))), 1) AS avg_pickup_duration
FROM customer_orders
	LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE cancellation IS NOT NULL
GROUP BY runner_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B2.PNG)

3) Is there any relationship between the number of pizzas and how long the order takes to prepare?

```sql
with order_counts as (
    select 
        customer_orders.order_id,
        count(customer_orders.order_id) as pizza_count,
        avg(EXTRACT(EPOCH FROM (runner_orders.pickup_time - customer_orders.order_time)) / 60) as pickup_duration
    from customer_orders
    left join runner_orders on customer_orders.order_id = runner_orders.order_id
    where cancellation is not null
    group by customer_orders.order_id
)

select 
    pizza_count,
    count(pizza_count) as num_orders,
    round(avg(pickup_duration), 1) as avg_pickup_duration
from order_counts
group by pizza_count
order by pizza_count;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B3.PNG)

4) What was the average distance travelled for each customer?

```sql
select 
	co.customer_id,
	round(avg(distance)) as avg_distance
from runner_orders as ro
inner join customer_orders as co on co.order_id = ro.order_id
group by co.customer_id
order by co.customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B4.PNG)

5) What was the difference between the longest and shortest delivery times for all orders?

```sql
select
	regexp_replace(duration, '[^0-9]', ''),
	*
from runner_orders;

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN distance TYPE FLOAT USING distance::FLOAT;

ALTER TABLE pizza_runner.runner_orders
ALTER COLUMN duration TYPE FLOAT USING duration::FLOAT;

select
    min(duration) as minimum_duration,
    max(duration) as maximum_duration,
    max(duration) - min(duration) as maximum_difference
from runner_orders;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B5.PNG)

6) What was the average speed for each runner for each delivery and do you notice any trend for these values?

```sql
select
    runner_id,
    distance as distance_km,
    round(duration / 60.0) as duration_hr,
    round(distance * 60.0 / duration) as average_speed
from runner_orders
where cancellation is not null
order by runner_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B6.PNG)

7) What is the successful delivery percentage for each runner?

```sql
with order_states as (
    select 
        runner_id,
        count(case when distance is not null then runner_id end) as successful_orders,
        count(case when distance is null then runner_id end) as failed_orders,
        count(runner_id) as total_orders
    from runner_orders
    group by runner_id
)

select 
    runner_id,
    round(successful_orders::decimal / total_orders * 100, 2) as success_percentage
from order_states
order by runner_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/B7.PNG)

C. INGREDIENT OPTIMIZATION

1) What are the standard ingredients for each pizza?

```sql
CREATE TABLE pizza_toppings_adjusted AS
with updated_toppings as
(
SELECT 
    pizza_recipes.pizza_id, 
    unnest(string_to_array(replace(toppings, ' ', ''), ','))::integer AS topping_id,
	pizza_name
FROM
    pizza_recipes
join pizza_names 
	on pizza_recipes.pizza_id = pizza_names.pizza_id
)

select
	pizza_id,
	updated_toppings.topping_id,
	pizza_toppings.topping_name,
	pizza_name
from updated_toppings
	left join pizza_toppings
		on updated_toppings.topping_id = pizza_toppings.topping_id
order by
	pizza_id,
	updated_toppings.topping_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/C1.PNG)

2) What was the most commonly added extra?

```sql
with adjusted_ingredients as
(
select 
	order_id,
	customer_id,
	pizza_id,
	exclusions,
	extras,
	unnest(string_to_array(exclusions, ',')) as exclusions_adjusted,
	unnest(string_to_array(extras, ',')) as extras_adjusted,
	order_time
from pizza_runner.customer_orders
)

select
	extras_adjusted as extra_id,
	count(extras_adjusted) as extras_count
from adjusted_ingredients
group by extras_adjusted
order by extras_count desc
limit 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/C2.PNG)

3) What was the most common exclusion?

```sql
with adjusted_ingredients as
(
select 
	order_id,
	customer_id,
	pizza_id,
	exclusions,
	extras,
	unnest(string_to_array(exclusions, ',')) as exclusions_adjusted,
	unnest(string_to_array(extras, ',')) as extras_adjusted,
	order_time
from pizza_runner.customer_orders
)

select
	exclusions_adjusted as exclusion_id,
	count(exclusions_adjusted) as exclusions_count
from adjusted_ingredients
group by exclusions_adjusted
order by exclusions_count desc
limit 1
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/C3.PNG)

D. PRICING AND RATINGS

1) If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes 
how much money has Pizza Runner made so far if there are no delivery fees?

```sql
WITH pizza_revenues AS (
    SELECT
        customer_orders.order_id,
        pizza_id,
        CASE 
            WHEN pizza_id = 1 THEN 12
            ELSE 10
        END AS pizza_revenue
    FROM customer_orders
    LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
    WHERE cancellation IS NOT NULL
)
SELECT 
    SUM(pizza_revenue) AS total_revenue
FROM pizza_revenues;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/D1.PNG)

2) What if there was an additional $1 charge for any pizza extras?

```sql
WITH pizza_revenues AS (
    SELECT
        customer_orders.order_id,
        pizza_id,
        exclusions,
        extras,
        CASE 
            WHEN pizza_id = 1 THEN 12
            ELSE 10
        END AS pizza_revenue
    FROM customer_orders
    LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
    WHERE cancellation IS NOT NULL
)
SELECT 
    SUM(CASE 
            WHEN extras IS NOT NULL THEN pizza_revenue + 1
            ELSE pizza_revenue 
        END) AS total_revenue
FROM pizza_revenues;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/D2.PNG)

3) The Pizza Runner team now wants to add an additional ratings system that allows
customers to rate their runner, how would you design an additional table for this new dataset 
generate a schema for this new table and insert your own data for ratings for each successful 
customer order between 1 to 5.

```sql
CREATE TABLE runner_ratings (
    order_id INTEGER,
    rating INTEGER CHECK (rating BETWEEN 1 AND 5),
    review VARCHAR(100)
);

-- Insert sample data into the runner_ratings table

INSERT INTO runner_ratings (order_id, rating, review) 
VALUES
    (1, 1, 'Really bad service'),
    (2, 1, NULL),
    (3, 4, 'Took too long...'),
    (4, 1, 'Runner was lost, delivered it AFTER an hour. Pizza arrived cold'),
    (5, 2, 'Good service'),
    (7, 5, 'It was great, good service and fast'),
    (8, 2, 'He tossed it on the doorstep, poor service'),
    (10, 5, 'Delicious!, he delivered it sooner than expected too!');

-- Display data from the runner_ratings table
SELECT * FROM runner_ratings;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/D3.PNG)

4) Using your newly generated table - can you join all of the information together 
to form a table which has the following information for successful deliveries?

```sql
-- customer_id
-- order_id
-- runner_id
-- rating
-- order_time
-- pickup_time
-- Time between order and pickup
-- Delivery duration
-- Average speed
-- Total number of pizzas

SELECT 
    customer_orders.customer_id,
    customer_orders.order_id,
    runner_orders.runner_id,
    runner_ratings.rating,
    customer_orders.order_time,
    runner_orders.pickup_time,
    ROUND(EXTRACT(EPOCH FROM (runner_orders.pickup_time - customer_orders.order_time)) / 60, 2) AS time_btw_order_and_pickup, -- Minutes between order and pickup
    runner_orders.duration,
    COUNT(customer_orders.pizza_id) AS total_number_of_pizzas
FROM customer_orders
LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
LEFT JOIN runner_ratings ON customer_orders.order_id = runner_ratings.order_id
WHERE runner_orders.cancellation IS NOT NULL
GROUP BY 
    customer_orders.customer_id, 
    customer_orders.order_id, 
    runner_orders.runner_id, 
    runner_ratings.rating, 
    customer_orders.order_time, 
    runner_orders.pickup_time, 
    runner_orders.duration;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%202%20-%20Pizza%20Runner/Screenshots/D4.PNG)

5) If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost 
for extras and each runner is paid $0.30 per kilometre traveled - how much money 
does Pizza Runner have left over after these deliveries?

```sql
SELECT
    ROUND(SUM(CASE 
                WHEN pizza_id = 1 THEN 12 ELSE 10 END)
                SUM(3 / 10 * runner_orders.distance, 2)) AS total_profit
FROM customer_orders
LEFT JOIN runner_orders ON customer_orders.order_id = runner_orders.order_id
WHERE runner_orders.cancellation IS NOT NULL;
```
