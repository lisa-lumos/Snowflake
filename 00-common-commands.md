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
list @%mytable; -- list files in table stage
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

-- warehouse ------------------------------------------------------------
alter warehouse mywh set SCALING_POLICY = 'ECONOMY';

-- snowpark-optimized wh
CREATE OR REPLACE WAREHOUSE snowpark_opt_wh WITH
  WAREHOUSE_SIZE = 'MEDIUM'
  WAREHOUSE_TYPE = 'SNOWPARK-OPTIMIZED';

-- query optimization service ------------------------------------------------------------
select parse_json(system$estimate_query_acceleration('8cd54bf0-1651-5b1c-ac9c-6a9582ebd20f')); -- for a specific query

-- for queries that benefit the most from the service
SELECT query_id, eligible_query_acceleration_time
FROM snowflake.account_usage.query_acceleration_eligible
ORDER BY eligible_query_acceleration_time DESC;

-- ... for a specific wh
SELECT query_id, eligible_query_acceleration_time
FROM snowflake.account_usage.query_acceleration_eligible
WHERE warehouse_name = 'mywh'
ORDER BY eligible_query_acceleration_time DESC;

-- for warehouses that benefit the most from the service
SELECT warehouse_name, SUM(eligible_query_acceleration_time) AS total_eligible_time
FROM snowflake.account_usage.query_acceleration_eligible
GROUP BY warehouse_name
ORDER BY total_eligible_time DESC;

-- enable the query acceleration service with a maximum scale factor of 0
alter warehouse my_wh set
  ENABLE_QUERY_ACCELERATION = true
  QUERY_ACCELERATION_MAX_SCALE_FACTOR = 0;

SELECT * FROM SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY
WHERE QUERY_ACCELERATION_PARTITIONS_SCANNED > 0
AND start_time >= DATEADD(hour, -24, CURRENT_TIMESTAMP())
ORDER BY QUERY_ACCELERATION_BYTES_SCANNED DESC;


-- clustering ------------------------------------------------------------
-- cluster by base columns
CREATE OR REPLACE TABLE t1 (c1 DATE, c2 STRING, c3 NUMBER) CLUSTER BY (c1, c2);

-- cluster by expressions
CREATE OR REPLACE TABLE t2 (c1 timestamp, c2 STRING, c3 NUMBER) CLUSTER BY (TO_DATE(C1), substring(c2, 0, 10));

-- cluster by paths in variant columns
CREATE OR REPLACE TABLE T3 (t timestamp, v variant) cluster by (v:"Data":id::number);

-- cluster by base columns
ALTER TABLE t1 CLUSTER BY (c1, c3);

ALTER TABLE t1 DROP CLUSTERING KEY;

ALTER TABLE t1 SUSPEND RECLUSTER;
ALTER TABLE t1 RESUME RECLUSTER;

SELECT TO_DATE(start_time) AS date,
  database_name,
  schema_name,
  table_name,
  SUM(credits_used) AS credits_used
FROM snowflake.account_usage.automatic_clustering_history
WHERE start_time >= DATEADD(month,-1,CURRENT_TIMESTAMP())
GROUP BY 1,2,3,4
ORDER BY 5 DESC;

-- tables, views ------------------------------------------------------------
CREATE TEMPORARY TABLE mytemptable (id NUMBER, creation_date DATE);
CREATE TRANSIENT TABLE mytranstable (id NUMBER, creation_date DATE);

ALTER TABLE t1 ADD SEARCH OPTIMIZATION ON EQUALITY(c1, c2, c3);
alter table test_table add search optimization;
DESCRIBE SEARCH OPTIMIZATION ON t1;
ALTER TABLE t1 DROP SEARCH OPTIMIZATION ON SUBSTRING(c2);
alter table test_table drop search optimization;

CREATE VIEW doctor_view AS
    SELECT patient_ID, patient_name, diagnosis, treatment FROM hospital_table;

CREATE OR REPLACE SECURE VIEW widgets_view AS
    SELECT w.*
        FROM widgets AS w
        WHERE w.id IN (SELECT widget_id
                           FROM widget_access_rules AS a
                           WHERE upper(role_name) = CURRENT_ROLE()
                      )
    ;

-- Example of a materialized view with a range filter
create materialized view v1 as
    select * from table1 where column_1 between 100 and 400;

ALTER MATERIALIZED VIEW vulnerable_pipes CLUSTER BY (installation_year);

SELECT * FROM TABLE(INFORMATION_SCHEMA.MATERIALIZED_VIEW_REFRESH_HISTORY());

CREATE TRANSIENT TABLE my_new_table LIKE my_old_table COPY GRANTS;
INSERT INTO my_new_table SELECT * FROM my_old_table;

CREATE TRANSIENT TABLE my_transient_table AS SELECT * FROM mytable;

CREATE TRANSIENT TABLE foo CLONE bar COPY GRANTS;






















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















```


































