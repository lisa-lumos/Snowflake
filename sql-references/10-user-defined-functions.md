# 10. User-defined functions

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

For a JavaScript UDF:
arguments:
SQL regular `NULL` => JavaScript `undefined`
SQL variant `null` => JavaScript `null`

Returned: 
JavaScript `undefined` => SQL `NULL`
JavaScript `null` => SQL variant `null`

Summary of above arguments and returned val:
SQL regular `NULL` <=> JavaScript `undefined`
SQL variant `null` <=> JavaScript `null`

Note: in sql -
- `cast(null as variant)` produces SQL `NULL`
- `PARSE_JSON('null')` produces SQL variant `null`


In the returned JS object to be casted to SQL variant:
- an `undefined` as the value in key-val pair, the whole pair will be omitted. 
- an `undefined` as an element in array, this value will be omitted. 


For a SQL UDF:
A reference to an schema object requires the function owner to have privileges to access that schema object. The invoker of the function only needs the privilege to use the function, they do not need to have access to the objects used in the function definition. (my thought: this works like a view)

























