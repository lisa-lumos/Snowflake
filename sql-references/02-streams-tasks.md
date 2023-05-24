# Streams and Tasks
## create stream
You can create stream on:
- table
- external table
- directory table
- view

To create a stream on table:
```sql
-- table
create [or replace] stream [if not exists]
  <name>
[copy grants]
on table <table_name>
[ at | before (
  timestamp => <timestamp> | 
  offset => <time_difference> | 
  statement => <id> | 
  stream => '<name>' ) 
]
[append_only = true | false ]
[show_initial_rows = true | false] -- 1st consumption of stream contains existing rows of table when stream created. default: false
[comment = '<string_literal>']
```

Creating the initial stream automatically enables change tracking on the table.

CREATE STREAM ... CLONE: The clone inherits the current offset. 

A stream can be queried multiple times to update multiple objects in the same transaction and it will return the same data.

For streams on shared tables, the retention period for a source table is not extended automatically to prevent any streams on the table from becoming stale.

Creating the first stream on a view using the view owner role (i.e. the role with the OWNERSHIP privilege on the view) enables change tracking on the view.

If change tracking has not been enabled on the source view and its underlying tables, then owner of the view and its underlying tables can create the initial stream on the view. Creating the initial stream automatically enables change tracking on the table. f the role was not granted the OWNERSHIP privilege on both the view and its underlying tables, then change tracking must be enabled manually on the applicable objects. 

If the owner of a non-secure view changes it to a secure view (ALTER VIEW ... SET SECURE), any stream on the view automatically enforces secure view constraints.

The retention period for the underlying tables is not extended automatically to prevent any streams on the secure view from becoming stale.

Examples:
```sql
create stream mystream on table mytable;

create stream mystream on table mytable 
before (timestamp => to_timestamp(40*365*86400));

create stream mystream on table mytable 
at (timestamp => to_timestamp_tz('02/02/2019 01:02:03', 'mm/dd/yyyy hh24:mi:ss'));

create stream mystream on table mytable 
at(offset => -60*5);

create stream mystream on table mytable 
at(stream => 'my_existing_stream'); -- similar to clone a stream

create or replace stream mystream -- recreate stream, retain cur offset
on table mytable 
at(stream => 'mystream');

create stream mystream 
on table mytable 
before(statement => '8e5d0ca9-005e-44e6-b858-a8f5b37c5726');

create stream mystream on view myview;

create external table my_ext_table (
  date_part date as to_date(substr(metadata$filename, 1, 10), 'yyyy/mm/dd'),
  ts timestamp as (value:time::timestamp),
  user_id varchar as (value:userid::varchar),
  color varchar as (value:color::varchar)
) 
partition by (date_part)
location=@my_ext_stage
auto_refresh = false
file_format=(type=json);

create stream my_ext_table_stream 
on external table my_ext_table 
insert_only = true;
alter external table my_ext_table refresh;
select * from my_ext_table_stream;

create stream dirtable_mystage_s 
on stage mystage;
alter stage mystage refresh;
select * from dirtable_mystage_s;
```

## create task
```sql
create [ or replace ] task [ if not exists ] 
  <name>
[ warehouse = <string>  |  user_task_managed_initial_warehouse_size = <string> ]
[ schedule = '<num> minute | using cron <expr> <time_zone>' ]
[ allow_overlapping_execution = true | false ] -- allow next run when current run is not completed. default: false
[ <session_parameter> = <value> [ , <session_parameter> = <value> ... ] ]
[ user_task_timeout_ms = <num> ] -- default: 1hr
[ suspend_task_after_num_failures = <num> ] -- default: 0 (do not suspend)
[ error_integration = <integration_name> ] -- send error notifications
[ copy grants ]
[ comment = '<string_literal>' ]
[ after <string> [ , <string> , ... ] ] -- parent tasks
[ when <boolean_expr> ] -- SYSTEM$STREAM_HAS_DATA(), do not require compute
as
  sql statement | call SP | sf scripting
```

Note that currently, Snowsight and the Classic Console do not support creating or modifying tasks to use Snowflake Scripting. Instead, use SnowSQL or another command-line client.

A schedule cannot be specified for child tasks in a DAG.

Validating the conditions of the WHEN expression does not require compute resources. The validation is instead processed in the cloud services layer.

Generally the compute time to validate the condition is insignificant compared to task execution time. As a best practice, align scheduled and actual task runs as closely as possible. Avoid task schedules that are wildly out of synch with actual task runs. For example, if data is inserted into a table with a stream roughly every 24 hours, do not schedule a task that checks for stream data every minute. 

Tasks run with the task owner's privileges. 

After creating a task, you must execute ALTER TASK ... RESUME so the task will run. 

Examples:
```sql
create task t1
  schedule = 'USING CRON 0 9-17 * * SUN America/Los_Angeles'
  user_task_managed_initial_warehouse_size = 'xsmall'
as
select current_timestamp;

create task mytask_hour
  warehouse = mywh
  schedule = 'USING CRON 0 9-17 * * SUN America/Los_Angeles'
as
select current_timestamp;

create task t1
  schedule = '60 minute'
  timestamp_input_format = 'yyyy-mm-dd hh24'
  user_task_managed_initial_warehouse_size = 'xsmall'
as
insert into mytable(ts) values(current_timestamp);

create task mytask_minute
  warehouse = mywh
  schedule = '5 minute'
as
insert into mytable(ts) values(current_timestamp);

create task mytask1
  warehouse = mywh
  schedule = '5 minute'
when
  system$stream_has_data('mystream')
as
  insert into mytable1(id,name) 
  select id, name 
  from mystream 
  where metadata$action = 'insert';

create task task5
  after task2, task3, task4
as
insert into t1(ts) values(current_timestamp);

create or replace procedure my_unload_sp()
  returns string not null
  language javascript
  as
  $$
    var my_sql_command = ""
    var my_sql_command = my_sql_command.concat("copy into @mystage","/",Date.now(),"/"," from mytable overwrite=true;");
    var statement1 = snowflake.createStatement( {sqlText: my_sql_command} );
    var result_set1 = statement1.execute();
  return my_sql_command; // Statement returned for info/debug purposes
  $$;

-- Create a task that calls the stored procedure every hour
create task my_copy_task
  warehouse = mywh
  schedule = '60 minute'
as
  call my_unload_sp();









```



























