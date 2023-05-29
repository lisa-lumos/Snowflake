# 12. Transactions
A transaction is a sequence of SQL statements that are committed or rolled back as a unit.

Transactions follow these rules:
- `Transactions are never nested.` You cannot create an outer transaction that would roll back an inner transaction that was committed, or create an outer transaction that would commit an inner transaction that had been rolled back.
- A transaction is associated with a single session. Multiple sessions cannot share the same transaction. 

A transaction can be started explicitly by executing a `BEGIN` statement. A transaction can be ended explicitly by executing `COMMIT` or `ROLLBACK`. 

Explicit transactions should contain only DML statements and query statements. DDL statements implicitly commit active transactions

Each DDL statement (such as CREATE/DROP TABLE, ALTER TABLE ADD COLUMN, ...) executes as a separate transaction. So you cannot roll back a DDL statement.

If a DDL statement is executed while a transaction is active, the DDL statement:
- First implicitly commits the active transaction.
- Then executes itself (DDL statement) as a separate transaction.

Snowflake supports an AUTOCOMMIT parameter. The default setting for AUTOCOMMIT is on.

While AUTOCOMMIT is enabled:
- individual statements are automatically committed if it succeeds, and automatically rolled back if it fails.
- Statements inside an explicit transaction are not affected by AUTOCOMMIT.

While AUTOCOMMIT is disabled:
- An implicit BEGIN TRANSACTION happens when:
  - The first DML/query statement after a transaction ends, or `ALTER SESSION UNSET AUTOCOMMIT`
- An implicit COMMIT happens when: 
  - The execution of a DDL statement, or `ALTER SESSION SET AUTOCOMMIT`
- An implicit ROLLBACK happens when:
  - The end of a session/SP

You cannot change AUTOCOMMIT settings inside a SP.

To avoid writing confusing code, avoid mixing implicit and explicit starts/ends in the same transaction. 

If a statement fails within a transaction, you can still commit the transaction, rather than roll back. If the transaction is committed, the changes by the successful statements are applied. The statements after the failed INSERT statement in this transaction might or might not be executed, depending upon how the statements are run and how errors are handled:
- If it is inside an SP, the failed statement throws an exception:
  - If the exception is not handled, this transaction is implicitly rolled back. 
  - If the exception is handled, and it commits the statements prior to the failed statement, then only the committed vals exist in the target table.
- If it is not inside a SP:
  - If executed through Snowsight, execution halts at the first error.
  - If executed by SnowSQL using the -f (filename) option, then the statements after the error are executed.

When a DML/CALL statement in a transaction fails, the changes made by that failed statement are rolled back. 

Snowflake recommends that multi-threaded client programs do at least one of the following:
- Use a separate connection for each thread.
- Execute the threads synchronously, to control the order in which steps are performed.

A transaction can be inside a SP, or a SP can be inside a transaction; however, a transaction cannot be partly inside and partly outside a SP, or started in one SP and finished in a different SP.

The SP inside a transaction:
- If the transaction is committed, then all statements inside the SP are committed.
- If the transaction is rolled back, then all statements inside the SP are rolled back.

You can execute any number of transactions inside a SP. 

Scoped Transactions: A SP that contains a transaction can be called from within another transaction. For example, a transaction inside a SP can include a call to another SP that contains a transaction. Snowflake does not treat the inner transaction as nested; instead, the inner transaction is a independent/separate transaction. 

In a SP:
- a `begin transaction;` without an explicit `commit/rollback` will be implicitly rolled back. 
- a `commit` without an explicit `begin transaction` will cause an error. 

Overlapping scoped transactions can cause a deadlock if they manipulate the same database object (e.g. table). Scoped transactions should be used only when necessary.

When AUTOCOMMIT is off, be especially careful combining implicit transactions and stored procedures. If you accidentally leave a transaction active at the end of a SP, the transaction is rolled back.

When AUTOCOMMIT is set to false, and the first DML after that happens inside an SP, and this SP did not end with a `commit/roll back`, then this DML and whatever after it in the SP is implicitly rolled back - even if there is a `commit` outside and after the SP. To avoid this, you can either:
- outside the SP, add a `begin transaction;` before entering the SP. 
- inside the SP, explicitly use `begin transaction` and `commit/roll back` 

Inside a SP, if `begin` is missing its `commit`, or vise versa, it will be rolled back and return an error. 

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


