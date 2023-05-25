# 8. Stored Procedures
You might want to use a stored procedure to automate a task that requires multiple SQL statements and is performed frequently. 

For example, imagine that you want to clean up a database by deleting data older than a specified date. You can write multiple DELETE statements, each of which deletes data from a specific table. You can put all of those statements in a single stored procedure and pass a parameter that specifies the cut-off date. Then you can simply call the procedure to clean up the database. As your database changes, you can update the procedure to clean up additional tables; if there are multiple users who use the cleanup command, they can call one procedure, rather than remember every table name and clean up each table individually.

```sql
create or replace procedure myproc(from_table string, to_table string, count int)
returns string
language python
runtime_version = '3.8'
packages = ('snowflake-snowpark-python')
handler = 'run'
as
$$
def run(session, from_table, to_table, count):
  session.table(from_table).limit(count).write.save_as_table(to_table)
  return "SUCCESS"
$$;

call myproc('table_a', 'table_b', 5);
```

You can write a handler in any of the following languages:
- Java (using the Snowpark API)
- JavaScript
- Python (using the Snowpark API)
- Scala (using the Snowpark API)
- Snowflake Scripting (SQL)






























