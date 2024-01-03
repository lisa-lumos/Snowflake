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

randstr(str_len, seed): Returns an alphanumberic string of specified length. 

uuid_string()

normal(...), uniform(...), zipf(...), 

seq1(), seq2(), seq4(), seq8(). The trailing number is integer width. Generates mono increasing ints. 

## Date & Time functions
date_from_parts(year_int, month_int, day_int): can take integers within these data part ranges, or out of the range, or be negative. E.g., the n-th day of the year. 

time_from_parts(hr_int, min_int, sec_int [nano_int]). 

timestamp_from_parts(...). 

date_part(...)/extract(... from ...): return a number. 

dayname()/monthname(): 3-letter day-of-week/month string, such as 'Fri', 'Jan'. 

last_day(): commonly used to return the last day of the mo, etc. 

next_day(), previous_day(): can get date of next Friday, etc. 

date_add(), datediff()

date_trunc(): returns the timestamp/date. 

snowflake.alert.last_successful_scheduled_time(): the timestamp representing the scheduled time for the most recent successful evaluation of the alert condition. 

snowflake.alert.scheduled_time(): the timestamp of the scheduled time of the current alert.

## Encryption functions
encrypt(): Encrypts a VARCHAR or BINARY value using a VARCHAR passphrase. Returns a binary value. 

decrypt(): Decrypts a BINARY value using a VARCHAR passphrase.

encrypt_raw()/decrypt_raw(): (uses a binary key)

## File functions
get_stage_location(@my_stage_name): Retrieves the URL for an external/internal named stage, using the stage name.

get_relative/absolute_path()

get_presigned_url(): Generates a pre-signed URL to a staged file.

build_scoped_file_url(): Generates a scoped Snowflake file URL to a staged file. Expires 24 hrs after first access. 

build_stage_file_url(): Generates a Snowflake file URL to a staged file. It doesn't expire. 

## Geospatial functions
skipped. 

## Hash functions
hash(...)/hash_agg(...): returns a signed integer. SF proprietary. CANNOT be used to create unique keys. Can take multiple cols as input. 

## Metadata functions
generate_column_description(): inter_schema() takes a set of staged files, that contains semi-structured data, its output feeds into generate_column_description(), who then returns a string that contains a list of cols. Which can then be manually referred to, when creating a table, etc, for that dataset. 

get_ddl(): Returns a ddl of an object, which can be used to recreate it. For UDFs and stored procedures, the output might be slightly different from the original DDL. 

## Numeric functions
div0(): like /, but returns 0, when divisor is 0. 

div0null(): like /, but returns 0, when divisor is 0, or null. 

abs/ceil/floor/mod/round/sign/truncate

cbrt/exp/factorial/power/sqrt/square

ln/log

acos/acosh/asin/asinh/atan/atan2/atanh/cos/cosh/cot/degrees/pi/radians/sin/sinh/tan/tanh

## Regular Expressions functions
my_col regexp/rlike 'my_pattern': Returns boolean, of whether pattern matches col provided. It supports more complex patterns than the "like" function. 

regexp_count(expression, 'my_pattern', ...): Returns the num of times a pattern occurs in a string. 

regexp_extract_all/regexp_substr_all(expression, 'my_pattern', ...): Returns an ARRAY, that contains all substrings that matches the pattern. 

regexp_instr(expression, 'my_pattern', ...): Returns the position of the specified occurrence of the pattern, from input. If no match is found, returns 0.

regexp_like/rlike(expression, 'my_pattern', ...): Returns true, if the expression matches the pattern.

regexp_replace(expression, 'my_pattern', 'my_replacement', ...): From input, replaces the pattern with replacement, and return it. 

regexp_substr(expression, 'my_pattern', ...): From input, returns the substr that matches the pattern. 

## Semi-Structured Data functions
check_json/xml(string_or_variant_expr): Checks the validity of a JSON/XML document, if valid, returns null, otherwise, error message. 

json_extract_path_text(col_name, 'path_name'): get the value for the corresponding path. 

parse_json/try_parse_json(my_str): converts the input string to a variant type.  

parse_xml(my_str): converts the input string to an object type.

strip_null_value(my_path): get the value at the path, and, if the val is JSON null, convert it to SQL null. 

### array creation
- array_construct(elem1, elem2, ...)
- array_construct_compact(elem1, elem2, null, ...)
- array_generate_range(start_int, end_int, step_size): create a new array

### array querying
- array_size(my_array)
- array_max/array_min(my_array): return the max/min elem
- array_contains(elem_to_look_for, my_array)
- array_position(my_elem, my_array): return the idx

### array manipulation
- array_append(my_array, my_new_elem): add a new elem to the array. 
- array_prepend(my_array, my_elem)
- array_insert(my_array, my_idx, my_elem): insert an elem to the array. 
- array_cat(array1, array2): return an array with concatenation. 
- array_compact(my_array): rmv nulls from the input array. 
- array_distinct(my_array): de-dupes the elems in the input array. 
- array_flatten(my_array): flattens an array of arrays, into a single array. (first-level flattening, not recursive)
- array_except(source_array, array_of_elems_to_exclude): rmvs specific elems from the source array. 
- array_intersection(array_1, array2)
- array_remove(my_array, my_elem)
- array_remove_at(my_array, my_idx)
- array_slice(my_array, start_idx, end_idx)
- array_sort(my_array, ...)
- array_to_string(my_array)
- arrays_overlap(array1, array2): returns true if they have a common elem
- arrays_to_object(my_key_array, my_val_array)

### object
- object_construct/object_construct_keep_null(key1, val1, key2, val2, ...)
- object_delete(my_object, key1, key2, ...)
- object_insert(my_object, my_key, my_val)
- object_pick(my_object, key1, key2, ...): return an object, containing the subset of key-val pairs specified, from the input obj

### map
map_cat
map_contains_key
map_delete/map_insert
map_keys
map_pick
map_size

### extraction
flatten(): parses a variant. Default parses one level. Can recursively parses all nested elems out. Can optionally recursively parse only object, or only array out. Can optionally specify a particular path to parse for each row. 

get()/get_path(): extract val from variant. 

get_ignore_case()

object_keys(): returns an array, containing the list of first-level-keys of the input object. 

xmlget(): extract an elem from xml. 

### Conversion/casting
as_...(my_variant): convert variant vals to other data types

### type predicates
is_...(my_variant): check if the variant is a certain datatype

typeof(my_variant): returns the type of the variant val

## String & Binary functions
ASCII(str)

concat(str1, str2, ...): same with ||

insert(source_str, idx1, idx2, str_to_be_inserted): delete chars between specified idxs, and insert a new str there. 

length/len()

lpad/rpad(): pads a string with characters from another string, so it reaches total length specified. 

ltrim/rtrim/trim(): rmvs leading/trailing/surrounding spaces

parse_url(url_string): returns a variant, key val pairs of host, parameters, path, port, query, scheme. 

repeat(string_to_repeat, n_times_to_repeat)

reverse(str)

space(length): create a str with spaces

split(str, delimiter): return an array of the separated substrs

split_part(str, delimiter, idx_to_return)

split_to_table(str, delimiter): instead of returning an array, it returns a table of rows that has each part

strtok/strtok_to_array/strtok_split_to_table(): similar to split, but takes multiple delimiters. 

translate(input_str, keys, vals): maps each char to another char. 

initcap(): format the first letter of each word to uppercase.

lower()/upper()

charindex(source_str, str_to_search_for): return the idx of the 1st occurrence.

contains(source_str, str_to_search_for)

editdistance(str1, str2)

startswith/endswith()

left/right(str, length_to_cutout)

my_str like all/any (pattern1, pattern2, ...): check if matches all/any the patterns. 

replace(source_str, pattern_to_find, replacement_str): removes all occurrences of the pattern, and optionally replace them. 

substr(source_str, start_idx, length)

compress(input_str, compress_method): returns a binary

decompress_binary/decompress_string()

base64_encode/hex_encode, ...

md5(str): can be used as a checksum function to detect data corruption

sha1/sha2(), ...

collate(my_str, collation_spec_str): can be used to sort/compare strs using other languages sort order

collation(my_str): returns the collation_spec_str of this str. 

## System functions
cleanup_database_role_grants(db_role_name, share_name): Revokes privileges on dropped objects from the share, and grants the database role to the share.

extract_semantic_categories(table_view_name): Returns a JSON, containing a set of categories (semantic and privacy) for each supported column in the specified table/view. 

snowflake.snowpark.show_python_packages_dependencies(python_runtime_version, packages_list)

explain_json(...): returns a table, input is the return result of explain_plan_json(...) function. 

get_query_operator_stats(query_id): Returns statistics, about individual query operators within a completed query. Use this to understand the structure of a query, and identify query operators (e.g. the join operator) that cause performance problems.

### the functions that starts with `system$`
abort_session/transaction(session_id/transaction_id)

cancel_all_queries(session_id)

cancel_query(query_id)

task_dependents_enable(root_task_name): Recursively resumes all dependent tasks tied to a specified root task. Need to be task owner to do this. 

user_task_cancel_ongoing_executions(task_name): Aborts a run of the specified task, that is currently executing.

wait(amount_of_time, time_unit): Waits for the specified amount of time before proceeding. 

allowlist(): Returns hostnames and port numbers to add to your firewall's allowed list, so that you can access Snowflake from behind your firewall. Because by default, your firewall might block access to Snowflake. 

clustering_depth(table_name, ...): Computes the average depth of the table, according to the specified columns, or the clustering key defined for the table.

clustering_information(table_name, ...): Returns clustering information, including average clustering depth, based on one or more columns in the table.

current_user_task_name(): Returns the task name that is currently executing.

estimate_search_optimization_costs(table_name, ...)

external_table_pipe_status(external_table_name): Retrieves a JSON, of the current refresh status for the internal pipe object, that is associated with an external table.

get_compute_pool_status(pool_name): for Snowpark Container Services.

get_directory_table_status(stage_name): for data replication, stage monitoring. 

get_job_logs(job_uuid, container_name, ...): for Snowpark Container Services.

get_job_status(), get_service_logs(), get_service_status(), registry_list_images(), ...

get_iceberg_table_information(iceberg_table_name): Returns the location of the root metadata file, and status of the latest snapshot for an Iceberg table.

get_predecessor_return_value(predecessor_task_name)

get_snowflake_platform_info(): Returns the IDs of the virtual network in which your Snowflake account is located.

get_tag(tag_name, obj_name[.col_name], obj_domain): return the corresponding tag value. 

get_tag_allowed_values(tag_name)

get_tag_on_current_column/table(tag_name): (this function can only be used in a masking policy condition, to dynamically evaluate the tag string value set on a column).

get_task_graph_config(): Returns the value of the configuration string for the cur task.

last_change_commit_time(obj_name): Returns a token, that can be used to detect whether a database table/view changed, between two calls to the function. The value can be used in applications, such as BI tools, to determine whether the underlying table data has changed. The value returned by the function is typically an "approximation" of the time that the database object was last changed, expressed as the UTC timestamp in nanoseconds, since the beginning of the epoch. 

log(level_str, message_str): Logs a message at the specified severity level, into the event table.

pipe_status(pipe_name): Retrieves a JSON representation of the current status of a pipe.

query_reference(select_statement): Returns a query reference, that you can pass to a stored procedure. Within the stored procedure, when you execute the query, the query is executed using the role that created this query reference.

reference(...): Returns a reference to an object (a table/view,/function). When you execute SQL actions on a reference to an object, the actions are performed using the role who created this reference.

set_return_value(): Explicitly sets the return value for a task. Another task that identifies this task as its predecessor can retrieve this return value.

set_span_attributes(...): Set the attribute name and value for a span, when using trace events from a handler, written in Snowflake Scripting.

show_budgets_in_account(): Returns the budgets in the account.

show_oauth_client_secrets(integration_name): Returns the client secrets in a string. The client ID and a client secret must be included in the authorization header to the OAuth token endpoint.

stream_get_table_timestamp(stream_name): Returns the timestamp in nanoseconds, of current offset for the specified stream.

stream_has_data(stream_name)

task_runtime_info(type_of_info_to_return): can return name of current task, current root task name, original scheduled timestamp, etc

typeof(expression): Returns a string, representing the SQL data type of an expression.

estimate_query_acceleration(query_id): returns a JSON object, that specifies if the query is eligible to benefit from the query acceleration service.

explain_plan_json(sql_query_text): return a string, of json-compatible format. 

explain_json_to_text(explain_output_in_json_format)

## Table functions
infer_schema(...): Automatically detects the file metadata schema in staged data files, and retrieves the column definitions. Supports Apache Parquet, Apache Avro, ORC, JSON, and CSV files. Can then be used to automatically create a table using this template. 

validate(table_name, job_id): Validates the files loaded in a previous execution of the COPY INTO table command, and returns all the errors encountered during the load. 








## Window functions




































