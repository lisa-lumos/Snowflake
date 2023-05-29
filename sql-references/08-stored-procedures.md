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

`You can write a handler in the following languages:`
- Java (using the Snowpark API)
- JavaScript
- Python (using the Snowpark API)
- Scala (using the Snowpark API)
- Snowflake Scripting (SQL)

For a role to use a SP, they must either be the owner, or have been granted USAGE privilege on the SP.

Although stored procedures allow nesting and recursion, the current maximum stack depth of nested calls for user-defined stored procedures is 5.

You can minimize the risk of SQL injection attacks by binding parameters rather than concatenating text.

## Caller and Owner Rights
A SP runs with either the caller's rights, or the owner's rights. At the time that the SP is created, the creator specifies whether the procedure runs with owner's rights, or caller's rights. `The default is owner's rights.` The owner can interchange these two rights by an ALTER PROCEDURE command. 

A caller's rights stored procedure can read/set/unset the caller's session parameters/variables. If a caller's rights stored procedure makes changes to the session, those changes can persist after the end of the CALL. 

An owner's rights stored procedure runs mostly with the privileges of the stored procedure's owner. Owner's rights stored procedures are not permitted to change session state (cannot view/set/unset session variables; can view some session parameters, but cannot set/unset them) of the caller. 


The primary advantage of an owner's rights stored procedure is that the owner can delegate specific administrative tasks, such as cleaning up old data, to another role without granting that role more general privileges, such as privileges to delete all data from a specific table.






















