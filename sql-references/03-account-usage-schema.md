# 3. Account usage schema
The Account Usage (AU) views and the corresponding views/functions in Snowflake Information Schema (IS) have same structures and naming conventions, but with some key differences:
- AU includes dropped objects, IS does not
- AU has latency (mostly 2hrs), IS do not
- AU has historical data of past 1yr, IS has 7 days to 6mo. 

By default, the SNOWFLAKE database is available only to the ACCOUNTADMIN role. To enable other roles to access it, ACCOUNTADMIN role must grant them IMPORTED PRIVILEGES on the snowflake db. 

## copy_history view (in account usage schema)
Shows Snowflake data loading history for the last 1 yr. Contains both bulk loading and snowpipe. 2hrs latency. 

```sql
select file_name, error_count, status, last_load_time 
from snowflake.account_usage.copy_history
order by last_load_time desc
limit 10;
```

## load_history view (in account usage schema)
Shows Snowflake data loading history for the last 1 yr. Contains both bulk loading only. Onw row for each file loaded. 1.5hr latency. 

The view only includes COPY INTO commands that executed to completion, with or without errors. No record is added if the transaction is rolled back, eg, if the ON_ERROR = ABORT_STATEMENT was used, and an error happened.

```sql
select file_name, last_load_time 
from snowflake.account_usage.load_history
order by last_load_time desc
limit 10;
```

## pipe_usage_history view (in account usage schema)
Shows the history of data loaded into Snowflake tables using Snowpipe within the last 1 yr. Displays the history of data loaded, and credits billed for your entire Snowflake account. 3 hrs of latency. 

## query_history view (in account usage schema)
Query Snowflake query history by various dimensions (time range, session, user, warehouse, etc.) within the last 1 yr. 45 min of latency. 













































