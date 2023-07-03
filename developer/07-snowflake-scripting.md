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




## Returning a Value



## Conditional Logic



## Loops



## Cursors



## RESULTSETs



## Exceptions



## Affected Rows



## Getting a Query ID



## Using Snowflake Scripting in SnowSQL or the Classic Console








































