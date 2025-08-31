# Credit Card Analytics with SQL

## Problem Statements & Solutions

### 1. 
**Objective:** Write a query to print top 5 cities with highest spends and their percentage contribution of total credit card spends.  

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
