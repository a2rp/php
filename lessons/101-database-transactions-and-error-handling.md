# 101 - Database transactions and error handling

## Learning goals

By the end of this lesson, you will be able to:

- Explain what a database transaction is and when related changes belong together.
- Use PDO beginTransaction(), commit(), and rollBack() correctly.
- Roll back safely when an operation fails.
- Check whether a transaction is still active.
- Keep business validation and database work inside a clear transaction boundary.
- Handle PDO exceptions without leaking database details.
- Recognize driver, table, DDL, concurrency, and external-side-effect limits.
- Avoid unsafe retry and nested-transaction patterns.

## 1. What is a transaction?

A transaction groups database operations into one logical unit. If all required operations succeed, the application commits them. If an operation fails before commit, the application rolls the transaction back.

For a bank transfer, subtracting money from one account and adding it to another are related changes. If only one update is saved, the database no longer represents the intended transfer. A transaction gives the application a way to make both changes together.

Transactions are often explained through ACID:

- **Atomicity:** The grouped work is treated as a unit: commit it or undo its uncommitted changes.
- **Consistency:** The application and database constraints should keep the data valid before and after the transaction.
- **Isolation:** Concurrent transactions should not produce results that violate the database's configured isolation behavior.
- **Durability:** A committed transaction is persisted according to the database's guarantees and configuration.

These guarantees depend on the database engine, storage configuration, and the operations being used. PDO provides a common PHP API, but it cannot make a non-transactional table or unsupported driver transactional.

## 2. Understand PDO's transaction methods

PDO starts in its normal auto-commit behavior. Start a transaction explicitly when multiple operations must succeed together:

~~~php
$pdo->beginTransaction();

// Execute related database operations here.

$pdo->commit();
~~~

- **beginTransaction():** Starts a transaction and disables auto-commit for that connection until the transaction ends.
- **commit():** Makes the transaction's changes permanent and returns the connection to auto-commit behavior.
- **rollBack():** Discards changes made since the transaction began and returns the connection to auto-commit behavior.
- **inTransaction():** Reports whether PDO currently considers the connection to be inside a transaction.

If the driver does not support transactions, beginTransaction() can fail. A successful call also cannot guarantee that every table or operation has transactional behavior; check the database engine and schema configuration.

Use PDO's transaction methods consistently. If application code starts a transaction by sending a raw SQL BEGIN command, PDO may not know it started one and cannot reliably manage that transaction through inTransaction() or its automatic cleanup behavior.

## 3. Use try/catch to commit or roll back

A transaction should have one clear owner and one clear success path. Commit only after every required operation completes:

~~~php
try {
    $pdo->beginTransaction();

    $firstStatement->execute($firstValues);
    $secondStatement->execute($secondValues);

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

This example rethrows the original failure. It does not report success after a failed operation.

The inTransaction() check matters because a failure can happen before the transaction starts or after it has ended. Calling rollBack() without an active transaction can itself fail.

At an application boundary, catch an exception only when you can respond usefully: log the technical details securely, map known failures to suitable application errors, and avoid sending SQL or connection details to a visitor.

## 4. Keep the transaction small and complete

Validate input and perform checks that do not require a database before opening the transaction when possible. Then begin the transaction, perform the database reads or writes needed for the operation, check their outcomes, and commit.

~~~php
if ($amountInCents <= 0) {
    throw new InvalidArgumentException('The amount must be positive.');
}

try {
    $pdo->beginTransaction();

    // Read or update the records required for this operation.
    // Check that each expected operation succeeded.

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

Avoid waiting for a person, a slow HTTP request, file upload, or unrelated work while a transaction is open. Long transactions can hold locks, use database resources, and increase contention.

A transaction only covers operations performed by the database connection in that transaction. Sending an email, writing a file, or calling another service is not automatically undone by rollBack().

## 5. Example: transfer between two accounts

Assume account balances are stored as integer cents in a transactional table. Integer minor units avoid floating-point rounding problems for this simple money example. A real financial application also needs carefully designed currency, authorization, audit, concurrency, and regulatory rules.

The transaction should subtract from the sender only when that account has sufficient funds, then credit the recipient. Both updates must affect exactly one expected row:

~~~php
$amountInCents = 2500;
$senderId = 10;
$recipientId = 20;

if ($amountInCents <= 0) {
    throw new InvalidArgumentException('The amount must be positive.');
}

if ($senderId === $recipientId) {
    throw new InvalidArgumentException('Choose two different accounts.');
}

try {
    $pdo->beginTransaction();

    $debit = $pdo->prepare(
        'UPDATE accounts
         SET balance_cents = balance_cents - :debit_amount
         WHERE id = :sender_id
           AND balance_cents >= :minimum_balance'
    );

    $debit->execute([
        'debit_amount' => $amountInCents,
        'sender_id' => $senderId,
        'minimum_balance' => $amountInCents,
    ]);

    if ($debit->rowCount() !== 1) {
        throw new RuntimeException(
            'The sender account is missing or has insufficient funds.'
        );
    }

    $credit = $pdo->prepare(
        'UPDATE accounts
         SET balance_cents = balance_cents + :credit_amount
         WHERE id = :recipient_id'
    );

    $credit->execute([
        'credit_amount' => $amountInCents,
        'recipient_id' => $recipientId,
    ]);

    if ($credit->rowCount() !== 1) {
        throw new RuntimeException('The recipient account was not found.');
    }

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

If the sender update or recipient update fails, the transaction is rolled back and the sender is not left debited by itself. The conditional sender update checks the balance as part of the write, but exact concurrency behavior still depends on the database and its isolation rules.

The example treats a missing account and insufficient balance as the same internal failure. A real application should deliberately decide which outcomes can safely be shown to the caller, and should check that the caller is authorized to operate on both accounts.

## 6. Check row counts and business outcomes

A SQL statement can execute without throwing and still affect no rows. For example, an UPDATE may match no record because an ID does not exist or a condition was not met.

~~~php
$statement->execute($parameters);

if ($statement->rowCount() !== 1) {
    throw new RuntimeException('The expected record was not changed.');
}
~~~

Use rowCount() only when its meaning is appropriate for the statement and driver. For UPDATE or DELETE, it is often useful for checking affected rows, but driver settings can affect whether unchanged matched rows are counted. SELECT row counts are not portable; fetch rows or run a deliberate COUNT query instead.

Decide what the application should do for each result:

- A required row is missing.
- A conditional update affects zero rows.
- More rows change than the operation expected.
- A uniqueness or foreign-key constraint rejects the write.

If an outcome means the larger operation must not proceed, throw or return a failure while the transaction is still active so the work can be rolled back.

## 7. Configure and handle PDO errors

Set the error mode explicitly when creating a connection, even though exception mode is the default in PHP 8 and later. Explicit configuration helps the code communicate its intended behavior and remains clear when reading older examples or different environments.

~~~php
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

$pdo = new PDO($dsn, $username, $password, $options);
~~~

In exception mode, PDO reports many database failures by throwing PDOException. A connection failure also throws PDOException. Catch database failures at a boundary that can handle them rather than adding repetitive catch blocks around every statement.

~~~php
try {
    $result = performDatabaseOperation($pdo);
} catch (PDOException $exception) {
    error_log('Database operation failed: ' . $exception->getMessage());

    // Return a generic error through the application's normal response flow.
}
~~~

Do not expose the exception message, SQL statement, DSN, credentials, file paths, or stack trace in a public response. Those details can help an attacker understand the system. Keep detailed diagnostics in access-controlled logs and avoid logging secrets or sensitive personal data.

Do not swallow an exception and continue as if the database work succeeded. Either handle a known failure deliberately or let it reach a centralized exception handler.

## 8. Preserve the original failure if rollback fails

A rollback can itself fail, for example if the connection was lost. If both the operation and rollback fail, avoid hiding the original failure with a second exception:

~~~php
try {
    $pdo->beginTransaction();

    // Perform all required database work.

    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        try {
            $pdo->rollBack();
        } catch (Throwable $rollbackException) {
            error_log(
                'Rollback also failed: ' . $rollbackException->getMessage()
            );
        }
    }

    throw $exception;
}
~~~

This pattern preserves the original throwable while recording that rollback also failed. The connection may no longer be usable, so higher-level code should not keep issuing queries through it.

A connection failure during commit can leave the client uncertain whether the database committed before the connection broke. Do not blindly repeat a non-idempotent operation. Design operations that can be safely identified or deduplicated when retries are required.

## 9. Understand database-specific limits

PDO's transaction API is shared, but transaction details differ by driver and database:

- Some database engines or table types do not support transactional writes.
- Some databases implicitly commit when certain DDL statements run, such as CREATE TABLE or DROP TABLE. A later rollback cannot undo that implicit commit.
- Isolation levels, locks, deadlocks, and conflict behavior are database-specific.
- A PDO transaction begins only for the connection on which beginTransaction() was called.
- A transaction does not automatically span multiple database connections.

Keep schema changes such as creating or dropping tables out of ordinary application transactions unless the target database guarantees the behavior you require. Check the storage engine used by each table.

For concurrent updates, use database constraints and conditional writes where appropriate, and understand your database's locking and isolation rules. Transactions do not eliminate the need to design for concurrency.

## 10. Nested transactions and savepoints

PDO does not provide a portable nested transaction API. Calling beginTransaction() again while a transaction is already active is not a general way to create an inner transaction.

Choose one component to own the transaction boundary. Lower-level functions can perform their SQL work without starting or committing their own transaction, allowing the caller to group several functions together.

Some databases support savepoints, which allow part of a transaction to be rolled back while preserving the outer transaction. Savepoint syntax and behavior are database-specific. Use them only behind a carefully designed database abstraction when the application needs that behavior.

## 11. Retry transient failures carefully

Databases can report transient failures such as deadlocks or serialization conflicts. A retry may be appropriate, but only when the application can identify the failure and repeat the whole transaction safely.

A careful retry policy should:

- Limit the number of attempts.
- Use a short backoff between attempts.
- Retry only known transient database errors.
- Start a fresh transaction for each attempt.
- Ensure repeated operations are idempotent or protected by a unique request key.
- Avoid sending emails or other external effects before a successful commit.

Do not retry every PDOException. Invalid SQL, constraint violations, invalid input, missing permissions, and programming errors are not fixed by repeating the same operation.

If an operation must coordinate a database commit with an external message or email, consider a durable outbox design: write the intended event in the same database transaction, then let a separate process deliver it and track completion. A database rollback cannot undo a message already sent to another service.

## Common mistakes

### Committing before all related writes finish

A later failure cannot roll back changes after commit. Commit only after all required database operations have succeeded.

### Rolling back without checking transaction state

If beginTransaction() failed or the transaction already ended, rollBack() can throw. Check inTransaction() before conditional rollback in an error path.

### Catching and hiding an exception

Returning a success response after a failure leaves callers with false information. Log and handle the error, or rethrow it.

### Including DDL in a transaction without checking the database

Some databases implicitly commit around DDL. A rollback may not undo the schema change or preceding data writes.

### Assuming a transaction covers email or HTTP requests

Only the database work in that transaction is governed by it. Use after-commit handling or a durable outbox for external effects.

### Starting nested transactions with repeated beginTransaction() calls

PDO does not offer portable nested transactions. Let one layer own the transaction, or use database-specific savepoints intentionally.

### Retrying every database error

Only retry known transient failures, with a bounded policy and safe operation semantics.

## Key takeaways

- A transaction groups related database operations into a unit that can be committed or rolled back.
- Use PDO beginTransaction(), commit(), rollBack(), and inTransaction() consistently.
- Commit only after every required database operation and business check succeeds.
- Roll back in an error path and preserve the original failure if rollback also fails.
- A transaction cannot make unsupported tables transactional or undo every DDL operation.
- Transactions do not cover emails, files, or other external services.
- Check the database-specific behavior for row counts, isolation, locks, implicit commits, and retries.
- Keep error details out of public responses, and never report success after a failed operation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [100 - Prepared statements and safe database queries](100-prepared-statements-and-safe-database-queries.md) | [Q&A](../qnas/101-database-transactions-and-error-handling.md) | [Example](../examples/101-database-transactions-and-error-handling.md) | [102 - Filtering, sorting, and paginating database results](102-filtering-sorting-and-paginating-database-results.md) |
