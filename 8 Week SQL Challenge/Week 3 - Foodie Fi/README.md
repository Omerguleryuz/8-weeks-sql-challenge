## FOODIE FI

A. DATA ANALYSIS

1) How many customers has Foodie-Fi ever had?

```sql
select 
	count(distinct customer_id) as customers
from subscriptions;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A1.PNG)


2) What is the monthly distribution of trial plan start_date values for our dataset 
use the start of the month as the group by value

```sql
select
	date_trunc('month', start_date) as month_start,
	count(distinct customer_id)
from subscriptions
where plan_id = 0
group by date_trunc('month', start_date)
order by month_start;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A2.PNG)

3) What plan start_date values occur after the year 2020 for our dataset? 
Show the breakdown by count of events for each plan_name

```sql
select 
	subscriptions.plan_id,
    plans.plan_name,
    count(subscriptions.plan_id)
from subscriptions
	left join plans
		on subscriptions.plan_id = plans.plan_id
where extract(year from start_date) > 2020
group by 
	subscriptions.plan_id,
	plans.plan_name	
order by plan_id asc;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A3.PNG)

4) What is the customer count and percentage of customers who have churned 
rounded to 1 decimal place?

```sql
with churned_customers as
(
select
	*
from subscriptions
where plan_id = 4
)

select
	count(distinct subscriptions.customer_id) as customer_count,
	round(count(distinct churned_customers.customer_id) * 1.0 / count(distinct subscriptions.customer_id), 1) as churn_rate
from subscriptions
	left join churned_customers
		on subscriptions.customer_id = churned_customers.customer_id;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A4.PNG)

5) How many customers have churned straight after their initial free trial
what percentage is this rounded to the nearest whole number?

```sql
with churned_customers as 
(
select
	customer_id,
    plan_id,
	row_number() over(partition by customer_id order by plan_id) as rn
from subscriptions
),

free_trials_and_churns as 
(
select 
	*,
    case 
		when (rn = 1 and plan_id = 0) then customer_id else null end as free_trials,
	case 
		when (rn = 2 and plan_id = 4) then customer_id else null end as churns
from churned_customers
)

select 
	count(distinct subscriptions.customer_id) as churned_customers_after_free_trial,
    round(count(distinct free_trials_and_churns.customer_id) * 1.0 / count(distinct subscriptions.customer_id), 0) as churn_rate,
    count(distinct free_trials_and_churns.customer_id),
    count(distinct subscriptions.customer_id)
    
from subscriptions
	left join free_trials_and_churns
		on free_trials_and_churns.customer_id = subscriptions.customer_id
where 
	free_trials_and_churns.churns is not null;
```

6) What is the number and percentage of customer plans after their initial free trial?

```sql
select
	plan_name,
    count(customer_id) as customer_count,
    round(100 * count(distinct customer_id) * 1.0 / (select
													count(distinct customer_id) as distinct_customers
													from subscriptions), 2) as customer_percentage
from subscriptions
join plans using (plan_id)
where plan_id != 0
group by plan_name
order by plan_name;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A6.PNG)

7) What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?

```sql
select
	plan_name,
    count(distinct customer_id) as customer_count,
    round(count(distinct customer_id) * 1.0 / (
									   select
											count(distinct customer_id)
									   from subscriptions
                                       where start_date <= '2020-12-31') * 100, 2) as percentage
from subscriptions
	left join plans
		on subscriptions.plan_id = plans.plan_id
where start_date <= '2020-12-31'
group by plan_name;
```

8) How many customers have upgraded to an annual plan in 2020?

```sql
select 
	plan_name,
    count(distinct customer_id) as customer_count
from subscriptions
	left join plans
		on subscriptions.plan_id = plans.plan_id
where extract(year from start_date) = 2020
	and plan_name = 'pro annual'
group by plan_name;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A8.PNG)

9) How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?

```sql
with dates as 
(
select
    customer_id,
    case when plan_id = 0 then start_date else null end as free_trial_date,
     case when plan_id = 3 then start_date else null end as annual_plan_date,
    case when plan_id = 3 then lag(start_date, 1) over(partition by customer_id order by start_date) 
    else null end as free_trial_start_date
from subscriptions
where plan_id in (0, 3)
)

select
	round(avg(annual_plan_date - free_trial_start_date), 2)
from dates
where annual_plan_date is not null;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A9.PNG)

10) Can you further breakdown this average value into 30 day periods
(i.e. 0-30 days, 31-60 days etc)

```sql
with dates as 
(
select
    customer_id,
    start_date,
    case when plan_id = 0 then start_date else null end as free_trial_date,
     case when plan_id = 3 then start_date else null end as annual_plan_date,
    case when plan_id = 3 then lag(start_date, 1) over(partition by customer_id order by start_date) 
    else null end as free_trial_start_date
from subscriptions
where plan_id in (0, 3)
)

select
	case 
		when annual_plan_date - free_trial_start_date <= 30 then '0-30 days'
		when annual_plan_date - free_trial_start_date <= 60 then '31-60 days'
		when annual_plan_date - free_trial_start_date <= 90 then '61-90 days'
		else '90+ days' 
	end as period,
	round(avg(annual_plan_date - free_trial_start_date), 2) as avg_days
from dates
where annual_plan_date is not null
group by period
order by period;
```

![alt text](https://github.com/Omerguleryuz/8-weeks-sql-challenge/blob/main/8%20Week%20SQL%20Challenge/Week%203%20-%20Foodie%20Fi/Screenshots/A10.PNG)

11) How many customers downgraded from a pro monthly to a basic monthly plan in 2020?

```sql
with next_plans as
(
select 
	*,
    lead(plan_id, 1) over(partition by customer_id order by start_date) as next_plan
from subscriptions
where extract(year from start_date) = 2020
)

select
	count(distinct customer_id) as downgraded_customers
from next_plans
where plan_id = 2
	and next_plan = 1;
```


D. Outside The Box Questions

1) How would you calculate the rate of growth for Foodie-Fi?

I would calculate the revenue monthly and calculate the percentage of difference.

2) What key metrics would you recommend Foodie-Fi management to track over 
time to assess performance of their overall business?

Revenue, customer count, customer change percentage, revenue change percentage, how many months 
do customers subscribe on average, distribution of revenue by plans	

3) What are some key customer journeys or experiences that you would 
analyse further to improve customer retention?

When do customers get bored, is there enough content, is pricing expensive,
should there be one more plan between pro monthly and pro annual (because of the huge price gap)
why do customers churn, is website fast enough to deliver the content

4) If the Foodie-Fi team were to create an exit survey shown to customers who wish to 
cancel their subscription, what questions would you include in the survey?

5) What business levers could the Foodie-Fi team use to reduce the customer 
churn rate? How would you validate the effectiveness of your ideas?

Loyalty programs could be integrated. The more months a customer stays as a subscriber the more 
incentive they will gain such as exclusive content, discounts, promotions, etc.
To validate the idea, we could run an A/B test for 6 months and compare churn rates of the two test groups.
If the idea works, then we could spend more time to enhance it and make it our main program.


