# Integration tests for databases and HTTP

## Question

How can you check that a PHP notes endpoint accepts an HTTP request, stores the note in a real database, and returns the stored note on a later request?

This example uses PDO with SQLite for a repeatable database integration test. It also runs a small PHP development server and sends real HTTP requests to the endpoint. The database is created only for this example and must not contain important data.

## Project files

~~~text
project/
├── public/
│   └── index.php
├── src/
│   ├── NoteRepository.php
│   └── schema.php
└── tests/
    ├── Integration/
    │   └── NoteRepositoryIntegrationTest.php
    └── http-smoke.php
~~~

The PHPUnit test needs PHPUnit installed in the project and the PDO SQLite extension enabled. The HTTP server also needs PDO SQLite. Check the PHP version and dependency constraints used by your project before installing a PHPUnit release.

## 1. Define the test schema

### File: src/schema.php

~~~php
<?php

declare(strict_types=1);

function createSchema(PDO $database): void
{
    $database->exec(
        'CREATE TABLE IF NOT EXISTS notes (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            title TEXT NOT NULL CHECK (length(trim(title)) > 0)
        )',
    );
}
~~~

The CHECK constraint makes the database reject a blank title, even if another caller bypasses request validation.

## 2. Create a repository that uses PDO

### File: src/NoteRepository.php

~~~php
<?php

declare(strict_types=1);

final class NoteRepository
{
    public function __construct(private PDO $database)
    {
    }

    public function create(string $title): int
    {
        $statement = $this->database->prepare(
            'INSERT INTO notes (title) VALUES (:title)',
        );
        $statement->execute(['title' => $title]);

        return (int) $this->database->lastInsertId();
    }

    public function find(int $id): ?array
    {
        $statement = $this->database->prepare(
            'SELECT id, title FROM notes WHERE id = :id',
        );
        $statement->execute(['id' => $id]);
        $note = $statement->fetch(PDO::FETCH_ASSOC);

        return $note === false ? null : $note;
    }
}
~~~

The repository receives its PDO connection instead of opening a new one. The test and endpoint can therefore pass in the connection they have configured. Prepared statements keep values separate from SQL text.
## 3. Test the repository against a real SQLite driver

### File: tests/Integration/NoteRepositoryIntegrationTest.php

~~~php
<?php

declare(strict_types=1);

use PHPUnit\Framework\TestCase;

require_once dirname(__DIR__, 2) . '/src/schema.php';
require_once dirname(__DIR__, 2) . '/src/NoteRepository.php';

final class NoteRepositoryIntegrationTest extends TestCase
{
    private PDO $database;
    private NoteRepository $notes;

    protected function setUp(): void
    {
        if (!in_array('sqlite', PDO::getAvailableDrivers(), true)) {
            self::markTestSkipped('The PDO SQLite driver is required.');
        }

        $this->database = new PDO('sqlite::memory:', options: [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]);

        createSchema($this->database);
        $this->notes = new NoteRepository($this->database);
    }

    public function testItPersistsAndFindsANote(): void
    {
        $id = $this->notes->create('Integration test');
        $note = $this->notes->find($id);

        self::assertNotNull($note);
        self::assertSame($id, (int) $note['id']);
        self::assertSame('Integration test', $note['title']);
    }

    public function testTheDatabaseRejectsAWhitespaceOnlyTitle(): void
    {
        $this->expectException(PDOException::class);

        $this->notes->create('   ');
    }
}
~~~

Each test gets a new in-memory connection and a new schema from setUp. The second test verifies the actual SQLite CHECK constraint rather than assuming that NOT NULL rejects an empty string.

Run it from the project root after PHPUnit is installed:

~~~sh
php vendor/bin/phpunit tests/Integration/NoteRepositoryIntegrationTest.php
~~~

Representative output:

~~~text
PHPUnit 13.4.x

..                                                                  2 / 2 (100%)

2 tests passed
~~~

The exact progress display and version vary. PHPUnit 13 requires a recent PHP runtime; use a PHPUnit release compatible with your project's PHP version. The code and output here are instructional and were not executed for this lesson.
## 4. Handle a small JSON HTTP API

### File: public/index.php

~~~php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/src/schema.php';
require_once dirname(__DIR__) . '/src/NoteRepository.php';

function respond(int $status, array $payload): void
{
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    echo json_encode($payload, JSON_THROW_ON_ERROR);
}

$databasePath = getenv('APP_DB_PATH');

if ($databasePath === false || $databasePath === '') {
    respond(500, ['error' => 'APP_DB_PATH is not configured']);
    return;
}

$database = new PDO('sqlite:' . $databasePath, options: [
    PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
    PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
]);

createSchema($database);
$notes = new NoteRepository($database);
$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);

if ($method === 'POST' && $path === '/notes') {
    try {
        $payload = json_decode(
            file_get_contents('php://input'),
            true,
            flags: JSON_THROW_ON_ERROR,
        );
    } catch (JsonException) {
        respond(400, ['error' => 'Request body must be valid JSON']);
        return;
    }

    $title = is_array($payload) ? ($payload['title'] ?? null) : null;

    if (!is_string($title) || trim($title) === '') {
        respond(422, ['error' => 'A non-empty title is required']);
        return;
    }

    $title = trim($title);
    $id = $notes->create($title);
    respond(201, ['data' => ['id' => $id, 'title' => $title]]);
    return;
}
if ($method === 'GET' && $path === '/notes') {
    $id = filter_var(
        $_GET['id'] ?? null,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1]],
    );

    if ($id === false) {
        respond(400, ['error' => 'A positive note id is required']);
        return;
    }

    $note = $notes->find((int) $id);

    if ($note === null) {
        respond(404, ['error' => 'Note not found']);
        return;
    }

    respond(200, ['data' => $note]);
    return;
}

respond(404, ['error' => 'Route not found']);
~~~

The endpoint validates JSON and input, then uses the same repository as the database test. In a full application, routing, validation, and response creation would usually live in framework components; this small front controller keeps the example self-contained.

## 5. Send real HTTP requests

### File: tests/http-smoke.php

~~~php
<?php

declare(strict_types=1);

function requestJson(string $method, string $url, ?array $payload = null): array
{
    $options = [
        'method' => $method,
        'ignore_errors' => true,
        'timeout' => 3,
    ];

    if ($payload !== null) {
        $options['header'] = 'Content-Type: application/json';
        $options['content'] = json_encode($payload, JSON_THROW_ON_ERROR);
    }

    $context = stream_context_create(['http' => $options]);
    $body = file_get_contents($url, false, $context);
    $statusLine = $http_response_header[0] ?? '';
    $statusParts = explode(' ', $statusLine);
    $status = (int) ($statusParts[1] ?? 0);

    if ($body === false || $status === 0) {
        throw new RuntimeException('The local HTTP server did not return a response.');
    }

    return [$status, json_decode($body, true, flags: JSON_THROW_ON_ERROR)];
}

$baseUrl = getenv('TEST_BASE_URL') ?: 'http://127.0.0.1:8085';
[$createStatus, $created] = requestJson(
    'POST',
    $baseUrl . '/notes',
    ['title' => 'Integration test'],
);

if ($createStatus !== 201) {
    throw new RuntimeException('Expected HTTP 201 when creating a note.');
}

$id = $created['data']['id'];
[$readStatus, $read] = requestJson('GET', $baseUrl . '/notes?id=' . $id);

if ($readStatus !== 200 || $read['data']['title'] !== 'Integration test') {
    throw new RuntimeException('The created note could not be read back.');
}

echo 'HTTP create: ' . $createStatus . PHP_EOL;
echo 'HTTP read: ' . $readStatus . PHP_EOL;
echo 'Created note: ' . $read['data']['title'] . PHP_EOL;
~~~
Start the PHP development server in one PowerShell terminal from the project root:

~~~powershell
$env:APP_DB_PATH = Join-Path $env:TEMP 'php-integration-test.sqlite'
Remove-Item -LiteralPath $env:APP_DB_PATH -ErrorAction SilentlyContinue
php -S 127.0.0.1:8085 -t public public/index.php
~~~

The database path points to a disposable file in the system temporary directory. The server creates its schema on the first request. Keep this server bound to loopback; do not expose it to a public network.

In a second terminal, from the same project root, run the HTTP client:

~~~powershell
$env:TEST_BASE_URL = 'http://127.0.0.1:8085'
php tests/http-smoke.php
~~~

Expected output:

~~~text
HTTP create: 201
HTTP read: 200
Created note: Integration test
~~~
The first request sends JSON to the running PHP server. The endpoint validates it, writes through PDO to SQLite, and returns HTTP 201. The second request reads the saved row and returns HTTP 200. This crosses both the HTTP process and database boundaries.

When finished, stop the server with Ctrl+C. Remove the temporary database if it is no longer needed:

~~~powershell
Remove-Item -LiteralPath $env:APP_DB_PATH -ErrorAction SilentlyContinue
~~~

## What this example verifies

- The repository executes real SQL through PDO.
- SQLite enforces the schema CHECK constraint.
- Each PHPUnit database test starts with a new in-memory database.
- The HTTP client sends a request to a separate local server process.
- The endpoint response status and persisted note are checked.
- Test data stays in a disposable temporary database.

The HTTP smoke client is an example script, not part of the PHPUnit suite. A larger application can automate server startup and cleanup, or use a framework HTTP client for most request-pipeline checks. Keep a server-level check when server configuration itself matters.

If requests fail, confirm PDO SQLite is enabled, the server is running, the port is available, and both terminals use the same TEST_BASE_URL. The sample code and representative output were not executed here.
## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [118 - Unit tests, fixtures, and data providers](../lessons/118-unit-tests-fixtures-and-data-providers.md) | [Q&A](../qnas/119-integration-tests-for-databases-and-http.md) | [Lesson](../lessons/119-integration-tests-for-databases-and-http.md) | [120 - Test doubles, stubs, and mocks](../lessons/120-test-doubles-stubs-and-mocks.md) |
