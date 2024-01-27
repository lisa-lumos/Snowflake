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






## References


## Ternary Logic


## Collation Support


## SQL Format Models


## Object Identifiers


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















