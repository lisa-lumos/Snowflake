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



## Constraints


## SQL Variables


## Bind Variables


## Transactions


## Table Literals


## Snowflake Database


## Snowflake Information Schema


## Metadata Fields


## Conventions


## Reserved Keywords















