# 100 - Prepared statements and safe database queries: Example

## Goal

Use PDO prepared statements to search, insert, update, and delete rows safely. The example also shows how to allowlist a dynamic sort choice and bind one value for each item in an IN list.

## Question

How can a PHP application keep changing values separate from SQL while still supporting search, sorting, and a variable number of IDs?

## Complete example

Create a file named prepared-statements-demo.php:

~~~text
project/
└── prepared-statements-demo.php
~~~

~~~php
<?php

declare(strict_types=1);

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
    $pdo = new PDO('sqlite::memory:', null, null, $options);

    $pdo->exec(
        'CREATE TABLE articles (
            id INTEGER PRIMARY KEY,
            title TEXT NOT NULL,
            status TEXT NOT NULL
        )'
    );

    // Add fixed lesson data with a prepared INSERT statement.
    $seed = $pdo->prepare(
        'INSERT INTO articles (title, status)
         VALUES (:title, :status)'
    );

    $seedArticles = [
        ['title' => 'Prepared statements keep database input safe', 'status' => 'published'],
        ['title' => 'PDO binds query values', 'status' => 'published'],
        ['title' => 'SQL identifiers need allowlists', 'status' => 'draft'],
    ];

    foreach ($seedArticles as $article) {
        $seed->execute([
            'title' => $article['title'],
            'status' => $article['status'],
        ]);
    }

    // Search for a value. The wildcard characters are part of LIKE behavior.
    $searchTerm = 'database';
    $search = $pdo->prepare(
        'SELECT id, title
         FROM articles
         WHERE title LIKE :pattern
         ORDER BY id'
    );
    $search->execute([
        'pattern' => '%' . $searchTerm . '%',
    ]);

    while (($article = $search->fetch()) !== false) {
        echo 'Search result: '
            . $article['id']
            . ' | '
            . $article['title']
            . PHP_EOL;
    }

    $search->closeCursor();

    // SQL-looking text stays a value when it is bound.
    $sqlLookingInput = "' OR 1=1 --";
    $exactSearch = $pdo->prepare(
        'SELECT id
         FROM articles
         WHERE title = :title'
    );
    $exactSearch->execute([
        'title' => $sqlLookingInput,
    ]);
    $exactMatches = $exactSearch->fetchAll();

    echo 'Rows matching SQL-looking exact value: '
        . count($exactMatches)
        . PHP_EOL;

    // SQL identifiers cannot be bound, so map choices to fixed SQL fragments.
    $requestedSort = 'title';
    $requestedDirection = 'asc';

    $allowedColumns = [
        'title' => 'title',
        'id' => 'id',
    ];
    $allowedDirections = [
        'asc' => 'ASC',
        'desc' => 'DESC',
    ];

    $sortColumn = $allowedColumns[$requestedSort] ?? 'id';
    $sortDirection = $allowedDirections[strtolower($requestedDirection)] ?? 'ASC';

    $sortSql = 'SELECT id, title
                FROM articles
                ORDER BY ' . $sortColumn . ' ' . $sortDirection;
    $sortStatement = $pdo->query($sortSql);
    $sortedArticles = $sortStatement->fetchAll();

    echo 'First title after sorting: '
        . $sortedArticles[0]['title']
        . PHP_EOL;

    // A single placeholder represents one value. Create one per ID.
    $articleIds = [1, 2];

    if ($articleIds === []) {
        echo 'IN query IDs: none' . PHP_EOL;
    } else {
        $placeholders = implode(
            ', ',
            array_fill(0, count($articleIds), '?')
        );

        $byIds = $pdo->prepare(
            'SELECT id, title
             FROM articles
             WHERE id IN (' . $placeholders . ')
             ORDER BY id'
        );
        $byIds->execute($articleIds);

        $matchingArticles = $byIds->fetchAll();
        $matchingIds = array_column($matchingArticles, 'id');

        echo 'IN query IDs: '
            . implode(', ', $matchingIds)
            . PHP_EOL;
    }

    // Insert a value supplied to the application.
    $insert = $pdo->prepare(
        'INSERT INTO articles (title, status)
         VALUES (:title, :status)'
    );
    $insert->execute([
        'title' => 'A newly added database article',
        'status' => 'draft',
    ]);

    $insertedId = (int) $pdo->lastInsertId();

    echo 'Inserted article ID: ' . $insertedId . PHP_EOL;

    // Bind an integer explicitly for the update.
    $update = $pdo->prepare(
        'UPDATE articles
         SET status = :status
         WHERE id = :id'
    );
    $update->bindValue(':status', 'published', PDO::PARAM_STR);
    $update->bindValue(':id', $insertedId, PDO::PARAM_INT);
    $update->execute();

    echo 'Updated rows: ' . $update->rowCount() . PHP_EOL;

    // Delete with a bound value as well.
    $delete = $pdo->prepare(
        'DELETE FROM articles
         WHERE id = :id'
    );
    $delete->execute([
        'id' => $insertedId,
    ]);

    echo 'Deleted rows: ' . $delete->rowCount() . PHP_EOL;
} catch (PDOException $exception) {
    // Keep diagnostic details in a protected log, not in a public response.
    error_log('Prepared statement example failed: ' . $exception->getMessage());
    fwrite(STDERR, 'The database example could not be completed.' . PHP_EOL);
    exit(1);
}
~~~

## Run the example

The PHP runtime must have PDO and the PDO_SQLITE driver enabled. Run the script from its directory:

~~~sh
php prepared-statements-demo.php
~~~

Expected output:

~~~text
Search result: 1 | Prepared statements keep database input safe
Rows matching SQL-looking exact value: 0
First title after sorting: PDO binds query values
IN query IDs: 1, 2
Inserted article ID: 4
Updated rows: 1
Deleted rows: 1
~~~

## How it works

1. **Connection and schema:** The example uses an in-memory SQLite database, so its sample data disappears when the script ends. The table definition is fixed SQL.
2. **Seed values:** The INSERT statement is prepared once and executed with a separate set of title and status values for each row.
3. **Search:** The search term is bound as a value. Percent signs are LIKE wildcards, so this search looks for the term anywhere in the title.
4. **SQL-looking input:** The text that resembles an injection attempt is bound to an exact-match query. It is treated as one string value and matches no seeded title.
5. **Dynamic sorting:** The selected column and direction come only from fixed allowlists. Other request values are never inserted into those SQL positions.
6. **Variable IN list:** The program creates one positional placeholder per ID and passes the IDs separately. The empty-list branch avoids generating an invalid or unexpectedly broad query.
7. **Insert, update, and delete:** Each changing value is passed through a parameter. The update binds the ID explicitly as an integer.
8. **Errors:** The exception detail is sent to the error log; the terminal gets a generic message.

## Important notes

- Prepared statement placeholders represent values. They cannot represent table names, column names, keywords, or whole SQL clauses.
- Do not build SQL by concatenating request values or by manually escaping them.
- Binding a LIKE pattern protects SQL syntax but does not make percent and underscore literal characters.
- Validate values against application rules, and separately check whether the current user is authorized to access or change the selected records.
- Treat rowCount() as a driver-specific affected-row result, not proof that the application operation had the intended business outcome.
- This is a local CLI learning example. It does not implement authentication, authorization, web request handling, or production database configuration.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [099 - Connecting to databases with PDO](../lessons/099-connecting-to-databases-with-pdo.md) | [Q&A](../qnas/100-prepared-statements-and-safe-database-queries.md) | Example | [101 - Database transactions and error handling](../lessons/101-database-transactions-and-error-handling.md) |
