# 3. Functions and Procedures
## Function or Procedure?
With a UDF, 
- Typically calculate and return a value
- Can be called as part of a sql statement
- Must return a value 
- Supported handler languages: Java, JS, Python, Scala, SQL

With a SP, 
- Generally perform administrative operations, by executing SQL statements
- Can perform typical db operations, such as typical queries and DML/DDL
- Optionally return a value
- Supported handler languages: Java, JS, Python, Scala, Snowflake Scripting

Note that every CREATE PROCEDURE statement must include a RETURNS clause that specifies a return type, even if the procedure does not explicitly return anything,in which case, it implicitly returns NULL.

The value returned by a SP, unlike those returned by a function, cannot be used directly in SQL.

Indirect ways to use the return value of a SP:
- Call a SP from inside of another SP
- RESULT_SCAN() after calling the SP
- Store the result set in a temp/perm table in the SP
- Return a VARIANT, if data set is small enough

UDF and SP are called differently: 
```sql
call mySP(argc1); -- call a SP
select myfunc(col1) from table1; -- use a udf
```

A single executable statement can call only one SP. In contrast, a single SQL statement can call multiple functions.

Unlike SPs, UDFs do not have access to an API that can perform database operations.

## Guidelines


## Packaging Handler Code


## Stored Procedures


## User-Defined Functions


## Logging and Tracing
































