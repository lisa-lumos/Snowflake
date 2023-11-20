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

Others: percentile_count(), percentile_disc(), ...



## Bitwise Expression functions


## Conditional Expression functions


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




































