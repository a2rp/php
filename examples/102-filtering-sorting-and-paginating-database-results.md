# 102 - Filtering, sorting, and paginating database results: Example

## Goal

Create a small PDO_SQLITE catalog query that validates optional request filters, counts the matching products, allows a safe sort choice, and returns one bounded page at a time.

## Question

How can a PHP list page use the same filters for its count and result queries, return stable pages, and avoid putting raw request values into SQL?

## Complete example

Create a file named catalog-pagination-demo.php:

~~~text
project/
└── catalog-pagination-demo.php
~~~

~~~php
<?php

declare(strict_types=1);

/**
 * Normalize a single positive integer and cap it at a configured maximum.
 */
function boundedPositiveInt(
    mixed $input,
    int $default,
    int $maximum
): int {
    if (!is_string($input) && !is_int($input)) {
        return $default;
    }

    $value = filter_var($input, FILTER_VALIDATE_INT);

    if ($value === false) {
        return $default;
    }

    return max(1, min($maximum, $value));
}

/**
 * Bind validated filter values with an explicit PDO type.
 */
function bindQueryParameters(
    PDOStatement $statement,
    array $parameters
): void {
    foreach ($parameters as $name => $value) {
        $type = is_int($value) ? PDO::PARAM_INT : PDO::PARAM_STR;
        $statement->bindValue(':' . $name, $value, $type);
    }
}

/**
 * Return one filtered and sorted page plus its page metadata.
 */
function findProducts(PDO $pdo, array $query): array
{
    $categoryId = null;
    $categoryInput = $query['category_id'] ?? null;

    if ($categoryInput !== null && $categoryInput !== '') {
        if (!is_string($categoryInput) && !is_int($categoryInput)) {
            throw new InvalidArgumentException(
                'Category must be a single integer value.'
            );
        }

        $categoryId = filter_var($categoryInput, FILTER_VALIDATE_INT);

        if ($categoryId === false || $categoryId < 1) {
            throw new InvalidArgumentException(
                'Category must be a positive integer.'
            );
        }
    }

    $searchInput = $query['search'] ?? '';

    if (!is_string($searchInput)) {
        throw new InvalidArgumentException(
            'Search must be a single text value.'
        );
    }

    $searchText = trim($searchInput);

    if (strlen($searchText) > 100) {
        throw new InvalidArgumentException(
            'Search must not exceed 100 bytes.'
        );
    }

    $conditions = [];
    $parameters = [];

    if ($categoryId !== null) {
        $conditions[] = 'category_id = :category_id';
        $parameters['category_id'] = $categoryId;
    }

    if ($searchText !== '') {
        $conditions[] = 'name LIKE :search_pattern';
        $parameters['search_pattern'] = '%' . $searchText . '%';
    }

    $whereSql = $conditions === []
        ? ''
        : ' WHERE ' . implode(' AND ', $conditions);

    // This count uses exactly the same filter conditions and values.
    $countStatement = $pdo->prepare(
        'SELECT COUNT(*)
         FROM products'
        . $whereSql
    );
    bindQueryParameters($countStatement, $parameters);
    $countStatement->execute();

    $totalRows = (int) $countStatement->fetchColumn();
    $countStatement->closeCursor();

    $page = boundedPositiveInt($query['page'] ?? null, 1, 10000);
    $pageSize = boundedPositiveInt($query['per_page'] ?? null, 20, 50);
    $totalPages = (int) ceil($totalRows / $pageSize);

    if ($totalPages > 0) {
        $page = min($page, $totalPages);
    } else {
        $page = 1;
    }

    $offset = ($page - 1) * $pageSize;

    // The user selects a key, never an SQL fragment.
    $sortOptions = [
        'newest' => 'created_at DESC, id DESC',
        'oldest' => 'created_at ASC, id ASC',
        'name' => 'name ASC, id ASC',
        'price-low' => 'price_cents ASC, id ASC',
    ];

    $sortKey = $query['sort'] ?? 'newest';

    if (!is_string($sortKey)) {
        $sortKey = 'newest';
    }

    $orderBy = $sortOptions[$sortKey] ?? $sortOptions['newest'];

    $resultStatement = $pdo->prepare(
        'SELECT id, name, price_cents, created_at
         FROM products'
        . $whereSql
        . ' ORDER BY ' . $orderBy
        . ' LIMIT :page_size OFFSET :row_offset'
    );

    bindQueryParameters($resultStatement, $parameters);
    $resultStatement->bindValue(
        ':page_size',
        $pageSize,
        PDO::PARAM_INT
    );
    $resultStatement->bindValue(
        ':row_offset',
        $offset,
        PDO::PARAM_INT
    );
    $resultStatement->execute();

    $products = $resultStatement->fetchAll();
    $resultStatement->closeCursor();

    return [
        'products' => $products,
        'totalRows' => $totalRows,
        'totalPages' => $totalPages,
        'page' => $page,
        'pageSize' => $pageSize,
    ];
}

function formatPrice(int $priceInCents): string
{
    return '$'
        . intdiv($priceInCents, 100)
        . '.'
        . str_pad(
            (string) ($priceInCents % 100),
            2,
            '0',
            STR_PAD_LEFT
        );
}

function printPage(string $label, array $result): void
{
    echo $label
        . ' '
        . $result['page']
        . ' of '
        . max(1, $result['totalPages'])
        . PHP_EOL;

    if ($result['products'] === []) {
        echo 'No products found.' . PHP_EOL;
        return;
    }

    foreach ($result['products'] as $product) {
        echo '- '
            . $product['id']
            . ': '
            . $product['name']
            . ' ('
            . formatPrice((int) $product['price_cents'])
            . ')'
            . PHP_EOL;
    }
}

$options = [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
];

try {
    $pdo = new PDO('sqlite::memory:', null, null, $options);

    $pdo->exec(
        'CREATE TABLE products (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            category_id INTEGER NOT NULL,
            price_cents INTEGER NOT NULL CHECK (price_cents >= 0),
            created_at TEXT NOT NULL
        )'
    );

    $insert = $pdo->prepare(
        'INSERT INTO products
            (id, name, category_id, price_cents, created_at)
         VALUES
            (:id, :name, :category_id, :price_cents, :created_at)'
    );

    $sampleProducts = [
        [
            'id' => 1,
            'name' => 'SQL Field Guide',
            'category_id' => 1,
            'price_cents' => 2200,
            'created_at' => '2025-01-02',
        ],
        [
            'id' => 2,
            'name' => 'PHP Reference Guide',
            'category_id' => 1,
            'price_cents' => 3500,
            'created_at' => '2025-01-05',
        ],
        [
            'id' => 3,
            'name' => 'Desk Lamp',
            'category_id' => 2,
            'price_cents' => 2799,
            'created_at' => '2025-01-06',
        ],
        [
            'id' => 4,
            'name' => 'Database Guide',
            'category_id' => 1,
            'price_cents' => 1200,
            'created_at' => '2025-01-08',
        ],
        [
            'id' => 5,
            'name' => 'PHP Basics',
            'category_id' => 1,
            'price_cents' => 1800,
            'created_at' => '2025-01-09',
        ],
    ];

    foreach ($sampleProducts as $product) {
        $insert->execute($product);
    }

    // These values stand in for validated query parameters from a web request.
    $query = [
        'category_id' => '1',
        'search' => 'Guide',
        'sort' => 'newest',
        'page' => '1',
        'per_page' => '2',
    ];

    $firstPage = findProducts($pdo, $query);

    echo 'Total matching products: '
        . $firstPage['totalRows']
        . PHP_EOL;
    printPage('Page', $firstPage);

    $query['page'] = '2';
    $secondPage = findProducts($pdo, $query);
    printPage('Page', $secondPage);
} catch (PDOException $exception) {
    error_log('Catalog query failed: ' . $exception->getMessage());
    fwrite(STDERR, 'The catalog could not be loaded.' . PHP_EOL);
    exit(1);
} catch (Throwable $exception) {
    error_log('Catalog request failed: ' . $exception->getMessage());
    fwrite(STDERR, 'The catalog request could not be completed.' . PHP_EOL);
    exit(1);
}
~~~

## Run the example

The PHP runtime must have PDO and the PDO_SQLITE driver enabled. Run the script from its directory:

~~~sh
php catalog-pagination-demo.php
~~~

Expected output:

~~~text
Total matching products: 3
Page 1 of 2
- 4: Database Guide ($12.00)
- 2: PHP Reference Guide ($35.00)
Page 2 of 2
- 1: SQL Field Guide ($22.00)
~~~

The category filter selects category 1. The search matches three names containing Guide. The newest sort puts IDs 4, 2, and 1 in that order, with ID as a tie-breaker. A page size of two returns the first two products on page 1 and the remaining product on page 2.

## How it works

1. **Request validation:** The example accepts only a single integer category, a string search, a recognized sort key, and bounded positive page values.
2. **Optional filters:** It adds a fixed category condition and/or fixed name condition. Their values are bound separately.
3. **Consistent count:** The COUNT query reuses the exact same WHERE clause and filter parameters as the result query.
4. **Page metadata:** It calculates the number of pages, clamps a too-large requested page to the last page when results exist, and computes the offset.
5. **Safe sorting:** The requested key selects one of four fixed ORDER BY expressions. Raw request text is never appended to SQL.
6. **Stable order:** Every option includes id as a unique tie-breaker.
7. **Typed page values:** LIMIT and OFFSET are bound as integers for PDO_SQLITE.
8. **Second page:** The script changes only the requested page number and runs the same validated query path again.
9. **Errors:** Database diagnostics go to the error log; terminal output gets a generic message.

## Important notes

- This example uses SQLite syntax and PDO_SQLITE. Check the target PDO driver's support for bound LIMIT and OFFSET values and use its documented SQL syntax.
- Percent signs in the search pattern act as LIKE wildcards. Binding the pattern does not make wildcard characters literal.
- A sort allowlist prevents request text from becoming SQL syntax. It does not replace authorization for the records being queried.
- A COUNT query and its result query can observe different data if rows change between them. The exact behavior depends on timing and transaction isolation.
- Offset pages can shift as rows are inserted or deleted between requests. For very deep sequential browsing, consider keyset pagination using the last row's sort values.
- The page and page-size caps are illustrative product choices. Also consider rate limits and appropriate indexes for frequently used filters and sorts.
- The example is a CLI teaching program, not a complete web endpoint. A web application must validate the actual request, authorize access, and format a safe HTTP response.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [101 - Database transactions and error handling](../lessons/101-database-transactions-and-error-handling.md) | [Q&A](../qnas/102-filtering-sorting-and-paginating-database-results.md) | Example | [103 - Using the MySQLi extension](../lessons/103-using-the-mysqli-extension.md) |
