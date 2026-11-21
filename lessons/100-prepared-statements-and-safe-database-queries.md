# 100 - Prepared statements and safe database queries

## Learning goals

By the end of this lesson, you will be able to:

- Explain why concatenating untrusted input into SQL is unsafe.
- Prepare a statement and bind values with PDO.
- Choose named or positional placeholders consistently.
- Understand why placeholders cannot represent table names or SQL keywords.
- Build a safe variable-length IN list with one placeholder per value.
- Apply prepared statements to SELECT, INSERT, UPDATE, and DELETE queries.
- Recognize why escaping and validation do not replace parameter binding.

## 1. Keep SQL syntax separate from data

SQL injection happens when data supplied to an application changes the structure or meaning of its SQL statement.

This is unsafe:

~~~php
$email = $_POST['email'] ?? '';

$sql = "SELECT id, display_name
        FROM customers
        WHERE email = '$email'";

$statement = $pdo->query($sql);
~~~

The request value is inserted directly into SQL syntax. A crafted value could change what the database executes.

Prepared statements keep the SQL structure separate from the values. Write the query with placeholders, then bind or pass the values when executing it.

~~~php
$statement = $pdo->prepare(
    'SELECT id, display_name
     FROM customers
     WHERE email = :email'
);

$statement->execute([
    'email' => $email,
]);

$customer = $statement->fetch();
~~~

Do not add quotes around a placeholder in the SQL. PDO supplies the value separately, with the appropriate handling for the database driver.

Prepared statements protect values in that statement. They do not automatically validate that a value is allowed by product rules, or that the current user may access the matched record.

## 2. Use named placeholders

Named placeholders make the connection between a SQL value and its meaning easy to read:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, status
     FROM articles
     WHERE author_id = :author_id
       AND status = :status
     ORDER BY id DESC'
);

$statement->bindValue(':author_id', $authorId, PDO::PARAM_INT);
$statement->bindValue(':status', $status, PDO::PARAM_STR);
$statement->execute();

$articles = $statement->fetchAll();
~~~

The colon is part of the marker in the SQL and can be included when binding. When passing an array to execute(), keys may be written with or without the colon. Use one style consistently.

For a small, one-time execution, passing an associative array is concise:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title
     FROM articles
     WHERE author_id = :author_id'
);

$statement->execute([
    'author_id' => $authorId,
]);
~~~

If the value needs an explicit PDO type, use bindValue() or bindParam(). Values passed through execute() are treated as strings by PDO; database drivers may still apply their own conversion rules.

Use a unique marker for each value. Reusing a named marker more than once in one statement is not supported consistently by all drivers and modes.

## 3. Choose between bindValue() and bindParam()

bindValue() binds the value as it is at the time of the call:

~~~php
$statement->bindValue(':status', $status, PDO::PARAM_STR);
~~~

bindParam() binds a PHP variable by reference. Its value is read when execute() runs:

~~~php
$statement->bindParam(':status', $status, PDO::PARAM_STR);
$status = 'published';
$statement->execute();
~~~

Use bindValue() for most straightforward input values. bindParam() is useful when the same prepared statement is executed repeatedly while a bound variable changes.

If you pass values to execute(), do not mix that with previous bindings unless you understand how the driver treats the supplied array. Choose one binding style for a statement execution and provide every required marker.

## 4. Positional placeholders are another option

Question-mark placeholders bind values by position:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title
     FROM articles
     WHERE author_id = ?
       AND status = ?'
);

$statement->execute([$authorId, $status]);
~~~

The first array value matches the first question mark, and the second matches the second. The number and order must match the SQL.

Do not mix named and positional placeholder styles in the same statement. Named markers can be easier to review in longer queries; positional markers can be concise for short queries. Use whichever style your project consistently prefers.

## 5. Bind values in every kind of data query

Prepared statements apply to reads and writes.

### SELECT

~~~php
$statement = $pdo->prepare(
    'SELECT id, title
     FROM articles
     WHERE status = :status
     ORDER BY id DESC'
);

$statement->execute(['status' => $status]);

while (($article = $statement->fetch()) !== false) {
    echo $article['title'] . PHP_EOL;
}
~~~

### INSERT

~~~php
$statement = $pdo->prepare(
    'INSERT INTO articles (author_id, title, status)
     VALUES (:author_id, :title, :status)'
);

$statement->execute([
    'author_id' => $authorId,
    'title' => $title,
    'status' => 'draft',
]);
~~~

### UPDATE

~~~php
$statement = $pdo->prepare(
    'UPDATE articles
     SET title = :title
     WHERE id = :id'
);

$statement->bindValue(':title', $title, PDO::PARAM_STR);
$statement->bindValue(':id', $articleId, PDO::PARAM_INT);
$statement->execute();
~~~

### DELETE

~~~php
$statement = $pdo->prepare(
    'DELETE FROM articles
     WHERE id = :id'
);

$statement->bindValue(':id', $articleId, PDO::PARAM_INT);
$statement->execute();
~~~

Binding values prevents them from changing SQL syntax. Authorization is still required: a valid article ID does not prove that the current user may update or delete that article.

## 6. Placeholders represent values, not SQL structure

A placeholder cannot stand for a table name, column name, sort direction, or arbitrary SQL fragment.

This does not safely select a column:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title FROM articles ORDER BY :column'
);
$statement->execute(['column' => $requestedColumn]);
~~~

The marker represents a value. It is not replaced with an SQL identifier; the database may treat it as a constant expression, producing meaningless ordering.

When a user may choose among sort options, map the request to fixed application-owned SQL fragments:

~~~php
$sortOptions = [
    'newest' => 'created_at DESC, id DESC',
    'oldest' => 'created_at ASC, id ASC',
    'title' => 'title ASC, id ASC',
];

$sortKey = $_GET['sort'] ?? 'newest';
$orderBy = $sortOptions[$sortKey] ?? $sortOptions['newest'];

$sql = 'SELECT id, title FROM articles ORDER BY ' . $orderBy;
$statement = $pdo->query($sql);
~~~

The request chooses only a key in a fixed map. It never supplies SQL syntax directly. Apply the same approach to table names and column names that must vary.

Validation is still useful: it can reject unsupported sort choices or values outside product rules. It is not a replacement for binding values.

## 7. Build variable-length IN lists safely

One placeholder represents one value. A single marker cannot bind an entire array for an IN clause.

Create exactly one placeholder per value, and bind all values:

~~~php
$articleIds = [4, 9, 15];

if ($articleIds === []) {
    $articles = [];
} else {
    $placeholders = implode(
        ', ',
        array_fill(0, count($articleIds), '?')
    );

    $sql = "SELECT id, title
            FROM articles
            WHERE id IN ($placeholders)
            ORDER BY id";

    $statement = $pdo->prepare($sql);
    $statement->execute($articleIds);
    $articles = $statement->fetchAll();
}
~~~

The placeholder string is generated from the count of values, not from their contents. The values themselves remain bound separately.

Handle an empty list explicitly. A generated IN () clause is not portable SQL and may be invalid.

## 8. Use LIKE values deliberately

A search pattern can be bound safely:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title
     FROM articles
     WHERE title LIKE :pattern'
);

$statement->execute([
    'pattern' => '%' . $searchText . '%',
]);
~~~

Binding keeps the search text from becoming SQL syntax. The percent and underscore characters still have their normal LIKE wildcard meaning. If the product requires those characters to match literally, escape them according to the database's LIKE rules and use a matching ESCAPE clause.

## 9. Do not manually escape values as a replacement

Functions such as PDO::quote() and manual backslash or quote replacement are not the recommended way to build dynamic queries. Escaping rules vary by SQL dialect and connection settings, and it is easy to make a mistake.

Keep the SQL structure fixed and bind the data values. Do not attempt to repair an unsafe concatenated query by adding more string filters.

## 10. Validate, authorize, and bind as separate steps

A safe database request usually needs all three:

1. **Validate:** Does the value have the expected type, format, length, or allowed range?
2. **Authorize:** May this caller access or change the requested record?
3. **Bind:** Is the value kept separate from the SQL syntax?

For example, an ID can be an integer and be bound safely, while still identifying another user's record. Authorization must check ownership or permission for that exact object.

Use least-privilege database credentials as another layer. Parameter binding addresses SQL injection in a statement; it does not make an overprivileged database account safe.

## 11. Handle errors and affected rows thoughtfully

With PDO exception mode, a failed prepare or execution can throw PDOException. Catch it at an appropriate application boundary, log diagnostic information securely, and return a generic message to the visitor.

Do not include raw SQL, credentials, database paths, or exception details in public output.

The affected-row count can help determine whether an UPDATE or DELETE matched data, but behavior can vary by database and settings. A zero count may mean no row matched or that the stored value was already the same. Define the desired behavior and check the database's semantics.

Prepared statements can be reused for repeated executions with different values:

~~~php
$statement = $pdo->prepare(
    'INSERT INTO audit_events (event_type, actor_id)
     VALUES (:event_type, :actor_id)'
);

foreach ($events as $event) {
    $statement->execute([
        'event_type' => $event['type'],
        'actor_id' => $event['actorId'],
    ]);
}
~~~

For a batch that must succeed as a unit, combine the work with a transaction. Transactions and robust error handling are covered in the next chapter.

## Common mistakes

### Quoting a placeholder in the SQL

Writing ':email' inside SQL quotes makes it ordinary text instead of a parameter marker. Leave the marker unquoted.

### Preparing after interpolation

Calling prepare() on a query that already contains concatenated request data does not undo the injection risk. Keep values out of the SQL string from the start.

### Trying to bind an identifier

Placeholders are for values. Use a fixed allowlist to select a known identifier or SQL fragment.

### Binding one array to one IN placeholder

A marker represents one value, not a collection. Generate one marker per list item and bind each value.

### Mixing named and positional markers

Use either named markers or question marks in a statement, not both.

### Assuming bound values are automatically authorized

Parameterization separates values from SQL. It does not check user permissions or business rules.

## Key takeaways

- Prepare SQL with placeholders and pass values separately.
- Use named or positional placeholders consistently and provide every value.
- Use bindValue() for a value snapshot and bindParam() when a variable should be read at execution time.
- Bind data in SELECT, INSERT, UPDATE, and DELETE statements.
- Placeholders cannot represent identifiers or SQL keywords; use a strict allowlist for those choices.
- Use one placeholder per value in an IN list and handle empty lists explicitly.
- Validation and authorization complement parameter binding; neither replaces it.
- Avoid manual escaping and keep raw database errors out of public responses.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [099 - Connecting to databases with PDO](099-connecting-to-databases-with-pdo.md) | [Q&A](../qnas/100-prepared-statements-and-safe-database-queries.md) | [Example](../examples/100-prepared-statements-and-safe-database-queries.md) | [101 - Database transactions and error handling](101-database-transactions-and-error-handling.md) |
