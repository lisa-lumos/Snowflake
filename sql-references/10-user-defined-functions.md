# 10. User-defined functions
A user-defined function (UDF) is a function you define so you can call it from SQL. A UDF typically extends or enhances SQL with functionality that SQL doesn't have or doesn't do well. A UDF also gives you a way to encapsulate functionality so that you can call it repeatedly from multiple places in code.

You can write a UDF that returns a single value (a scalar UDF), or that returns a tabular value (a user-defined table function, or UDTF).

Supported languages:
- Java
- JavaScript
- Python
- Scala
- SQL

```sql
create or replace function addone(i int) -- the function name of the udf
returns int
language python
runtime_version = '3.8'
handler = 'addone_py' -- the function name in the body
as
$$
  def addone_py(i):
    return i+1
$$;

select addone(3);
```


































