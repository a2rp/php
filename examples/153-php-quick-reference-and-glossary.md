# PHP quick reference and glossary — Example

This example combines common PHP syntax in a runnable command-line reading-list summary, then shows small copy-ready patterns for web forms, HTML output, PDO, JSON, and version-specific syntax.

The main script runs on PHP 8.0 or later. The version-specific section calls out syntax that needs a newer runtime.

## Goal

Run a PHP script that:

- Stores a few book records in arrays.
- Validates an optional command-line filter.
- Counts items by status.
- Uses functions, types, loops, closures, and array helpers.
- Calculates reading progress.
- Encodes the result as JSON.
- Reports expected failures safely.

## Project structure

~~~text
php-quick-reference-example/
└── php-quick-reference.php
~~~

## Create the runnable script

Create **php-quick-reference.php**:

~~~php
<?php

declare(strict_types=1);

const STATUSES = ['planned', 'reading', 'finished'];

$books = [
    [
        'title' => 'Everyday PHP',
        'status' => 'reading',
        'pages' => 320,
        'pages_read' => 120,
    ],
    [
        'title' => 'Web Security Notes',
        'status' => 'finished',
        'pages' => 210,
        'pages_read' => 210,
    ],
    [
        'title' => 'Clean Code Workbook',
        'status' => 'planned',
        'pages' => 180,
        'pages_read' => 0,
    ],
];

function countBooksByStatus(array $books): array
{
    $counts = array_fill_keys(STATUSES, 0);

    foreach ($books as $book) {
        $status = $book['status'] ?? null;

        if (!is_string($status) || !in_array($status, STATUSES, true)) {
            throw new InvalidArgumentException('A book has an unknown status.');
        }

        $counts[$status]++;
    }

    return $counts;
}

function progressPercent(array $book): int
{
    $pages = $book['pages'] ?? null;
    $pagesRead = $book['pages_read'] ?? null;

    if (
        !is_int($pages)
        || !is_int($pagesRead)
        || $pages < 1
        || $pagesRead < 0
        || $pagesRead > $pages
    ) {
        throw new InvalidArgumentException('A book has invalid page counts.');
    }

    return (int) round(($pagesRead / $pages) * 100);
}

$filter = $argv[1] ?? 'all';
$allowedFilters = array_merge(['all'], STATUSES);

if (!is_string($filter) || !in_array($filter, $allowedFilters, true)) {
    fwrite(
        STDERR,
        'Usage: php php-quick-reference.php [all|planned|reading|finished]'
        . PHP_EOL
    );
    exit(2);
}

$visibleBooks = $filter === 'all'
    ? $books
    : array_values(array_filter(
        $books,
        static fn(array $book): bool => $book['status'] === $filter
    ));

try {
    $counts = countBooksByStatus($visibleBooks);

    $currentlyReading = array_map(
        static fn(array $book): array => [
            'title' => $book['title'],
            'progress_percent' => progressPercent($book),
        ],
        array_values(array_filter(
            $visibleBooks,
            static fn(array $book): bool => $book['status'] === 'reading'
        ))
    );

    $summary = [
        'filter' => $filter,
        'total' => count($visibleBooks),
        'by_status' => $counts,
        'currently_reading' => $currentlyReading,
    ];

    echo 'Reading list: ', count($visibleBooks), ' book(s)', PHP_EOL;
    echo 'Status counts:', PHP_EOL;

    foreach ($counts as $status => $count) {
        echo '- ', $status, ': ', $count, PHP_EOL;
    }

    if ($currentlyReading !== []) {
        echo 'Currently reading:', PHP_EOL;

        foreach ($currentlyReading as $book) {
            echo '- ', $book['title'], ' (', $book['progress_percent'], '%)', PHP_EOL;
        }
    }

    echo 'JSON summary:', PHP_EOL;
    echo json_encode(
        $summary,
        JSON_PRETTY_PRINT | JSON_THROW_ON_ERROR
    ), PHP_EOL;
} catch (InvalidArgumentException $exception) {
    fwrite(STDERR, 'Data error: ' . $exception->getMessage() . PHP_EOL);
    exit(1);
} catch (JsonException $exception) {
    fwrite(STDERR, 'Could not encode the summary as JSON.' . PHP_EOL);
    exit(1);
}
~~~

The dataset is internal sample data. If the same values came from a request or database, validate them at that boundary too.

## Run it

Run the default report:

~~~powershell
php php-quick-reference.php
~~~

Expected output:

~~~text
Reading list: 3 book(s)
Status counts:
- planned: 1
- reading: 1
- finished: 1
Currently reading:
- Everyday PHP (38%)
JSON summary:
{
    "filter": "all",
    "total": 3,
    "by_status": {
        "planned": 1,
        "reading": 1,
        "finished": 1
    },
    "currently_reading": [
        {
            "title": "Everyday PHP",
            "progress_percent": 38
        }
    ]
}
~~~

The percentage is rounded from 120 pages read out of 320.

Filter the report:

~~~powershell
php php-quick-reference.php finished
~~~

This reports one finished book. Try an unsupported value:

~~~powershell
php php-quick-reference.php archived
~~~

The script prints a usage message to STDERR and exits with code 2. It does not silently accept an unknown filter.

## What the script demonstrates

| Feature | Where it appears |
| --- | --- |
| Strict type declaration | At the top of the PHP file |
| Constant | STATUSES |
| Indexed and associative arrays | The book records |
| Scalar and return types | Both helper functions |
| foreach | Status counts and reading results |
| Closure | The static arrow functions passed to array helpers |
| Null coalescing | Optional array values and command-line filter |
| Strict comparison | in_array(..., true) and status checks |
| Exceptions | Invalid internal data and JSON failures |
| JSON encoding | Final summary output |
| CLI output and exit status | echo, fwrite(STDERR), and exit |

## Safe HTML output pattern

For a server-rendered page, escape each dynamic string for HTML:

~~~php
function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

$title = $book['title'];
echo '<h2>' . e($title) . '</h2>';
~~~

If the title contains markup characters, the browser displays them as text instead of interpreting them as HTML. Do not reuse this helper for JavaScript, CSS, or SQL contexts.

## Form input and validation pattern

A browser form is not a trusted boundary. Check the submitted type and enforce server rules:

~~~php
$rawTitle = $_POST['title'] ?? null;
$errors = [];

if (!is_string($rawTitle)) {
    $errors['title'] = 'Enter a title.';
} else {
    $title = trim($rawTitle);

    if ($title === '') {
        $errors['title'] = 'A title is required.';
    } elseif (mb_strlen($title, 'UTF-8') > 160) {
        $errors['title'] = 'The title must be no longer than 160 characters.';
    }
}
~~~

For a real state-changing form, also verify a session CSRF token before saving. Keep validation and output escaping as separate steps.

## PDO prepared statement pattern

Bind values rather than concatenating them into SQL:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, status FROM books WHERE status = :status'
);
$statement->execute(['status' => $status]);
$books = $statement->fetchAll();
~~~

A placeholder represents a value, not an SQL column name. For dynamic sorting, map allowed user choices to fixed SQL fragments.

## JSON request and response pattern

~~~php
try {
    $payload = json_decode($rawBody, false, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    http_response_code(400);
    echo json_encode([
        'error' => [
            'code' => 'invalid_json',
            'message' => 'The request body is not valid JSON.',
        ],
    ]);
}
~~~

A production endpoint should also check the request size and Content-Type, verify the decoded shape, validate each field, and return a consistent status and response format. Do not expose exception details in the response.

## Object-oriented pattern

A small class can keep related data and behavior together:

~~~php
interface Summarizable
{
    public function summary(): string;
}

final class Book
{
    public function __construct(
        public string $title,
        private string $author
    ) {
    }

    public function summary(): string
    {
        return $this->title . ' by ' . $this->author;
    }
}
~~~

Constructor property promotion requires PHP 8.0 or later. Keep properties private when callers should not change them directly; public promotion is concise for simple data objects.

## Version-specific syntax examples

Enums require PHP 8.1 or later:

~~~php
enum BookStatus: string
{
    case Planned = 'planned';
    case Reading = 'reading';
    case Finished = 'finished';
}
~~~

Readonly classes require PHP 8.2 or later:

~~~php
readonly class BookSummary
{
    public function __construct(
        public string $title,
        public string $status
    ) {
    }
}
~~~

The pipe operator requires PHP 8.5 or later:

~~~php
$slug = $title
    |> trim(...)
    |> strtolower(...);
~~~

Do not copy syntax into a project that supports an earlier PHP version. Check the runtime used by local development, CI, and production.

## Verification checklist

- Run the script with no argument and compare the total and status counts.
- Run it with reading and finished filters.
- Run it with an unsupported filter and confirm a non-zero exit.
- Change a sample status to an unsupported value and confirm the data check reports an error.
- Change pages_read to exceed pages and confirm progress validation reports an error.
- Run php -l php-quick-reference.php to check syntax.
- Confirm the runtime version before trying the enum, readonly class, or pipe examples.
- For web patterns, verify invalid forms are rejected, output is escaped, and SQL uses bound parameters.

The command-line script is an executable reference example. The web snippets are focused patterns to adapt inside an application with its own routes, templates, error handling, and security requirements.

## Expected result

The main script prints a readable status summary and a JSON representation. A valid status filter narrows the report. An invalid filter or invalid internal record produces a clear error on STDERR and a non-zero exit code.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Course |
| --- | --- | --- | --- |
| [152 - Capstone: plan, build, test, and document an application](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) | [Q&A](../qnas/153-php-quick-reference-and-glossary.md) | [Lesson](../lessons/153-php-quick-reference-and-glossary.md) | End of course |
