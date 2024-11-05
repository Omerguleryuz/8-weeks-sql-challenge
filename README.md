# 8-weeks-sql-challenge

Case Study Questions
 
1. What is the total amount each customer spent at the restaurant?

```sql
select 	
    customer_id,
    sum(price) as total_spent
from sales
left join menu
on sales.product_id = menu.product_id
group by customer_id;
