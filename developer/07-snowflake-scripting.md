# 7. Snowflake scripting
An extension to Snowflake SQL, that adds support for procedural logic.

## Blocks
Basic structure:
```sql
declare -- optional. For variables/cursors/resultsets/exceptions
  ... (variable declarations, cursor declarations, etc.) ...
begin
  ... (snowflake scripting and sql statements) ...
exception
  ... (statements for handling exceptions) ...
end;
```

The keyword BEGIN that starts a block, is different from the keyword BEGIN that starts a transaction. To minimize confusion, start transactions with BEGIN TRANSACTION.

Any db objects that you create in a block, can be used outside of the block.

Example:
```sql
declare -- these vars cannot be used outside of the block
    radius_of_circle float;
    area_of_circle float;
begin
    radius_of_circle := 3;
    area_of_circle := pi() * radius_of_circle * radius_of_circle;
    return area_of_circle;
end;
```

You can use a block in a SP definition. If you use SnowSight, you do not need `$$` to enclose the code (compared with legacy console, and SnowSQL). 

If you don't want to store the block in a SP, you can define/use an anonymous block, which is a block that is not part of a SP. In SnowSight, The BEGIN statement of an anonymous block defines the block, also executes the block. 

## Variables
When you declare a variable, you must specify the data type of the variable explicitly/implicitly, either in the declare block, or in the begin ... end block.

Example in Snowsight: 
```sql
declare
    profit number(38, 2) default 0.0; -- var declared in the declare block
begin
    let cost number(38, 2) := 100.0; -- var declared in the begin-end block
    let revenue number(38, 2) default 110.0;

    profit := revenue - cost;
    return profit;
end;
```

Same example in legacy-console/SnowSQL:
```sql
execute immediate $$ -- this is the wrap
    declare
        profit number(38, 2) default 0.0;
    begin
        let cost number(38, 2) := 100.0;
        let revenue number(38, 2) default 110.0;

        profit := revenue - cost;
        return profit;
    end;
$$
;
```

When a variable is declared in the DECLARE section of a block, its scope is that block, and any blocks nested in that block. 

If a block declares an object, and there is an object declared in an outer block with the same name, then the inner one takes precedence. 

### Binding
Use a var in a sql statement. 

```sql
insert into my_table (x) values (:my_variable) -- use var in a sql statement

select count(*) from identifier(:table_name) -- use var as name of an object

return profit; -- var in expression do not need colon prefix

let select_statement := 'select * from invoices where id = ' || id_variable; -- var in expression do not need colon prefix
```

### Set vars using as SELECT statement
```sql
declare
  id_variable integer;
  name_variable varchar;
begin
  select id, name into :id_variable, :name_variable from some_data where id = 1;
  return id_variable || ' ' || name_variable;
end;
```

### Variables examples
```sql
create procedure duplicate_name(pv_name varchar)
returns varchar
language sql
as
begin
  declare
    pv_name varchar;
  begin
    pv_name := 'middle block variable';
    declare
      pv_name varchar;
    begin
      pv_name := 'innermost block variable';
      insert into names (v) values (:pv_name);
    end;
    -- because the innermost and middle blocks have separate variables
    -- named "pv_name", the insert below inserts the value
    -- 'middle block variable'.
    insert into names (v) values (:pv_name);
  end;
  -- this inserts the value of the input parameter.
  insert into names (v) values (:pv_name);
  return 'completed.';
end;

call duplicate_name('parameter');

```

## Returning a Value
You can return a value of of either:
- A SQL data type, such as `return my_variable; `
- A table, such as `returns table(my_result_set); `

## Conditional Logic
"if" statement: 
```sql
begin
  let count := 1;
  if (count < 0) then
    return 'negative value';
  elseif (count = 0) then
    return 'zero';
  else
    return 'positive value';
  end if;
end;
```

"case" statement:
```sql
-- simple case statements
declare
  expression_to_evaluate varchar default 'default value';
begin
  expression_to_evaluate := 'value a';
  case (expression_to_evaluate)
    when 'value a' then
      return 'x';
    when 'value b' then
      return 'y';
    when 'value c' then
      return 'z';
    when 'default value' then
      return 'default';
    else
      return 'other';
  end;
end;

-- searched case statements
declare
  a varchar default 'x';
  b varchar default 'y';
  c varchar default 'z';
begin
  case
    when a = 'x' then
      return 'a is x';
    when b = 'y' then
      return 'b is y';
    when c = 'z' then
      return 'c is z';
    else
      return 'a is not x, b is not y, and c is not z';
  end;
end;
```

## Loops
```sql
-- for loop, counter based, example
declare
  counter integer default 0;
  maximum_count integer default 5;
begin
  for i in 1 to maximum_count do
    counter := counter + 1;
  end for;
  return counter;
end;

-- for loop, cursor based, example
create or replace table invoices (price number(12, 2));
insert into invoices (price) values
  (11.11),
  (22.22);

declare
  total_price float;
  c1 cursor for select price from invoices;
begin
  total_price := 0.0;
  for record in c1 do
    total_price := total_price + record.price;
  end for;
  return total_price;
end;

-- while loop, example
begin
  let counter := 0;
  while (counter < 5) do
    counter := counter + 1;
  end while;
  return counter;
end;

-- repeat loop, example
begin
  let counter := 5;
  let number_of_iterations := 0;
  repeat
    counter := counter - 1;
    number_of_iterations := number_of_iterations + 1;
  until (counter = 0)
  end repeat;
  return number_of_iterations;
end;

-- loop loop, example
begin
  let counter := 5;
  loop
    if (counter = 0) then
      break;
    end if;
    counter := counter - 1;
  end loop;
  return counter;
end;
```

You can explicitly terminate a loop early, by executing the `BREAK` command. 

You can use the `CONTINUE` (or `ITERATE`) command to jump to the end of an iteration of a loop, skipping the remaining statements in it. The loop continues at the start of the next iteration.

You can specify which loop to continue, ec:
```sql
begin
  let inner_counter := 0;
  let outer_counter := 0;
  loop
    loop
      if (inner_counter < 5) then
        inner_counter := inner_counter + 1;
        continue outer;
      else
        break outer;
      end if;
    end loop inner;
    outer_counter := outer_counter + 1;
    break;
  end loop outer;
  return array_construct(outer_counter, inner_counter);
end;
```

## Cursors
Use it to iterate through query results, one row at a time.

```sql
declare
  id integer default 0;
  minimum_price number(13,2) default 22.00;
  maximum_price number(13,2) default 33.00;
  c1 cursor for select id from invoices where price > ? and price < ?; -- ? are bind parameters
begin
  -- Although cursor definition has a query inside, the query is not executed until you "open" the cursor
  open c1 using (minimum_price, maximum_price);
  -- Use "fetch" to retrieve the current row, and advance the pointer to the next row.
  fetch c1 into id;
  return id;
end;
```

When using a cursor in a FOR loop, you do not need to "open" the cursor explicitly.

If you declare a cursor for a RESULTSET obj, the query is executed when you associate the RESULTSET with the query. Therefore opening the cursor will not re-execute it.

As with any SQL query, if the query definition does not contain an ORDER BY, then the result set has no defined order. 

If you try to FETCH a row after the last row, you get NULL.

```sql
declare
  c1 cursor for select * from invoices;
begin
  open c1;
  -- returns a table from the cursor
  return table(resultset_from_cursor(c1));
end;
```

To close a cursor: `close my_cursor;`

## RESULTSETs
A pointer, that points to the result set of a query.

Cursor vs RESULTSET: 
| Cursor      | RESULTSET   |
| ----------- | ----------- |
| Query is executed with its OPEN command  | Query is executed when you assign the query to it       |
| Support binding in OPEN cmd   | Do not support OPEN cmd        |

```sql
-- assign query directly
declare
  res resultset;
begin
  res := (select col1 from mytable order by col1);
  ...
;

-- assign query via a variable
declare
  res resultset;
  col_name varchar;
  select_statement varchar;
begin
  col_name := 'col1';
  select_statement := 'select ' || col_name || ' from mytable';
  res := (execute immediate :select_statement);
  return table(res); -- return the RESULTSET via a table
end;

-- access RESULTSET with a cursor
declare
  ...
  res resultset default (select col1 from mytable order by col1);
  c1 cursor for res;
...
;

```

## Exceptions
Snowflake Scripting raises an exception if an error occurs while executing a statement, such as when it attempts to DROP a non-existing table. An exception prevents the next lines of code from executing.

If the block (in which the exception occurred) has a handler for that exception, then execution resumes at the handler.

An exception handler can contain its own exception handler, in case an exception occurs while handling another exception.

```sql
declare
  -- declare your own exception
  my_exception exception (-20002, 'Raised MY_EXCEPTION.');
begin
  let counter := 0;
  let should_raise_exception := true;
  if (should_raise_exception) then
    raise my_exception;
  end if;
  counter := counter + 1;
  return counter;
end;

-- return:
-- -20002 (P0001): Uncaught exception of type 'MY_EXCEPTION' on line 8 at position 4 : Raised MY_EXCEPTION.

```

You can handle customized exceptions, and built-in exceptions. Currently, Snowflake provides these built-in exceptions:
- STATEMENT_ERROR: an error while executing a statement. e.g., you attempt to drop a non-existent table.
- EXPRESSION_ERROR: an error related to an expression. e.g., you create an expression that evaluates to a VARCHAR, and attempt to assign it to a FLOAT.

```sql
declare
  my_exception exception (-20002, 'Raised MY_EXCEPTION.');
begin
  let counter := 0;
  let should_raise_exception := true;
  if (should_raise_exception) then
    raise my_exception;
  end if;
  counter := counter + 1;
  return counter;
exception
  when statement_error then
    return object_construct('Error type', 'statement_error',
                            'sqlcode', sqlcode,
                            'sqlerrm', sqlerrm,
                            'sqlstate', sqlstate);
  when my_exception then
    return object_construct('Error type', 'my_exception',
                            'sqlcode', sqlcode,
                            'sqlerrm', sqlerrm,
                            'sqlstate', sqlstate);
  when other then
    return object_construct('Error type', 'Other error',
                            'sqlcode', sqlcode,
                            'sqlerrm', sqlerrm,
                            'sqlstate', sqlstate);
END;

-- result:
-- +--------------------------------------+
-- | anonymous block                      |
-- |--------------------------------------|
-- | {                                    |
-- |   "Error type": "MY_EXCEPTION",      |
-- |   "SQLCODE": -20002,                 |
-- |   "SQLERRM": "Raised MY_EXCEPTION.", |
-- |   "SQLSTATE": "P0001"                |
-- | }                                    |
-- +--------------------------------------+
```

When you need to raise the same exception that you caught in your exception handler, simply execute the RAISE command.

## Affected Rows
After each DML command is executed, Snowflake Scripting sets the 3 global variables, which can be used, such as `RETURN SQLROWCOUNT;`, `IF (SQLNOTFOUND = true) THEN ...`: 
- SQLROWCOUNT: num of rows affected by the last dml statement. 
- SQLFOUND: TRUE if the statement affected >=1 rows. 
- SQLNOTFOUND: TRUE if the statement affected 0 rows. 

## Getting a Query ID
If you need to access the query ID of the last query that was executed, use the global variable SQLID. 

## Using Snowflake Scripting in SnowSQL or the Classic Console
Use delimiters around the start and end of a Snowflake Scripting block, if you are using SnowSQL, or the Classic Console.

If you are writing an anonymous block, pass the block as a string literal, to the EXECUTE IMMEDIATE command. 
