## Danny's Diner
 
1. What is the total amount each customer spent at the restaurant?

```sql
SELECT
    customer_id,
    SUM(price) AS total_spent
FROM sales
    LEFT JOIN menu ON sales.product_id = menu.product_id
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/1.PNG)

2. How many days has each customer visited the restaurant?

```sql
SELECT 
    customer_id,
    COUNT(DISTINCT order_date) AS visit_count
FROM sales
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/2.PNG)

3. What was the first item from the menu purchased by each customer?

```sql
WITH first_purchase AS (
    SELECT
        customer_id,
        product_name,
        order_date,
        DENSE_RANK() OVER(PARTITION BY customer_id ORDER BY order_date) AS dr
    FROM sales
        LEFT JOIN menu ON sales.product_id = menu.product_id
)
SELECT 
    customer_id,
    product_name
FROM first_purchase
WHERE dr = 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/3.PNG)

4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```sql
SELECT
    product_name,
    COUNT(sales.product_id) AS purchase_count
FROM sales
    LEFT JOIN menu ON sales.product_id = menu.product_id
GROUP BY product_name
ORDER BY purchase_count DESC
LIMIT 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/4.PNG)

5. Which item was the most popular for each customer?

```sql
WITH popular_products AS (
    SELECT
        customer_id,
        product_name,
        COUNT(product_name) AS product_count,
        RANK() OVER(PARTITION BY customer_id ORDER BY COUNT(product_name) DESC) AS rn
    FROM sales
        LEFT JOIN menu ON sales.product_id = menu.product_id
    GROUP BY customer_id, product_name
)
SELECT
    customer_id,
    product_name
FROM popular_products
WHERE rn = 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/5.PNG)

6. Which item was purchased first by the customer after they became a member?

```sql
WITH order_ranking AS (
    SELECT 
        sales.customer_id,
        order_date,
        product_name,
        RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date) AS rn
    FROM sales 
        LEFT JOIN members ON sales.customer_id = members.customer_id
        LEFT JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date >= join_date
)
SELECT 
    customer_id,
    product_name
FROM order_ranking
WHERE rn = 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/6.PNG)

7. Which item was purchased just before the customer became a member?

```sql
WITH order_ranking AS (
    SELECT 
        sales.customer_id,
        order_date,
        join_date,
        product_name,
        RANK() OVER(PARTITION BY sales.customer_id ORDER BY order_date DESC) AS rn
    FROM sales 
        LEFT JOIN members ON sales.customer_id = members.customer_id
        LEFT JOIN menu ON sales.product_id = menu.product_id
    WHERE order_date < join_date
)
SELECT
    customer_id,
    order_date,
    join_date,
    product_name
FROM order_ranking
WHERE rn = 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/7.PNG)

8. What is the total items and amount spent for each member before they became a member?

```sql
SELECT
    sales.customer_id,
    COUNT(sales.product_id) AS total_items,
    SUM(price) AS total_spent
FROM sales
    LEFT JOIN menu ON menu.product_id = sales.product_id
    LEFT JOIN members ON sales.customer_id = members.customer_id
WHERE order_date < join_date
GROUP BY sales.customer_id
ORDER BY sales.customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/8.PNG)

9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?

```sql
WITH product_scores AS (
    SELECT 
        sales.customer_id,
        sales.product_id,
        product_name,
        price,
        CASE 
            WHEN product_name = 'sushi' THEN 20 
            ELSE 10 
        END AS score_multiplier
    FROM sales
        LEFT JOIN menu ON sales.product_id = menu.product_id
)
SELECT 
    customer_id,
    SUM(price * score_multiplier) AS total_points
FROM product_scores
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/9.PNG)

10. In the first week after a customer joins the program (including their join date) they earn 2x points on 
all items, not just sushi - how many points do customer A and B have at the end of January?

```sql
WITH score_multipliers AS (
    SELECT 
        sales.customer_id,
        order_date,
        join_date,
        sales.product_id,
        product_name,
        price,
        CASE 
            WHEN product_name = 'sushi' OR order_date <= join_date + INTERVAL '7 days' THEN 20 
            ELSE 10 
        END AS score_multiplier
    FROM sales 
        LEFT JOIN menu ON sales.product_id = menu.product_id
        LEFT JOIN members ON sales.customer_id = members.customer_id
    WHERE order_date < '2021-02-01'
        AND sales.customer_id IN ('A', 'B')
)
SELECT 
    customer_id,
    SUM(price * score_multiplier) AS total_score
FROM score_multipliers
GROUP BY customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%201%20-%20Danny's%20Diner/Screenshots/10.PNG)







