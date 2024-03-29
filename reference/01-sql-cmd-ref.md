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

-- having
select department_id
from employees
group by department_id
having count(*) < 10;

-- qualify. Filters window functions after they are computed. Can be replace by cte
select 
  col1, 
  col2, 
  row_number() over (partition by col1 order by col2) as row_num
from qt
qualify row_num = 1
;
with vals as (
  select 
    min(col1) as filtered_col1
  from tbl1
  group by col2
  having filtered_col1 > 3;
)
select 
  col2, 
  sum(col3) over (partition by col2) as r
from tbl2
where col3 < 4
group by col2, col3
having sum(col1) > 3
qualify r in (select * from vals)

-- order by: NULL values are higher than non-NULL values.
select col1
from values (1), (null), (2), (null), (3)
order by column1 nulls first; -- then nulls come first

-- limit
select col1 from testtable 
order by col1 
limit 3 offset 3
;
```

## Query Operators
```sql
-- Leading digits (L): num of digits to the left of the decimal point
-- Scale (S): num of digits to the right of the decimal point
-- Precision (P): P = L + S. Total num of digits. Always limited to 38, in sf.
-- e.g.: for the decimal(8,2) data type, precision is 8, scale is 2, and leading digits is 6.

-- set operators
-- Make sure:
-- 1. each query selects the same num of columns
-- 2. the data type of each column is consistent from different sources
-- 3. the meanings of the columns should match

-- intersect: return the intersection rows, does deduplication
select ...
intersect
select ...

-- minus(same with except): rmv intersection rows, does deduplication
select ...
minus
select ...

-- union: union does deduplication, union all does not do it. 
select ...
union [all]
select ...
```

## General DDL
To to create/manipulate/modify objects in Snowflake. 
```sql
-- alter
alter account/session set param_name = param_val;
alter obj_type obj_name ...;

-- comment. Adds/overrides comment for an object
comment [if exists] on obj_type obj_name is 'my obj comment';
comment [if exists] on column my_table.my_column is 'my col comment';

-- create
create [or replace] obj_type [if not exists] obj_name ...
; -- CREATE TABLE ... CLONE includes the COPY GRANTS option
create database/schema/table obj_name 
  clone source_obj_name ...
; -- with/without time travel
-- when clone from db/schema, views inside them do not have ddl change (points to source tables/views, same as source views). 

-- describe
describe obj_type obj_name;

-- drop
-- 1. restrict: do not drop if foreign key points to the obj. 
-- 2. cascade: drop the obj, and drop any foreign keys that point to this obj.
drop obj_type [if exists] obj_name [restrict/cascade];

-- undrop. Relies on time travel. 
undrop obj_type obj_name;

-- show
-- 1. uses cloud services layer only
-- 2. only show objs you have access to (roles with MANAGE GRANTS privilege can see every object in the account)
-- 3. recommend to provide scope
-- 4. return max of 10k records. To see more, go to information_schema
show obj_type_plural [like 'my_pattern'] [in obj_type [obj_name]];

-- use. Sets the role/wh/db/sc for the current session
use obj_type obj_name;
```

## General DML
```sql
-- insert. Can use OVERWRITE param to do truncate and insert in one transaction. 
insert [overwrite] into employees
values -- multi-row insert
  ('Lysandra','Reeves','1-212-759-3751','New York',10018),
  ('Michael','Arnett','1-650-230-8467','San Francisco',94116)
;
insert into table1 (col_number, col_string, col_variant)
select 
  4, 
  'fourier', 
  parse_json('{ "key1": "value1", "key2": "value2" }')
;
insert into employees (first_name, last_name)
with cte as(
  select 
    contractor_first as first_name,
    contractor_last as last_name
  from contractors
)
select first_name, last_name
from cte
;

-- merge. 
-- Inserts/updates/deletes vals in a table, 
-- based on vals in a 2nd table/subquery, when it is like a change log.
-- ERROR_ON_NONDETERMINISTIC_MERGE session parameter is true by default,
-- which will return an error if multiple rows in my_log matches the condition
merge into my_table using my_log on my_table.col1 = my_log.col1
when matched and ... then update/delete ...
when matched then update/delete ...
when not matched and ... then insert ... values ...
when not matched then insert ... values ...
;

-- update
-- ERROR_ON_NONDETERMINISTIC_MERGE session parameter is true by default,
-- which will return an error if multiple rows in my_log matches the condition
update target_table
set v = src_table.v
from src_table
where target_table.k = src_table.k;

-- delete
delete from bicycles 
where bicycle_id = 105
;
delete from table1 
using table2 
where table1.k = table2.k

-- truncate
-- also deletes the load metadata for the table, for manual load
truncate [table] [if exists] my_table;
```

## All Commands 
skipped. 

## Accounts
Note that all show ... commands lives on cloud services layer. 
```sql
create/drop/undrop account ...; -- need to be ORGADMIN
 
show organization accounts ...;  -- need to be ORGADMIN. 

-- for reader accounts. 
create/drop managed account ...;  -- need to be accountadmin, or "create account" privilege on account object
show managed accounts ...;  -- need to be accountadmin, or "monitor usage" global privilege. 

alter account set/unset ...; 
-- need to be accountadmin, for account/session/object parameters, resource monitors, tags. 
-- need to be securityadmin, for network_policy account param. 
-- need to be orgadmin, for rename account, enable orgadmin role for the account. 

-- lists both native function and user-defined functions. 
show functions ...;
```

## Users, Roles, & Privileges
```sql
create user ...; -- need to be useradmin+, or "create user" privilege on the account object
alter user ...; -- admin can do this to all users; individual users can modify default warehouse/namespace/role/session-params of themselves. 
drop user ...;
describe user ...; -- user's owner+ can see other users, or user can see themselves
show users ...; -- security admin+, or needs "manage grants" global privilege 

create database role ...; -- need to be user admin+, or db owner/user, or have "create database role" privilege on the db.  
alter database role ...; -- need to be db role owner+
drop database role ...;
show database roles in database ...; -- need to be db user
grant database role ... to role ...; -- grant the db role to an account/db role. Need to be the owner of the source db role, or have global "manage grants" privilege. 
revoke database role ... from role ...;
grant database role ... to share ...;
revoke database role ... from share ...;

create role ...; -- need to be useradmin+, or "create role" privilege on account obj
alter role ...; -- need to be the role owner+
drop role ...; -- Objects owned by the dropped role is transferred to the role that executes the DROP command. If a role has a future privilege as a grantor/grantee, the role can only be dropped by a role with the "manage grants" privilege. 
show roles ...; 
grant role to role/user ...; -- need to be owner of role, or security admin, or have "manage grants" privilege. 
revoke role from role/user ...; 
use role ...; -- set the cur primary role for the session
use secondary roles all/none; -- use all roles available as secondary role, or disable secondary roles. 

-- You must use an application role, to grant/revoke privileges on objects in an application
grant ... on ... to application role ...;
revoke ... on ... from application role ...;
grant ... on ... to role ...;
grant ... on ... to database role ...;
revoke ... on ... from role ...;
revoke ... on ... from database role ...;
grant ... on ... to share ...;
revoke ... on ... from share ...;
grant ownership on ... to role ...;
grant ownership on ... to database role ...;

show grants [on ...]; -- list all privileges you have on this object
show grants to ...; -- list all privileges/roles this role/user/share has
show grants of ...; -- list all users/roles granted to this role/share...

```

## Integrations
skipped

## Security Policies
```sql
create network policy ...; -- need to be security admin+, or "create network policy" privilege on the account object
alter network policy ...; -- need to be the network policy owner+
drop network policy ...; -- need to be security admin+
describe network policy ...; -- need to be the network policy owner+
show network polices ...; -- need to be the network policy owner+

-- After creating a password policy object, apply it to an account using ALTER ACCOUNT, or a user using ALTER USER.
create password policy ...; -- need to have "create password policy" privilege on the schema object
alter password policy ...; -- need to be the password policy owner+
drop password policy ...; -- need to be the password policy owner+
describe password policy ...; -- need to be the password policy owner+, or have "apply password policy" on the account object. 
show password policies ...; -- need to be the password policy owner+, or have "apply password policy" on the account object. 

-- Session policies requires Enterprise Edition &+
-- defines the idle session timeout period in minutes.
-- After creating a session policy object, apply it to an account using ALTER ACCOUNT, or a user using ALTER USER.
create session policy ...; -- need to have "create session policy" privilege on the schema object
alter session policy ...; -- need to be the session policy owner+
drop session policy ...; -- need to be the session policy owner+
describe session policy ...; -- need to be the session policy owner+, or have "apply session policy" privilege on the account object
show session polices ...; -- need to be the session policy owner+, or have "apply session policy" privilege on the account object
```

## Replication & Failover
```sql
-- list all the accounts in your org that enabled replication, and their region
show replication accounts ...; -- need to be account admin
-- list all the dbs that enabled replication, and their region
show replication databases ...; -- need usage/monitor on the db
-- list all the regions, in which accounts can be created.
show regions ...;

-- replication
-- Database and share replication are available to all accounts.
-- Replication of other account objects & failover/failback require Business Critical Edition &+

-- Create a replication group in the source/target account, in the same org
create replication group ...; 
-- need to be account admin, or "create replication group" on the account object. 
-- To add a db, need to have "monitor" privilege on the db; 
-- for a share, need to be its owner. 

alter/drop replication group ...;
show databases/shares in replication group ...;
show replication groups ...;

create/alter/drop failover group ...;
show failover groups ...;
show databases/shares in failover group ...;

create/alter/drop connection ...; -- for client redirect, during disaster recovery, or account migration
show connections ...;
```

## Sessions
```sql
alter session set/unset ...; -- have types, allow boolean/number/string.
show parameters in ...; -- show all params in account/session/objs

set var_name = my_expression; -- set session variable
unset var_name; 
show variables ...; -- list all vars defined in cur session

explain my_statement; -- get the execution plan for the statement, in tab/text/json format
describe result 'my_query_id'/last_query_id(); -- describe the cols in query result
```

## Transactions
```sql
-- All transactions have a system-generated internal ID.
-- When you query a stream within an explicit transaction, 
-- the stream is queried at the timestamp when the transaction began, 
-- rather than when the statement was run. 

-- begins a transaction in the current session, can give it a name
begin ...;  -- same as "start transaction; ", "begin transaction", "begin work"

show transactions;
select current_transaction();

-- To complete a transaction, a COMMIT/ROLLBACK command must be explicitly executed.
commit; -- same as "commit work"

select last_transaction();

describe transaction my_transaction_id;

rollback; -- same as "rollback work"

-- lists all running transactions for the current user, that have locks on resources. 
-- If account admin runs it, it shows result across the account. 
-- can then use system$abort_transaction() to abort a specific transaction.
show locks; 

-- list all running transactions for the current user
-- If account admin runs it, it shows result across the account. 
-- can then use system$abort_transaction() to abort a specific transaction.
show transactions;
```

## Virtual Warehouses & Resource Monitors
```sql
create warehouse ...; -- Need to be sysadmin+, or with "create warehouse" privilege on the account object
alter warehouse ...; -- Need to have usage/operate/monitor/modify privilege on the wh
describe warehouse ...;
drop warehouse ...;
show warehouses ...; -- list all whs that you have access to

-- After a resource monitor is created, 
-- it must be assigned to a warehouse/account, 
-- before it can perform any monitoring actions
create resource monitor ...; -- Need to be account admin. 
alter resource monitor ...;
drop resource monitor ...;
show resource monitors ...; -- list all resource monitors that you have access to
```

## Databases, Schemas, & Shares
```sql
create database ...; -- need to be sysadmin+, or with "create database" privilege on the account
alter database ...; -- must be db owner, may require more privileges depends on work
describe database ...; -- shows schemas in the db, etc
drop database ...;
undrop database ...;
show databases; -- list all dbs you have access to, optionally including dropped ones in time travel

create schema ...;
alter schema ...; -- must be schema owner, and have "create schema" privilege on the db
describe schema ...; -- lists objects in the schema
drop schema ...;
undrop schema ...;
show schemas; -- list all schemas you have access to, optionally including dropped ones in time travel

create share; -- creates an empty share. Need to be account admin, or have "create share" privilege on the account object
alter share; -- need to be owner of the share, or have "create share" privilege
drop share; -- need to be the owner of the share
describe share; -- list the objects in a share. Need to be account admin. 
show shares; -- list all inbound/outbound shares. Need to be account admin, or with "import share" privilege
```

## Tables, Views, & Sequences
```sql
show objects ...; -- list tables/views that you have access to

-- need to have "create table" privilege on the schema object
-- a schema cannot contain tables/views with the same name
create table ...;
create table ... as select ...; -- create table with a select query
create table ... using template ...; -- create table with infer_schema() from staged files 
create table ... like ...; -- create table with same schema as an existing table, without data
create table ... clone ...; -- create table via cloning from another table, can use time travel

-- Inside a transaction, any DDL statement commits the current transaction, 
-- before executing the DDL statement itself. 
-- The DDL statement then runs in its own transaction.
-- The next statement after the DDL statement starts a new transaction.

-- For compatibility with other databases, Snowflake provides constraint properties.
-- Note, they are not enforced or maintained by Snowflake.
-- except the "not null" constraint
create table ... constraint ...; 

alter table ...;
-- rename table
-- alter/add/drop/rename col 
-- set table properties, such as clustering, copy options, add masking policy, search optimization, ...
-- set/unset tag, add/drop row access policy

-- Adding a new column with a default value containing a function is not currently supported.
-- If a new column with a default value is added to a table, all of the existing rows are populated with the default value.


alter table ... alter column ...;

drop/undrop table ...;
show tables [like ...] [in account/database/schema ...]; -- list tables in scope
show columns [like ...] [in account/database/schema/table/view ...]; -- list cols in scope
show primary keys [in account/database/schema/table/view ...]; -- list pks in scope
describe table/view ... [type = columns/stage]; -- can use describe table for view, and vise versa

describe search optimization on ...; -- shows search optimization info for a table and its cols

-- dynamic tables
-- Dynamic tables are updated as underlying database objects change. 
-- Change tracking must be enabled on all its underlying objects.
-- Snowflake will attempt to enable change tracking on all underlying objects, when a dynamic table is created.
-- min lag behind base table could be 1 min
-- If the dynamic table A depends on dynamic table B, 
-- the min lag for A must >= the lag for B.
create dynamic table ...; -- need to have "create dynamic table" privilege on the schema
alter dynamic table ... suspend/resume; -- suspend/resume its refresh, and dynamic tables that rely on it
alter dynamic table ... refresh; -- can also refresh it even if its suspended
alter dynamic table ... set ...; -- need "operate" privilege on the table to alter it
describe dynamic table ...; -- need "monitor" privilege on the table
drop dynamic table ...; -- your role must be its owner
show dynamic tables ...; -- list the dynamic tables you have access to (have monitor privilege on them)

-- external table
-- It reads from one or more files from external stage when queried,
-- and outputs the data in one variant col.
-- Refreshing the external table metadata synchronizes the metadata, 
-- with the current list of data files in the specified stage path.
-- This is required for the metadata to register any existing data files
-- in the named external stage specified in the "location = ...".
-- Recommend to manually batch refresh, if path has >= 1m files.
-- auto_refresh updates metadata when new/updated files are available (need event notification). 
-- "copy grants" option specifies whether retain the replaced ext tables's permissions in the new ext table, if using "create or replace". 
-- Partition columns optimize query performance,
-- by pruning out the data files that do not need to be scanned.
-- You can set partition using expression automatically, or add/rmv it manually
create external table ...; -- need "create external table" privilege on the schema
alter external table ... refresh ...;
alter external table ... add/remove files ...; -- for manual refresh
alter external table ... set/unset ...;
-- An external table cannot be recovered using Time Travel; 
-- also, there is no UNDROP EXTERNAL TABLE command. 
-- A dropped external table must be recreated.
drop external table ... [cascade/restrict];
show external tables ...;
describe external table ...;

-- event table
create event table ...;
alter table ...;
show event tables ...;
desc event table ...;

-- view
-- When a view is created, 
-- unqualified references to tables and other database objects are resolved in the view's schema, not in the session's current schema.
create view ...;
alter/drop/desc view ...;
show views ...;

--  materialized view
create materialized view as ...; -- need "create materialized view" privilege on the schema
alter/drop/desc materialized view ...;
show materialized views ...;

-- sequence
-- A sequence does not necessarily produce a gap-free sequence
create/alter/drop/desc sequence ...;
show sequences ...;
```

## Functions, Procedures, & Scripting
```sql
-- UDFs
-- Depending on the handler language, 
-- you can either include the handler code in the CREATE FUNCTION statement 
-- or refer the handler location on a stage from CREATE FUNCTION, 
-- where the handler is precompiled, or as a source code.
-- UDFs are identified and resolved by the combination of the name and argument types.
create/alter/drop/desc function ...;
show user functions ...;

-- external functions
create/alter/drop/desc function ...;
show external functions ...;

-- Stored procedure
-- Stored procedures are not atomic; 
-- if one statement in a stored procedure fails, 
-- the other statements in the stored procedure are not necessarily rolled back.
-- Owner's rights stored procedures have less access to the caller's environment, -- and Snowflake defaults to this higher level of privacy/security. 
-- Stored procedures support overloading
create procedure ...;
call ... [into :my_var];
with ... as procedure ... call ...; -- for anonymous procedure, use like a CTE
alter/drop/desc procedure ...;
show procedures ...;

-- snowflake scripting
execute immediate my_stmt using my_bind_var1, my_bind_var2, ...;
execute immediate from myFilePath; -- run statements in a file in stage, allow for a version-controlled file
```

## Streams & Tasks
```sql
-- stream
-- A stream can be queried multiple times to update multiple objects in one transaction, and it will return the same data.
-- The stream offset is advanced, when it is used in a DML statement.
-- For streams on shared tables, the retention period for source table is not extended automatically
-- change tracking of an object can only be enabled by its owner, or higher role
-- A stream on a view breaks, if the source view or underlying tables are dropped/recreated
create stream ... on table/view/stage/external table ...;
alter stream ... set/unset ...;
desc stream ...;
drop stream ...;
show streams ...;

-- task
create task ... as ...;
desc task ...;
alter/drop task ...;
-- manually trigger an async single run of a scheduled task. 
-- A successful run of a root task triggers a cascading run of child tasks in the DAG, as though the root task had run on its defined schedule.
execute task ...; -- RETRY LAST creates a new graph run, which begins execution at the last failed task(s).
show tasks ...;
```

## Classes & Instances
```sql
show classes ...;
drop instance ...; -- rmv an instance of a class
```

## Security
```sql
-- network policy
-- When an IP is in both ALLOWED_IP_LIST and BLOCKED_IP_LIST, it is blocked.
create network policy ...; -- need to be security admin or +, or have "create network policy" privilege on the account object
alter/drop/desc network policy ...;
show network policies ...;  

-- network rule
create network rule ...; -- need to be security admin or +, or have "create network rule" privilege on the schema object
alter/drop/desc network rule ...;
show network rules ...;

-- packages policy
create packages policy ...;
alter/drop/desc packages policy ...;
show packages policies ...;

-- password policy
create password policy ...; -- need "create password policy" privilege on the schema object
alter/drop/desc password policy ...;
show password policies ...;

-- session policy (sets session timeout in secs)
-- Needs Enterprise edition and +
create session policy ...; -- need "create session policy" privilege on the schema object
alter/drop/desc session policy ...;
show session policies ...;

-- secret
create secret ...; -- need "create secret" privilege on the schema object
alter/drop/desc secret ...;
show secrets ...;
```

## Data Governance
```sql
-- column-level security
-- can be normal masking policy (e.g., relies on only one col)
-- or conditional masking policy (e.g., an additional col participate in decision making)
create masking policy ... as ... returns ...; -- need "create masking policy" privilege on the schema object
alter/drop/desc masking policy ...;
show masking policies ...;

-- row access policy
-- If a database object has both a row access policy, and one or more masking policy, the row access policy is evaluated first.
-- The same column in an obj cannot live in both masking policy signature and row access policy signature for that obj.
create row access policy ... as ...;
alter/drop/desc row access policy ...;
show row access policies ...;

-- tag
-- tag can have masking policies attached to it
create tag ...; -- need "create tag" privilege on the schema object
alter/drop/undrop tag ...;
show tags ...;
```

## Data Loading & Unloading
```sql
-- stage
-- An internal/external stage can include a directory table. 
-- Directory tables store a catalog of staged files in cloud storage.
-- you can choose to "match by column name" for csv/JSON/Avro/ORC/Parquet files for data loading
-- copy options: ...
create/alter/drop/desc stage ...;
show stages ...;

-- file format
create/alter/desc/drop file format ...;
show file formats ...;

-- pipe
create/alter/desc/drop pipe ...;
show pipes ...;

-- snowpipe streaming
show channels ...;

-- loading/unloading
copy into ... from ...;
```

## File Staging
```sql
-- file staging
put ... @...;
get @... ...;
list ...;
remove ...; -- rmv fils from internal/external stage
```

## Alerts
```sql
create/alter/drop/desc/execute alert ...;
show alerts ...;
```

## Native Apps Framework
skipped. 

## Streamlit
skipped.
