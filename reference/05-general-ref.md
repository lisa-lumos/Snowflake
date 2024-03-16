# 5. General Reference
## Parameters
I only listed the interesting ones. 

allow_client_mfa_caching: Specifies whether an MFA token can be saved in the client-side operating system keystore, to promote continuous, secure connectivity, without users needing to respond to an MFA prompt at the start of each connection attempt to Snowflake.

allow_id_token: Specifies whether a connection token can be saved in the client-side operating system keystore, to promote continuous, secure connectivity, without users needing to enter login credentials at the start of each connection attempt to Snowflake.

enable_unredacted_query_syntax_error: Controls whether query text is redacted if a SQL query fails due to a syntax or parsing error. Need to explicitly set it to true, to see the query text for queries that fail, due to a syntax or parsing error.

event_table: Specifies the name of the event table, for logging messages from stored procedures and UDFs, in this account.

max_concurrency_level: Specifies the concurrency level for SQL statements executed by a warehouse. If the number is reached, new queries either queued, or additional cluster start. Note that Snowflake automatically allocates resources for each statement when it is submitted, and the allocated amount is dictated by the individual requirements of the statement. 

max_data_extension_time_in_days: By default, if the DATA_RETENTION_TIME_IN_DAYS setting for a source table is less than 14 days, and a stream has not been consumed, Snowflake temporarily extends this period to the stream's offset, up to a maximum of 14 days, regardless of the Snowflake Edition for your account. 

min_data_retention_time_in_days: Minimum number of days, for which Snowflake retains historical data for performing Time Travel actions on an object. 

search_path: Specifies the path to search, to resolve unqualified object names in queries.

statement_timeout_in_seconds: Amount of time, in seconds, after which a running SQL statement is canceled by the system. Default is 2 days. 

suspend_task_after_num_failures: Default is 10. 

transaction_default_isolation_level: Default is READ COMMITTED. 

use_cached_result: Can be used for evaluate query performance improvements. 

user_task_timeout_ms: default is 1 hr. 

## References
A reference can be used to authorize access on objects, to a SP, application, or class instance, that does not have access to those objects by default.

A reference is a string that can be used as an identifier. The identifier resolves to the object being referenced.

A reference encapsulates the following:
1. The object name.
2. The role used to create the object reference, and any active secondary roles if applicable.
3. The privileges on the object, when the reference is created.

Examples of reference use cases:
- An owner's rights SP need to insert data in a table owned by the caller.
- An application performs data analytics, and requires read access to a table.
- An instance of the SNOWFLAKE.ML.ANOMALY_DETECTION class requires read access to a view, for training the anomaly detection ML model.

A reference identifies an object by name - If an object is renamed after a reference is created, the reference is invalid. However, if a new object with the same name is created, the reference might be valid.

The lifespan of a reference can be specified at creation time, it can either be transient or persistent.

A reference can become invalid for any of these reasons:
- The object it references is renamed.
- The role that created the reference is dropped.
- The role that created the reference no longer has privileges on the object.

## Ternary Logic
When used in expressions (e.g. SELECT list), UNKNOWN results are returned as NULL values.

When used as a predicate (e.g. WHERE clause), UNKNOWN results evaluate to FALSE.

## Collation Support
Collation allows you to specify rules for comparing strings, which can be used to compare/sort data according to a particular language, or other user-specified rules.

Text strings in Snowflake are stored using the UTF-8 character set and, by default, strings are compared according to their Unicode codes.

Collation control is granular. You can explicitly specify the collation to use for account/db/schema/table/table-col/statement.

Using collation can affect the performance of various database operations:
1. Operations involving comparisons might be slower.
2. Micro-partition pruning might be less efficient.
3. Using collation in a WHERE predicate that is different from the collation specified for the column,might result in reduced pruning efficiency, or completely eliminating pruning.

Collation Limitations: 
- Collation is Supported only for resulting Strings up to 8MB
- No support with collation with UDFs
- Collation Not Supported for Strings in variant/array/object

## SQL Format Models
In Snowflake, SQL format models (literals containing format strings) are used to specify how numeric values are converted to text strings, and vice versa. They can be specified as arguments in the to_char/to_varchar and to_decimal/to_number/to_numeric conversion functions. 

## Object Identifiers
It just means object name. 

### Requirements
Unquoted object ids:
- Start with a letter (A-Z, a-z) or an underscore (_).
- Contain only letters, underscores, decimal digits (0-9), and dollar signs ($).

Quoted ids:
- Can contain/start with anything. 

### Literals and Variables as Identifiers
In Snowflake SQL statements, in addition to referring to objects by name, you can also use a string literal, session variable, bind variable, or Snowflake Scripting variable to refer to an object. 

For example, you can use a session variable, that is set to the name of a table in the FROM clause of a SELECT statement.

e.g.: `identifier('my_db.my_schema.my_table')`. 

### Object Name Resolution
When the database name is omitted, the object name is resolved with the current database in the context. 

The name of the current database is returned by the current_database() function.

When the schema name is omitted using `<database_name>..<object_name>`, the object name is resolved with the public default schema. Note that using it in this way is not recommended.  

In DDL and DML statements, unqualified objects are augmented with the current database and schema. The current schema is maintained similarly to the current database.

When a session is initiated, the current schema is initialized based on the connection's settings. When the current database is changed, the current schema defaults to the value of an internal property (normally set to PUBLIC).

In queries, unqualified object names are resolved through a search path, which usually contains the current schema, but can also contain other schemas.

The search path is stored in the session-level parameter SEARCH_PATH (basically the search path of schemas). You can modify it. The default value of the search path is `$current, $public`.

To see the schemas that will be searched for unqualified objects in queries, use the current_schemas() function. 

The SEARCH_PATH is not used inside views or UDFs. All unqualified objects in a view or UDF definition will be resolved in the view's or UDF's schema only.

## Constraints
Snowflake supports defining and maintaining constraints, but does not enforce them, except for the NOT NULL constraint. 

Snowflake supports the following constraint types, from the ANSI SQL standard:
- UNIQUE
- PRIMARY KEY
- FOREIGN KEY
- NOT NULL

For Snowflake Time Travel, when previous versions of a table are copied, the current version of the constraints on the table are used, because Snowflake does not store previous versions of constraints in table metadata.

Constraints can be defined on a single column, or on multiple columns in the same table. For multi-column constraints (i.e. compound primary keys or unique keys), the columns are ordered, and each column has a corresponding key sequence.

## SQL Variables
Examples:
- `set my_var = 10;`
- `select $my_var;`
- `create table identifier($my_table_name) (i integer);`
- `show variables; `
- `unset my_var;`

## Bind Variables
To execute a sql statement that should contain user input info, you can concatenate strings and form a statement, or, you can use bind variables, such as `insert into my_table (c1, c2) values (?, ?);`. bind variables can prevent SQL injection attacks. 

## Transactions
A transaction is a sequence of SQL statements that are committed or rolled back as a unit. A transaction can include both reads and writes.

Transactions follow these rules:
- Transactions are never nested. For example, you cannot create an outer transaction that would roll back an inner transaction that was committed, or create an outer transaction that would commit an inner transaction that had been rolled back.
- A transaction is associated with a single session. 

Explicit transactions should contain only DML statements and query statements. 

DDL statements implicitly commit active transactions. Each DDL statement executes as a separate transaction.

If a DDL statement is executed while a transaction is active, the DDL statement:
1. First, implicitly commits the active transaction.
2. Then, executes the DDL statement as a separate transaction.

To avoid writing confusing code, you should avoid mixing implicit and explicit starts and ends in the same transaction. 

Snowflake recommends that multi-threaded client programs do at least one of the following:
- Use a separate connection for each thread.
- Execute the threads synchronously rather than asynchronously, to control the order in which steps are performed.

A transaction can be inside a stored procedure, or a stored procedure can be inside a transaction; however, a transaction cannot be partly inside and partly outside a stored procedure, or started in one stored procedure and finished in a different stored procedure.

A stored procedure that contains a transaction can be called from within another transaction. An outer ROLLBACK/COMMIT does not undo an inner COMMIT/ROLLBACK.

READ COMMITTED is the only isolation level currently supported for tables. When a statement is executed inside a multi-statement transaction:
- A statement sees only data that was committed before the statement began. Two successive statements in the same transaction can see different data if another transaction is committed between the execution of the first and the second statements.
- A statement does see the changes made by previous statements executed within the same transaction, even though those changes are not yet committed.

Most INSERT and COPY statements write only new partitions. Those statements often can run in parallel with other INSERT and COPY operations, and sometimes can run in parallel with an UPDATE, DELETE, or MERGE statement.

Best practices:
- In general, one transaction should contain only related statements.
- Having less statements in a transaction can improve performance in some cases. In Snowflake, as in most databases, managing transactions consumes resources. For example, inserting 10 rows in one transaction is generally faster and cheaper than inserting one row each in 10 separate transactions. 
- Overly large number of statements in a transaction can reduce parallelism or increase deadlocks.
- Snowflake recommends keeping AUTOCOMMIT enabled, and using explicit transactions as much as possible.

Every Snowflake transaction is assigned a unique transaction id. 

## Table Literals
Table literals are used to pass the name of a table or a placeholder value (instead of a table name) to a query. 

```sql
set my_var = 'my_table';
select * from table($my_var);
```

## Snowflake Database
### Account Usage
access_history: which root/parent query's which child query executed by which user accessed/modified which direct/base objects, used which policies. 

aggregate_access_history: similar data to the access_history View, aggregated for repeated queries in one-minute intervals.

aggregate_query_history: contains similar data to the query_history view, but is aggregated in one-minute intervals for repeated SQL statements. Who executed which query under which db/sc context using which role, which wh, and run-time info typically included in query profile, such as bytes spilled to local/remote storage, etc. 

aggregation_policies: An aggregation policy is a schema-level object, that controls what type of query can access a table/view. When an aggregation policy is applied to a table, queries against that table must aggregate data into groups of a minimum size, in order to return results, thereby preventing a query from returning information from an individual record. 

alert_history: For the past year, which alert in which location included which query, with which condition, when was it executed to check for the condition. 

automatic_clustering_history: in a specified time span, clustering a table took how many credits; how many bytes/rows was reclustered. 

class_instances

classes

columns: which table has which cols, its data type, whether nullable, default val, number precision, comment, if this col is deleted and when. 

complete_task_graphs: for each completed DAG runs, what was its state, root task, who ran it (schedule/manual/retry), first errored task and message, scheduled time, start time, completed time. 

copy_history: for the past year, manual and snowpipe copy into tbl, which file path, stage, num of rows, error message, status, loaded into which table, loaded by which pipe. 

data_classification_latest: which table, classified result, when it happened. 

data_transfer_history: within the past year, when, how many bytes transferred from which source cloud to which different target cloud, which operation caused the transfer. 

database_replication_usage_history: when, which db was replicated, used how many credits, transferred how many bytes. 

database_storage_usage_history: for the past year, for each day, for each db, its avg bytes, avg failsafe bytes, avg hybrid table bites. If the db was dropped, when it happened. 

databases: who owns which database, is it transient, when was it created/altered, was it deleted, if so, when; retention time, whether it is from a share.  

element_types: if a table has cols of structured array type, what is this col's detailed elem type.

event_usage_history: credits and bytes related for data ingestion. 

external_access_history: from which source to target cloud each external access happened, sent/received how many bytes, what is their host name, IP, what is the query that initialized it. 

fields: if a table has cols of structured object type, what is this col's detailed key and val types.

file_formats: for each file format object, who is its owner, what is in this file format, like skip_header, delimiter, etc; when was it created, and last altered. 

functions: for each UDF, what is its return type, its body, its language, when was created/updated, its owner, is it external, etc. 

grants_to_roles: for each role, show each privileges granted to it, when was each privilege granted, who granted it. 

grants_to_users: for each user, what roles they have, when was it granted, who granted it to this user. 

hybrid_tables: for each hybrid table, who is its owner, how many rows/bytes it has, its retention time, when it was created/altered/deleted. 

hybrid_tables_usage_history: credits used for hybrid tables. 

index_columns: for each index, which db/sc/tbl/col it belongs to, who is the owner, its status, when it was created/deleted. 

indexes: same with above. 

load_history: manual copy into history. Loaded into which table, file name loaded, number of rows, etc. 

lock_wait_history: history of transactions that wait on locks. Which object was locked by which query at which time, which query/transaction requested the lock at when. 

login_history: which user attempted to login at when, using which client, was it successful, what error they had. 

masking_policies: for each masking policy, who is its owner, when was it created/altered/deleted, its body, signature and return type. 

materialized_view_refresh_history: Each time a materialized view is refreshed, when it started/ended, for which table, used how many credits. 

metering_daily_history: type of service that consumed credits, on which day it happened, how many compute credits used, how many cloud services credits used, actually credits billed. 

metering_history: type/name of service that consumed credits, its start/end time, how many compute credits used, how many cloud services credits used, total credits used. 

network_policies: for each network policy, its name/owner/comment, when it was created/altered/deleted. 

network_rule_references

network_rules: for each network rule, its owner/comment, when it was created/altered/deleted. 

object_dependencies: for each object, such as a view, what are its base tables. 

password_policies: for each password policy, its owner/comment/content-specs, its created/altered/deleted date. 

pipe_usage_history: which pipe at start/end time loaded how many files, used how many credits, inserted how many bytes. 

pipes: for each pipe, its sqs notification channel name, does it auto ingest, it definition, owner, when it was created/altered/deleted. 

policy_references: for each policy, which objects it refers to, its tag

procedures: for each procedure, its return type, its language, body, owner, when it was created/altered/deleted. 

projection_policies: for each projection policy, its owner, return type, body, etc. 

query_acceleration_eligible: for each query, its query text, start/end time, wh name and size, num of secs eligible for the query acceleration service, max useful scale factor. 

query_acceleration_history: which warehouse used how many credits between start/end time. 

query_history: for each query, its id/body/context/type/session/user/role/wh/wh_size/wh_type/query_tag/status/error/start_and_end_time/bytes_scanned/rows/partitions_scanned/bytes_spilled/query_acceleration_info/transaction_info. 

referential_constraints: for each constraint, its match option, rules, created/altered/deleted time. 

replication_group_refresh_history: for each replication group, its start/end time, bytes info, object count, snapshot time, error.  

replication_group_usage_history: for each replication group, its start/end time, credits used, bytes transferred. 

replication_usage_history: (deprecated)

resource_monitors: (under READER_ACCOUNT_USAGE schema) for each resource monitor for reader account, its created time, quota, used credits, owner, wh, notify/suspend/etc policy. 

roles: for each role, its owner, when it was created/deleted. 

row_access_policies: for each row access policy, its owner, signature, return type, body. When it was created/altered/deleted. 

schemata: for each schema, its retention time, is_transient, crated/altered/deleted date. 

search_optimization_history: for which table, at which span, its search optimization consumed how many credits. 

sequences: for each sequence, its owner/data-type/min-max-val/increment, etc. When it was created/altered/deleted. 

serverless_task_history: for each serverless task, its start/end time, number of credits it consumed. 

services

session_policies

sessions

snowpark_container_services_history

snowpipe_streaming_client_history

snowpipe_streaming_file_migration_history

stage_storage_usage_history

stages

storage_usage

table_constraints

table_storage_metrics

tables

tag_references

tags

task_history

task_versions

users

views

warehouse_events_history

warehouse_load_history

warehouse_metering_history












### Data Sharing Usage



### Organization Usage



### SNOWFLAKE Database Roles



### Snowflake Classes




## Snowflake Information Schema


## Metadata Fields


## Conventions


## Reserved Keywords















