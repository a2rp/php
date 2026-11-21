# 101 - Database transactions and error handling: Example

## Goal

Model a transfer between two accounts using PDO and SQLite. The first transfer succeeds and commits. A second transfer fails because its recipient does not exist, so its earlier debit is rolled back.

## Question

How can a PHP application ensure that a transfer's debit and credit succeed together, and how can it verify that a failed transfer did not leave a partial balance change?

## Complete example

Create a file named transaction-demo.php:

~~~text
project/
└── transaction-demo.php
~~~

~~~php
<?php

declare(strict_types=1);

function formatCents(int $cents): string
{
    $wholeUnits = intdiv($cents, 100);
    $fraction = str_pad(
        (string) ($cents % 100),
        2,
        '0',
        STR_PAD_LEFT
    );

    return $wholeUnits . '.' . $fraction;
}

/**
 * Transfer a positive amount in cents from one account to another.
 */
function transfer(
    PDO $pdo,
    int $senderId,
    int $recipientId,
    int $amountInCents
): void {
    if ($amountInCents <= 0) {
        throw new InvalidArgumentException('The amount must be positive.');
    }

    if ($senderId === $recipientId) {
        throw new InvalidArgumentException(
            'Choose two different accounts.'
        );
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
            throw new DomainException(
                'Sender account is missing or has insufficient funds.'
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
            throw new DomainException('Recipient account not found.');
        }

        $pdo->commit();
    } catch (Throwable $exception) {
        if ($pdo->inTransaction()) {
            try {
                $pdo->rollBack();
            } catch (Throwable $rollbackException) {
                error_log(
                    'Transaction rollback failed: '
                    . $rollbackException->getMessage()
                );
            }
        }

        throw $exception;
    }
}

/**
 * Return account balances keyed by account ID.
 *
 * @return array<int, int>
 */
function readBalances(PDO $pdo): array
{
    $statement = $pdo->query(
        'SELECT id, balance_cents
         FROM accounts
         ORDER BY id'
    );

    $balances = [];

    while (($row = $statement->fetch()) !== false) {
        $balances[(int) $row['id']] = (int) $row['balance_cents'];
    }

    $statement->closeCursor();

    return $balances;
}

function printBalances(string $label, array $balances): void
{
    echo $label
        . ': account 1 = '
        . formatCents($balances[1])
        . '; account 2 = '
        . formatCents($balances[2])
        . PHP_EOL;
}

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
    // This database is temporary and exists only for this script run.
    $pdo = new PDO('sqlite::memory:', null, null, $options);

    $pdo->exec(
        'CREATE TABLE accounts (
            id INTEGER PRIMARY KEY,
            owner TEXT NOT NULL,
            balance_cents INTEGER NOT NULL CHECK (balance_cents >= 0)
        )'
    );

    $seed = $pdo->prepare(
        'INSERT INTO accounts (id, owner, balance_cents)
         VALUES (:id, :owner, :balance_cents)'
    );

    $seedAccounts = [
        ['id' => 1, 'owner' => 'Ari', 'balance_cents' => 10000],
        ['id' => 2, 'owner' => 'Bo', 'balance_cents' => 5000],
    ];

    foreach ($seedAccounts as $account) {
        $seed->execute([
            'id' => $account['id'],
            'owner' => $account['owner'],
            'balance_cents' => $account['balance_cents'],
        ]);
    }

    transfer($pdo, 1, 2, 2500);
    echo 'First transfer committed.' . PHP_EOL;
    printBalances('Balances after commit', readBalances($pdo));

    try {
        // Account 999 does not exist. The debit must be rolled back.
        transfer($pdo, 1, 999, 1000);
        echo 'Second transfer committed.' . PHP_EOL;
    } catch (DomainException $exception) {
        echo 'Second transfer rolled back: '
            . $exception->getMessage()
            . PHP_EOL;
    }

    printBalances('Balances after rollback', readBalances($pdo));
} catch (PDOException $exception) {
    // Detailed diagnostics stay in the protected log.
    error_log('Transaction example failed: ' . $exception->getMessage());
    fwrite(STDERR, 'The database example could not be completed.' . PHP_EOL);
    exit(1);
} catch (Throwable $exception) {
    error_log('Unexpected transaction example failure: ' . $exception->getMessage());
    fwrite(STDERR, 'The transaction example could not be completed.' . PHP_EOL);
    exit(1);
}
~~~

## Run the example

The PHP runtime must have PDO and the PDO_SQLITE driver enabled. Run the script from its directory:

~~~sh
php transaction-demo.php
~~~

Expected output:

~~~text
First transfer committed.
Balances after commit: account 1 = 75.00; account 2 = 75.00
Second transfer rolled back: Recipient account not found.
Balances after rollback: account 1 = 75.00; account 2 = 75.00
~~~

The second attempt debits account 1 before it discovers that account 999 is missing. The transaction rolls that debit back, so the final balance stays at 75.00 for account 1.

## How it works

1. **Temporary database:** PDO connects to an in-memory SQLite database, creates the table, and inserts two fixed sample accounts.
2. **Validation:** transfer() rejects a non-positive amount and a transfer to the same account before opening a transaction.
3. **Start:** beginTransaction() starts the unit of database work.
4. **Conditional debit:** The UPDATE subtracts only when the sender exists and has at least the requested balance. The amount is bound as data.
5. **Check outcomes:** rowCount() verifies that the expected account row changed. A missing sender, insufficient balance, or missing recipient raises a domain error.
6. **Credit and commit:** The recipient is credited only after the debit check succeeds. The transaction commits only after both updates succeed.
7. **Rollback:** Any throwable before a successful commit triggers a rollback if PDO reports that a transaction is active. If rollback itself fails, the rollback problem is logged and the original failure is rethrown.
8. **Safe error response:** The top-level handler logs diagnostic details and writes a generic message to standard error instead of exposing the exception to a public visitor.
9. **Balance display:** Balances are stored and formatted as integer cents to avoid floating-point arithmetic in this teaching example.

## Important notes

- This is a local learning example, not a production money-transfer system.
- A production transfer also needs authorization for both accounts, audit history, appropriate currency handling, concurrency design, and carefully reviewed failure semantics.
- PDO transactions only cover database operations on the connection that started the transaction.
- Verify that the production database and its account table use transactional storage.
- Some databases implicitly commit around DDL. This example creates its table before starting either transfer.
- Transaction isolation, locking, and affected-row behavior can differ by database and driver.
- A connection failure during commit can leave the result uncertain. Do not blindly retry a non-idempotent transfer; production systems need an idempotency or deduplication design.
- Emails and external API calls cannot be rolled back by this database transaction. Use after-commit processing or a durable outbox when external effects must be coordinated.
- The example does not retry deadlocks or other transient failures.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [100 - Prepared statements and safe database queries](../lessons/100-prepared-statements-and-safe-database-queries.md) | [Q&A](../qnas/101-database-transactions-and-error-handling.md) | Example | [102 - Filtering, sorting, and paginating database results](../lessons/102-filtering-sorting-and-paginating-database-results.md) |
