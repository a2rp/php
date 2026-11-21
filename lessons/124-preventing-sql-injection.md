# 124 - Preventing SQL injection

SQL injection happens when untrusted input changes the meaning of a database query. Prevent it by keeping SQL instructions separate from values supplied by users or other external sources.

This chapter explains how to recognize unsafe query construction, use prepared statements with PDO and MySQLi, handle query parts that placeholders cannot represent, and verify the protection across an application.

## Learning goals

By the end of this chapter, you should be able to:

- Explain how SQL injection changes query meaning.
- Replace concatenated values with prepared statements.
- Use named and positional placeholders correctly.
- Safely choose dynamic column names and sort directions from fixed allowlists.
- Handle LIKE searches and variable-length IN lists.
- Recognize why validation, escaping, ORMs, and stored procedures do not replace safe query construction.
- Add practical tests and database permissions that reduce risk.

## How SQL injection happens

A query contains two different kinds of information:

1. **SQL instructions**, such as which table to read and how rows should be filtered.
2. **Data values**, such as an email address submitted in a form.

The database should interpret the first as instructions and the second only as data. Injection becomes possible when the application builds SQL by joining an instruction string with untrusted text.

Consider this unsafe example:

~~~php
$email = $_POST['email'] ?? '';

$sql = "SELECT id, email FROM users WHERE email = '" . $email . "'";
$result = $pdo->query($sql);
~~~

The value in $email is inserted into the SQL text before the database parses it. A carefully crafted value may escape the intended string and add SQL syntax. The exact effect depends on the query, database, driver, and application, but it can expose records, change data, or bypass an intended filter.

Treat data from all of these sources as untrusted:

- Form fields, query parameters, and route parameters.
- Cookies and request headers.
- Uploaded files and imported data.
- Values read from a database if a user could have influenced them.
- Data returned by another service.

A value does not become safe merely because it came from a logged-in user or an earlier database write.

## The main defense: prepared statements

A prepared statement gives the database a query template and supplies values separately. A placeholder marks where one complete value belongs.

With PDO:

~~~php
$email = $_POST['email'] ?? '';

$statement = $pdo->prepare(
    'SELECT id, email FROM users WHERE email = :email'
);

$statement->execute([
    'email' => $email,
]);

$user = $statement->fetch(PDO::FETCH_ASSOC);
~~~

The query structure remains fixed. The supplied email is treated as a value, even if it contains quote marks or text that resembles SQL syntax.

A complete request should also handle the result and errors appropriately:

~~~php
try {
    $statement = $pdo->prepare(
        'SELECT id, email FROM users WHERE email = :email'
    );
    $statement->execute(['email' => $email]);

    $user = $statement->fetch(PDO::FETCH_ASSOC);

    if ($user === false) {
        $user = null;
    }
} catch (PDOException $exception) {
    error_log($exception->getMessage());
    http_response_code(500);
    $user = null;
}
~~~

Configure PDO to report database errors as exceptions when creating the connection. Show users a generic response; keep diagnostic details in access-controlled logs, and do not log passwords, tokens, or other secrets.

## Named and positional placeholders

PDO supports named placeholders:

~~~php
$statement = $pdo->prepare(
    'SELECT id, email FROM users WHERE email = :email'
);
$statement->execute(['email' => $email]);
~~~

It also supports positional placeholders:

~~~php
$statement = $pdo->prepare(
    'SELECT id, email FROM users WHERE email = ?'
);
$statement->execute([$email]);
~~~

Use one placeholder style in a statement. Do not mix named markers and question marks in the same query. Keep the order of values in the execute array aligned with positional markers.

A placeholder represents a complete data value. Do not put quotes around it:

~~~php
// Correct
$statement = $pdo->prepare(
    'SELECT id FROM users WHERE email = :email'
);

// Incorrect: the marker is inside a quoted SQL string
$statement = $pdo->prepare(
    "SELECT id FROM users WHERE email = ':email'"
);
~~~

Do not substitute a value into the query yourself before calling prepare. Preparing a query after concatenating untrusted text does not undo the unsafe construction.

## Placeholders cannot stand for SQL structure

A placeholder cannot represent a table name, column name, SQL keyword, operator, or sort direction. This does not work as a way to choose a column:

~~~php
$statement = $pdo->prepare(
    'SELECT id, email FROM users ORDER BY :column'
);
$statement->execute(['column' => $requestedColumn]);
~~~

The marker is a value, not SQL syntax. The database will not treat the supplied value as a column identifier.

When users can choose among query structures, map their choice to fixed strings that the application owns:

~~~php
$sortColumns = [
    'newest' => 'created_at',
    'name' => 'display_name',
];

$sortDirections = [
    'ascending' => 'ASC',
    'descending' => 'DESC',
];

$column = $sortColumns[$requestedSort] ?? 'created_at';
$direction = $sortDirections[$requestedDirection] ?? 'DESC';

$sql = 'SELECT id, display_name FROM users ORDER BY '
    . $column . ' ' . $direction;

$statement = $pdo->query($sql);
~~~

Only the fixed allowlist values enter the SQL structure. Never add a raw request value to the query. Use placeholders for the ordinary data values in the same query.

For numeric values such as a page size, validate the permitted range and bind the value with the appropriate integer type when the driver requires it:

~~~php
$pageSize = filter_var(
    $_GET['limit'] ?? 20,
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 100]]
);

if ($pageSize === false) {
    $pageSize = 20;
}

$statement = $pdo->prepare(
    'SELECT id, display_name FROM users LIMIT :limit'
);
$statement->bindValue(':limit', $pageSize, PDO::PARAM_INT);
$statement->execute();
~~~

Driver behavior for particular SQL features can differ. Check the database driver used by the application and exercise the query in integration tests.

## Search patterns with LIKE

Bind the whole search pattern as a value. Build the pattern in PHP instead of putting the placeholder inside SQL quotes:

~~~php
$pattern = '%' . $searchTerm . '%';

$statement = $pdo->prepare(
    'SELECT id, display_name FROM users WHERE display_name LIKE :pattern'
);
$statement->execute(['pattern' => $pattern]);
~~~

The percent signs above are search wildcards. Binding prevents the term from becoming SQL syntax, but percent signs and underscores in the term can still act as LIKE wildcards. If the product requirement is to search for those characters literally, escape wildcard characters according to the database's LIKE rules and add an appropriate ESCAPE clause. That is a search-semantics concern separate from SQL injection.

Do not write LIKE '%:term%': a placeholder is not expanded inside a quoted SQL string.

## Variable-length IN lists

One placeholder represents one value. It does not expand an array:

~~~php
// This does not bind one placeholder to every value in the array.
$statement->execute(['ids' => $ids]);
~~~

Create one marker for each validated item and bind each item separately:

~~~php
$ids = [4, 9, 15];

if ($ids === []) {
    $rows = [];
} else {
    $markers = implode(', ', array_fill(0, count($ids), '?'));
    $statement = $pdo->prepare(
        'SELECT id, email FROM users WHERE id IN (' . $markers . ')'
    );

    $statement->execute($ids);
    $rows = $statement->fetchAll(PDO::FETCH_ASSOC);
}
~~~

The generated query fragment contains only a number of fixed question-mark markers determined by the list length. The actual IDs remain bound values. Decide explicitly what an empty list means rather than generating invalid SQL such as IN (). Apply reasonable limits to list length so one request cannot create an excessive query.

## MySQLi prepared statements

MySQLi also supports prepared statements. This example uses positional markers:

~~~php
$statement = $mysqli->prepare(
    'SELECT id, email FROM users WHERE email = ?'
);

$statement->bind_param('s', $email);
$statement->execute();

$result = $statement->get_result();
$user = $result->fetch_assoc();
~~~

The type string passed to bind_param describes the values: s for string, i for integer, d for float, and b for binary data. MySQLi binding uses variables by reference, so keep the variables available through execution. Handle statement failures using the application's configured error mode and error-handling policy.

Choose PDO or MySQLi based on the application's database needs and conventions. The security property comes from separating SQL structure from values, not from the library name.

## Why other techniques are not enough

### Validation is useful but does not replace binding

Validate values to enforce the application's rules: a page number may need to be a positive integer, and a sort choice may need to be one of a few supported options. Validation reduces invalid input and helps the application behave predictably.

Still bind values, including values that passed validation. A value can be valid for the product and still contain characters that would be unsafe if concatenated into SQL.

### Manual escaping is fragile

Avoid building query strings with manual quote escaping, addslashes(), or a database-specific escaping function as the primary defense. Correct escaping depends on the SQL context, character set, driver, and database behavior. A missed case can restore the vulnerability.

Prepared statements are the normal way to pass values. Escaping for HTML output is a different operation and must happen at the point where data is rendered.

### ORMs and query builders need safe use

An ORM or query builder can provide parameterized methods, but unsafe raw-query methods can still concatenate input. Inspect how raw fragments are constructed, and use the library's parameter-binding feature for values. A library does not make an interpolated string safe automatically.

### Stored procedures can also be vulnerable

A stored procedure is not automatically safe. Its body may construct and execute dynamic SQL from concatenated values. Parameterize values inside the procedure and review any dynamic SQL just as carefully as application queries.

## Least privilege and safe error handling

Use a database account dedicated to the application and grant only the permissions it needs. A read-only feature should not use an account that can drop tables or modify unrelated data. Separate accounts or roles can limit the impact of a mistake or compromise.

Do not show raw database errors to visitors. Error text can reveal query details, schema names, file paths, or environment information. Return a suitable generic response, log protected diagnostics, and ensure the application does not continue as if a failed security check succeeded.

Prepared statements prevent values from changing SQL syntax. They do not decide whether a user is allowed to access a particular record. Authorization must still be enforced, for example by constraining a lookup to the authenticated user's permitted records.

## A practical review checklist

When reviewing a database feature:

1. Find every query that uses request data or data influenced by a request.
2. Replace concatenated values with prepared statements.
3. Check dynamic identifiers, sort choices, and SQL fragments against fixed allowlists.
4. Handle empty collections and limits for generated placeholder lists.
5. Validate business rules and authorize access independently of SQL parameterization.
6. Review ORM raw queries, stored procedures, and database functions for dynamic SQL.
7. Test that quote marks and SQL-shaped text are handled as ordinary data.
8. Confirm errors do not disclose database internals and logs do not contain secrets.
9. Confirm the database account has only the permissions the application needs.

## Key takeaways

- SQL injection occurs when untrusted text changes SQL instructions.
- Use prepared statements for every untrusted data value.
- Placeholders represent values, not table names, column names, operators, or keywords.
- Choose dynamic query structure only from fixed application-owned allowlists.
- Validation, ORMs, and stored procedures do not replace safe parameterization.
- Enforce authorization separately and use least-privileged database accounts.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [123 - Secure coding and threat modeling](123-secure-coding-and-threat-modeling.md) | [Q&A](../qnas/124-preventing-sql-injection.md) | [Example](../examples/124-preventing-sql-injection.md) | [125 - Preventing cross-site scripting](125-preventing-cross-site-scripting.md) |