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
inner join car c on cl.car_id = c.id;
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
inner join avg_value_by_car_use v on c.car_use = v.car_use
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

```sql
-- Market Comparison: For every claim, show the claim_amt alongside the average claim amount for that specific car_type.
-- need to compute average claim amount based on car type -> use partition without specific sorting
with avg_amt_by_car_type as (
    select cl.id as claim_id,
           cl.client_id,
           c.car_type, 
           cl.claim_amt,
           avg(cl.claim_amt) over (partition by c.car_type) as avg_claim_amt
    from claim cl
    inner join car c on cl.car_id = c.id
),
-- Risk Ranking: Within each state, rank the clients by their total claim amounts.
-- need to compute total claim amount for each client and state -> use group by instead of window function since we do not need any breakdown of details
client_total_claim_amt as (
    select cli.id as client_id,
           concat(cli.first_name, ' ', cli.last_name) as client_name,
           a.state,
           sum(cl.claim_amt) as client_claim_amt
    from claim cl
    inner join client cli on cl.client_id = cli.id
    inner join address a on cli.address_id = a.id
    group by cli.id, client_name, a.state
),
--Efficiency: Only show the top 2 highest-claiming clients per state.
-- need to compute rank based on state and claim amount-> use partition and sort by claim amount
rank_clients_by_claim as (
    select state,
           client_id,
           client_claim_amt,
           RANK() OVER (partition by state order by client_claim_amt desc) as state_rank
    from client_total_claim_amt
    qualify state_rank <= 2
    order by state, state_rank
)
--Final Output: The table should include: Client Name, State, Car Type, Total Claimed, State Rank.
select t.client_name,
       r.state,
       STRING_AGG(distinct a.car_type, ', ' order by a.car_type asc) as car_types, -- to display multiple car types from the same client in a single row
       r.client_claim_amt as total_claimed,
       r.state_rank
from client_total_claim_amt t
inner join rank_clients_by_claim r on t.client_id = r.client_id
inner join avg_amt_by_car_type a on a.client_id = t.client_id
group by t.client_name, r.state, r.client_claim_amt, r.state_rank
order by r.state, r.state_rank
```
