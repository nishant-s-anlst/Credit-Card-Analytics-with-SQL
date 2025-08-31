# Credit Card Analytics with SQL

## Data Source
https://www.kaggle.com/datasets/thedevastator/analyzing-credit-card-spending-habits-in-india

## Problem Statements & Solutions

### 1. 
**Objective:** To find top 5 cities with highest spends and their percentage contribution of total credit card spends.  

```sql
with total_spent_cte as(
select SUM(amount) as total_spent
from credit_card_transcations
)
select top 5
city,
sum(amount) as expense,
round((sum(amount)/t.total_spent)*100,2) as pct_contribution
from credit_card_transcations
cross join total_spent_cte t
group  by city,t.total_spent
order by expense desc;
```
### 2. 
**Objective:** To print highest spend month and amount spent in that month for each card type.

```sql
with cte1 as (
select card_type,
datepart(month,transaction_date) as month,
datepart(year,transaction_date)  as year,
SUM(amount) as monthly_expense
from credit_card_transcations
group by card_type,datepart(month,transaction_date), datepart(year,transaction_date)
)
select * from (
select *,
RANK() over (partition by card_type order by monthly_expense desc) as rn
from cte1) as r
where rn=1;
```

### 3.
**Objective:** write a query to print the transaction details(all columns from the table) for each card type when it reaches a cumulative of 1000000 total spends(We should have 4 rows in the o/p one for each card type).

```sql
with running_totals as(
select *,
SUM(amount) over (partition by card_type order by transaction_date,transaction_id) as cum_sum
from credit_card_transcations
)
select * from (
select *,
RANK() over (partition by card_type order by cum_sum) as rn
from running_totals
where cum_sum>=1000000) as r
where rn=1;
```

### 4.
**Objective:** write a query to find city which had lowest percentage spend for gold card type.

```sql
with cte1 as(
select city,
sum(case when card_type='gold' then amount else 0 end) as gold_spend,
sum(amount) as total_spend
from credit_card_transcations
group by city 
)
select top 1
*,
round((gold_spend/total_spend)*100,2) as gold_contribution
from cte1
where gold_spend <>0
order by gold_contribution asc;
```
### 5.
**Objective:** write a query to print 3 columns:  city, highest_expense_type , lowest_expense_type (example format : Delhi , bills, Fuel).

```sql
with cte1 as(
select 
city,
exp_type,
sum(amount) as total_spend,
row_number() over (partition by city order by sum(amount)) as low_rn,
row_number() over (partition by city order by sum(amount) desc) as high_rn
from credit_card_transcations
group by exp_type,city)

select city,
min(case when low_rn=1 then exp_type end) as lowest_expense_type,
max(case when high_rn=1 then exp_type end) as highest_expense_type
from cte1
group by city;
```
### 6.
**Objective:** write a query to find percentage contribution of spends by females for each expense type.

```sql
with cte1 as(
select 
exp_type,
SUM(case when gender='f' then amount end) as female_spend,
SUM(amount) as total_spend
from credit_card_transcations
group by exp_type
)
select *,
round((female_spend/total_spend)*100,2) as female_contribution
from cte1;
```
### 7.
**Objective:** which card and expense type combination saw highest month over month growth in Jan-2014.

```sql
with cte1 as (
select 
card_type,
exp_type,
DATEPART(month,transaction_date) as mt,
DATEPART(year,transaction_date) as yt,
SUM(amount) as expense
from credit_card_transcations
group by card_type,exp_type,
DATEPART(month,transaction_date), DATEPART(year,transaction_date)
),cte2 as(
select *,
LAG(expense,1) over(partition by card_type, exp_type order by yt, mt) as prev_month_expense
from cte1)

select top 1 *,
round((expense-prev_month_expense)/prev_month_expense*100,2) as mom_growth
from cte2
where mt=1 and yt=2014
order by mom_growth desc;
```

### 8. 
**Objective:** during weekends which city has highest total spend to total no of transcations ratio.

```sql 
select top 1 city,
SUM(amount)/COUNT(*) as ratio
from credit_card_transcations
where DATENAME(weekday,transaction_date) in ('saturday','sunday')
group by city
order by ratio desc;
```
### 9. 
**Objective:** which city took least number of days to reach its 500th transaction after the first transaction in that city.

```sql
with cte1 as(
select 
*,
row_number() over (partition by city order by transaction_date) as rn
from credit_card_transcations) 
,cte2 as(
select *,
LAG(transaction_date,1) over (partition by city order by transaction_date) as first_transaction_date
from cte1 
where rn in(1,500)
)
select top 1 city, DATEDIFF(day,first_transaction_date,transaction_date) as days_to_500 from cte2
where first_transaction_date is not null
order by days_to_500;
```
