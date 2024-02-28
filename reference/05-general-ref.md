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

















### Data Sharing Usage



### Organization Usage



### SNOWFLAKE Database Roles



### Snowflake Classes




## Snowflake Information Schema


## Metadata Fields


## Conventions


## Reserved Keywords















