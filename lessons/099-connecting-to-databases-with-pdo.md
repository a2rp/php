# 099 - Connecting to databases with PDO

## Learning goals

By the end of this lesson, you will be able to:

- Explain what PDO and a PDO database driver do.
- Check that the required PDO driver is available to the PHP runtime.
- Build a data source name (DSN) and open a database connection.
- Configure consistent error handling and a useful fetch mode.
- Read results from a PDO statement and understand when to use query() or exec().
- Handle transactions and release database resources deliberately.
- Keep credentials and database errors out of public responses.

## 1. What PDO provides

PHP Data Objects (PDO) is a common interface for connecting to relational database systems. PDO itself is not a database server and does not install every database driver.

A PHP installation needs:

1. The PDO extension.
2. A PDO driver for the database, such as PDO_SQLITE, PDO_MYSQL, or PDO_PGSQL.
3. A reachable database and credentials if the selected database requires them.

PDO gives PHP code a consistent connection and statement API across drivers. SQL syntax and some behaviors still depend on the database engine. PDO is not an ORM: you still write SQL and design the database schema.

## 2. Check which drivers are available

Use PDO::getAvailableDrivers() to see which drivers the current PHP runtime can use:

~~~php
var_dump(PDO::getAvailableDrivers());
~~~

The CLI and web server may load different php.ini files or extensions. If a driver appears in the terminal but not in a web request, inspect the configuration used by that runtime. Avoid publishing a phpinfo() page on a public server.

If the required driver is missing, enable or install the matching extension for the PHP version and restart the relevant PHP process. Installing PDO alone may not install the database-specific driver.

## 3. Build a DSN for the selected database

A Data Source Name (DSN) tells PDO which driver and connection details to use. Its format depends on that driver.

SQLite can use a file or an in-memory database:

~~~php
$fileDsn = 'sqlite:' . __DIR__ . '/var/app.sqlite';
$memoryDsn = 'sqlite::memory:';
~~~

A MySQL connection typically includes a host, database name, and character set:

~~~php
$dsn = 'mysql:host=127.0.0.1;dbname=app;charset=utf8mb4';
~~~

A PostgreSQL DSN has its own format:

~~~php
$dsn = 'pgsql:host=127.0.0.1;port=5432;dbname=app';
~~~

The exact DSN options are driver-specific. Keep user credentials as separate constructor arguments instead of inserting them into a DSN string.

An in-memory SQLite database is temporary and normally disappears when its connection closes. A file-backed SQLite database persists data, so its path and file permissions need to be managed deliberately.

## 4. Create a connection with explicit options

Pass the DSN, username, password, and options to PDO:

~~~php
$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

$pdo = new PDO($dsn, $username, $password, $options);
~~~

Exception mode makes database failures raise PDOException instead of requiring every call to check a false return value. Set it explicitly so the code's intended behavior is clear across environments.

PDO::FETCH_ASSOC returns each fetched row as an array indexed by column names. This avoids the duplicate numeric and named keys returned by PDO::FETCH_BOTH, the default fetch mode.

Connection failure can throw PDOException during construction regardless of the later statement error mode. Handle connection failure at the application boundary and show users a generic service error rather than raw connection details.

## 5. Keep credentials in runtime configuration

Do not commit production usernames, passwords, or private connection details to the repository. Supply them through the deployment environment or a protected secrets manager.

For example, verify that required environment values exist before connecting:

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

$pdo = new PDO($dsn, $username, $password, [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);
~~~

This example checks for missing values, but real configuration should also validate host and database settings. Do not print environment values or include them in exception messages returned to a visitor.

Use an application database account with only the permissions the application needs. Connection encryption and certificate settings depend on the database driver and deployment; configure them according to that driver's documentation when traffic crosses an untrusted network.

## 6. Run a static query and fetch rows

PDO::query() prepares and executes a SQL statement that has no parameters. It returns a PDOStatement whose rows can be fetched.

~~~php
$sql = '
    SELECT id, email, display_name
    FROM customers
    ORDER BY id
';

$statement = $pdo->query($sql);

while (($customer = $statement->fetch()) !== false) {
    echo $customer['id'] . ': ';
    echo $customer['display_name'] . PHP_EOL;
}

$statement->closeCursor();
~~~

fetch() returns one row at a time. When there are no more rows, it returns false. Iterating this way avoids building an array of every row in memory.

For a small, bounded result set, fetchAll() can be convenient:

~~~php
$customers = $pdo
    ->query('SELECT id, email, display_name FROM customers ORDER BY id')
    ->fetchAll();

foreach ($customers as $customer) {
    echo $customer['email'] . PHP_EOL;
}
~~~

Do not fetch an unbounded result set into memory just for convenience. Use pagination or iteration for larger results.

## 7. Use exec() for statements without result rows

PDO::exec() executes a statement that does not return a result set, such as a fixed schema command or a fixed update:

~~~php
$affectedRows = $pdo->exec(
    "UPDATE customers SET display_name = 'Ada Lovelace' WHERE id = 1"
);
~~~

exec() returns the number of affected rows, or false when error handling is not configured to throw. The meaning of an affected-row count can vary by database and statement. It is not a substitute for checking the business outcome the application requires.

Do not use exec() with SQL assembled from request data. For queries with dynamic values, use prepared statements. The next chapter focuses on preparing and executing safe parameterized queries.

## 8. Understand statements and result values

PDO returns database values according to the driver and its configuration. Do not assume every driver returns numeric columns as PHP integers or every boolean as a PHP boolean. Validate and convert values at the application boundary when the code needs a specific PHP type.

Use clear aliases in SQL when joined tables have repeated column names:

~~~sql
SELECT
    orders.id AS order_id,
    customers.id AS customer_id,
    customers.display_name
FROM orders
INNER JOIN customers
    ON customers.id = orders.customer_id;
~~~

If a statement has a result set, fetch it or close its cursor before reusing the connection for another operation. Some drivers cannot run another query while an earlier result is still active.

Do not rely on rowCount() to count SELECT results; behavior for result sets varies by driver. Fetch rows or issue a deliberate COUNT query when a count is needed.

## 9. Use transactions for related changes

PDO can begin, commit, and roll back a transaction when the selected driver and storage engine support transactions.

~~~php
try {
    $pdo->beginTransaction();

    $pdo->exec(
        'UPDATE accounts
         SET balance_cents = balance_cents - 500
         WHERE id = 1'
    );

    $pdo->exec(
        'UPDATE accounts
         SET balance_cents = balance_cents + 500
         WHERE id = 2'
    );

    $pdo->commit();
} catch (Throwable $error) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $error;
}
~~~

In an application, check that the expected accounts exist and that the first account has enough funds before committing. Parameterized queries should be used for values that come from the application or request; the next lesson covers them.

Transactions are useful when a set of changes must succeed or fail together. Transaction support depends on the database engine, table type, and configuration. Some databases implicitly commit certain schema changes, so do not assume every operation can be rolled back.

Catch and handle the failure at the appropriate application boundary. Do not swallow an exception and report success after rolling back.

## 10. Handle errors without exposing internals

A PDOException may contain SQL state, driver messages, database names, and other operational details. These details are useful to a protected log but should not be copied into a public response.

~~~php
try {
    $pdo = new PDO($dsn, $username, $password, $options);
} catch (PDOException $exception) {
    error_log('Database connection failed: ' . $exception->getMessage());

    http_response_code(500);
    echo 'The service is temporarily unavailable.';
    exit;
}
~~~

In a larger application, a central exception handler should log diagnostic information and return a consistent public error response. Redact secrets and personal data from logs. Do not expose raw SQL errors in production.

## 11. Manage connection and statement lifetimes

A PDO connection remains available while the object is referenced. In a normal request-based application, let the connection be owned by a service or request scope and release it when that work is complete.

A statement can keep a cursor and database resources open. Fetch the needed rows or call closeCursor() when appropriate. For very large result sets, iterate rather than using fetchAll().

Persistent connections are an optional driver feature. They can change connection reuse and state assumptions. Do not enable them as a default performance fix; first measure the application and understand how the database resets session state.

## 12. Avoid common connection mistakes

### Assuming PDO automatically supplies a driver

PDO needs the specific driver for the chosen database. Confirm the driver exists in the same PHP runtime that runs the application.

### Putting credentials in source code

Secrets in source control can remain in history even after deletion. Read credentials from protected runtime configuration and rotate exposed secrets.

### Returning database errors to visitors

Raw errors can reveal internal names and query details. Log them securely and return a generic response.

### Fetching every row into memory

fetchAll() is convenient for small bounded results but can exhaust memory for large queries. Iterate or paginate.

### Using query() with dynamic values

query() does not accept bound placeholders as an execution mechanism. Use prepare() and execute() for dynamic values; the next chapter explains this pattern.

### Assuming SQL and PDO behavior are identical across drivers

DSNs, SQL syntax, type conversion, transactions, affected-row counts, and some attributes depend on the driver and database. Check the selected engine's behavior.

## Key takeaways

- PDO provides a common PHP interface, while database-specific drivers provide actual connections.
- Check the driver in the same PHP runtime that will run the code.
- Build a driver-specific DSN and pass credentials separately.
- Set exception mode and a clear fetch mode explicitly.
- Use query() for static SQL that has no input values; use exec() for statements without result rows.
- Fetch bounded results, release cursors, and do not rely on rowCount() for SELECT counts.
- Use a transaction when multiple changes must succeed together.
- Keep secrets and raw database diagnostics out of public responses.
- Use prepared statements for dynamic values; the next chapter covers them in detail.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [098 - Relational databases and SQL fundamentals](098-relational-databases-and-sql-fundamentals.md) | [Q&A](../qnas/099-connecting-to-databases-with-pdo.md) | [Example](../examples/099-connecting-to-databases-with-pdo.md) | [100 - Prepared statements and safe database queries](100-prepared-statements-and-safe-database-queries.md) |
