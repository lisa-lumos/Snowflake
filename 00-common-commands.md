# Common Commands
```sql
describe user lisa; 

-- grants ------------------------------------------------------------
grant usage on database mydb to role analyst;

grant usage, create file format, create stage, create table on schema mydb.public to role analyst;

grant operate, usage on warehouse mywh to role analyst;

-- create/drop objects ------------------------------------------------------------
create or replace database mydatabase;
create or replace temporary table mycsvtable (
  id integer,
  last_name string,
  first_name string);

create or replace temporary table myjsontable (
  json_data variant);

create or replace warehouse mywarehouse with
  warehouse_size='X-SMALL'
  auto_suspend = 120
  auto_resume = true
  initially_suspended=true;

create or replace file format mycsvformat
  type = 'CSV'
  field_delimiter = '|'
  skip_header = 1;

create or replace file format myjsonformat
  type = 'JSON'
  strip_outer_array = true;

create or replace stage my_csv_stage
  file_format = mycsvformat;

create or replace stage my_json_stage
  file_format = myjsonformat;

drop database if exists mydatabase;
drop warehouse if exists mywarehouse;


-- staging files ------------------------------------------------------------
create or replace stage my_stage -- create a internal named stage, associate stage with named file format
file_format = my_csv_format;

create or replace stage my_stage -- create a named stage, associate stage with anonymous file format
file_format = (type = 'CSV' field_delimiter = '|' skip_header = 1);

create stage my_stage 
url='s3://mybucket/US/California/san_diego/' 
credentials=(aws_key_id='1a2b3c' aws_secret_key='4x5y6z');

-- (Linux/Unix)
put file:///data/data.csv @~/staged; -- put local file in the /data dir into user stage in specified folder named "staged" 
put file:///data/data.csv @%mytable; -- ... into table stage
put file:///data/data.csv @my_stage; -- ... into named stage
put file:///tmp/load/contacts*.csv @my_csv_stage auto_compress=true;
put file:///tmp/load/contacts.json @my_json_stage auto_compress=true;

-- (Windows OS), to do the same as above
put file://c:\data\data.csv @~/staged; 
put file://c:\data\data.csv @%mytable;
put file://c:\data\data.csv @my_stage;
put file://c:\temp\load\contacts*.csv @my_csv_stage auto_compress=true;
put file://c:\temp\load\contacts.json @my_json_stage auto_compress=true;

list @~;        -- list files in user stage
list @%my_table; -- list files in table stage
list @my_stage; -- list files in named stage

-- bulk loading files ------------------------------------------------------------
-- this requires a warehouse
copy into mytable -- from files with filename that has mydata as prefix
from @%mytable/US/california/san_diego/2016/06/01/11/mydata;

copy into mytable -- from files specified in the path
from @%mytable/US/california/san_diego/2016/06/01/11/
files=('mydata1.csv', 'mydata1.csv');

copy into mytable -- from files with matched filenames in the path, delete files after copied successfully
from @%mytable/US/california/san_diego/2016/06/01/11/
pattern='.*mydata[^[0-9]{1,3}$$].csv'
purge = true;

copy into mytable -- from all files in specified path in user stage
from @~/staged 
file_format = (format_name = 'my_csv_format');

copy into mytable -- from all files from the table stage (from clause omitted)
file_format = (type = csv field_delimiter = '|' skip_header = 1);

copy into mytable -- from all files in named stage, using file format associated with this stage
from @my_stage;

copy into mycsvtable
from @my_csv_stage/contacts1.csv.gz
file_format = (format_name = mycsvformat)
on_error = 'skip_file';

copy into mycsvtable
from @my_csv_stage
file_format = (format_name = mycsvformat)
pattern='.*contacts[1-5].csv.gz'
on_error = 'skip_file';

copy into mytable(c2) -- copy the first column in the file to c2 column in table
from (select t.$1 from @mystage/myfile.csv.gz t);

copy into home_sales(city, zip, sale_date, price) -- skip columns when loading
from (select t.$1, t.$2, t.$6, t.$7 from @mystage/sales.csv.gz t)
file_format = (format_name = mycsvformat);

copy into home_sales(city, zip, sale_date, price) -- use substr when loading
from (select substr(t.$2,4), t.$1, t.$5, t.$4 from @mystage t)
file_format = (format_name = mycsvformat);

copy into casttb(col1, col2, col3) -- convert data types (casting) when loading
from (
  select to_binary(t.$1, 'utf-8'),to_decimal(t.$2, '99.9', 9, 5),to_timestamp_ntz(t.$3)
  from @~/datafile.csv.gz t
)
file_format = (type = csv);

copy into mytable (col1, col2, col3) -- include sequence val as col1 when loading
from (
  select seq1.nextval, $1, $2
  from @~/myfile.csv.gz t
)

copy into mytable -- use validation before loading
from @my_stage
validation_mode = 'RETURN_ALL_ERRORS';

copy into mytable -- ignore the space before opening quotes before values
from @%mytable
file_format = (type = csv trim_space=true field_optionally_enclosed_by = '0x22');

copy into myjsontable
from @my_json_stage/contacts.json.gz
file_format = (format_name = myjsonformat)
on_error = 'skip_file';

copy into mytable -- load each array elem in JSON file into its own row as variant
from @~/test1.json 
file_format = (type = 'JSON' strip_outer_array = true strip_null_values = true);

copy into home_sales(city, postal_code, sq_ft, sale_date, price) -- load JSON file with transformation into different columns
from (
  select
    $1:location.city::varchar,
    $1:location.zip::varchar,
    $1:dimensions.sq_ft::number,
    $1:sale_date::date,
    $1:price::number
  from @mystage/sales.json.gz t
);

create or replace table flattened_source -- load JSON file via flatten table function
(seq string, key string, path string, index string, value variant, element variant)
as select
  seq::string, 
  key::string, 
  path::string, 
  index::string, 
  value::variant, 
  this::variant
from @mystage/sales.json.gz, table(flatten(input => parse_json($1)));

copy into splitjson(col1, col2) -- split ip address into arrays of 4 elements
from (
  select split($1:ip_address.router1, '.'),split($1:ip_address.router2, '.')
  from @mystage/ipaddress.json.gz t
);

copy into parquet_col -- load parquet via transformation into different cols
from (
  select
    $1:o_custkey::number,
    $1:o_orderdate::date,
    $1:o_orderstatus::varchar,
    $1:o_totalprice::varchar
  from @mystage/mydata.parquet
);

-- remove staged files
remove @mystage/path1/subpath2; -- rmv all files under this path
remove @my_csv_stage pattern='.*.csv.gz';
remove @my_json_stage pattern='.*.json.gz';

select * -- check copy history for a table
from table(information_schema.copy_history(
  table_name=>'MYTABLE', 
  start_time=> dateadd(hours, -1, current_timestamp())
  )
);

-- query staged files ------------------------------------------------------------
create or replace file format myformat  -- create a named file format
type = 'csv' 
field_delimiter = '|';

select t.$1, t.$2 -- select first and second column based on file format
from @mystage1 (file_format => 'myformat', pattern=>'.*data.*[.]csv.gz') t;

select ascii(t.$1), ascii(t.$2) -- get ASCII code for the first char for each col
from @mystage1 (file_format => myformat) t;

create or replace file format my_json_format -- create a named file format
type = 'json';

select parse_json($1):a.b from @mystage2/data1.json.gz; -- select x1 and x2 from JSON: {"a": {"b": "x1","c": "y1"}}, {"a": {"b": "x2","c": "y2"}}

-- query metadata of staged files ------------------------------------------------------------
select metadata$filename, metadata$file_row_number, t.$1, t.$2 
from @mystage1 (file_format => myformat) t;

select metadata$filename, metadata$file_row_number, parse_json($1) 
from @mystage2/data1.json.gz;

create or replace table table1 ( -- create a table with first 2 cols to store metadata
  filename varchar,
  file_row_number varchar,
  col1 varchar,
  col2 varchar
);

copy into table1(filename, file_row_number, col1, col2) -- put metadata cols into table
  from (
    select metadata$filename, metadata$file_row_number, t.$1, t.$2 
    from @mystage1/data1.csv.gz (file_format => myformat) t
  );

-- retrieve errors
create or replace table save_copy_errors 
as select * from table(validate(mycsvtable, job_id=>'<query_id>'));

select * from save_copy_errors;

-- bulk unloading data ------------------------------------------------------------
copy into @mystage/myfile.csv.gz from mytable
file_format = (type=csv compression='gzip')
single=true
max_file_size=4900000000;

copy into @mystage
from null_empty1
  file_format = (format_name = 'my_csv_format');

copy into @mystage
from (select object_construct('id', id, 'first_name', first_name, 'last_name', last_name, 'city', city, 'state', state) from mytable)
file_format = (type = json);

copy into @mystage/myfile.parquet 
from (select id, name, start_date from mytable)
file_format=(type='parquet')
header = true;

copy into @mystage
from (select cast(c1 as tinyint) ,
             cast(c2 as smallint) ,
             cast(c3 as int),
             cast(c4 as bigint) from mytable)
file_format=(type=parquet);



-- warehouse ------------------------------------------------------------
alter warehouse mywh set scaling_policy = 'economy';

-- snowpark-optimized wh
create or replace warehouse snowpark_opt_wh with
  warehouse_size = 'medium'
  warehouse_type = 'snowpark-optimized';

-- query optimization service ------------------------------------------------------------
select parse_json(system$estimate_query_acceleration('8cd54bf0-1651-5b1c-ac9c-6a9582ebd20f')); -- for a specific query

-- for queries that benefit the most from the service
select query_id, eligible_query_acceleration_time
from snowflake.account_usage.query_acceleration_eligible
order by eligible_query_acceleration_time desc;

-- ... for a specific wh
select query_id, eligible_query_acceleration_time
from snowflake.account_usage.query_acceleration_eligible
where warehouse_name = 'mywh'
order by eligible_query_acceleration_time desc;

-- for warehouses that benefit the most from the service
select warehouse_name, sum(eligible_query_acceleration_time) as total_eligible_time
from snowflake.account_usage.query_acceleration_eligible
group by warehouse_name
order by total_eligible_time desc;

-- enable the query acceleration service with a maximum scale factor of 0
alter warehouse my_wh set
  enable_query_acceleration = true
  query_acceleration_max_scale_factor = 0;

select * from snowflake.account_usage.query_history
where query_acceleration_partitions_scanned > 0
and start_time >= dateadd(hour, -24, current_timestamp())
order by query_acceleration_bytes_scanned desc;


-- clustering ------------------------------------------------------------
-- cluster by base columns
create or replace table t1 (c1 date, c2 string, c3 number) cluster by (c1, c2);

-- cluster by expressions
create or replace table t2 (c1 timestamp, c2 string, c3 number) cluster by (to_date(c1), substring(c2, 0, 10));

-- cluster by paths in variant columns
create or replace table t3 (t timestamp, v variant) cluster by (v:"Data":id::number);

-- cluster by base columns
alter table t1 cluster by (c1, c3);

alter table t1 drop clustering key;

alter table t1 suspend recluster;
alter table t1 resume recluster;

select to_date(start_time) as date,
  database_name,
  schema_name,
  table_name,
  sum(credits_used) as credits_used
from snowflake.account_usage.automatic_clustering_history
where start_time >= dateadd(month,-1,current_timestamp())
group by 1,2,3,4
order by 5 desc;

-- tables, views ------------------------------------------------------------
create temporary table mytemptable (id number, creation_date date);
create transient table mytranstable (id number, creation_date date);

alter table t1 add search optimization on equality(c1, c2, c3);
alter table test_table add search optimization;
describe search optimization on t1;
alter table t1 drop search optimization on substring(c2);
alter table test_table drop search optimization;

create view doctor_view as
    select patient_id, patient_name, diagnosis, treatment from hospital_table;

create or replace secure view widgets_view as
    select w.*
        from widgets as w
        where w.id in (select widget_id
                           from widget_access_rules as a
                           where upper(role_name) = current_role()
                      )
    ;

-- example of a materialized view with a range filter
create materialized view v1 as
    select * from table1 where column_1 between 100 and 400;

alter materialized view vulnerable_pipes cluster by (installation_year);

select * from table(information_schema.materialized_view_refresh_history());

create transient table my_new_table like my_old_table copy grants;
insert into my_new_table select * from my_old_table;

create transient table my_transient_table as select * from mytable;

create transient table foo clone bar copy grants;


-- data types ------------------------------------------------------------
create table my_table (my_variant_column variant);
copy into my_table ... file format = (type = 'json') ...

insert into my_table (my_variant_column) select parse_json('{...}');

-- Operators : and . and [] return VARIANT values containing strings
select src:dealership -- <column>:<level1_element>
-- src is case insensitive, but dealership is
from car_sales
order by 1;

select src:"company name" from partners; -- use "" for attribute names with spaces etc
select zipcode_info:"94987" from addresses;
select measurements:"#sPerSquareInch" from english_metrics;

select src:salesperson.name -- <column>:<level1_element>.<level2_element>.<level3_element>
from car_sales
order by 1;

select src:customer[0].name, src:vehicle[0] -- customer field is an array
from car_sales
order by 1;

select src:vehicle[0].price::number * 0.10 as tax -- use :: to cast vals
from car_sales
order by tax;

select src:dealership, src:dealership::varchar
from car_sales
order by 2;

select
  value:name::string as "customer name",
  value:address::string as "address"
from
  car_sales
, lateral flatten(input => src:customer); -- flatten table function to parse arrays

-- +--------------------+-------------------+
-- | Customer Name      | Address           |
-- |--------------------+-------------------|
-- | Joyce Ridgely      | San Francisco, CA |
-- | Bradley Greenbloom | New York, NY      |
-- +--------------------+-------------------+

select 'The First Employee Record is '|| -- select from a staged json file
    s.$1:root[0].employees[0].firstname||
    ' '||s.$1:root[0].employees[0].lastname
from @%customers/contacts.json.gz (file_format => 'my_json_format') as s;

copy into <table>
from @~/<file>.json
file_format = (type = 'json' strip_outer_array = true);

-- enable directory tables for a stage
create stage mystage
  directory = (enable = true)
  file_format = myformat;

create stage mystage
  url='s3://load/files/'
  storage_integration = my_storage_int
  directory = (enable = true);

alter stage mystage refresh;
select * from directory(@mystage);





-- SnowSQL ------------------------------------------------------------
> snowsql
> !set prompt_format=[#FF0000][user].[role].[#00FF00][database].[schema].[#0000FF][warehouse]>
> !q


-- parameters ------------------------------------------------------------
-- account parameters (only can be set at account level)
alter account set min_data_retention_time_in_days = 5; -- some admins can set these

-- session parameters (can be set at account, user and session levels)
--   alter session > alter user > alter account)
alter user lisa set default_role = sysadmin; -- some admins and users(only for themselves) can both set these
alter user lisa set default_warehouse = compute_wh;
alter session set lock_timeout = 3600; -- users can use this to set session parameters within their sessions
alter session unset lock_timeout; -- set back to default

-- object parameters (can be set at both account and object level)
--   alter table/pipe... > alter schema > alter database > alter account; 
--   alter warehouse > alter account
alter account set data_retention_time_in_days = 5; -- for time travel
alter table t1 set data_retention_time_in_days = 5;

show parameters; -- show only session parameters
show parameters in database db1; -- show object parameters for this db
show parameters in warehouse wh1; -- show object parameters for this wh
show parameters in account; -- show all account, session and object parameters
show parameters like 'time%' in account; -- show all params start with "time"

-- CDC ------------------------------------------------------------
create secure view v change_tracking = true as select col1, col2 from t;
alter view v2 set change_tracking = true;
create table t (col1 string, col2 number) change_tracking = true;
alter table t1 set change_tracking = true;


-- queries ------------------------------------------------------------
-- joins
select 
  p.project_id, 
  project_name, 
  employee_id, 
  employee_name, 
  e.project_id
from 
  projects as p 
  join employees as e
  on e.project_id = p.project_id
order by p.project_id, e.employee_id;

-- recursive CTEs
with recursive 
managers(indent, employee_id, manager_id, employee_title)
as (
  select -- starting level
    '' as indent, 
    employee_id, 
    manager_id, 
    title as employee_title
  from employees
  where title = 'president'

  union all

  select -- the next level
    indent || '--- ',
    employees.employee_id, 
    employees.manager_id, 
    employees.title
  from 
    employees 
    join 
    managers
    on employees.manager_id = managers.employee_id
)
select 
  indent || employee_title as title, 
  employee_id, 
  manager_id
from 
  managers
;

-- window functions
select 
  day, 
  sales_today, 
  rank() over (order by sales_today desc) as rank,
  sum(sales_today) over (order by day rows between unbounded preceding and current row) as "sales so far",
  sum(sales_today) over () as total_sales,
  avg(sales_today) over (order by day rows between 2 preceding and current row) as "3-day moving average"
from store_sales_2
order by day;

-- +-----+-------------+------+------------------------+-------------+----------------------+
-- | DAY | SALES_TODAY | RANK | SALES SO FAR           | TOTAL_SALES | 3-DAY MOVING AVERAGE |
-- |-----+-------------+------+------------------------+-------------+----------------------|
-- |   1 |          10 |    5 |                     10 |          84 |               10.000 |
-- |   2 |          14 |    3 |                     24 |          84 |               12.000 |
-- |   3 |           6 |    6 |                     30 |          84 |               10.000 |
-- |   4 |           6 |    6 |                     36 |          84 |                8.666 |
-- |   5 |          14 |    3 |                     50 |          84 |                8.666 |
-- |   6 |          16 |    2 |                     66 |          84 |               12.000 |
-- |   7 |          18 |    1 |                     84 |          84 |               16.000 |
-- +-----+-------------+------+------------------------+-------------+----------------------+

select -- running sum for each month
  month(sales_date) as month_num,
  sum(quantity) over (partition by month(sales_date) order by sales_date) as monthly_cumulative_sum_quantity
from sales
order by sales_date;
-- +-----------+---------------------------------+
-- | MONTH_NUM | MONTHLY_CUMULATIVE_SUM_QUANTITY |
-- |-----------+---------------------------------+
-- |         1 |                               1 |  -- sum = 1
-- |         1 |                               4 |  -- sum = 1 + 3
-- |         1 |                               9 |  -- sum = 1 + 3 + 5
-- |         2 |                               2 |  -- sum = 0 + 2 (new month)
-- +-----------+---------------------------------+

select -- sliding window of size 2, for each month
  month(sales_date) as month_num,
  quantity,
  sum(quantity) over (partition by month(sales_date) 
                      order by sales_date
                      rows between 1 preceding and current row) as monthly_sliding_sum_quantity
from sales
order by sales_date;

-- match recognize
select * from stock_price_history
  match_recognize(
    partition by company
    order by price_date
    measures
      match_number() as match_number,
      first(price_date) as start_date,
      last(price_date) as end_date,
      count(*) as rows_in_sequence,
      count(row_with_price_decrease.*) as num_decreases,
      count(row_with_price_increase.*) as num_increases
    one row per match
    after match skip to last row_with_price_increase
    pattern(row_before_decrease row_with_price_decrease+ row_with_price_increase+)
    define
      row_with_price_decrease as price < lag(price),
      row_with_price_increase as price > lag(price)
  )
order by company, match_number;

-- sequences
create or replace sequence seq1;
select seq1.nextval a, seq1.nextval b from dual; -- return diff vals
select seqref.a a, seqref.a b from (select seq1.nextval a from dual) seqref; -- return same val

create or replace sequence seq1;
create or replace table foo (k number default seq1.nextval, v number);
-- insert rows with unique keys generated by seq1 and explicit values
insert into foo (v) values (100);
insert into foo values (default, 101);
-- new keys are distinct from preexisting keys.
insert into foo (v) select v from foo;
-- insert row with explicit values for both columns
insert into foo values (1000, 1001);
select * from foo;
-- +------+------+
-- |    K |    V |
-- |------+------|
-- |    1 |  100 |
-- |    2 |  101 |
-- |    3 |  100 |
-- |    4 |  101 |
-- | 1000 | 1001 |
-- +------+------+

-- primary data tables
create or replace table people (id number, firstname string, lastname string);
create or replace table contact (id number, p_id number, c_type string, data string);
create or replace sequence people_seq;
create or replace sequence contact_seq;
create or replace table input (json variant);
insert all
  when 1=1 then
    into contact values (
      c_next, 
      p_next, 
      contact_value:contacttype, 
      contact_value:contactdata
    )
  when contact_index = 0 then
    into people values (
      p_next, 
      person_value:firstname, 
      person_value:lastname
    )

select * from
(
  select 
    f1.value person_value, 
    f2.value contact_value, 
    f2.index contact_index, 
    p_seq.nextval p_next, 
    c_seq.nextval c_next
  from 
    input, 
    lateral flatten(input.json) f1, 
    table(getnextval(people_seq)) p_seq,
    lateral flatten(f1.value:contacts) f2, 
    table(getnextval(contact_seq)) c_seq
);

-- persisted query results (result cache)
show tables;
select -- show the tables that are empty.
  "schema_name", 
  "name" as "table_name", 
  "rows"
from table(result_scan(last_query_id()))
where "rows" = 0;

-- distinct counts
create or replace table 
  daily_uniques
as
select
  visitdate,
  hll_export(hll_accumulate(sourceip)) as hll_sourceip
from uservisits
group by visitdate;

select -- aggregation of the HLL structures is much faster than over the base data
  extract(year from visitdate) as visit_year,
  extract(month from visitdate) as visit_month,
  hll_estimate(hll_combine(hll_import(hll_sourceip))) as distinct_ips
from daily_uniques
where visitdate between '2000-01-01' and '2000-12-31'
group by 1,2
order by 1,2;

-- minhash
create or replace table mhtab1(c1 number,c2 double,c3 text,c4 date);
create or replace table mhtab2(c1 number,c2 double,c3 text,c4 date);
insert into mhtab1 values
    (1, 1.1, 'item 1', to_date('2016-11-30')),
    (2, 2.31, 'item 2', to_date('2016-11-30')),
    (3, 1.1, 'item 3', to_date('2016-11-29')),
    (4, 44.4, 'item 4', to_date('2016-11-30'));
insert into mhtab2 values
    (1, 1.1, 'item 1', to_date('2016-11-30')),
    (2, 2.31, 'item 2', to_date('2016-11-30')),
    (3, 1.1, 'item 3', to_date('2016-11-29')),
    (4, 44.4, 'item 4', to_date('2016-11-30')),
    (6, 34.23, 'item 6', to_date('2016-11-29'));
select approximate_similarity(mh) from
    ((select minhash(100, *) as mh from mhtab1)
    union all
    (select minhash(100, *) as mh from mhtab2));

-- +----------------------------+
-- | APPROXIMATE_SIMILARITY(MH) |
-- |----------------------------|
-- |                       0.79 |
-- +----------------------------+

-- shares ------------------------------------------------------------
show shares;
-- listing
create secure view paid_v as -- return rows if customer paid
  select *
  from my_table
  where system$is_listing_purchased() = true;

create secure view paid_v as -- return rows that is_free or if consumer paid
  select *
  from my_table
  where
    is_free
    or
    system$is_listing_purchased() = true;

create secure view paid_v as -- return rows within past 7 days or if consumer paid
  select *
  from my_table
  where
    (timestamp > current_timestamp() - interval '7 days')
    or
    system$is_listing_purchased() = true;

execute using 
  share_context(system$is_listing_purchased=>'false') -- test if share created properly
as
  select *
  from example_db.example_sc.purchased_view
;

execute using 
  share_context(system$is_listing_purchased=>'true')
as
  select *
  from example_db.example_sc.purchased_view
;

-- use database roles in a share
use role accountadmin;
create database role d1.r1;
create database role d1.r2;
grant usage on schema d1.s1 to database role d1.r1;
grant select on view d1.s1.v1 to database role d1.r1;
grant usage on schema d1.s1 to database role d1.r2;
grant select on view d1.s1.v2 to database role d1.r2;
show grants to database role d1.r1;
show grants to database role d1.r2;
create share share1;
grant usage on database d1 to share share1;
grant database role d1.r1 to share share1;
grant database role d1.r2 to share share1;
alter share share1 add accounts = org1.consumer1,org1.consumer2;

-- manage database roles
alter database role d1.r1 rename to d1.r3;
drop database role d1.r2;

-- create a share by directly granting prilileges
use role accountadmin;
create share sales_s;
grant usage on database sales_db to share sales_s;
grant usage on schema sales_db.aggregates_eula to share sales_s;
grant select on table sales_db.aggregates_eula.aggregate_1 to share sales_s;

show grants to share sales_s; -- share has these privileges
alter share sales_s add accounts=xy12345, yz23456;
show grants of share sales_s; -- shared to these accounts

-- for validating shared objects
alter session set simulated_data_sharing_consumer = xy12345;

-- to let consumer create their own streams
alter table ... change_tracking = true

-- revoke
revoke select on view sales_db.aggregates_eula.agg_secure from share sales_s;

-- share data from multiple databases
-- sample database database1
create database database1;
create schema database1.sch;
create table database1.sch.table1 (id int);
create view database1.sch.view1 as select * from database1.sch.table1;
-- sample database database2
create database database2;
create schema database2.sch;
create table database2.sch.table2 (id int);
-- sample database to be shared
create database database3;
create schema database3.sch;
create table database3.sch.table3 (id int);
create secure view database3.sch.view3 as -- sample view to be shared.
select view1.id as view1id, table2.id as table2id, table3.id as table3id
from database1.sch.view1 view1,
     database2.sch.table2 table2,
     database3.sch.table3 table3;
create share share1;
grant usage on database database3 to share share1;
grant usage on schema database3.sch to share share1;
grant reference_usage on database database1 to share share1;
grant reference_usage on database database2 to share share1;
grant select on view database3.sch.view3 to share share1;

-- replicating shares across rgions/cloud-platforms
-- in primary account:
use role accountadmin;
create role myrole;
grant create replication group on account to role myrole;
use role myrole;
create replication group myrg
    object_types = databases, shares
    allowed_databases = db1, db2
    allowed_shares = s1
    allowed_accounts = myorg.myaccount2, myorg.myaccount3
    replication_schedule = '10 minute';
grant replicate on replication group myrg to role my_replication_role;
show replication accounts;
show replication groups;
-- in secondary account:
use role accountadmin;
create role myrole;
grant create replication group on account to role myrole;
use role myrole;
create replication group myrg as replica of myorg.myaccount1.myrg;
grant replicate on replication group myrg to role my_replication_role;
use role my_replication_role;
alter replication group myrg refresh; -- manual refresh

-- control data access in a share
use role sysadmin;
create or replace table mydb.private.sensitive_data (
    name string,
    date date,
    time time(9),
    bid_price float,
    ask_price float,
    bid_size int,
    ask_size int,
    access_id string /* granularity for access */ )
    cluster by (date);
insert into mydb.private.sensitive_data
    values('AAPL',dateadd(day,  -1,current_date()), '10:00:00', 116.5, 116.6, 10, 10, 'STOCK_GROUP_1'),
          ('AAPL',dateadd(month,-2,current_date()), '10:00:00', 116.5, 116.6, 10, 10, 'STOCK_GROUP_1'),
          ('MSFT',dateadd(day,  -1,current_date()), '10:00:00',  58.0,  58.9, 20, 25, 'STOCK_GROUP_1'),
          ('MSFT',dateadd(month,-2,current_date()), '10:00:00',  58.0,  58.9, 20, 25, 'STOCK_GROUP_1'),
          ('IBM', dateadd(day,  -1,current_date()), '11:00:00', 175.2, 175.4, 30, 15, 'STOCK_GROUP_2'),
          ('IBM', dateadd(month,-2,current_date()), '11:00:00', 175.2, 175.4, 30, 15, 'STOCK_GROUP_2');
create or replace table mydb.private.sharing_access (
  access_id string,
  snowflake_account string
);
insert into mydb.private.sharing_access values('STOCK_GROUP_1', CURRENT_ACCOUNT());
insert into mydb.private.sharing_access values('STOCK_GROUP_2' '<consumer_account>');
create or replace secure view mydb.public.paid_sensitive_data as
    select name, date, time, bid_price, ask_price, bid_size, ask_size
    from mydb.private.sensitive_data sd
    join mydb.private.sharing_access sa on sd.access_id = sa.access_id
    and sa.snowflake_account = current_account();
grant select on mydb.public.paid_sensitive_data to public; -- to public role

select count(*) from mydb.private.sensitive_data;
select * from mydb.private.sensitive_data;
select count(*) from mydb.public.paid_sensitive_data;
select * from mydb.public.paid_sensitive_data;
select * from mydb.public.paid_sensitive_data where name = 'AAPL';
alter session set simulated_data_sharing_consumer=<account_name>;
select * from mydb.public.paid_sensitive_data;

use role accountadmin;
create or replace share mydb_shared
  comment = 'Example of using Secure Data Sharing with secure views';
show shares;
/* Option 1: Create a database role */
create database role mydb.dr1;
grant usage on database mydb to database role mydb.dr1;
grant usage on schema mydb.public to database role mydb.dr1;
grant select on mydb.public.paid_sensitive_data to database role mydb.dr1;
grant database role mydb.dr1 to share mydb_shared;
/* Option 2: Grant privileges on the database objects to include in the share.  */
grant usage on database mydb to share mydb_shared;
grant usage on schema mydb.public to share mydb_shared;
grant select on mydb.public.paid_sensitive_data to share mydb_shared;
show grants to share mydb_shared;

-- for consumers
use role accountadmin;
show shares;
desc share <provider_account>.mydb_shared;
create database mydb_shared1 from share <provider_account>.mydb_shared;
/* Option 1 */
grant database role mydb_shared1.db1 to role custom_role1;
/* Option 2 */
grant imported privileges on database mydb_shared1 to custom_role1;

use role custom_role1;
show views;
use warehouse <warehouse_name>;
select * from paid_sensitive_data;

-- consume shared data
show shares;
desc share xy12345.sales_s;
create database snow_sales from share xy12345.sales_s;

-- reader accounts
use role accountadmin;
grant create account on account to role sysadmin; -- delegate to sysadmin
use role accountadmin;
create managed account reader_acct1
admin_name = user1, 
admin_password = 'sdfed43da!44' ,
type = reader;

use role accountadmin;
drop managed account reader_acct1;

use role accountadmin;
show managed accounts; -- view all reader accounts

alter user ra_user1 reset password; -- your user can login with the link
alter user ra_user2 reset password;

use role accountadmin;
grant import share on account to sysadmin;

-- data exchange
use role accountadmin;
grant imported privileges on data exchange mydataexchange to sysadmin;

use role accountadmin;
grant create data exchange listing on account to role sysadmin;
grant create data exchange listing on account to sysadmin with grant option;

-- override default share restrictions for business critical account
use role accountadmin;
grant override share restrictions on account to role sysadmin;
use role sysadmin;
alter share <share_name> add accounts = <consumer_account_name> SHARE_RESTRICTIONS=false;

-- alerts ------------------------------------------------------------
use role accountadmin;
create role my_alert_role;
grant execute alert on account to role my_alert_role;
grant role my_alert_role to user my_user;
grant create alert on schema my_schema to role my_alert_role;
grant usage on schema my_schema to role my_alert_role;
grant usage on database my_database to role my_alert_role;
grant usage on warehouse my_warehouse to role my_alert_role;
create or replace alert myalert
  warehouse = mywarehouse
  schedule = '1 minute'
  if( exists(
    select gauge_value from gauge where gauge_value>200))
  then
    insert into gauge_value_exceeded_history values (current_timestamp());
alter alert myalert resume;

grant database role snowflake.alert_viewer to role alert_role;
create or replace alert alert_new_rows -- sends an email if any new rows were added to my_table between the time [when the last successfully evaluated alert was scheduled] and the time [when the current alert has been scheduled]
  warehouse = my_warehouse
  schedule = '1 minute'
  if (exists (
      select *
      from my_table
      where row_timestamp between snowflake.alert.last_successful_scheduled_time()
       and snowflake.alert.scheduled_time()
  ))
  then call system$send_email(...);

alter alert myalert suspend;
alter alert myalert resume;

alter alert my_alert set warehouse = my_other_warehouse;
alter alert my_alert set schedule = '2 minutes';
alter alert my_alert modify condition exists (select gauge_value from gauge where gauge_value>300);
alter alert my_alert modify action call my_procedure();

drop alert myalert;

show alerts; -- list the alerts that were created in the current schema
desc alert myalert;

select *
from
  table(
    information_schema.alert_history(
      scheduled_time_range_start => dateadd('hour',-1,current_timestamp())
    )
  )
order by scheduled_time desc;

-- email notifications ------------------------------------------------------------
create notification integration my_email_int
  type=email
  enabled=true
  allowed_recipients=('first.last@example.com','first2.last2@example.com');
grant usage on integration my_email_int to role my_sp_owner_role;
call system$send_email(
    'my_email_int', -- notification integration name
    'person1@example.com, person2@example.com',
    'Email Alert: Task A has finished.',
    'Task A has successfully finished.\nStart Time: 10:10:32\nEnd Time: 12:15:45\nTotal Records Processed: 115678'
);

-- network policies ------------------------------------------------------------
show parameters like 'network_policy' in account;
show parameters like 'network_policy' in user jsmith;

-- session policies ------------------------------------------------------------
select * -- return a row for each user assigned the session policy
from table(
  my_db.information_schema.policy_references(
    policy_name => 'my_db.my_schema.session_policy_prod_1'
  )
);

-- Access control ------------------------------------------------------------
use role useradmin;
create role db_hr_r;
create role db_fin_r;
create role db_fin_rw;
create role accountant;
create role analyst;
use role securityadmin;
-- grant read-only permissions on database hr to db_hr_r role.
grant usage on database hr to role db_hr_r;
grant usage on all schemas in database hr to role db_hr_r;
grant select on all tables in database hr to role db_hr_r;
-- grant read-only permissions on database fin to db_fin_r role.
grant usage on database fin to role db_fin_r;
grant usage on all schemas in database fin to role db_fin_r;
grant select on all tables in database fin to role db_fin_r;
-- grant read-write permissions on database fin to db_fin_rw role.
grant usage on database fin to role db_fin_rw;
grant usage on all schemas in database fin to role db_fin_rw;
grant select,insert,update,delete on all tables in database fin to role db_fin_rw;
grant role db_fin_rw to role accountant;
grant role db_hr_r to role analyst;
grant role db_fin_r to role analyst;
grant role accountant,analyst to role sysadmin;
grant role accountant to user user1;
grant role analyst to user user2;

grant select on future tables in schema s1 to role r1;
grant select on future tables in schema s1 to role r2;
grant select on all tables in schema s1 to role r2;
revoke select on future tables in schema s1 from role r1;
revoke select on all tables in schema s1 from role r1;

-- view granted privileges
show grants on schema database_a.schema_1;
show grants to role r1;

-- enabling non-account admins to monitor usage & billing history
grant monitor usage on account to role custom;
grant imported privileges on database snowflake to role custom;

-- client side encryption ------------------------------------------------------------
create stage encrypted_customer_stage -- stage with master key
url='s3://customer-bucket/data/'
credentials=(aws_key_id='ABCDEFGH' aws_secret_key='12345678')
encryption=(master_key='eSxX...=');
create table users (id bigint, name varchar(500), purchases int);
copy into users from @encrypted_customer_stage/users; -- load
create table most_purchases as select * from users order by purchases desc limit 10;
copy into @encrypted_customer_stage/most_purchases from most_purchases; -- unload

-- encryption key ------------------------------------------------------------
alter account set periodic_data_rekeying = true;

-- tag ------------------------------------------------------------
create tag cost_center
  allowed_values 'finance', 'engineering';
select get_ddl('tag', 'cost_center');
alter tag cost_center
  add allowed_values 'marketing';
alter tag cost_center
  drop allowed_values 'engineering';
select system$get_tag_allowed_values('governance.tags.cost_center');

use role useradmin;
create role tag_admin;
use role accountadmin;
grant create tag on schema mydb.mysch to role tag_admin;
grant apply tag on account to role tag_admin;
use role useradmin;
grant role tag_admin to user jsmith;
use role tag_admin;
use schema mydb.mysch;
create tag cost_center;
use role tag_admin;
create warehouse mywarehouse with tag (cost_center = 'sales');
use role tag_admin;
alter warehouse wh1 set tag cost_center = 'sales';
alter table hr.tables.empl_info
  modify column job_title
  set tag visibility = 'public';

select * from snowflake.account_usage.tags
order by tag_name;
select system$get_tag('cost_center', 'my_table', 'table');

select * -- account level
from table(
  snowflake.account_usage.tag_references_with_lineage(
    'my_db.my_schema.cost_center'
  )
);
select * from snowflake.account_usage.tag_references -- account level
order by tag_name, domain, object_id;
select * -- database level
from table(
  my_db.information_schema.tag_references(
    'my_table',
    'table'
  )
);
select * -- database level
from table(
  information_schema.tag_references_all_columns(
    'my_table',
    'table'
  )
);

-- data classfication
select system$get_tag_allowed_values('snowflake.core.semantic_category');
select system$get_tag_allowed_values('snowflake.core.privacy_category');
select extract_semantic_categories('hr_data'); -- returns variant in JSON
select -- convert result to a table
    f.key::varchar as column_name,
    f.value:"privacy_category"::varchar as privacy_category,  
    f.value:"semantic_category"::varchar as semantic_category,
    f.value:"extra_info":"probability"::number(10,2) as probability,
    f.value:"extra_info":"alternates"::variant as alternates
  from
  table(flatten(extract_semantic_categories('hr_data')::variant)) as f;

use role accountadmin;
grant database role snowflake.governance_admin to role data_engineer;
grant apply tag on account to role data_engineer;
use role data_engineer;
alter table hr.tables.empl_info
  modify column email
  set tag snowflake.core.semantic_category = 'email';

select * from snowflake.account_usage.tag_references
    where tag_name = 'privacy_category'
    order by object_database, object_schema, object_name, column_name;
select * from
  table(
    my_db.information_schema.tag_references(
      'my_db.my_schema.hr_data.fname',
      'column'
    )
  )
;
select * from
  table(
    my_db.information_schema.tag_references_all_columns(
      'my_db.my_schema.hr_data',
      'table'
    )
  )
;
select system$get_tag(
  'snowflake.core.privacy_category',
  'hr_data.fname',
  'COLUMN'
  )
;

create table hr_data (
    age integer,
    email_address varchar,
    fname varchar,
    lname varchar
);
use role accountadmin;
create role data_engineer;
grant usage on database my_db to role data_engineer;
grant usage on schema my_db.my_schema to role data_engineer;
grant select, update on table my_db.my_schema.hr_data to role data_engineer;
grant database role snowflake.governance_admin to role data_engineer;
use role accountadmin;
create role policy_admin;
grant apply masking policy on account to role policy_admin;
grant database role snowflake.governance_viewer to role policy_admin;
use role data_engineer;
select extract_semantic_categories('my_db.my_schema.hr_data');
call associate_semantic_category_tags(
   'my_db.my_schema.hr_data',
    extract_semantic_categories('my_db.my_schema.hr_data')
);
use role data_engineer;
alter table my_db.my_schema.hr_data
  modify column fname
  set tag snowflake.core.semantic_category='name';
alter table my_db.my_schema.hr_data
modify column fname
set tag snowflake.core.privacy_category='identifier';

use role policy_admin;
select * from snowflake.account_usage.tag_references
  where tag_name = 'privacy_category'
  and tag_value = 'identifier';
alter table my_db.my_schema.hr_data
modify column fname
set masking policy governance.policies.identifier_mask;

use role securityadmin;
select * from snowflake.account_usage.tag_references
    where tag_name = 'privacy_category'
    and tag_value= 'identifier';
revoke select on table my_db.my_schema.hr_data from role analyst;

-- masking policy ------------------------------------------------------------
-- dynamic data masking
create masking policy employee_ssn_mask as (val string) returns string ->
  case
    when current_role() in ('payroll') then val
    else '******'
  end;
create masking policy email_visibility as -- conditional masking, uses a conditional column that maps to visibitlity in the table
(email varchar, visibility string) returns varchar ->
  case
    when current_role() = 'admin' then email
    when visibility = 'public' then email
    else '***masked***'
  end;

-- external tokenization
create masking policy employee_ssn_detokenize as (val string) returns string ->
  case
    when current_role() in ('payroll') then ssn_unprotect(val) -- detokenize val
    else val -- sees tokenized data
  end;

-- table
create or replace table user_info (ssn string masking policy ssn_mask);
alter table if exists user_info modify column ssn_number set masking policy ssn_mask;
create or replace table user_info (email string masking policy email_visibility) using (email, visibility);
alter table if exists user_info modify column email
set masking policy email_visibility using (email, visibility);
-- view
create or replace view user_info_v (ssn masking policy ssn_mask_v) as select * from user_info;
alter view user_info_v modify column ssn_number set masking policy ssn_mask_v;
create or replace view user_info_v (email masking policy email_visibility) using (email, visibility) as select * from user_info;
alter view user_info_v modify column email
set masking policy email_visibility using (email, visibility);

alter table t1 modify column c1 unset masking policy;
alter table t1 modify column c1 set masking policy p2;
alter table t1 modify column c1 set masking policy p2 force; -- auto replace old policy
alter table t1 modify column c1 set masking policy policy1 using (c1, c3, c4) force;

create or replace materialized view user_info_mv -- materialized view
  (ssn_number masking policy ssn_mask)
as select ssn_number from user_info;

alter table t1 modify column value set masking policy p1; -- for an external tbl

select * from snowflake.account_usage.masking_policies -- discover policies
order by policy_name;
select * from snowflake.account_usage.policy_references -- assignments
order by policy_name, ref_column_name;
select *
from table(
  my_db.information_schema.policy_references(
    'my_table',
    'table'
  )
);

-- tag based masking
use role securityadmin;
grant apply masking policy on account to role tag_admin;
alter tag security set masking policy ssn_mask;
alter tag security unset masking policy ssn_mask;
alter tag security set masking policy ssn_mask_2;
alter tag security set masking policy ssn_mask_2 force;

-- row access policy ------------------------------------------------------------
create or replace row access policy rap_it
as (empl_id varchar) returns boolean ->
  'it_admin' = current_role()
;

create table sales ( -- table
  customer   varchar,
  product    varchar,
  spend      decimal(20, 2),
  sale_date  date,
  region     varchar
)
with row access policy sales_policy on (region);
create view sales_v -- view
with row access policy sales_policy on (region)
as select * from sales;
alter table t1 add row access policy rap_t1 on (empl_id);
alter view v1 add row access policy rap_v1 on (empl_id);

create table sales ( -- table
  customer   varchar,
  product    varchar,
  spend      decimal(20, 2),
  sale_date  date,
  region     varchar
);
create table security.salesmanagerregions ( -- mapping table (maps roles to their regions)
  sales_manager varchar, -- role name
  region        varchar  -- matching region for this role
);
use role securityadmin;
create role mapping_role;
grant select on table security.salesmanagerregions to role mapping_role;
use role schema_owner_role;
-- Users with the sales_executive_role role can view all rows.
-- Users with the sales_manager roles can view rows based on the salesmanagerregions mapping table.
create or replace row access policy security.sales_policy
as (sales_region varchar) returns boolean ->
  'sales_executive_role' = current_role()
      or exists (
            select 1 from salesmanagerregions
              where sales_manager = current_role()
                and region = sales_region
          )
;
grant ownership on row access policy security.sales_policy to mapping_role;
grant apply on row access policy security.sales_policy to role sales_analyst_role;
use role securityadmin; -- applu policy on table
alter table sales add row access policy security.sales_policy on (region);
grant select on table sales to role sales_manager_role;
use role sales_manager_role; -- test the policy
select product, sum(spend)
from sales
where year(sale_date) = 2020
group by product;

select * from snowflake.account_usage.row_access_policies
order by policy_name;
select * from snowflake.account_usage.policy_references
order by policy_name, ref_column_name;
select *
from table(
  mydb.information_schema.policy_references(
    policy_name=>'mydb.policies.rap1'
  )
);
select *
from table(
  mydb.information_schema.policy_references(
    ref_entity_name => 'mydb.tables.t1',
    ref_entity_domain => 'table'
  )
);

-- access history ------------------------------------------------------------
select user_name
       , query_id
       , query_start_time
       , direct_objects_accessed
       , base_objects_accessed
from access_history -- in account usage schema
order by 1, 3 desc
;

-- object dependencies ------------------------------------------------------------
select 
  referencing_object_name, 
  referencing_object_domain, 
  referenced_object_name, 
  referenced_object_domain
from snowflake.account_usage.object_dependencies
where 
  referenced_object_name = 'sales_staging_table' 
  and 
  referenced_object_domain = 'external table';

-- parameter management ------------------------------------------------------------
show parameters in account;
alter account set date_output_format = 'dd/mm/yyyy';
show parameters like 'date_output%' in account;
alter account unset date_output_format;

-- user management ------------------------------------------------------------
use role useradmin;
create role policy_admin;
use role securityadmin;
grant usage on database security to role policy_admin;
grant usage on schema security.policies to role policy_admin;
grant create password policy on schema security.policies to role policy_admin;
grant apply password policy on account to role policy_admin;
grant apply password policy on user jsmith to role policy_admin; -- set policy for a user only
use role securityadmin;
grant role policy_admin to user jsmith;
use role policy_admin;
use schema security.policies;
create password policy password_policy_prod_1
    password_min_length = 12
    password_max_length = 24
    password_min_upper_case_chars = 2
    password_min_lower_case_chars = 2
    password_min_numeric_chars = 2
    password_min_special_chars = 2
    password_max_age_days = 999
    password_max_retries = 3
    password_lockout_time_mins = 30
    comment = 'production account password policy';
alter account set password policy security.policies.password_policy_prod_1;
alter user jsmith set password policy security.policies.password_policy_user;
alter account unset password policy;
alter account set password policy security.policies.password_policy_prod_2;
alter user jsmith set must_change_password = true;

create user janesmith password = 'abc123' default_role = myrole must_change_password = true;
grant role myrole to user janesmith;
alter user janesmith set password = 'abc123' must_change_password = true;
alter user janesmith reset password; -- generate a url to share to user

alter user janesmith set disabled = true;
alter user janesmith set disabled = false;
alter user janesmith set mins_to_unlock= 0; -- reset min to unlock after 

alter user janesmith set client_session_keep_alive = true;
alter user janesmith set last_name = 'jones';
alter user janesmith set default_warehouse = mywarehouse default_namespace = mydatabase.myschema default_role = myrole default_secondary_roles = ('all');

desc user janeksmith;
drop user janesmith;

-- behavior change release management ------------------------------------------------------------
select system$behavior_change_bundle_status('2021_02'); -- see whether enabled
select system$enable_behavior_change_bundle('2021_02'); -- enable behavior change
select system$disable_behavior_change_bundle('2021_02');

-- time travel ------------------------------------------------------------
create table mytable(col1 number, col2 date) data_retention_time_in_days=90;
alter table mytable set data_retention_time_in_days=30;

select * from my_table at(timestamp => 'Fri, 01 May 2015 16:20:00 -0700'::timestamp_tz);
select * from my_table at(offset => -60*5); -- in secs
select * from my_table before(statement => '8e5d0ca9-005e-44e6-b858-a8f5b37c5726');

create table restored_table clone my_table
  at(timestamp => 'Sat, 09 May 2015 01:01:00 +0300'::timestamp_tz);
create schema restored_schema clone my_schema at(offset => -3600);
create database restored_db clone my_db
  before(statement => '8e5d0ca9-005e-44e6-b858-a8f5b37c5726');

-- list dropped objects
show tables history like 'load%' in mytestdb.myschema;
show schemas history in mytestdb;
show databases history;

undrop table mytable;
undrop schema myschema;
undrop database mydatabase;

-- performance optimization ------------------------------------------------------------
select query_id, -- show to n longest-running queries
  row_number() over(order by partitions_scanned desc) as query_id_int,
  query_text,
  total_elapsed_time/1000 as query_execution_time_seconds,
  partitions_scanned,
  partitions_total,
from snowflake.account_usage.query_history q
where warehouse_name = 'my_warehouse' and to_date(q.start_time) > dateadd(day,-1,to_date(current_timestamp()))
  and total_elapsed_time > 0 --only get queries that actually used compute
  and error_code is null
  and partitions_scanned is not null
order by total_elapsed_time desc
limit 50;

select to_date(start_time) as date, -- total warehouse load
  warehouse_name,
  sum(avg_running) as sum_running,
  sum(avg_queued_load) as sum_queued
from snowflake.account_usage.warehouse_load_history
where to_date(start_time) >= dateadd(month,-1,current_timestamp())
group by 1,2
having sum(avg_queued_load) >0;

select -- longest running tasks
datediff(seconds, query_start_time,completed_time) as duration_seconds,*
from snowflake.account_usage.task_history
where state = 'succeeded'
  and query_start_time >= dateadd (day, -1, current_timestamp())
order by duration_seconds desc;

-- change a wh to a snowpark-optimized wh
alter warehouse suspend;
alter warehouse my_analytics_wh set warehouse_type='snowpark-optimized';

alter warehouse my_wh set warehouse_size = large;

alter warehouse my_wh set
  enable_query_acceleration = true
  query_acceleration_max_scale_factor = 0;

alter warehouse my_wh set auto_suspend = 600;

alter warehouse my_wh set max_concurrency_level = 4;
































```

