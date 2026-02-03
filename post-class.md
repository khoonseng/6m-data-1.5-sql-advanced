# Assignment

## Brief

Write the SQL statements for the following questions.

## Instructions

Paste the answer as SQL in the answer code section below each question.

### Question 1

Using the `claim` and `car` tables, write a SQL query to return a table containing `id, claim_date, travel_time, claim_amt` from `claim`, and `car_type, car_use` from `car`. Use an appropriate join based on the `car_id`.

Answer:

```sql
select cl.id,
       cl.claim_date,
       cl.travel_time,
       cl.claim_amt,
       c.car_type,
       c.car_use
from claim cl
inner join car c
on cl.car_id = c.id;
```

### Question 2

Write a SQL query to compute the running total of the `travel_time` column for each `car_id` in the `claim` table. The resulting table should contain `id, car_id, travel_time, running_total`.

Answer:

```sql
SELECT id,
       car_id,
       travel_time,
       sum(travel_time) over (partition by car_id order by car_id) as running_total
from claim
order by car_id
```

### Question 3

Using a Common Table Expression (CTE), write a SQL query to return a table containing `id, resale_value, car_use` from `car`, where the car resale value is less than the average resale value for the car use.

Answer:

```sql
with avg_value_by_car_use as (
    select car_use,
           round(avg(resale_value),2) as average_value
    from car
    group by car_use
)
SELECT c.id,
       c.resale_value,
       c.car_use,
       v.average_value as avg_value
from car c
inner join avg_value_by_car_use v
on c.car_use = v.car_use
where c.resale_value < v.average_value
order by c.id
```

# **Level Up: The Insurance Auditor Project**

Scenario:  
You are the Lead Data Analyst for "SafeDrive Insurance". The CEO suspects that certain car types in specific cities are disproportionately expensive.  
Your Task:  
Create a comprehensive SQL report that answers the following in a single script (using CTEs):

1. **Market Comparison:** For every claim, show the claim\_amt alongside the **average claim amount for that specific car\_type**.  
2. **Risk Ranking:** Within each state, rank the clients by their total claim amounts.  
3. **Efficiency:** Only show the **top 2** highest-claiming clients per state.  
4. **Final Output:** The table should include: Client Name, State, Car Type, Total Claimed, State Rank.

Submission:  
A single .sql file with comments explaining your logic.
