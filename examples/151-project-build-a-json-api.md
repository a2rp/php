# Project: build a JSON API — Example

Build a small task API with PHP 8, PDO, and SQLite. It accepts JSON and supports listing, creating, reading, updating, and deleting tasks.

This is a local learning project. It has no authentication, so do not expose it to the internet. A real service must authenticate clients and authorize access to each task.

## What you will build

| Method | Path | Result |
| --- | --- | --- |
| GET | /api/tasks | List tasks |
| POST | /api/tasks | Create a task |
| GET | /api/tasks/{id} | Read one task |
| PATCH | /api/tasks/{id} | Update selected task fields |
| DELETE | /api/tasks/{id} | Delete a task |

Successful JSON responses use a data envelope. Errors use a consistent error envelope. The server validates the request before it accesses the database.

## Requirements

- PHP 8.0 or later.
- PDO with the PDO_SQLite driver enabled.
- The mbstring extension for Unicode-aware title and description length checks.
- PowerShell for the request examples below, or another HTTP client.

Check PHP extensions with:

~~~powershell
php -m
~~~

Look for PDO, pdo_sqlite, and mbstring in the output.

## Project structure

~~~text
task-api/
├── .gitignore
├── bin/
│   └── init-db.php
├── database/
│   └── schema.sql
├── public/
│   ├── api.php
│   └── router.php
├── src/
│   ├── bootstrap.php
│   ├── task-repository.php
│   └── task-validation.php
└── var/
    └── .gitkeep
~~~

Only the public directory is served by the development server. The SQLite database stays in var, outside the document root.

## 1. Ignore the generated database

Create **.gitignore**:

~~~gitignore
/var/*.sqlite
/var/*.sqlite-*
!/var/.gitkeep
~~~

The database is generated locally and should not be committed. In a real application, plan database backups and migrations separately.

## 2. Define the database schema

Create **database/schema.sql**:

~~~sql
CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL CHECK (length(title) BETWEEN 1 AND 200),
    description TEXT NOT NULL DEFAULT '' CHECK (length(description) <= 2000),
    status TEXT NOT NULL DEFAULT 'open'
        CHECK (status IN ('open', 'done')),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
~~~

SQLite constraints protect the database if another code path forgets a validation rule. The PHP layer still validates requests so clients receive useful errors.

## 3. Connect to SQLite and return JSON

Create **src/bootstrap.php**:

~~~php
<?php

declare(strict_types=1);

const MAX_BODY_BYTES = 65536;

function database(): PDO
{
    $directory = dirname(__DIR__) . '/var';

    if (
        !is_dir($directory)
        && !mkdir($directory, 0775, true)
        && !is_dir($directory)
    ) {
        throw new RuntimeException('Could not create the database directory.');
    }

    $pdo = new PDO('sqlite:' . $directory . '/tasks.sqlite', options: [
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        PDO::ATTR_EMULATE_PREPARES => false,
    ]);

    $pdo->exec('PRAGMA foreign_keys = ON');
    $pdo->exec('PRAGMA busy_timeout = 5000');

    return $pdo;
}

function sendJson(int $status, array $payload = [], array $headers = []): void
{
    $body = '';

    if ($status !== 204) {
        try {
            $body = json_encode(
                $payload,
                JSON_THROW_ON_ERROR | JSON_UNESCAPED_UNICODE
            );
        } catch (JsonException $exception) {
            $status = 500;
            $body = '{"error":{"code":"internal_error","message":"An unexpected server error occurred."}}';
        }
    }

    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    header('Cache-Control: no-store');

    foreach ($headers as $name => $value) {
        header($name . ': ' . $value);
    }

    if ($body !== '') {
        echo $body;
    }

    exit;
}

function sendError(
    int $status,
    string $code,
    string $message,
    array $fields = [],
    array $headers = []
): void {
    $error = [
        'code' => $code,
        'message' => $message,
    ];

    if ($fields !== []) {
        $error['fields'] = $fields;
    }

    sendJson($status, ['error' => $error], $headers);
}

function readJsonObject(): stdClass
{
    $contentType = strtolower(trim(
        explode(';', $_SERVER['CONTENT_TYPE'] ?? '', 2)[0]
    ));

    if ($contentType !== 'application/json') {
        sendError(
            415,
            'unsupported_media_type',
            'Send this request with Content-Type: application/json.'
        );
    }

    $rawBody = file_get_contents(
        'php://input',
        false,
        null,
        0,
        MAX_BODY_BYTES + 1
    );

    if ($rawBody === false) {
        sendError(400, 'invalid_request_body', 'The request body could not be read.');
    }

    if (strlen($rawBody) > MAX_BODY_BYTES) {
        sendError(413, 'request_too_large', 'The request body is too large.');
    }

    try {
        $payload = json_decode($rawBody, false, 512, JSON_THROW_ON_ERROR);
    } catch (JsonException $exception) {
        sendError(400, 'invalid_json', 'The request body is not valid JSON.');
    }

    if (!$payload instanceof stdClass) {
        sendError(400, 'json_object_required', 'Send one JSON object in the request body.');
    }

    return $payload;
}
~~~

The request size is bounded before decoding. JSON syntax errors become a safe 400 response, and a valid JSON array is rejected because this API expects one object.

## 4. Validate task fields

Create **src/task-validation.php**:

~~~php
<?php

declare(strict_types=1);

function validateTask(stdClass $payload, bool $partial, ?array $current = null): array
{
    $input = get_object_vars($payload);
    $allowedFields = ['title', 'description', 'status'];
    $unknownFields = array_diff(array_keys($input), $allowedFields);
    $errors = [];

    if ($unknownFields !== []) {
        $errors['_request'] = 'The request contains unsupported fields.';
    }

    if ($partial && $input === []) {
        $errors['_request'] = 'Provide at least one field to update.';
    }

    $task = $current ?? [
        'title' => '',
        'description' => '',
        'status' => 'open',
    ];

    if (!$partial && !array_key_exists('title', $input)) {
        $errors['title'] = 'A title is required.';
    }

    foreach ($allowedFields as $field) {
        if (!array_key_exists($field, $input)) {
            continue;
        }

        if (!is_string($input[$field])) {
            $errors[$field] = 'This field must be a string.';
            continue;
        }

        $value = trim($input[$field]);

        if ($field === 'title') {
            if ($value === '') {
                $errors['title'] = 'Enter a title.';
            } elseif (mb_strlen($value, 'UTF-8') > 200) {
                $errors['title'] = 'The title must be no longer than 200 characters.';
            }
        }

        if ($field === 'description' && mb_strlen($value, 'UTF-8') > 2000) {
            $errors['description'] = 'The description must be no longer than 2,000 characters.';
        }

        if ($field === 'status' && !in_array($value, ['open', 'done'], true)) {
            $errors['status'] = 'Choose either open or done.';
        }

        $task[$field] = $value;
    }

    return [
        'task' => $task,
        'errors' => $errors,
    ];
}
~~~

Create requests need a title. Description and status have defaults. PATCH starts with the current task and changes only fields present in the request. Unknown fields are rejected instead of silently becoming database columns.

## 5. Keep SQL operations in one repository

Create **src/task-repository.php**:

~~~php
<?php

declare(strict_types=1);

function formatTask(array $row): array
{
    $row['id'] = (int) $row['id'];

    return $row;
}

function listTasks(PDO $pdo): array
{
    $statement = $pdo->query(
        'SELECT id, title, description, status, created_at
         FROM tasks
         ORDER BY id ASC'
    );

    return array_map('formatTask', $statement->fetchAll());
}

function findTask(PDO $pdo, int $id): ?array
{
    $statement = $pdo->prepare(
        'SELECT id, title, description, status, created_at
         FROM tasks
         WHERE id = :id'
    );
    $statement->execute(['id' => $id]);
    $row = $statement->fetch();

    return $row === false ? null : formatTask($row);
}

function createTask(PDO $pdo, array $task): array
{
    $statement = $pdo->prepare(
        'INSERT INTO tasks (title, description, status)
         VALUES (:title, :description, :status)'
    );
    $statement->execute([
        'title' => $task['title'],
        'description' => $task['description'],
        'status' => $task['status'],
    ]);

    return findTask($pdo, (int) $pdo->lastInsertId())
        ?? throw new RuntimeException('The created task could not be loaded.');
}

function updateTask(PDO $pdo, int $id, array $task): array
{
    $statement = $pdo->prepare(
        'UPDATE tasks
         SET title = :title, description = :description, status = :status
         WHERE id = :id'
    );
    $statement->execute([
        'title' => $task['title'],
        'description' => $task['description'],
        'status' => $task['status'],
        'id' => $id,
    ]);

    return findTask($pdo, $id)
        ?? throw new RuntimeException('The updated task could not be loaded.');
}

function deleteTask(PDO $pdo, int $id): bool
{
    $statement = $pdo->prepare('DELETE FROM tasks WHERE id = :id');
    $statement->execute(['id' => $id]);

    return $statement->rowCount() > 0;
}
~~~

All client-provided values are bound as parameters. The SQL column names and statement structure remain fixed. A prepared statement separates a value from SQL syntax; it does not perform authorization.

## 6. Match API routes and methods

Create **public/router.php**:

~~~php
<?php

declare(strict_types=1);

$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);

if (is_string($path) && str_starts_with($path, '/api/')) {
    require __DIR__ . '/api.php';
    return true;
}

return false;
~~~

The built-in server calls this router for requests. API paths go to one front controller; normal file requests are passed through. This router is for local development. A production web server needs equivalent routing rules.

Create **public/api.php**:

~~~php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/src/bootstrap.php';
require_once dirname(__DIR__) . '/src/task-repository.php';
require_once dirname(__DIR__) . '/src/task-validation.php';

function methodNotAllowed(array $methods): void
{
    sendError(
        405,
        'method_not_allowed',
        'This method is not allowed for the requested resource.',
        headers: ['Allow' => implode(', ', $methods)]
    );
}

$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);
$method = strtoupper($_SERVER['REQUEST_METHOD'] ?? 'GET');

if (!is_string($path)) {
    sendError(400, 'invalid_path', 'The request path is invalid.');
}

$isCollection = $path === '/api/tasks';
$itemId = null;

if (!$isCollection) {
    if (!preg_match('~\A/api/tasks/([0-9]+)\z~D', $path, $matches)) {
        sendError(404, 'not_found', 'The requested endpoint was not found.');
    }

    $itemId = filter_var(
        $matches[1],
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1]]
    );

    if ($itemId === false) {
        sendError(400, 'invalid_id', 'The task ID must be a positive integer.');
    }
}

try {
    $pdo = database();

    if ($isCollection) {
        if ($method === 'GET') {
            sendJson(200, ['data' => listTasks($pdo)]);
        }

        if ($method === 'POST') {
            $result = validateTask(readJsonObject(), partial: false);

            if ($result['errors'] !== []) {
                sendError(
                    422,
                    'validation_failed',
                    'The request contains invalid fields.',
                    $result['errors']
                );
            }

            $task = createTask($pdo, $result['task']);

            sendJson(
                201,
                ['data' => $task],
                ['Location' => '/api/tasks/' . $task['id']]
            );
        }

        methodNotAllowed(['GET', 'POST']);
    }

    if (!in_array($method, ['GET', 'PATCH', 'DELETE'], true)) {
        methodNotAllowed(['GET', 'PATCH', 'DELETE']);
    }

    $task = findTask($pdo, $itemId);

    if ($task === null) {
        sendError(404, 'task_not_found', 'The requested task was not found.');
    }

    if ($method === 'GET') {
        sendJson(200, ['data' => $task]);
    }

    if ($method === 'PATCH') {
        $result = validateTask(readJsonObject(), partial: true, current: $task);

        if ($result['errors'] !== []) {
            sendError(
                422,
                'validation_failed',
                'The request contains invalid fields.',
                $result['errors']
            );
        }

        sendJson(200, ['data' => updateTask($pdo, $itemId, $result['task'])]);
    }

    if ($method === 'DELETE') {
        if (!deleteTask($pdo, $itemId)) {
            sendError(404, 'task_not_found', 'The requested task was not found.');
        }

        sendJson(204);
    }
} catch (Throwable $exception) {
    error_log('[task-api] ' . $exception);
    sendError(500, 'internal_error', 'An unexpected server error occurred.');
}
~~~

The collection allows GET and POST. A task item allows GET, PATCH, and DELETE. Unsupported methods return 405 with an Allow header. Unexpected exceptions are logged; the response does not reveal internal details.

## 7. Initialize and start the local API

Create **bin/init-db.php**:

~~~php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/src/bootstrap.php';

$schemaPath = dirname(__DIR__) . '/database/schema.sql';
$schema = file_get_contents($schemaPath);

if ($schema === false) {
    throw new RuntimeException('Could not read the database schema.');
}

$pdo = database();
$pdo->exec($schema);

echo "Database initialized." . PHP_EOL;
~~~

Run these commands from the task-api directory:

~~~powershell
php bin/init-db.php
php -S 127.0.0.1:8000 -t public public/router.php
~~~

The server binds to the loopback address, so it is available only on your own computer. Keep this development server off public networks.

## 8. Send requests from PowerShell

Open another PowerShell terminal. Create a task:

~~~powershell
$createRequest = @{
  Method = 'Post'
  Uri = 'http://127.0.0.1:8000/api/tasks'
  ContentType = 'application/json'
  Headers = @{ Accept = 'application/json' }
  Body = (@{
    title = 'Read the PHP API chapter'
    description = 'Try the task endpoints'
    status = 'open'
  } | ConvertTo-Json)
}
$created = Invoke-RestMethod @createRequest
$created.data
~~~

The response has status 201 and a body like:

~~~json
{
  "data": {
    "id": 1,
    "title": "Read the PHP API chapter",
    "description": "Try the task endpoints",
    "status": "open",
    "created_at": "2026-11-21 12:00:00"
  }
}
~~~

The timestamp depends on when the row was inserted. The response also includes Location: /api/tasks/1.

List tasks:

~~~powershell
Invoke-RestMethod -Method Get -Uri 'http://127.0.0.1:8000/api/tasks'
~~~

Read the created task:

~~~powershell
$taskId = $created.data.id
Invoke-RestMethod -Method Get -Uri "http://127.0.0.1:8000/api/tasks/$taskId"
~~~

Update only the status:

~~~powershell
$patchRequest = @{
  Method = 'Patch'
  Uri = "http://127.0.0.1:8000/api/tasks/$taskId"
  ContentType = 'application/json'
  Body = (@{ status = 'done' } | ConvertTo-Json)
}
$updated = Invoke-RestMethod @patchRequest
$updated.data
~~~

Delete the task:

~~~powershell
Invoke-WebRequest -Method Delete -Uri "http://127.0.0.1:8000/api/tasks/$taskId"
~~~

A successful delete returns 204 No Content and has no JSON response body.

## 9. Inspect a validation error

Try creating a task with a blank title:

~~~powershell
$invalidRequest = @{
  Method = 'Post'
  Uri = 'http://127.0.0.1:8000/api/tasks'
  ContentType = 'application/json'
  Body = (@{ title = '   ' } | ConvertTo-Json)
}

try {
  Invoke-RestMethod @invalidRequest
} catch {
  $_.ErrorDetails.Message
}
~~~

The API returns status 422 with an error body like:

~~~json
{
  "error": {
    "code": "validation_failed",
    "message": "The request contains invalid fields.",
    "fields": {
      "title": "Enter a title."
    }
  }
}
~~~

Malformed JSON returns 400. A non-JSON Content-Type returns 415. A missing task returns 404. These distinct statuses help clients decide whether to fix the request, change the method, or handle a missing resource.

## Expected behavior

- GET /api/tasks initially returns 200 with an empty data array.
- A valid POST returns 201, includes the new task, and sets Location.
- GET for that task returns 200.
- PATCH changes only supplied fields and returns 200 with the updated task.
- DELETE returns 204 with an empty body.
- A request for the deleted task returns 404.
- Invalid task fields return 422 with field-level details.
- Unsupported methods return 405 and an Allow header.
- Unexpected server failures return a generic 500 response; diagnostic details go to the server log.

## Important limitations

- The API has no authentication or per-task authorization. Anyone who can reach it can manage all tasks.
- CORS is not access control. Add server-side authentication and authorization before deployment.
- If authentication uses cookies, add appropriate CSRF protection.
- The sample does not implement pagination, rate limiting, API versioning, migrations, or idempotency keys.
- The PHP built-in server is for local development. Configure production routing, HTTPS, logging, backups, and access controls separately.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [150 - Project: build a database-backed CRUD application](../lessons/150-project-build-a-database-backed-crud-application.md) | [Q&A](../qnas/151-project-build-a-json-api.md) | [Lesson](../lessons/151-project-build-a-json-api.md) | [152 - Capstone: plan, build, test, and document an application](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) |
