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



## RESULTSETs



## Exceptions



## Affected Rows



## Getting a Query ID



## Using Snowflake Scripting in SnowSQL or the Classic Console








































