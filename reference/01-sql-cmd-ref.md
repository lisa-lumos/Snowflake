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

-- lateral join. (can be re-written using regular inner join)
create table departments (
  department_id integer, 
  name varchar
);
create table employees (
  employee_id integer, 
  last_name varchar, 
  department_id integer, 
  project_names array
);
insert into departments 
  (department_id, name) 
values 
  (1, 'Engineering'), 
  (2, 'Support')
;
insert into employees 
  (employee_id, last_name, department_id) 
values 
  (101, 'Richards', 1),
  (102, 'Paulson',  1),
  (103, 'Johnson',  2)
;
select * 
from departments as d, 
  lateral ( -- inline view
    select * 
    from employees as e 
    where e.department_id = d.department_id -- refers a col from a table expression, which lives outside of this inline view
  ) as iv2
order by employee_id;
-- <-------- from d -------------> <-------------------- from e ---------------------------->

-- +---------------+-------------+-------------+-----------+---------------+---------------+
-- | DEPARTMENT_ID | NAME        | EMPLOYEE_ID | LAST_NAME | DEPARTMENT_ID | PROJECT_NAMES |
-- |---------------+-------------+-------------+-----------+---------------+---------------|
-- |             1 | Engineering |         101 | Richards  |             1 | NULL          |
-- |             1 | Engineering |         102 | Paulson   |             1 | NULL          |
-- |             2 | Support     |         103 | Johnson   |             2 | NULL          |
-- +---------------+-------------+-------------+-----------+---------------+---------------+
update employees 
set project_names = array_construct('proj1', 'proj2') 
where employee_id = 101
;
update employees 
set project_names = array_construct('proj1', 'proj3')
where employee_id = 102
;
select * from employees;
-- EMPLOYEE_ID  LAST_NAME  DEPARTMENT_ID  PROJECT_NAMES
-- 101          Richards   1              ["proj1", "proj2"]
-- 102          Paulson    1              ["proj1", "proj3"]
-- 103          Johnson    2              null

-- the use of lateral flatten
select 
  *
from employees as emp, 
  lateral flatten(input => emp.project_names) as proj_names
order by employee_id;
-- EMPLOYEE_ID  LAST_NAME  DEPARTMENT_ID  PROJECT_NAMES        SEQ  KEY   PATH   INDEX   VALUE    THIS
-- 101          Richards   1              ["proj1", "proj2"]   1    null  [0]    0       "proj1"  ["proj1", "proj2"]
-- 101          Richards   1              ["proj1", "proj2"]   1    null  [1]    1       "proj2"  ["proj1", "proj2"]
-- 102          Paulson    1              ["proj1", "proj3"]   2    null  [0]    0       "proj1"  ["proj1", "proj3"]
-- 102          Paulson    1              ["proj1", "proj3"]   2    null  [1]    1       "proj3"  ["proj1", "proj3"]


-- match_recognize is often used to detect events in time series

-- pivot example. Transform vals in a col to col headers
create or replace table monthly_sales(
  empid int, 
  amount int, 
  month text
) as select * 
from values
  (1, 10000, 'JAN'),
  (1, 400, 'JAN'),
  (2, 4500, 'JAN'),
  (2, 35000, 'JAN'),
  (1, 5000, 'FEB'),
  (1, 3000, 'FEB'),
  (2, 200, 'FEB'),
  (2, 90500, 'FEB'),
  (1, 6000, 'MAR'),
  (1, 5000, 'MAR'),
  (2, 2500, 'MAR'),
  (2, 9500, 'MAR'),
  (1, 8000, 'APR'),
  (1, 10000, 'APR'),
  (2, 800, 'APR'),
  (2, 4500, 'APR');

select * 
from monthly_sales
  pivot(
    sum(amount) -- values in each new col
    for month in ('JAN', 'FEB', 'MAR', 'APR') -- agg by these vals
  ) as p (EMP_ID_renamed, JAN, FEB, MAR, APR) -- new col names
order by empid;
-- +-------+-------+-------+-------+-------+
-- | EMPID | 'JAN' | 'FEB' | 'MAR' | 'APR' |
-- |-------+-------+-------+-------+-------|
-- |     1 | 10400 |  8000 | 11000 | 18000 |
-- |     2 | 39500 | 90700 | 12000 |  5300 |
-- +-------+-------+-------+-------+-------+

-- unpivot example. Transforms cols to rows
-- example setup
create or replace table monthly_sales(
  empid int, 
  dept text, 
  JAN int, 
  FEB int, 
  MAR int, 
  APRIL int
);

insert into monthly_sales values
  (1, 'electronics', 100, 200, 300, 100),
  (2, 'clothes', 100, 300, 150, 200),
  (3, 'cars', 200, 400, 100, 50)
;
-- EMPID DEPT        JAN FEB MAR APRIL
-- 1     electronics 100 200 300 100
-- 2     clothes     100 300 150 200
-- 3     cars        200 400 100 50

select * 
from monthly_sales
  unpivot(
    sales -- name of the new col
    for month -- name of the new dimensional col
      in (JAN, FEB, MAR, APRIL) -- list of column names to go into the new dimensional col
  )
order by empid;
-- +-------+-------------+-------+-------+
-- | EMPID | DEPT        | MONTH | SALES |
-- |-------+-------------+-------+-------|
-- |     1 | electronics | JAN   |   100 |
-- |     1 | electronics | FEB   |   200 |
-- |     1 | electronics | MAR   |   300 |
-- |     1 | electronics | APRIL |   100 |
-- |     2 | clothes     | JAN   |   100 |
-- |     2 | clothes     | FEB   |   300 |
-- |     2 | clothes     | MAR   |   150 |
-- |     2 | clothes     | APRIL |   200 |
-- |     3 | cars        | JAN   |   200 |
-- |     3 | cars        | FEB   |   400 |
-- |     3 | cars        | MAR   |   100 |
-- |     3 | cars        | APRIL |    50 |
-- +-------+-------------+-------+-------+

-- VALUES example
select * from values 
  (1, 'one'), 
  (2, 'two'), 
  (3, 'three')
;
-- +---------+---------+
-- | COLUMN1 | COLUMN2 |
-- |---------+---------|
-- |       1 | one     |
-- |       2 | two     |
-- |       3 | three   |
-- +---------+---------+
select column1, $2 from values 
  (1, 'one'), 
  (2, 'two'), 
  (3, 'three')
;
-- +---------+-------+
-- | COLUMN1 | $2    |
-- |---------+-------|
-- |       1 | one   |
-- |       2 | two   |
-- |       3 | three |
-- +---------+-------+
select v1.$2, v2.$2
from 
  values (1, 'one'), (2, 'two') as v1 -- join values
  inner join values (1, 'one'), (3, 'three') as v2
  on v2.$1 = v1.$1
;
select c1, c2 -- alias the values' column name
from values 
  (1, 'one'), 
  (2, 'two') as v1 (c1, c2)
;

-- return a random subset of rows from table, using sample
select * from my_table sample bernoulli (10.2); -- each row has 10.2% chance of being returned, use bernoulli method for sampling
select * from my_table sample system (3.1) seed (82); -- each row has 3.1% chance of being returned, use system method (appx, and faster) for sampling, fix the seed val
select * from my_table sample (10 rows); -- sample and return 10 rows. Slower. 

-- where clause
-- In most contexts, the boolean expression NULL = NULL returns NULL, not TRUE. Use IS [ NOT ] NULL to compare NULL values.

-- group by. Takes col name, position number, expression
-- GROUP BY ALL: group by all non aggregated items in the SELECT.
select -- still, not recommend to have dup alias with existing col names
  sum(salary), 
  any_value(employment_state) as state -- state is already a col name in employees table
from employees
group by state -- SF uses the state col in table, not the alias in select. 
;

-- group by cube/rollup. Can be re-written using group by.
-- group by grouping sets. Can be re-written using group by.
















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


























































