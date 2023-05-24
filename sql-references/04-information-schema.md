# Information schema

## query_history() (in information schema)
Family of table functions. Returns query activity of last 7 days. 
- query_history() returns queries within a specified time range.
- query_history_by_session() returns queries within a specified session and time range.
- query_history_by_user() returns queries submitted by a specified user within a specified time range.
- query_history_by_warehouse() returns queries executed by a specified warehouse within a specified time range.

```sql
select * -- last 100 queries run by the current user, or run by any user on any wh on which the current user has the MONITOR privilege:
from table(information_schema.query_history())
order by start_time;

select * -- run in the current session
from table(information_schema.query_history_by_session())
order by start_time;

select *
from table(
  information_schema.query_history(
    dateadd('hours',-1,current_timestamp()),
    current_timestamp()
  )
)
order by start_time;

select *
from table(
  information_schema.query_history(
    end_time_range_start=>to_timestamp_ltz('2017-12-4 12:00:00.000 -0700'),
    end_time_range_end=>to_timestamp_ltz('2017-12-4 12:30:00.000 -0700')
  )
);
```

## validate_pipe_load() (in information schema)
Validate data files processed by Snowpipe within a time range. Returns details about any errors encountered during an attempted data load into Snowflake tables. 
This function returns pipe activity within the last 14 days.

```sql
validate_pipe_load(
  pipe_name => '<string>'
  , start_time => <constant_expr>
  [, end_time => <constant_expr> ] 
)
```

If Snowpipe encountered no errors while processing data files within the specified time range, the function returns no results.

Example:
```sql
select * from table(
  validate_pipe_load(
    pipe_name=>'my_db.public.mypipe',
    start_time=>dateadd(hour, -1, current_timestamp())
  )
);
```

## copy_history() (in information schema)
A table function inside information schema of any db. Show the data loading history (both bulk loading, and snowpipe) for a table within the past 14 days. 

```sql
copy_history(
  table_name => '<string>'
  , start_time => <constant_expr>
  [, end_time => <constant_expr> ] 
)
```

- For bulk data loads, this function returns results for a role that has MONITOR privilege on your Snowflake account, or a role with USAGE privilege on schema and database and any privilege on table.
- For Snowpipe data loads, you also need usage on db/schema that has the pipe. 

Example:
```sql
select *
from table(
  information_schema.copy_history(
    table_name=>'mytable', 
    start_time=> dateadd(hours, -1, current_timestamp())
  )
);
```

## task_history() (in information schema)
Shows the history of task usage within a specified date range. Returns task activity within the last 7 days or the next scheduled execution within the next 8 days.

It returns results only for the ACCOUNTADMIN role, the task owner, or a role with the global MONITOR EXECUTION privilege. 

To query only those tasks that have already completed or are currently running, include `where query_id is not null` as a filter. The QUERY_ID column in the TASK_HISTORY output is populated only when a task has started running.

```sql
select *
from table(
  information_schema.task_history()
)
order by scheduled_time;

select *
from table(
  information_schema.task_history(
    scheduled_time_range_start=>to_timestamp_ltz('2018-11-9 12:00:00.000 -0700'),
    scheduled_time_range_end=>to_timestamp_ltz('2018-11-9 12:30:00.000 -0700')
  )
);

select *
from table(
  information_schema.task_history(
    scheduled_time_range_start=>dateadd('hour',-1,current_timestamp()),
    result_limit => 10,
    task_name=>'MYTASK'
  )
);

```








































