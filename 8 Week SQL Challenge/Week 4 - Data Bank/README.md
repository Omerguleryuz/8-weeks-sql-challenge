## DATA BANK

A. Customer Nodes Exploration

1) How many unique nodes are there on the Data Bank system?

```sql
SELECT
    COUNT(DISTINCT node_id) AS unique_nodes
FROM customer_nodes;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/A1.PNG)

2) What is the number of nodes per region?

```sql
SELECT
    region_name,
    COUNT(DISTINCT node_id) AS nodes
FROM customer_nodes
LEFT JOIN regions
    ON customer_nodes.region_id = regions.region_id
GROUP BY region_name;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/A1.PNG)

3) How many customers are allocated to each region?

```sql
SELECT
    region_name,
    COUNT(customer_id) AS customers
FROM customer_nodes
LEFT JOIN regions
    ON customer_nodes.region_id = regions.region_id
GROUP BY region_name;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/A3.PNG)

4) How many days on average are customers reallocated to a different node?

```sql
WITH node_days AS (
    SELECT 
        customer_id, 
        node_id,
        end_date - start_date AS days_in_node
    FROM data_bank.customer_nodes
    WHERE end_date != '9999-12-31'
), 
total_node_days AS (
    SELECT 
        customer_id,
        node_id,
        SUM(days_in_node) AS total_days_in_node
    FROM node_days
    GROUP BY customer_id, node_id
)

SELECT ROUND(AVG(total_days_in_node)) AS avg_node_reallocation_days
FROM total_node_days;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/A4.PNG)

5) What is the median, 80th, and 95th percentile for reallocation days per region?

```sql
WITH reallocation_days AS (
    SELECT 
        region_id,
        end_date - start_date AS days_in_node
    FROM customer_nodes
    WHERE end_date != '9999-12-31'
),
percentile_calculations AS (
    SELECT
        region_id,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY days_in_node) AS median,
        PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY days_in_node) AS percentile_80,
        PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY days_in_node) AS percentile_95
    FROM reallocation_days
    GROUP BY region_id
)

SELECT 
    r.region_name, 
    p.median, 
    p.percentile_80, 
    p.percentile_95
FROM percentile_calculations p
JOIN regions r ON p.region_id = r.region_id;
```

B. Customer Transactions

1) What is the unique count and total amount for each transaction type?

```sql
SELECT
    txn_type, 
    COUNT(customer_id) AS transaction_count, 
    SUM(txn_amount) AS total_amount
FROM data_bank.customer_transactions
GROUP BY txn_type;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/B1.PNG)

2) What is the average total historical deposit counts and amounts for all customers?

```sql
SELECT
    round(AVG(transaction_count),2) AS avg_total_deposit_count,
    round(AVG(total_amount),2) AS avg_total_deposit_amount
FROM (
    SELECT 
        customer_id,
        COUNT(*) AS transaction_count,
        SUM(txn_amount) AS total_amount
    FROM data_bank.customer_transactions
    WHERE txn_type = 'deposit'
    GROUP BY customer_id
) AS deposits;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/B2.PNG)

3) For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

```sql
SELECT
    DATE_TRUNC('month', txn_date) AS month,
    COUNT(DISTINCT customer_id) AS eligible_customers
FROM data_bank.customer_transactions
GROUP BY DATE_TRUNC('month', txn_date)
HAVING COUNT(*) FILTER (WHERE txn_type = 'deposit') > 1
   AND COUNT(*) FILTER (WHERE txn_type IN ('purchase', 'withdrawal')) >= 1;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/B3.PNG)

4) What is the closing balance for each customer at the end of the month?

```sql
WITH monthly_balance AS (
    SELECT
        customer_id,
        DATE_TRUNC('month', txn_date) AS month,
        SUM(txn_amount) AS monthly_amount
    FROM data_bank.customer_transactions
    GROUP BY customer_id, DATE_TRUNC('month', txn_date)
)

SELECT
    customer_id,
    month,
    SUM(monthly_amount) OVER (PARTITION BY customer_id ORDER BY month) AS closing_balance
FROM monthly_balance;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%204%20-%20Data%20Bank/Screenshots/B4.PNG)

5) What is the percentage of customers who increase their closing balance by more than 5%?

```sql
WITH balance_change AS (
    SELECT
        customer_id,
        month,
        closing_balance,
        LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY month) AS previous_balance
    FROM (
        SELECT
            customer_id,
            month,
            SUM(txn_amount) AS closing_balance
        FROM data_bank.customer_transactions
        GROUP BY customer_id, month
    ) AS monthly_closing_balance
)

SELECT
    ROUND(100.0 * COUNT(*) FILTER (WHERE closing_balance > 1.05 * previous_balance) / COUNT(*), 2) AS percentage_increased_balance
FROM balance_change;
```




















