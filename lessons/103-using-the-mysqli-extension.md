# 103 - Using the MySQLi extension

## Learning goals

By the end of this lesson, you will be able to:

- Explain what MySQLi is and when it is an appropriate database API.
- Connect to MySQL with explicit error reporting and a connection character set.
- Use object-oriented MySQLi consistently.
- Run fixed SQL with query() and parameterized SQL with prepared statements.
- Bind MySQLi statement parameters with the correct type string.
- Fetch prepared-statement results with get_result() or bind_result().
- Use affected_rows and insert_id appropriately.
- Group related writes in a transaction.
- Handle errors and close statement and connection resources safely.

## 1. What is MySQLi?

MySQLi means MySQL Improved. It is a PHP extension for communicating with MySQL-compatible database servers. It provides both object-oriented and procedural APIs.

MySQLi is specifically for MySQL. It is not a general interface for SQLite or PostgreSQL. PDO is a separate PHP API that supports multiple database drivers, including PDO_MYSQL.

This lesson uses object-oriented MySQLi consistently. Procedural functions such as mysqli_connect() and mysqli_prepare() provide equivalent operations; choose one style for a codebase and follow it consistently.

Before using it, confirm the mysqli extension is enabled in the PHP runtime that runs your application. The web server and PHP command line can load different configurations.

## 2. Configure errors and connect

Enable strict MySQLi error reporting before creating the connection. This makes database failures throw mysqli_sql_exception, which can be handled through PHP exception flow.

~~~php
mysqli_report(MYSQLI_REPORT_ERROR | MYSQLI_REPORT_STRICT);

$host = getenv('DB_HOST');
$username = getenv('DB_USER');
$password = getenv('DB_PASSWORD');
$database = getenv('DB_NAME');

if (
    $host === false
    || $username === false
    || $password === false
    || $database === false
) {
    throw new RuntimeException('Database configuration is incomplete.');
}

$port = 3306;

$mysqli = new mysqli(
    $host,
    $username,
    $password,
    $database,
    $port
);

$mysqli->set_charset('utf8mb4');
~~~

Keep credentials in protected runtime configuration such as environment variables or a secret manager, not in committed PHP source. Use set_charset('utf8mb4') to configure the connection character set; do not rely on issuing a text query to set it.

Check the actual server address, database name, user permissions, network policy, and TLS configuration for the deployment environment. A local development connection and a production connection often need different settings.

## 3. Run fixed SQL with query()

Use query() for a fixed SQL statement that does not include request data:

~~~php
$result = $mysqli->query(
    'SELECT id, title
     FROM articles
     ORDER BY id'
);

while ($article = $result->fetch_assoc()) {
    echo $article['id']
        . ': '
        . $article['title']
        . PHP_EOL;
}

$result->free();
~~~

For a query that returns rows, query() returns a result object in strict exception mode. Fetch rows with methods such as fetch_assoc(), fetch_row(), or fetch_all(). Free the result when it is no longer needed, especially in long-running scripts.

For INSERT, UPDATE, or DELETE statements that do not return a result set, query() returns a success value. The connection's affected_rows property reports rows changed, deleted, inserted, or matched according to the statement and MySQL behavior.

Never concatenate a value from a request into query(). Use a prepared statement for changing values.

## 4. Prepare statements with placeholders

MySQLi prepared statements use positional question-mark markers. Each marker represents one data value:

~~~php
$statement = $mysqli->prepare(
    'SELECT id, title, status
     FROM articles
     WHERE author_id = ?
       AND status = ?
     ORDER BY id DESC'
);

$authorId = 42;
$status = 'published';

$statement->bind_param('is', $authorId, $status);
$statement->execute();
~~~

The first argument to bind_param() is a type string with one character for each marker:

| Type | Meaning |
| --- | --- |
| i | Integer |
| d | Float or double |
| s | String |
| b | Blob data sent in packets |

The type string length must match the number of placeholders. The variables passed after it must also match the number and order of markers.

bind_param() binds variables by reference. It reads their current values when execute() runs:

~~~php
$statement = $mysqli->prepare(
    'UPDATE articles
     SET status = ?
     WHERE id = ?'
);

$status = 'draft';
$articleId = 12;

$statement->bind_param('si', $status, $articleId);
$statement->execute();
~~~

Do not pass a temporary expression such as a function call where bind_param() expects a variable. Assign the value to a variable first.

Prepared statements protect values from changing SQL syntax. They do not validate business rules or determine whether the current user is allowed to access a row.

## 5. Fetch rows returned by a prepared statement

There are two common ways to read a prepared SELECT result.

### Use get_result() when mysqlnd is available

~~~php
$statement = $mysqli->prepare(
    'SELECT id, title
     FROM articles
     WHERE status = ?
     ORDER BY id'
);

$status = 'published';
$statement->bind_param('s', $status);
$statement->execute();

$result = $statement->get_result();

while ($article = $result->fetch_assoc()) {
    echo $article['id']
        . ': '
        . $article['title']
        . PHP_EOL;
}

$result->free();
$statement->close();
~~~

get_result() returns a mysqli_result that supports convenient array-based fetching. It is available only when PHP uses the mysqlnd driver.

### Use bind_result() for a result-binding approach

~~~php
$statement = $mysqli->prepare(
    'SELECT id, title
     FROM articles
     WHERE status = ?
     ORDER BY id'
);

$status = 'published';
$statement->bind_param('s', $status);
$statement->execute();

$statement->bind_result($id, $title);

while ($statement->fetch()) {
    echo $id . ': ' . $title . PHP_EOL;
}

$statement->close();
~~~

Bind one output variable for each selected column after execute() and before fetch(). The fetched values are assigned into those variables for each row.

Use one result approach for a statement; do not combine get_result() and store_result() for the same result.

## 6. Insert, update, and delete safely

Use placeholders for values in every kind of data statement:

~~~php
$insert = $mysqli->prepare(
    'INSERT INTO articles (author_id, title, status)
     VALUES (?, ?, ?)'
);

$authorId = 42;
$title = 'Learning MySQLi';
$status = 'draft';

$insert->bind_param('iss', $authorId, $title, $status);
$insert->execute();

$newArticleId = $mysqli->insert_id;
echo 'Created article ID: ' . $newArticleId . PHP_EOL;

$insert->close();
~~~

For UPDATE and DELETE, put all changeable values in placeholders and keep the appropriate access condition in the query:

~~~php
$update = $mysqli->prepare(
    'UPDATE articles
     SET status = ?
     WHERE id = ?
       AND author_id = ?'
);

$newStatus = 'published';
$articleId = 15;
$currentAuthorId = 42;

$update->bind_param(
    'sii',
    $newStatus,
    $articleId,
    $currentAuthorId
);
$update->execute();

if ($update->affected_rows !== 1) {
    throw new RuntimeException('The expected article was not updated.');
}

$update->close();
~~~

Including the owner condition can be part of an authorization check, but the complete application still needs an intentional permission model and safe response behavior.

A zero affected-row count can mean no row matched or, depending on statement settings and database behavior, that the stored value did not change. Interpret it according to the operation's expected result.

## 7. Build an IN list with one marker per value

One question mark represents one value, not a whole array. For a variable list, generate the marker count and bind each item:

~~~php
$articleIds = [4, 9, 15];

if ($articleIds === []) {
    $articles = [];
} else {
    $placeholders = implode(
        ', ',
        array_fill(0, count($articleIds), '?')
    );

    $statement = $mysqli->prepare(
        'SELECT id, title
         FROM articles
         WHERE id IN (' . $placeholders . ')
         ORDER BY id'
    );

    $types = str_repeat('i', count($articleIds));
    $statement->bind_param($types, ...$articleIds);
    $statement->execute();

    $result = $statement->get_result();
    $articles = $result->fetch_all(MYSQLI_ASSOC);

    $result->free();
    $statement->close();
}
~~~

The array values remain bound. The generated SQL contributes only a comma-separated set of question-mark markers. Handle an empty list explicitly rather than creating an invalid IN () clause or removing the filter.

## 8. Use transactions for related changes

MySQL transactions require a transactional storage engine for the tables involved. InnoDB is the usual choice for transactional application tables.

~~~php
$transactionStarted = false;

try {
    $mysqli->begin_transaction();
    $transactionStarted = true;

    $debit->execute();
    $credit->execute();

    $mysqli->commit();
    $transactionStarted = false;
} catch (Throwable $exception) {
    if ($transactionStarted) {
        try {
            $mysqli->rollback();
        } catch (Throwable $rollbackException) {
            error_log(
                'MySQLi rollback failed: '
                . $rollbackException->getMessage()
            );
        }
    }

    throw $exception;
}
~~~

Commit only after all required operations and outcome checks succeed. The flag prevents a rollback attempt if begin_transaction() failed and preserves the original failure if rollback also fails.

Some MySQL statements, especially certain DDL statements, can cause an implicit commit. A rollback cannot undo work that the server has already implicitly committed. Keep schema changes outside ordinary data transactions unless the database behavior is understood.

## 9. Handle errors without exposing internals

Strict error reporting lets application code catch mysqli_sql_exception or a broader Throwable at a suitable boundary:

~~~php
try {
    // Connect and perform the required database operations.
} catch (mysqli_sql_exception $exception) {
    error_log('MySQL operation failed: ' . $exception->getMessage());

    // Return a generic application error instead of database details.
}
~~~

Do not show SQL, database names, host details, file paths, credentials, or full exception messages to public visitors. Keep useful diagnostics in protected logs and avoid logging secrets or unnecessary personal data.

Do not catch an error and continue as if the query succeeded. Either handle a known failure deliberately or allow it to reach the application's centralized error handler.

## 10. Close resources and understand result buffering

Close prepared statements when their work is done:

~~~php
$statement->close();
$mysqli->close();
~~~

Free mysqli_result objects after fetching. PHP will normally release objects when they are no longer referenced, but explicit cleanup makes resource ownership clear and matters in long-running processes.

Buffered results consume client memory. For large result sets, process rows incrementally or use an appropriate unbuffered query strategy, then finish or free the result before issuing another query on that connection.

A prepared statement also uses server resources. Close it when finished, particularly in a long-running worker.

## 11. MySQLi and PDO at a glance

| Question | MySQLi | PDO |
| --- | --- | --- |
| Which databases can it connect to? | MySQL-compatible servers | Multiple databases through drivers |
| Placeholder styles | Positional question marks | Named or positional markers |
| Parameter binding | bind_param() with a type string and variables by reference | bindValue(), bindParam(), or execute() values |
| Fetching prepared results | get_result() with mysqlnd, or bind_result() | PDOStatement fetch methods |
| Transactions | begin_transaction(), commit(), rollback() | beginTransaction(), commit(), rollBack() |

Both APIs support prepared statements and transactions. Choose based on project needs and existing conventions; neither API removes the need for validation, authorization, or safe error handling.

## Common mistakes

### Putting request values into SQL strings

Prepared statements are the normal way to supply values. Manual escaping is not a substitute for parameter binding.

### Giving bind_param() the wrong type string

Use one type character and one variable for each placeholder, in the same order as the SQL markers.

### Passing a temporary expression to bind_param()

bind_param() requires variables by reference. Assign a computed value to a variable before binding.

### Using get_result() without mysqlnd

get_result() is mysqlnd-only. Use bind_result() when that driver capability is unavailable.

### Treating identifiers as placeholders

Table names, column names, and SQL keywords cannot be bound as values. Use a strict allowlist for any supported dynamic SQL structure.

### Assuming every MySQL table supports transactions

Check the storage engine. A transaction cannot provide rollback for non-transactional table changes.

### Returning database exception details to a visitor

Log details securely and send a generic application response.

## Key takeaways

- MySQLi is a MySQL-specific PHP extension with object-oriented and procedural APIs.
- Configure strict error reporting, runtime credentials, and the connection character set.
- Use query() only for fixed SQL without request values.
- Use prepared statements and bind_param() for changing values.
- Match the bind_param() type string and variable count to the placeholders.
- get_result() requires mysqlnd; bind_result() is an alternative.
- Use InnoDB tables and MySQLi transaction methods for related writes.
- Close statements and results, and keep database errors out of public output.
- Prepared statements do not replace validation or authorization.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [102 - Filtering, sorting, and paginating database results](102-filtering-sorting-and-paginating-database-results.md) | [Q&A](../qnas/103-using-the-mysqli-extension.md) | [Example](../examples/103-using-the-mysqli-extension.md) | [104 - Schema design and database migrations](104-schema-design-and-database-migrations.md) |
