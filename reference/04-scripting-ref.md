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

## fetch ... into ...
Uses the specified cursor to fetch one or more rows.

## for ... end for
loop

## if ... then elseif ... then ... else ... end if
conditional

## let ... := ...
Assigns an expression to a Snowflake Scripting variable, cursor, or RESULTSET.

## loop ... end loop
The user must explicitly exit the loop, by using BREAK or RETURN inside it.

## null;
NULL can be used as a "no-op" (no operation) statement (uncommon):
- The NULL statement can be executed only inside Snowflake Scripting scripts.
- A NULL statement in an exception handler ensures that the code continues executing, rather than aborting, if there is no higher-level handler.
- A NULL statement in a branch does nothing; however, it communicates to the reader, that the branch condition was not overlooked or accidentally omitted.
- Before using the NULL statement, consider alternatives.

## open ...
Opens a cursor.

## raise ...;
Raises an exception.

## repeat ... until ... end repeat;
Works like a do ... while in programming languages. 

## return ...;


## while ... do ... end while;
loop. 
