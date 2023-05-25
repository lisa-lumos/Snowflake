# 12. Transactions
A transaction is a sequence of SQL statements that are committed or rolled back as a unit.

Transactions follow these rules:
- `Transactions are never nested.` For example, you cannot create an outer transaction that would roll back an inner transaction that was committed, or create an outer transaction that would commit an inner transaction that had been rolled back.
- A transaction is associated with a single session. Multiple sessions cannot share the same transaction. 

A transaction can be started explicitly by executing a `BEGIN` statement. A transaction can be ended explicitly by executing `COMMIT` or `ROLLBACK`. 

Explicit transactions should contain only DML statements and query statements. DDL statements implicitly commit active transactions

Each DDL statement executes as a separate transaction.

If a DDL statement is executed while a transaction is active, the DDL statement:
- First implicitly commits the active transaction.
- Then executes the DDL statement as a separate transaction.

Because a DDL statement is its own transaction, you cannot roll back a DDL statement. 

Snowflake supports an AUTOCOMMIT parameter. The default setting for AUTOCOMMIT is on.

While AUTOCOMMIT is enabled:
- statements are automatically committed if it succeeds, and automatically rolled back if it fails.
- Statements inside an explicit transaction are not affected by AUTOCOMMIT.

While AUTOCOMMIT is disabled:
- An implicit BEGIN TRANSACTION is executed at:
  - The first DML statement or query statement after a transaction ends
  - The first DML statement or query statement after disabling AUTOCOMMIT
- An implicit COMMIT is executed at: 
  - The execution of a DDL statement
  - The execution of ALTER SESSION SET AUTOCOMMIT
- An implicit ROLLBACK is executed at:
  - The end of a session
  - The end of a SP

To avoid writing confusing code, avoid mixing implicit and explicit starts and ends in the same transaction. 

If a statement fails within a transaction, you can still commit, rather than roll back, the transaction. If the transaction is committed, the changes by the successful statements are applied. The statements after the failed INSERT statement might or might not be executed, depending upon how the statements are run and how errors are handled.

When a DML statement or CALL statement in a transaction fails, the changes made by that failed statement are rolled back. 

Snowflake recommends that multi-threaded client programs do at least one of the following:
- Use a separate connection for each thread.
- Execute the threads synchronously to control the order in which steps are performed.

A transaction can be inside a stored procedure, or a stored procedure can be inside a transaction; however, a transaction cannot be partly inside and partly outside a stored procedure, or started in one stored procedure and finished in a different stored procedure.

The stored procedure inside a transaction follows the rules of the enclosing transaction:
- If the transaction is committed, then all the statements inside the procedure are committed.
- If the transaction is rolled back, then all statements inside the procedure are rolled back.

You can execute any number of transactions inside a stored procedure. 

Scoped Transactions: A stored procedure that contains a transaction can be called from within another transaction. For example, a transaction inside a stored procedure can include a call to another stored procedure that contains a transaction. (My question - why do not sf call them nested transactions?)

Overlapping scoped transactions can cause a deadlock if they manipulate the same database object (e.g. table). Scoped transactions should be used only when necessary.

When AUTOCOMMIT is off, be especially careful combining implicit transactions and stored procedures. If you accidentally leave a transaction active at the end of a stored procedure, the transaction is rolled back.

READ COMMITTED is the only isolation level currently supported for tables.

When a statement is executed inside a multi-statement transaction:
- A statement sees only data that was committed before the statement began. Two successive statements in the same transaction can see different data if another transaction is committed between the execution of the first and the second statements.
- A statement does see the changes made by previous statements executed within the same transaction, even though those changes are not yet committed.

A blocked statement either acquires a lock on the resource it was waiting for or times out waiting for the resource to become available. The amount of time (in seconds) that a statement should block can be configured by setting the LOCK_TIMEOUT parameter in seconds.

Snowflake detects deadlocks and chooses the most recent statement which is part of the deadlock as the victim. 

To allow a statement error within a transaction to abort the transaction, set the TRANSACTION_ABORT_ON_ERROR parameter at the session or account level.

QUERY_HISTORY View (in Account usage schema) can show transaction_blocked_time for a query. To find blocker transactions for the queries identified from the previous step, query the LOCK_WAIT_HISTORY view (in Account usage schema) for rows with those query IDs. 

If a transaction is running in a session and the session disconnects abruptly, preventing the transaction from committing or rolling back, the transaction is left in a detached state, including any locks that the transaction is holding on resources. If this happens, you might need to abort the transaction. The user who started the transaction or an account administrator can call the system function, SYSTEM$ABORT_TRANSACTION().

If the transaction is not aborted by the user:
- If it blocks another transaction from acquiring a lock on the same table and is idle for 5 minutes, it is automatically aborted and rolled back.
- If it does not block other transactions from modifying the same table and is older than 4 hours, it is automatically aborted and rolled back.

Inserting 10 rows in one transaction is generally faster and cheaper than inserting one row each in 10 separate transactions. Combining multiple statements into a single transaction can improve performance.

Snowflake recommends keeping AUTOCOMMIT enabled and using explicit transactions as much as possible.

Every Snowflake transaction is assigned a unique transaction id. 

If a scoped transaction was committed, even if the outer level transaction that contains it was rolled back later, the scoped transaction is still committed. 


