# 099 - Connecting to databases with PDO: Example

## Goal

Connect to a local SQLite database through PDO, create a small table, add sample rows, fetch and display the results, count rows, and group multiple writes in a transaction.

## Question

How can a PHP script confirm that the SQLite PDO driver is available, connect with consistent settings, execute static SQL, fetch rows, and roll back a multi-step operation if it fails?

## Complete example

Create this structure:

~~~text
project/
└── pdo-demo.php
~~~

### 1. PHP program: pdo-demo.php

~~~php
<?php

declare(strict_types=1);

if (!in_array('sqlite', PDO::getAvailableDrivers(), true)) {
    fwrite(
        STDERR,
        "The PDO_SQLITE driver is not available in this PHP runtime." . PHP_EOL
    );
    exit(1);
}

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
    // This database exists only for this script run.
    $pdo = new PDO('sqlite::memory:', null, null, $options);
} catch (PDOException $exception) {
    error_log('PDO connection failed: ' . $exception->getMessage());
    fwrite(STDERR, "Could not connect to the lesson database." . PHP_EOL);
    exit(1);
}

try {
    $pdo->exec(
        'CREATE TABLE notes (
            id INTEGER PRIMARY KEY,
            title TEXT NOT NULL
        )'
    );

    // These fixed values are included only as local lesson data.
    $pdo->exec(
        "INSERT INTO notes (title) VALUES ('First PDO note')"
    );

    $pdo->exec(
        "INSERT INTO notes (title) VALUES ('Second PDO note')"
    );

    try {
        $pdo->beginTransaction();

        $pdo->exec(
            "INSERT INTO notes (title) VALUES ('Transaction note A')"
        );

        $pdo->exec(
            "INSERT INTO notes (title) VALUES ('Transaction note B')"
        );

        $pdo->commit();
    } catch (Throwable $exception) {
        if ($pdo->inTransaction()) {
            $pdo->rollBack();
        }

        throw $exception;
    }

    $statement = $pdo->query(
        'SELECT id, title FROM notes ORDER BY id'
    );

    while (($note = $statement->fetch()) !== false) {
        echo $note['id'] . ': ' . $note['title'] . PHP_EOL;
    }

    $statement->closeCursor();

    $countStatement = $pdo->query('SELECT COUNT(*) FROM notes');
    $count = (int) $countStatement->fetchColumn();
    $countStatement->closeCursor();

    echo 'Total notes: ' . $count . PHP_EOL;
} catch (Throwable $exception) {
    error_log('PDO lesson query failed: ' . $exception->getMessage());
    fwrite(STDERR, "The lesson database operation failed." . PHP_EOL);
    exit(1);
}
~~~

### 2. Run the program

Run it from the project directory:

~~~bash
php pdo-demo.php
~~~

Expected terminal output:

~~~text
1: First PDO note
2: Second PDO note
3: Transaction note A
4: Transaction note B
Total notes: 4
~~~

If the PDO_SQLITE driver is not enabled for this PHP runtime, the program prints a clear message to standard error and exits before connecting.

## How it works

1. **Driver check:** PDO::getAvailableDrivers() verifies that the current PHP runtime has the SQLite driver.
2. **Connection:** The sqlite::memory: DSN creates a temporary SQLite database for this run. The username and password are null because this local SQLite connection does not use them.
3. **Options:** Exception mode turns database failures into catchable PDOException errors. FETCH_ASSOC returns rows keyed by column name.
4. **Schema and fixed seed data:** exec() creates the table and inserts two static sample values. They are fixed lesson data, not user input.
5. **Transaction:** Two related inserts are committed together. If an operation throws before the commit, the catch block rolls back while a transaction is active.
6. **Fetching:** query() executes a static SELECT. fetch() returns one row at a time until it returns false.
7. **Count:** fetchColumn() reads the first column from the COUNT result. The explicit integer conversion gives the output a predictable format.
8. **Cursor cleanup:** closeCursor() releases the result resources when each statement is finished.
9. **Safe error reporting:** Detailed errors go to the script's error log; the terminal receives a generic message.

## Optional persistent SQLite database

The main example uses an in-memory database so it starts from clean data every run. To use a file instead, create a writable directory outside the public web root and change the DSN:

~~~php
$databasePath = __DIR__ . '/var/lesson.sqlite';
$pdo = new PDO('sqlite:' . $databasePath, null, null, $options);
~~~

Ensure the var directory exists and is writable by the PHP process. A file-backed database persists across runs, so do not assume its contents reset automatically.

## Optional MySQL connection shape

A server database uses its own driver-specific DSN and credentials. Read secrets from protected runtime configuration:

~~~php
$host = getenv('DB_HOST');
$database = getenv('DB_NAME');
$username = getenv('DB_USER');
$password = getenv('DB_PASSWORD');

if (
    $host === false
    || $database === false
    || $username === false
    || $password === false
) {
    throw new RuntimeException('Database configuration is incomplete.');
}

$dsn = "mysql:host={$host};dbname={$database};charset=utf8mb4";

$pdo = new PDO($dsn, $username, $password, $options);
~~~

This is only the connection shape; it requires the PDO_MYSQL driver, a reachable MySQL service, and correctly supplied environment settings.

## Important notes

- This example requires PDO and the PDO_SQLITE driver in the PHP runtime running the script.
- The in-memory database disappears when the script exits.
- The sample uses static SQL values only. Never concatenate request values into SQL. The next lesson covers prepared statements and parameter binding.
- Do not expose exception messages, DSNs, usernames, passwords, or database host details in a public response.
- Set a suitable fetch mode and error mode explicitly so application behavior is easy to understand.
- SQL syntax, DSNs, data conversion, transaction behavior, and connection options vary by database driver.
- Production credentials should come from protected runtime configuration, not source files or committed example values.
- This local CLI example has no HTTP response or user authentication. It demonstrates PDO mechanics, not a complete web application.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [098 - Relational databases and SQL fundamentals](../lessons/098-relational-databases-and-sql-fundamentals.md) | [Q&A](../qnas/099-connecting-to-databases-with-pdo.md) | Example | [100 - Prepared statements and safe database queries](../lessons/100-prepared-statements-and-safe-database-queries.md) |
