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
### Handler code inline with sql, or on a stage?
Handler code lives in-line with sql:
- Easy to implement/deploy
- Has an upper limit on the source code size
- Change the code via ALTER FUNCTION/PROCEDURE

If the in-line handler source code needs to be compiled (Java/Scala), Snowflake manages compiled output as such:
- If the SQL statement (such as CREATE FUNCTION) uses TARGET_PATH to specify a output location (such as for the JAR file), Snowflake compiles the code once, and keeps the compiled output for future use. This can result in faster execution on repeated calls.
- If the SQL statement does not specify a output location, Snowflake re-compiles the code each time it was called. Snowflake automatically cleans up the file after the SQL statement finishes.

Handler code lives in stage:
- Can use compiled code
- Can use code that is too big to paste into sql statement
- One handler file can contain many handler functions, so many functions/procedures can use them. 
- When code is large/complex, using existing testing/debugging tools are convenient. 

If you delete or rename the handler file, you can no longer call the function/procedure. To update your handler file:
- First, ensure that no calls are being made to the function/procedure that uses the handler.
- Use the PUT command to upload a new handler file. Use the PUT command OVERWRITE=TRUE clause to overwrite the old handler file.

### Security Practices


### Secure UDFs and Procedures


### Pushdown Optimization and Data Visibility


### Designing for Snowflake Constraints


### Data Type Mappings


### Naming Conventions


### Uploading Dependencies





## Packaging Handler Code


## Stored Procedures


## User-Defined Functions


## Logging and Tracing
































