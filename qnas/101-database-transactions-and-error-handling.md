# 101 - Database transactions and error handling: Q&A

## Multiple-choice questions

### 1. What is the main purpose of a database transaction?

- **A.** To group related database operations so they can succeed or be undone together
- **B.** To encrypt every value stored in the database
- **C.** To replace input validation
- **D.** To make every database driver behave identically

**Answer: A.** A transaction lets an application commit the required database work as one unit or roll back its uncommitted changes when the operation cannot be completed.

### 2. Which PDO method starts a transaction?

- **A.** PDO::commit()
- **B.** PDO::beginTransaction()
- **C.** PDO::inTransaction()
- **D.** PDO::prepare()

**Answer: B.** beginTransaction() starts a transaction on that PDO connection. The driver and database table configuration must support transactional work.

### 3. What does PDO::commit() do?

- **A.** Makes the active transaction's changes permanent
- **B.** Starts a second nested transaction
- **C.** Deletes all changes from the current transaction
- **D.** Rolls back the last SQL statement only

**Answer: A.** Commit completes the transaction successfully and returns the connection to its normal auto-commit behavior.

### 4. When should PDO::rollBack() be called?

- **A.** When required work fails before the transaction has committed
- **B.** After every successful SELECT
- **C.** Before beginTransaction()
- **D.** After sending an email

**Answer: A.** Rollback discards uncommitted transaction changes. It cannot undo work that was already committed or an external action such as an email.

### 5. What does PDO::inTransaction() tell the application?

- **A.** Whether PDO considers the connection to be in an active transaction
- **B.** Whether every table in the database supports transactions
- **C.** Whether the last query changed a row
- **D.** Whether the database is online

**Answer: A.** It is useful before a conditional rollback because an error may happen before a transaction begins or after it ends.

### 6. Why use a try/catch block around a transaction?

- **A.** To commit on success and roll back active uncommitted work after a failure
- **B.** To ignore all database errors
- **C.** To make unsafe SQL concatenation safe
- **D.** To prevent every possible database outage

**Answer: A.** The error path should roll back when needed and then handle or rethrow the failure. It should not report success after an operation failed.

### 7. Which statement about PDO exception mode is correct?

- **A.** It throws PDOException for many database failures; it is the default since PHP 8, but setting it explicitly can make intent clear
- **B.** It disables all database errors
- **C.** It is available only with SQLite
- **D.** It automatically retries every failed transaction

**Answer: A.** Exception mode supports a clear error-handling path. Explicit configuration makes the intended behavior visible across codebases and older PHP examples.

### 8. What is the purpose of checking inTransaction() before rolling back in a catch block?

- **A.** Avoid calling rollback when no transaction is active
- **B.** Prove that the transaction will roll back successfully
- **C.** Find out whether a SELECT returned rows
- **D.** Re-enable SQL injection protection

**Answer: A.** Calling rollBack() without an active transaction can itself fail. The check guards that call, though the rollback can still fail if the connection has a problem.

### 9. If rollback itself throws after the original database operation failed, what should robust code consider?

- **A.** Preserve the original failure and record the rollback failure as well
- **B.** Report success
- **C.** Retry rollback forever
- **D.** Display both exception messages publicly

**Answer: A.** A rollback failure is useful diagnostic information, but it should not silently replace the original cause. Keep both details in protected logs.

### 10. Does a database transaction roll back an email already sent by the application?

- **A.** Yes, every external action is undone
- **B.** No, the database cannot reverse an action performed by an email service
- **C.** Only when PDO exception mode is enabled
- **D.** Only when the message contains an attachment

**Answer: B.** A PDO transaction governs database work on its connection. Send external effects after commit or use a durable pattern such as an outbox.

### 11. Can every PDO driver and database table be assumed to support transactions?

- **A.** Yes, PDO guarantees it
- **B.** No, support depends on the driver, database, and table or storage configuration
- **C.** Only for SELECT statements
- **D.** Only when using named placeholders

**Answer: B.** PDO gives a common API, but the underlying database and table must provide the transactional behavior the application needs.

### 12. What can happen when some databases run DDL such as CREATE TABLE inside a transaction?

- **A.** The DDL may cause an implicit commit
- **B.** PDO automatically converts it to a SELECT
- **C.** The schema change is always undone by rollBack()
- **D.** The database disables all constraints permanently

**Answer: A.** Some databases implicitly commit around certain schema changes. Keep DDL out of application transactions unless the target database's behavior is understood.

### 13. Does calling beginTransaction() a second time create a portable nested PDO transaction?

- **A.** Yes, PDO creates a nested transaction automatically
- **B.** No, PDO does not provide portable nested transactions through repeated beginTransaction() calls
- **C.** It creates a savepoint on every database
- **D.** It commits the first transaction

**Answer: B.** Let one layer own the transaction boundary. Some databases support savepoints, but their syntax and behavior are database-specific.

### 14. Why should an application not leave a transaction open while waiting for an HTTP request or user input?

- **A.** It can hold locks and database resources longer, increasing contention
- **B.** It makes placeholders stop working
- **C.** It automatically commits the transaction
- **D.** It changes every string into an integer

**Answer: A.** Keep transactions focused on the required database work. Long-running transactions can interfere with other connections and consume resources.

### 15. Should rowCount() be treated as a portable count of rows returned by SELECT?

- **A.** Yes, it always returns the same value across drivers
- **B.** No, SELECT row-count behavior varies; fetch rows or use COUNT when needed
- **C.** Only if the query has ORDER BY
- **D.** Only if the database has one table

**Answer: B.** Fetch the results or execute a deliberate COUNT query to count selected rows. Affected-row behavior for writes should also be interpreted with driver semantics in mind.

### 16. Is every PDOException a reason to retry a transaction?

- **A.** Yes, all database errors are temporary
- **B.** No, retry only known transient failures with a bounded, safe policy
- **C.** Yes, but only five times
- **D.** No, database operations should never be retried

**Answer: B.** Invalid SQL, constraint violations, permission errors, and programming bugs generally will not be fixed by repeating the operation. Deadlocks or serialization conflicts may be retryable when designed for it.

### 17. What must be true before retrying a transaction that may have partially executed?

- **A.** The whole operation can safely be repeated, with a fresh transaction and bounded attempts
- **B.** The code prints the exception to the user
- **C.** The first transaction is left open
- **D.** The operation sends an email before retrying

**Answer: A.** Retry the complete unit, not an arbitrary middle statement. Idempotency or deduplication is important, especially when the result of a failed commit is uncertain.

### 18. What is a safe public response to an unexpected database failure?

- **A.** A generic error, with technical diagnostics logged securely
- **B.** The full SQL, DSN, and stack trace
- **C.** A success message to avoid worrying the user
- **D.** The database password

**Answer: A.** Technical details can reveal internal paths, schema, SQL, or infrastructure. Keep them in protected logs and return an appropriate non-sensitive response.

## Short-answer questions

### 19. Describe the normal success and failure paths of a transaction.

**Answer:** Begin the transaction, perform and check all required database operations, then commit on success. If required work fails before commit, roll back the active transaction. **Explanation:** The commit belongs after the full set of required work, not between dependent operations.

### 20. Why should a transaction have one clear owner?

**Answer:** One owner can define when the transaction starts, what it includes, and who commits or rolls it back. **Explanation:** If several layers independently begin or end transactions, a helper can accidentally commit work that its caller still expects to be reversible.

### 21. Why validate input before opening a transaction when possible?

**Answer:** It avoids holding a database transaction open for input that can be rejected without querying or changing data. **Explanation:** Perform database-dependent checks inside the transaction when they must be consistent with the writes.

### 22. Why should transactions be kept short?

**Answer:** Long transactions can hold locks and database resources and can block or conflict with other work. **Explanation:** Do not wait for user interaction, remote HTTP calls, or slow unrelated operations while a transaction is open.

### 23. Do transactions replace prepared statements?

**Answer:** No. Transactions group database operations; prepared statements keep dynamic values separate from SQL syntax. **Explanation:** Use both when an operation needs multiple related queries that include variable data.

### 24. Does a transaction check whether the current user may change a record?

**Answer:** No. Application authorization must be checked separately. **Explanation:** A transaction controls how database changes are committed; it does not decide whether a caller is permitted to make them.

### 25. Why can a DDL change survive a rollback on some databases?

**Answer:** Some databases implicitly commit when certain schema statements run. **Explanation:** Learn the target database behavior and keep schema changes outside ordinary data transactions unless rollback semantics are guaranteed.

### 26. What is an outbox pattern used for?

**Answer:** It stores an event or message record in the same database transaction as the business change, then a separate process delivers it. **Explanation:** This helps coordinate database changes with external services when those services cannot participate in the database transaction.

### 27. Why should a commit failure caused by a lost connection not trigger an automatic blind retry?

**Answer:** The application may not know whether the database committed before the connection failed. **Explanation:** Repeating a non-idempotent operation could apply it twice; use an idempotency key or another deduplication design.

### 28. What information belongs in secure diagnostic logs rather than a public response?

**Answer:** Relevant exception details and database diagnostics that operators need, while excluding credentials and unnecessary sensitive data. **Explanation:** Public output should not reveal SQL, DSNs, file paths, stack traces, or internal infrastructure details.

## Practical scenarios

### 29. A transfer debits the sender, then the recipient update fails. What should happen?

**Answer:** Roll back the active transaction so the sender debit is not committed by itself. **Explanation:** Both account changes represent one logical operation and should only persist together.

### 30. The database updates succeed, then an email send fails before commit. What design issue should be considered?

**Answer:** Do not assume the database can undo an email; consider committing the database work and delivering an outbox event after commit. **Explanation:** External systems do not participate in PDO's transaction.

### 31. A catch block always calls rollBack(), even if beginTransaction() failed. What should change?

**Answer:** Check inTransaction() before the rollback call. **Explanation:** A begin failure can leave no active transaction, and calling rollBack() then can raise another exception.

### 32. A helper function calls beginTransaction(), but its caller already started one. What is the safer design?

**Answer:** Make one layer own the transaction and let the helper perform its database work within that boundary. **Explanation:** Repeated beginTransaction() calls are not a portable nested-transaction feature; use database-specific savepoints only when deliberately designed.

### 33. A migration runs CREATE TABLE, then a later query fails and the code calls rollBack(). Can the application assume the table creation was undone?

**Answer:** No. Some database systems implicitly commit around DDL. **Explanation:** Verify the target database's transaction semantics and separate schema migration work from application data transactions when needed.

### 34. A deadlock occurs during a multi-step operation. What should a retry implementation do?

**Answer:** If that specific failure is documented as transient, roll back if possible and retry the entire operation in a fresh transaction with a bounded attempt count and backoff. **Explanation:** Ensure the operation is safe to repeat and do not retry unrelated errors such as invalid SQL or failed authorization.

### 35. A web endpoint returns PDOException::getMessage() to the browser. What should change?

**Answer:** Log useful details securely and return a generic public error response. **Explanation:** The exception message can disclose SQL, schema, paths, or database configuration.

### 36. An UPDATE executes without an exception but rowCount() is zero. Does that always mean the database failed?

**Answer:** No. The WHERE clause may have matched no rows, a condition may have failed, or driver behavior may affect affected-row counts. **Explanation:** Define the expected business result and interpret rowCount() according to the database and statement.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [100 - Prepared statements and safe database queries](../lessons/100-prepared-statements-and-safe-database-queries.md) | [Lesson](../lessons/101-database-transactions-and-error-handling.md) | [Example](../examples/101-database-transactions-and-error-handling.md) | [102 - Filtering, sorting, and paginating database results](../lessons/102-filtering-sorting-and-paginating-database-results.md) |
