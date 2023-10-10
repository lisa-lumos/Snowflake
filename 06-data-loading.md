# 6. Data loading
## Overview
A stage: the location of data files in cloud storage. A named external stage lives in a schema. 

Snowflake supports 3 types of internal stages: 
- User: belong to a single user, cannot be dropped, do not support file format options because it may need to handle any file format. User need insert privilege on target table. `@~`
- Table: belong to a table, cannot be dropped. Do not support transform while loading. Only table owner can use it. `@%table_name`
- Named:  a db object lives inside a schema. Most flexible. Privileges can be granted/revoked, ownership can be transferred. `@stage_name`

`You can do bulk loading, and continuous loading, from any sort of stages to tables.` `You can directly load from S3 bucket without using an external stage.`  

Bulk loading uses user-provided virtual warehouses, specified in the COPY statement. 

Data transformations supported in COPY INTO command:
- Column reordering & omission & casts
- Truncating text strings that exceed the target column length

Continuous loading using snowpipe:
- used to load small volumes of data incrementally
- loads data within minutes
- uses serverless compute resources, so automatically resized & scaled
- can be used in data pipelines with streams and tasks, for complex transformations

The Snowflake Connector for Kafka can connect to an Apache Kafka server, load data from topics into an internal stage, and load data files into tables (manual or pipe). Example: a Kafka topic has 3 partitions, and each partition loads files into their respective folder in a internal stage, then the REST API for snowpipe is called, and the 3 pipes would load data into one table concurrently, from their respective folder. 

For semi-structured data (Apache Parquet, Apache Avro, and ORC files), snowflake supports auto detection of schema, which retrieves the column definitions (names, data types, and ordering of columns in the files). Use function `INFER_SCHEMA` and `GENERATE_COLUMN_DESCRIPTION`. Create tables with the column definitions derived from a set of staged files using the `CREATE TABLE ... USING TEMPLATE`. 

Instead of loading data into sf table, you can query then directly in the stage also. Useful if you only want to query a subset of data. 

You can create external stages/tables on software and devices, on premises or in a private cloud, that is highly compliant with Amazon S3 API (Amazon S3-compatible Storage). 

Note:
- Metadata for pipe and manual bulk load lives in different places, if pipe already loaded the files, and you run copy into command manually on these files, you will get duplicated records in the table. 

## Feature Summary
For files to load to SF, the default encoding character set is UTF-8. 

gzip is the default compression format for uncompressed files to be loaded to SF stage. For already compressed files, `gzip, bzip2, deflate, raw_deflate` are supported, and can be auto-detected. Auto-detection is not yet supported for `Brotli and Zstandard`, you need to specify it during staging and loading. 

The files sits in the SF stage are encrypted using `128-bit keys by default`, if the files came unencrypted at the beginning. If they are already encrypted, you will need the encryption key. 

## Considerations
### Preparing Your Data Files
The number of data files that are processed in parallel is determined by the amount of compute resources in a warehouse.

File sizing (for both bulk and pipe loading): 
- Data files >= 100-250 MB compressed. 
- Do not load large files (>= 100GB).
- Can use split command in Linux to partition large files

Semi-structured:
- Supported semi-structured data formats: `JSON, Avro, ORC, Parquet, XML (preview)`
- The VARIANT data type imposes a `16 MB size limit` on individual rows. 
- Use the `STRIP_OUTER_ARRAY` file format option for the COPY INTO command to load the records into separate table rows. 
- Extract semi-structured data elements containing "null" values into relational columns before loading them; or, set the file format option `STRIP_NULL_VALUES` to TRUE when loading it.

Snowpipe:
- Snowpipe is designed to load new data within a minute after a file notification is sent. 
- If it takes >= 1min to accumulate MBs of data in your source app, recommend to create 1 file/min, using Amazon Kinesis Firehose etc to batch data files.

csv files:
- UTF-8 encoding is the default, but can specify other types
- field vals that contain delimiter should be single/double-quoted
- if data contains single/double quotes, they need to be escaped
- carriage returns should be single/double-quoted
- num of cols in each row should be same

### Planning a Data Load
Dedicate separate warehouses for loading and querying (to optimize performance for each). 

Unless bulk loading >= 100s or 1000s of files concurrently, a smaller warehouse (Small, Medium, Large) is generally sufficient.

### Staging Data
You can include paths in a stage definition; you can also use put command to put the file into a certain folder (path) in the stage. 

Partition the data into logical paths such as geographical location with date. 

### Loading Data
COPY INTO command supports you to choose files by:
- path/prefix
- specifying a list of file names
- use regex pattern matching

Snowflake maintains data loading metadata for each table, this metadata lives inside the table, including:
- Name & size & ETag of the files loaded into this table
- Num of rows parsed in the file
- Last load timestamp for each file
- Loading error info for the file

This metadata is used to prevent data duplication during loading for the target table. `It expires after 64 days for bulk loading`. `For files with unknown status, they are skipped by default.` 

When loading staged data, narrow the path to the most granular level, such as using pattern options or use common prefix of the files. 

### Managing Regular Data Loads
After staged files are loaded, consider removing them (PURGE copy option or the REMOVE command). 

## Preparing to Load Data
If you regularly load similarly-formatted data, recommend named file formats. 

File format / copy options priority: COPY INTO statement > Stage definition > Table definition. 

File format options are `not cumulative`, while copy options `are cumulative`. 

COMPRESSION file format option describes how your data files are already compressed in the stage. Set it by 2 ways:
- As a file format option in COPY INTO.
- As a file format option inside a named file format or a stage, which can be referred in COPY INTO.

Copy options determine the error handling behavior, maximum data size etc. 

## Loading Using the Web Interface (Limited)
The Web Interface (classic console) for loading data (combines put and copy command into one operation) is only intended for loading small numbers of files of limited size (up to 50 MB). 

## Bulk loading from Local File System
PUT -> COPY INTO. 

Bulk loads are always performed in a single transaction.
### Choosing an Internal Stage
checked. 
### Staging Files
checked. 
### Copying Data from a Local Filesystem
Before loading data, you can validate that the data in the uploaded files will load correctly by using the `VALIDATION_MODE` parameter in COPY INTO command. 

The ON_ERROR copy option for the COPY INTO command indicates what action to perform if errors are encountered in a file during loading.

## Bulk loading from Amazon S3
### Allowing the Virtual Private Cloud IDs
Your sf account has a unique VPC id, you can create an Amazon S3 policy allowing this id, then provide an IAM role for sf to access the specific S3 bucket. 
### Configuring Secure Access
You can use `storage integrations` to allow Snowflake to read/write data from/to an Amazon S3 bucket referenced in a stage. This avoids the need for passing explicit cloud provider credentials like secret keys or access tokens. It stores an AWS IAM user ID. An admin in your org grants the IAM user permissions to the AWS account.

Different stage objs can refer different buckets and paths and use the same storage integration for authentication. SF provisions a single IAM user for your entire sf account. All S3 storage integrations use that IAM user.
### AWS Data File Encryption
Snowflake supports either `client-side encryption` (Requires a MASTER_KEY of 128/256-bit and base64 encoded) or `server-side encryption`. 
### Creating an S3 Stage
skipped
### Copying Data from an S3 Stage
skipped
## Bulk loading from Google Cloud Storage
skipped
## Bulk loading from Microsoft Azure
skipped
## Bulk loading Troubleshooting
Data load failures:
- view the copy history for the table
- use validation_mode in COPY INTO statement: `VALIDATION_MODE=RETURN_ALL_ERRORS`

If you use CURRENT_TIMESTAMP() inside a COPY INTO command, it is evaluated at compile time, rather than when the record is actually inserted into the table. The latter is reflected in the LOAD_TIME column in the COPY_History view/function. 


## Snowpipe 
2 mechanisms for detecting the staged files:
- using cloud messaging (auto-ingest)
- calling Snowpipe REST endpoints
## Snowpipe Overview
Cross-cloud support is currently only available to accounts hosted on AWS.
## Snowpipe Auto Ingest
Event notifications received while a pipe is paused are retained for 14 days. 

auto_ingest parameter should be set to true. 
### Automating for Amazon S3
skipped
### Automating for Google Cloud Storage
skipped
### Automating for Microsoft Azure
skipped
## Snowpipe REST Endpoints
Calling the public REST endpoints to load data. Calls to the public Snowpipe REST endpoints use key-based authentication, not the typical username/password authentication, because the ingestion service does not maintain client sessions.

Your client app / aws Lambda function calls a public REST endpoint with a list of data filenames and a referenced pipe name. If new data files matching the list exists, they are queued for loading. 

auto_ingest param should be set to false. 

Use case: if files are constantly being loaded into an internal stage (using Python or Java), and we want snowpipe to pick up these files and load in to tables in snowflake. The problem is, internal stage do not have event notification feature, to still be able to use snowpipe, we need to call its REST API from the code. 

### Preparing to Load Data
The Snowpipe service requires the Java/Python(3.6+) SDK. 

The Snowpipe REST endpoints require key pair authentication with JSON Web Token (JWT). 
### Load Data Using REST API
skipped
### Load Data Using AWS Lambda
skipped
### Snowpipe REST API
skipped
## Snowpipe Error Notifications
Snowpipe can push error notifications to a cloud messaging service when it encounters errors while loading data. 

Snowpipe error notifications only work when the ON_ERROR copy option is set to SKIP_FILE (the default). Use the NOTIFICATION_HISTORY table function to query the history of notifications sent through Snowpipe. 

### Enabling for Amazon SNS
skipped
### Enabling for Microsoft Azure Event Grid
skipped
### Enabling for Google Pub/Sub
skipped
## Snowpipe Troubleshooting
1. check the pipe status
2. view the copy history of the table
3. validate the data files

Snowflake uses file loading metadata (which lives in the pipes) to prevent reloading the same files in one table. Snowpipe prevents loading files with the same name even if they were later modified. Staged files with the same name as files that were already loaded are ignored, even if they have been modified, e.g. if new rows were added or errors in the file were corrected. If the files are modified and staged again after 14 days, Snowpipe loads the data again, potentially resulting in duplicate records in the target table.

## Snowpipe Managing
`Pipe does not support the PURGE copy option.` To remove staged files that you no longer need, recommend periodically executing the REMOVE command to delete the files. Alternatively, configure any lifecycle management features provided by your cloud storage service provider.

ALTER PIPE ... REFRESH statement loads the files that are staged within the past 7 days, and checks the load history for both the target table and the pipe to ensure the same files are not loaded twice.


## Snowpipe Costs
Query either of the following:
- PIPE_USAGE_HISTORY table function (in Information Schema).
- PIPE_USAGE_HISTORY View

`Managed service, with additional cost of 0.06 credit/1k files being loaded.` 

## Snowpipe Streaming - Overview (preview)
Calling the Streaming API from your `Java application code` to `insert` rows of data to Snowflake tables directly, without staged files in the middle. This architecture results in lower load latencies and costs, which makes it great for real-time data streams. Do not require pipe objects. 

The API ingests rows via one or more channels. Each channel points to only one table, but a table can have multiple channels pointing to it. 

## Snowpipe Streaming - Configuring
API calls rely on key pair authentication with JSON Web Token (JWT).

## Snowpipe Streaming - Recommendations
Recommend calling the API with fewer clients that write more data per second.

## Snowpipe Streaming - Costs
Similar to other serverless features. 

## Snowpipe Streaming - Kafka Connector with Snowpipe Streaming
Optionally replace Snowpipe with Snowpipe Streaming in your data loading chain from Kafka. Low latency and low cost. 

## Amazon S3-compatible Storage (Preview)
The industry-standard Amazon S3 REST API enables programmatic access to storage buckets and objects. To access storage outside of the public cloud, you can create external stages in Snowflake that store the S3-compliant API endpoint, bucket name/path/credentials. Then you can load and unload data from and to the storage locations.

## Querying Data in Staged Files
sf supports using SQL to query data files in an internal stage or named external (S3 etc) stage. Useful for inspecting the file content before loading / after unloading.

## Querying Metadata for Staged Files
Snowflake automatically generates metadata for files in stages. This metadata is "stored" in virtual columns that can be:
- Queried using a standard SELECT statement.
- Loaded into a table, along with the regular data columns, using COPY INTO (can only insert new rows, cannot modify existing row).

The metadata columns are: 
- METADATA$FILENAME: current row's file path
- METADATA$FILE_ROW_NUMBER: row num for each row
- METADATA$FILE_CONTENT_KEY: checksum of file for cur row
- METADATA$FILE_LAST_MODIFIED: cur row's file's last modified timestamp in NTZ
- METADATA$START_SCAN_TIME: cur row's start to scan time in LTZ

## Transforming Data During Load
All file formats supported by COPY INTO can use transformations, except JSON has to be "Newline delimited JSON" standard format. 

`The VALIDATION_MODE param does not support transforming data during a load.`

## Continuous data pipelines Overview
It automates many manual steps in transforming and optimizing continuous data loads. Usually, the raw data files are first loaded temporarily into a staging table, then transformed using SQL statements, then inserted into the destination reporting tables. The efficient way is to transform only new/modified data. 

A stream object records the delta (inserts etc, DML) of CDC info for a table - allows querying/consuming the changes at the row level, between two transactional points of time.

A task object defines a recurring schedule for executing a SQL statement (and SP calls) - can be chained together for successive execution of complex periodic processing. Can use streams by calling SYSTEM$STREAM_HAS_DATA. Users can define a tree-like structure of tasks to process/move data.

## Continuous data pipelines - Dynamic table
Dynamic tables are the building blocks of declarative data transformation pipelines. Instead of defining data transformation steps as a series of tasks, and then monitoring dependencies and scheduling, you can simply define the end state of the transformation using dynamic tables, and leave the complex pipeline management to Snowflake.

A dynamic table materializes the results of a query that you specify. 

Dynamic tables can be used as the source of a stream. When used together, a stream based on a dynamic table works like any other stream.

The automated process computes the changes that were made to the base objects, and merges those changes into the dynamic table. 

You can set up a dynamic table to query other dynamic tables.



### Understanding the Dynamic Table Refresh
Dynamic table content is based on the results of a specific query. When the underlying data on which the dynamic table is based on changes, the table is updated to reflect those changes. These updates are referred to as a "refresh".

When a dynamic table is created, it is initially populated using data from the underlying tables (via a full refresh, could take time to finish). The refresh ensures snapshot isolation. 

To track progress of the initial full refresh, query the refresh history using `information_schema.dynamic_table_refresh_history()`, where `refresh_trigger = INITIAL`; or user the `show dynamic tables` command, and check `refresh_mode` column.

During the initialization of a new version of an existing dynamic table, the existing version remains accessible, until the new version is available, ensuring that failing to create a new version doesn't affect the current instance.

The dynamic table refresh process operates in 1 of 2 ways:
1. Incremental refresh. It analyzes the dynamic table's query, and calculates changes since the last refresh. It then "merges" these changes into the table. Types of queries that support incremental refreshes:
   - WITH (for CTEs)
   - Expressions in SELECT (including deterministic system functions, and immutable UDFs)
   - FROM (source tables, views, and other dynamic tables)
   - OVER (supports all window functions)
   - WHERE (including deterministic system functions, and immutable UDFs)
   - JOIN (inner/outer/cross joins)
   - UNION ALL
   - GROUP BY
2. Full refresh. When the automated process can't perform an incremental refresh, it conducts a full refresh. This involves executing the query for the dynamic table, and completely replacing the previous materialized results.

Types of queries that do NOT support incremental refreshes:
- LATERAL joins
- Subqueries outside of FROM clauses (e.g., WHERE EXISTS)
- VOLATILE UDFs

The "constructs used in the query" determine whether an incremental refresh can be used. After you create a dynamic table, you can monitor the table, to determine whether incremental or full refreshes are used to update that table.

Target lag is specified in 1 of 2 ways:
1. Measure of freshness (target_lag). Defines the maximum amount of time that the dynamic table's content should lag behind updates to the base tables. Specified with the TARGET_LAG parameter.
2. DOWNSTREAM. Specifies that the dynamic table should be refreshed, when its downstream dynamic tables need to refresh. 

How data is refreshed when dynamic tables depend on other dynamic tables. Target lag is not a guarantee - it is a target that Snowflake attempts to meet. In order to keep data consistent in cases when one dynamic table depends on another, the process refreshes all dynamic tables in an account at compatible times. This means that at any given time, when you query a set of dynamic tables that depend on each other, you are querying the same "snapshot" of the data across these tables. Note that the target lag of a dynamic table cannot be shorter than the target lag of the dynamic tables it depends on. If refreshes take too long, the scheduler may skip refreshes to try to stay up to date. However, snapshot isolation is preserved.

### Dynamic tables compared to streams & tasks, and materialized views
| Streams and Tasks | Dynamic Tables |
|----------|-------------|
| imperative | declarative |
| you define schedule | sf determines schedule to meet lag requirements |
| can use non-deterministic code, SPs, other tasks. SP can use UDFs and external functions | select statement, cannot use external functions |
| can use streams to incrementally refresh target | automated refresh |

| Materialized Views | Dynamic Tables |
|----------|-------------|
| To improve query performance | To build multi-level data pipelines |
| Can be automatically used in queries against the base table | not auto |
| Can only use a single base table | Can use a complex query |
| Data in there is always current | Data there have a lag |

### Understanding the Costs of Dynamic Tables
Incur cost in 3 ways:
- Storage
- Cloud services compute. For periodic checks to determine whether changes happened, if so, triggering refreshes. Only bills if this cost daily is > 10% of daily warehouse cost. 
- Virtual warehouse compute. For initialization and actually doing refreshes. 

Snowflake recommends testing dynamic tables using dedicated warehouses to understand related costs.

### Dynamic Table States
A dynamic table will be suspended, if there are 5 continuous refresh errors. 

To view the scheduling state of a dynamic table, call the DYNAMIC_TABLE_GRAPH_HISTORY() table function, and examine the SCHEDULING_STATE column.

### Dynamic Table and Streams
Streams can be created on dynamic tables, much like streams on traditional tables. However, if the table experiences a full refresh, a stream event/row will be generated for every row in the dynamic table.

### About working with Dynamic Tables
#### Create Dynamic Tables
Dynamic tables differ from traditional tables, certain query constructs and functions are not allowed. Search optimization, clustering, and the query acceleration service are not supported. 

Some non-deterministic functions are supported, but only for full refreshes. 

#### About managing Dynamic Tables
```sql
show dynamic tables like 'product_%' in schema mydb.myschema;
desc dynamic table product;
```

Dynamic tables can be shared. 
```sql
grant select on all dynamic tables in schema mydb.public to share share1;
grant select on dynamic table mydb.public to share share1;
```

Dynamic tables can be created on shared data. 
```sql
-- create a dynamic table to ingest shared data
create or replace dynamic table my_dynamic_table
target_lag = '1 day'
warehouse = my_wh
as
  select * from shared_db.public.mydb
;
```

Creating a dynamic table on top of a shared dynamic table is currently not supported.

Creating a dynamic table on top of a shared secure view that references an upstream dynamic table is currently not supported.

#### Manage Dynamic Tables Refresh


#### Monitor Dynamic Tables






## Continuous data pipelines - Streams
A stream itself does not contain any table data - it only stores source object's offset and returns CDC records, using the source's versioning history. The stream rely on both the offset and the change tracking metadata stored in the table (several hidden cols created when stream for the table was first created, or when ALTER TABLE ... CHANGE_TRACKING = TRUE is executed).

For streams on views, change tracking must be enabled explicitly for the view & underlying tables to add the hidden columns to these tables.

A stream provides the minimal set of changes from its current offset to the current version of the table. Multiple queries can independently consume (select from) the same change data from a stream without changing the offset. A stream advances the offset only when it is used in a DML transaction. 

When you query a stream within a transaction, it follows repeatable read isolation transaction isolation level. When DML transaction completes successfully, the stream offset advances. To ensure multiple statements access the same change records in the stream (lock the stream), surround them with an explicit transaction statement (BEGIN .. COMMIT). 

Stream differs from the read committed mode supported for tables, in which statements see any changes made by previous statements executed within the same transaction, even though those changes are not yet committed.

Stream has 3 additional cols:
- `METADATA$ACTION`: the DML operation INSERT/DELETE.
- `METADATA$ISUPDATE`: in the stream, updates are represented as a pair of DELETE and INSERT records with  METADATA$ISUPDATE = TRUE. Because streams record the overall differences between two offsets, so if a row is added and then updated, the delta change is a new row. The METADATA$ISUPDATE row records a FALSE value.
- `METADATA$ROW_ID`: the unique and immutable ID for the row.

3 stream types:
- Standard. tracks all DML changes (insert/update/delete). Support tables, directory tables, views. Do not work with geospatial data - recommend creating append-only streams on them.
- Append-only. tracks row inserts only. Support standard tables, directory tables, views. 
- Insert-only. tracks row inserts only. Support `external tables` only. Overwritten/appended files are handled as new files - the stream does not record the diff of the old and new file versions.

A stream (unconsumed records inside it) becomes stale (gone) when its offset is outside of the effective data retention period for its source table, or the underlying tables for a source view. To track new change records for the table, recreate the stream. If the data retention period for a table is < 14d, and a stream has not been consumed, Snowflake extends to 14d by default (`MAX_DATA_EXTENSION_TIME_IN_DAYS` param), aka effective data retention time for this stream, regardless of the Snowflake edition. This could incur storage costs. 

To view the current staleness status of a stream, execute DESCRIBE STREAM / SHOW STREAMS. 

Recreating an object drops its history, which makes any stream relies on it go stale. 

If a table and its stream are cloned, the cloned stream offset is when it was cloned. 

Renaming a source obj does not break its stream (references are used instead of obj names).

Recommend to create a separate stream for each DML consumer (task, script, ... that consumes the same change data) for an obj.

`Streams works for local views and shared secure views, but NOT for materialized views.` Underlying tables must be native tables with change tracking enabled (using ALTER TABLE ... CHANGE_TRACKING = TRUE), and the view may only have projections, filters, inner/cross joins, union all, system scalar functions (group by, qualify, limit, etc are not supported). 

The CHANGES clause in SELECT enables querying change tracking metadata between two points in time, without having a stream. Using a stream is sufficient for most use cases, CHANGES clause is useful in infrequent cases when you need to manage the offset for arbitrary periods of time. 

The main cost associated with a stream is the processing time used by a virtual warehouse to query the stream. These charges appear on your bill as familiar Snowflake credits.

### Managing streams
Only the object owner role of a view or underlying tables can enable change tracking.

`Creating a stream on the view you own enables change tracking on the view, and all its underlying tables you own. If you don't own a underlying table, the owner must explicitly enable change tracking for it.` 

During the enabling operation of CDC for a table, the table is locked for writes until the command finishes running. 

### Examples of streams
Read, skip. 

## Continuous data pipelines - Tasks
A task can execute following types of SQL code:
- Single SQL statement
- Call to a stored procedure
- Procedural logic using Snowflake Scripting

Can be combined with streams for continuous ELT workflows.

Can also be used independently to do periodic work, such as for periodic reports by inserting/merging rows into a report table.

Tasks require compute resources to execute SQL code. Individual tasks can have either of these:
- `Serverless/managed tasks using snowflake-managed resources`. Snowflake determines the size of the compute resources for a run based on the previous runs of this task. Max size is 2xl, can be used in parallel by diff tasks. Omit the WAREHOUSE parameter when CREATE TASK (the role needs EXECUTE MANAGED TASK privilege). Cannot work for UDFs with Java/Python inside, or SPs in Scala. 
- `User-managed tasks using user-specified virtual warehouse`. Specify the WAREHOUSE parameter when CREATE TASK. 

Query the TASK_HISTORY Account Usage view to find the average run time of a task (avg difference between scheduled and completed time). The warehouse size you choose should be large enough to run multiple child tasks that are triggered simultaneously by predecessor tasks. Choose an appropriate warehouse size for task to complete its workload within the defined schedule interval.

serverless or user-managed task, which one to choose:
- Choose serverless tasks if: you have too few tasks to run concurrently; tasks runtime < 1min; tasks with stable runtime; need to fully adhere to the schedule (has no queue period)
- Choose user-managed tasks if: you can fully utilize one warehouse; have unpredictable runtime/loads; no need to fully adhere to the schedule

A standalone task or the root task in a DAG generally runs on a schedule. You can define the schedule when creating a task (using CREATE TASK) or later (using ALTER TASK). If a task is still running when it is already the next scheduled execution time, then that scheduled run is skipped (by default).

The cron expression in a task definition supports specifying a time zone (daylight saving time is included in some timezones, need to be careful with this). Easiest way to avoid DST confusion is to use a timezone that do not use DST, such as UTC. 

A `Directed Acyclic Graph (DAG)` is a series of tasks with a single root task, organized by their dependencies. DAGs flow in a single direction. Each task (not the root) can have multiple predecessor tasks (dependencies); and each task can have multiple subsequent/child tasks that depend on it. `A task runs only after all of its predecessor tasks have successfully completed`.

The root task have a schedule that starts a run of the DAG. 

Specify the predecessor tasks when creating a new task (using CREATE TASK ... AFTER) or later (using ALTER TASK ... ADD AFTER).

A DAG is limited to a max of 1000 tasks. A single task can have a max of 100 predecessor tasks and 100 child tasks.

You can include a conclude task in a DAG calling an external function to trigger a remote messaging service to send a notification that all prev tasks have successfully completed.

`All tasks in a DAG must have the same task owner and be stored in the same db and schema.`

Transferring ownership of a task severs the dependency between this task and any predecessor and child tasks, unless all tasks ownership is transferred all at once (by dropping the old owner role so the tasks are transferred to the dropper, or by GRANT OWNERSHIP on all tasks in a schema to a diff role). 

By default, Snowflake allow only one instance of a DAG to run at a time. The next run of a root task is scheduled only after the whole DAG have finished running. This is controlled by `ALLOW_OVERLAPPING_EXECUTION` parameter on the root task. Overlapping runs are fine when overlapping runs of a DAG do not produce incorrect/duplicate data. 

TASK_DEPENDENTS table function shows all dependents of a task in a DAG. 

When a DAG runs with some suspended child tasks, the run ignores those sleeping tasks. A child task with many predecessors runs as long as all its resumed predecessors successfully completes.

To recursively resume all tasks in a DAG, query the `system$task_dependents_enable` function rather than resuming each task individually (using ALTER TASK ... RESUME).

To modify/recreate any task in a DAG, the root task must first be suspended (using ALTER TASK ... SUSPEND).

If the SP definition called by a task changes during a DAG run, the new code can be used in current run.

To retrieve the history of task versions, query TASK_VERSIONS Account Usage view. 

A task supports all session parameters. You can set them for a task. A task does not support account or user params.

You can configure a task or DAG to auto suspend after n consecutive failed/timeout runs. Set the `SUSPEND_TASK_AFTER_FAILURES` param on a [standalone task] or [the DAG root task to be effective if one task in DAG meets this condition]. 

Root task can also be manually triggered (good for testing), its successful run triggers the rest of the DAG. 

Any 3rd-party services that can authenticate into your Snowflake account and authorize SQL actions can execute the EXECUTE TASK command to run tasks.

Ways to view task history:
- TASK_HISTORY table function in information schema
- CURRENT_TASK_GRAPHS table function in information schema
- COMPLETE_TASK_GRAPHS table function in information schema
- COMPLETE_TASK_GRAPHS View in Account Usage

`To recover the management costs of Snowflake-provided compute resources, 1.5x multiplier to resource consumption is applied.` Note that the serverless compute model could still reduce compute costs over user-managed warehouses; in some cases significantly.

To retrieve credit usage for a task, use the SERVERLESS_TASK_HISTORY table function in info schema, or SERVERLESS_TASK_HISTORY view in Account Usage.

Tasks run with the privileges of the task owner, whether [scheduled] or [run manually by EXECUTE TASK by another role with OPERATE privilege].

In addition to the task owner, a role that has the OPERATE privilege on the task can suspend/resume the task.

Recommend to create a custom role (e.g. taskadmin) and grant the EXECUTE TASK privilege to this role. Then this role can be granted to any task owner role so they can alter their own tasks. To no longer allow the task owner role to execute the task, just revoke taskadmin role from the task owner role. 

Make sure the SQL statement that you use in a task works as expected before you create the task. Tasks are intended to automate SQL statements or SPs that have already been tested thoroughly.

### Enabling Error Notifications for tasks
A task can push error notifications to a cloud messaging service, using:
- Amazon SNS
- Microsoft Azure Event Grid
- Google Pub/Sub 

To enable a task to send error notifications, you must associate the task with a `notification integration`: CREATE TASK ... ERROR_INTEGRATION = ... as ..., or ALTER TASK ... SET ...

### Troubleshooting tasks
1. verify that the task indeed did not run using TASK_HISTORY table function
2. verify that the task is RESUMED
3. verify that the task owner has sufficient permissions
4. verify the conditions for the task to run, like streams actually has data. Historical data for a stream can be queried using an AT | BEFORE clause.

`There is a 1hr default limit on one run of a task.` Increase the wh size accordingly if it exceeds this limit, or, increase this timeout limit param for this task using `ALTER TASK ... SET USER_TASK_TIMEOUT_MS = ....` 

## Continuous data pipelines - Examples
read & skipped.

