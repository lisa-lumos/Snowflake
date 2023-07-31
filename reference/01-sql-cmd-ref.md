# 1. SQL command reference
## Query Syntax
```sql
-- Example of using a recursive CTE, to generate a Fibonacci series:
with recursive current_f (previous_val, current_val) as (
  select 
    1, 0
  union all 
  select 
    current_val, 
    current_val + previous_val  
  from current_f
  where current_val + previous_val < 10
)

select current_val  
from current_f 
order by current_val;

-- Top example:
select top 5
  *
from my_table
order by my_col
;

-- Snowflake scripting, select a single row to a set of variables:
select 
  <expression1>,
  <expression2>
into 
  :var1, 
  :var2 
from ...
where ...

-- use time travel in select
select * 
from my_table at(
  timestamp => 'Fri, 01 May 2015 16:20:00 -0700'::timestamp
);

select * 
from my_table 
at(offset => -60*5) as t  -- select data from 5 min ago
where t.flag = 'valid';

-- query the change tracking metadata for a table/view 
-- within a specified interval of time 
-- without using a stream
-- Change tracking must be enabled on the source table 
select *
from t1
  changes(information => default) -- full delta
  at(timestamp => $ts1)
;
select *
from t1
  changes(information => append_only) -- only new rows
  at(timestamp => $ts1)
  end(timestamp => $ts2);
;

-- connect by 
select employee_id, manager_id, title
  from employees
    start with title = 'president'
    connect by
      manager_id = prior employee_id
order by employee_id;
-- +-------------+------------+----------------------------+
-- | EMPLOYEE_ID | MANAGER_ID | TITLE                      |
-- |-------------+------------+----------------------------|
-- |           1 |       NULL | President                  |
-- |          10 |          1 | Vice President Engineering |
-- |          20 |          1 | Vice President HR          |
-- |         100 |         10 | Programmer                 |
-- |         101 |         10 | QA Engineer                |
-- |         200 |         20 | Health Insurance Analyst   |
-- +-------------+------------+----------------------------+

-- You CANNOT use "join" keyword on lateral table function:
select ... from my_table 
join table(flatten(input=>[col_a])) -- NOT allowed
on ... ;

-- Without "join" is allowed on lateral table function:
select ... from my_table,
table(flatten(input=>[col_a]))
on ... ;

-- Natural join produces the same output as the corresponding inner join, 
-- except its output doesn't include a 2nd copy of the join column




```


## Query Operators


## General DDL


## General DML


## All Commands 


## Accounts


## Users, Roles, & Privileges


## Integrations


## Security Policies


## Replication & Failover


## Sessions


## Transactions


## Virtual Warehouses & Resource Monitors


## Databases, Schemas, & Shares


## Tables, Views, & Sequences


## Functions, Procedures, & Scripting


## Streams & Tasks


## Data Governance


## Secret


## Data Loading & Unloading


## File Staging


## Alerts


## Native Apps Framework


























































