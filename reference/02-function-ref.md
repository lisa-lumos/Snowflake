# 2. Function Reference
## Aggregate functions
Common ones: avg(), count(), count_if(), median(), sum(), 

any_value(): Non deterministic. Optimizes the group by statements when you do not need to agg on that col.

corr(): Correlation coefficient for not null pairs in a group. 

count(): Returns either the num of non-NULL records for the specified columns, or the total num of records.

count_if(): Returns num of rows that match a condition

listagg(): Returns a string that is delimiter-separated concatenated input strings, skips nulls. 

min()/max(): Ignores nulls, unless they are all nulls. 

min_by()/max_by(): find the row containing th min/max val of a col, and return the val of another col of that row. 

mode(): Returns the most frequent value. 

booland_agg()/boolor_agg()/boolxor_agg(): Returns the logical AND/OR/XOR val of all non-null boolean records in a group. Note that bool xor returns true when exactly one record in the group is true - different from bitwise xor. 

hash_agg(): Returns an aggregated hash value over the set of input rows and columns (if include multiple cols). Can be used for detecting changes to a set of values, without comparing the individual old and new values. For rows of vals in a col, it produces the same hash_agg value, regardless of the order of the rows. It does not ignore NULL inputs. 

array_agg(): Turns input rows into one array. Can be distinct vals, can choose to order them. 

object_agg(): Turns pairs of a col of (unique strings) keys and a col of (variant) vals into one object. 

array_union_agg(): Returns an array, that contains all distinct elements from rows of an array type column; may have duplicates, num of duplicates for that val is same with max num of duplicates among the input arrays for that val. Uses "uses multiset semantics", it is rarely used. Ignores nulls. 

array_unique_agg(): Similar to array_union_agg(), but with no duplicates in the output. 

approx_count_distinct()/HLL(): Returns distinct count using HyperLogLog approximation. 


Others: percentile_count(), percentile_disc(), ...

## Bitwise Expression functions
skipped. 

## Conditional Expression functions
[not] between ... and ...: equivalent to >= ... and <= ...

booland()/boolnot()/boolor()/boolxor(): Input is numeric datatypes. If the number is 0, is is considered False, otherwise as True. Then perform the specified (in the function name)logical operation for two numbers.  

case:
```sql
case
  when my_condition1 then ...
  when my_condition2 then ...
  ...
  else ...
end

case my_expression
  when my_val1 then ...
  when my_val2 then ...
  ...
  else ...
end
```

coalesce(a, b, c, ...): Returns the first non-NULL expression. 

decode(my_expression_or_column, key1, val1, key2, val2, ...., default_val): Maps the calculated result from key to values. 

equal_null(a, b): Similar to the "=" expression, but it is NULL-safe, meaning, it thinks null is equal to null. 

greatest(a, b, c, ...): Compared with max() with compares across rows, greatest() compares across cols for each row in its input, and it supports all data types. eturns null if any val in the list is null. 

iff(my_condition, a, b): Simplified if-then-else expression. 

ifnull(a, b): if a is null, return b, otherwise return the original a. 

value [not] in (a, b, ...) / value [not] in (my_subquery): Here, null is not equal to null. 

... is [not] distinct from ...: Same with equal_null(). This function is null-safe. "is distinct from" means "not equal to". 

is [not] null

is_null_value(my_variant_value): checks if a variant is json null. 

least(a, b, c, ...): opposite to greatest(). Returns null if any val in the list is null. 

nullif(a, b): if a = b, return null, otherwise returns a. Always return null is a is null. 

nullifzero(a): if a is 0, return null; otherwise return a. 

nvl(a, b): Null-replacing function. If a is null, return b, otherwise returns a. e.g.: nvl(my_col, 0) means map all null vals to 0s; zeroifnull(my_col) does the same.  

nvl2(a, b, c): Null-replacing function. if a is null, return c; otherwise return b. nvl(my_col, my_col+100, 0) means map all null vals to 0s, and rest of the vals to val+100. 

regr_valx(a, b)/regr_valy(a, b): Null-preserving function. 

zeroifnull(a): Null-replacing function. If a is null, return 0. 

## Context functions
current_client(): Returns the version of the driver/client. 

current_date(), current_timestamp(), current_time(), localtime(), localtimestamp(). 

current_ip_address(): current ip of the client connected to sf. 

current_region(): current account region

current_version(): current sf version major_version.minor_version.patch_version

sysdate(): current timestamp in UTC. 

all_user_names(): Array of all user names in the current account.

current_account(): account locator.

current_account_name(): account name. 

current_organization_name()

current_role(): the name of the primary role in use for the current session, when the primary role is an account-level role; NULL if the role in use for the current session is a database role.

current_available_roles(): a string that contains all account-level roles available to the current user. 

current_secondary_roles(): all secondary roles being used. 

current_session(): unique sys id for the cur sf session. 

current_statement(): sql text of cur statement that is executing.

current_transaction(): transaction id of the cur transaction. 

current_user()

getvariable('MY_VARIABLE'): the val of my_variable. 

last_query_id([my_offset]): positive offset val starts from 1st query in the session, negative offset val starts from now and go backwards in time. 

last_transaction(): prv transaction id that has completed. 

current_database(), current_schema(), current_warehouse(). 

current_role_type(): role, or database_role.

current_schemas(): active search path schemas. 

invoker_role(): account-level role name, or null. For masking policy. 

invoker_share(): share name that accesses the table/view. For making policy. 

is_database_role_in_session('my_database_role'): check whether this db role is active in the session.  

is_granted_to_invoker_role('my_role_name'): check if this role name is granted to the invoker role. Can be used in masking policy. 

is_role_in_session('my_role_name'): check whether this account level role is active in the session. 

policy_context(...): Simulates the query result for table or view columns protected by a masking policy, a table or view protected by a row access policy, or both. 

snowflake.alert.get_condition_query_uuid(): check the results of the statement for the condition.

## Conversion functions
cast(... as ...), ::, 

try_cast(... as ...): only works for casting string to other types. 

try_to_...()

convert_timezone(source_tz, target_tz, source_ts_ntz)

convert_timezone(target_tz, source_ts): implicitly uses session time zone in source timezone, if source_ts doesn't already have a timezone inside. 

to_array(): returns a single elem array of the value. 

to_object(), to_variant(). 

## Data Generation functions
random([seed]): Returns a pseudo-random 64-bit integer. The seed only works for the same row - expect numbers to change across rows. 











## Date & Time functions
snowflake.alert.last_successful_scheduled_time(): the timestamp representing the scheduled time for the most recent successful evaluation of the alert condition. 

snowflake.alert.scheduled_time(): the timestamp of the scheduled time of the current alert.

## Encryption functions


## File functions


## Geospatial functions


## Hash functions


## Metadata functions


## Numeric functions


## Regular Expressions functions


## Semi-Structured Data functions


## String & Binary functions


## System functions


## Table functions


## Window functions




































