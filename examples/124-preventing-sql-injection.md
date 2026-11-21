# 124 - Preventing SQL injection: Example

## Question

Write a PHP user lookup that accepts an email address without letting the input change the SQL query. Show that a value containing SQL-shaped text is handled as a literal, and safely support a requested sort choice.

## Why concatenating values is unsafe

This pattern is vulnerable because the request value becomes part of the SQL text:

~~~php
$sql = "SELECT id, email FROM users WHERE email = '" . $email . "'";
$rows = $pdo->query($sql)->fetchAll(PDO::FETCH_ASSOC);
~~~

For example, an input such as `' OR '1'='1' -- ` can change the meaning of a query built this way. Do not run this version. Replace value concatenation with a prepared statement.

## Complete example

Save the following as a PHP file and run it from the command line in an environment with PDO SQLite enabled. The database is created in memory, so the example does not need a database server or write any records to disk.

~~~php
<?php

declare(strict_types=1);

function findUserByEmail(PDO $pdo, string $email): ?array
{
    $statement = $pdo->prepare(
        'SELECT id, email FROM users WHERE email = :email'
    );

    $statement->execute([
        'email' => $email,
    ]);

    $user = $statement->fetch(PDO::FETCH_ASSOC);

    return $user === false ? null : $user;
}

/**
 * @return list<array{id: int, email: string}>
 */
function findUsersInSafeOrder(
    PDO $pdo,
    string $requestedSort,
    string $requestedDirection
): array {
    $allowedColumns = [
        'id' => 'id',
        'email' => 'email',
    ];

    $allowedDirections = [
        'asc' => 'ASC',
        'desc' => 'DESC',
    ];

    $column = $allowedColumns[$requestedSort] ?? 'id';
    $directionKey = strtolower($requestedDirection);
    $direction = $allowedDirections[$directionKey] ?? 'ASC';

    $sql = 'SELECT id, email FROM users ORDER BY '
        . $column . ' ' . $direction;

    return $pdo->query($sql)->fetchAll(PDO::FETCH_ASSOC);
}

$pdo = new PDO('sqlite::memory:');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$pdo->exec(
    'CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        email TEXT NOT NULL UNIQUE
    )'
);

$insert = $pdo->prepare(
    'INSERT INTO users (email) VALUES (:email)'
);

$insert->execute(['email' => 'alice@example.test']);
$insert->execute(['email' => 'bob@example.test']);

$knownUser = findUserByEmail($pdo, 'alice@example.test');
$sqlShapedEmail = "' OR '1'='1' -- ";
$unexpectedUser = findUserByEmail($pdo, $sqlShapedEmail);

$orderedUsers = findUsersInSafeOrder(
    $pdo,
    'email; DROP TABLE users --',
    'ASC'
);

$userCount = (int) $pdo
    ->query('SELECT COUNT(*) FROM users')
    ->fetchColumn();

if ($knownUser !== null) {
    echo 'Known address lookup: FOUND (id=' . $knownUser['id'] . ')' . PHP_EOL;
} else {
    echo "Known address lookup: NO MATCH" . PHP_EOL;
}

if ($unexpectedUser === null) {
    echo 'SQL-shaped value treated literally: NO MATCH' . PHP_EOL;
} else {
    echo 'SQL-shaped value treated literally: MATCH' . PHP_EOL;
}

echo 'Unknown sort key fallback: first row id='
    . $orderedUsers[0]['id'] . PHP_EOL;
echo 'User records after all lookups: ' . $userCount . PHP_EOL;
~~~

The functions use only fixed query text and bound values. The sort key and direction are selected from arrays controlled by the application. The unexpected sort string does not enter the SQL query; it selects the safe default column.

## Expected output

~~~text
Known address lookup: FOUND (id=1)
SQL-shaped value treated literally: NO MATCH
Unknown sort key fallback: first row id=1
User records after all lookups: 2
~~~

The SQL-shaped value does not make the lookup return both records. The unknown sort key does not become SQL syntax, and both stored rows remain available afterward.

## Explanation

### The lookup binds the email

The query contains a named placeholder for the email. The call to execute supplies the email as a separate value. If that value contains quotes or SQL operators, the database compares it as data; it does not parse it as part of the query.

The function returns one matching row or null. The caller does not need to inspect or sanitize SQL metacharacters to preserve the query structure.

### The sort choice uses an allowlist

Placeholders cannot represent column names or sort directions. The function therefore maps the requested key to one of two application-owned names and maps the direction to ASC or DESC. Any unknown value falls back to a known choice.

Only those fixed strings are inserted into the ORDER BY clause. If the query also had a user-supplied filter value, that filter would still use a placeholder.

### The test checks behavior, not input spelling

The SQL-shaped email is deliberately passed to the same lookup function as an ordinary value. It matches no row, and the row count stays at two. This demonstrates that the value did not change the filter or execute as SQL.

The sample uses in-memory SQLite to make the behavior easy to inspect. In a real application, use its configured database driver and add integration tests for that driver.

## What this example demonstrates

- User-controlled values belong in prepared-statement parameters.
- A placeholder represents a value, not SQL syntax.
- Dynamic identifiers and directions must come from a fixed allowlist.
- A suspicious-looking value should remain a literal value.
- SQL injection prevention does not replace authorization checks or least-privilege database permissions.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [123 - Secure coding and threat modeling](../lessons/123-secure-coding-and-threat-modeling.md) | [Q&A](../qnas/124-preventing-sql-injection.md) | [Lesson](../lessons/124-preventing-sql-injection.md) | [125 - Preventing cross-site scripting](../lessons/125-preventing-cross-site-scripting.md) |