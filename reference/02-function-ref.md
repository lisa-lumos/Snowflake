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



## Context functions


## Conversion functions


## Data Generation functions


## Date & Time functions


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




































