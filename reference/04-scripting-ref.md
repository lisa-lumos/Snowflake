# 4. Scripting Reference
## begin ... end;
Define a Snowflake Scripting block.

Blocks can be nested.

## break
Terminates a loop. 

Can append a label to break out of more than one level of a nested loop/branch.

## case
Multiple conditions. 

## close
Closes the specified cursor.

## continue
Go directly to the next iteration of the loop. 

## declare
Declares one or more Snowflake Scripting variables, cursors, RESULTSETs, or exceptions.

## exception
Specifies how to handle exceptions raised in the Snowflake Scripting block.

The WHEN OTHER THEN clause catches any exception not yet specified.

If a stored procedure is intended to return a value, then it should return a value from each possible path, including each WHEN clause of the exception handler.

## fetch
Uses the specified cursor to fetch one or more rows.

## for


## if


## let


## loop


## null


## open


## raise


## repeat


## return


## while




































