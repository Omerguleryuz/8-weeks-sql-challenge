# 8-weeks-sql-challenge

### Danny's Diner Case Study Questions
 
 1. What is the total amount each customer spent at the restaurant?

```sql
select 	
    customer_id,
    sum(price) as total_spent
from sales
left join menu
on sales.product_id = menu.product_id
group by customer_id;
```

![alt text](https://github.com/hilalguleryuz/northwind_data_analysis_capstone_project/blob/main/Screenshots/SS_1.png)
