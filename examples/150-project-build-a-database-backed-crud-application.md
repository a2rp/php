# Project: build a database-backed CRUD application — Example

## Goal

Build a small task manager that stores tasks in SQLite and supports create, read, update, and delete. The example uses prepared SQL statements, server-side validation, CSRF tokens, post/redirect/get, and escaped HTML output.

This is a single-user learning application. Do not expose it publicly until authentication and per-task authorization are added.

## Requirements

- PHP 8.0 or later.
- PDO with the PDO_SQLite driver.
- The mbstring extension for character-count validation.
- A writable project **var/** directory.
- A production server configured to expose only **public/**.

## Project structure

~~~text
task-manager/
├── .gitignore
├── bin/
│   └── init-db.php
├── database/
│   └── schema.sql
├── public/
│   ├── index.php
│   └── assets/
│       └── styles.css
├── src/
│   ├── bootstrap.php
│   ├── task-repository.php
│   └── task-validation.php
└── var/
    └── tasks.sqlite       generated locally; do not commit
~~~

The SQLite file and PHP source are outside the public document root. The database initialization command creates the **var/** directory when needed.

## 1. Ignore generated database files

### File: .gitignore

~~~gitignore
/var/*.sqlite
/var/*.sqlite-journal
/var/*.sqlite-shm
/var/*.sqlite-wal
~~~

The database contains application data, so keep it out of version control. A real application also needs a protected backup and restore plan.

## 2. Define the task table

### File: database/schema.sql

~~~sql
CREATE TABLE IF NOT EXISTS tasks (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL CHECK (length(title) BETWEEN 1 AND 200),
    description TEXT NOT NULL DEFAULT '',
    status TEXT NOT NULL DEFAULT 'open'
        CHECK (status IN ('open', 'done')),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
~~~

The database enforces important rules as a second layer. The PHP handler still validates input so visitors receive understandable feedback.

## 3. Initialize the local database

### File: bin/init-db.php

~~~php
<?php
declare(strict_types=1);

$projectRoot = dirname(__DIR__);
$varDirectory = $projectRoot . '/var';

if (!is_dir($varDirectory)) {
    if (!mkdir($varDirectory, 0770, true) && !is_dir($varDirectory)) {
        throw new RuntimeException('Could not create the application data directory.');
    }
}

$schemaPath = $projectRoot . '/database/schema.sql';
$schema = file_get_contents($schemaPath);

if ($schema === false) {
    throw new RuntimeException('Could not read the database schema.');
}

$pdo = new PDO('sqlite:' . $varDirectory . '/tasks.sqlite');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$pdo->exec($schema);

echo "Database initialized.\n";
~~~

Run the initializer once from the project root:

~~~sh
php bin/init-db.php
~~~

The schema uses CREATE TABLE IF NOT EXISTS so rerunning the initializer does not delete existing tasks. For later schema changes, use ordered migrations rather than treating this initializer as a migration system.

## 4. Shared connection, escaping, session, and CSRF helpers

### File: src/bootstrap.php

~~~php
<?php
declare(strict_types=1);

function databaseConnection(): PDO
{
    $databasePath = dirname(__DIR__) . '/var/tasks.sqlite';

    if (!is_file($databasePath)) {
        throw new RuntimeException(
            'Database is missing. Run php bin/init-db.php from the project root.'
        );
    }

    $pdo = new PDO('sqlite:' . $databasePath);
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);

    return $pdo;
}

function startApplicationSession(): void
{
    if (session_status() === PHP_SESSION_ACTIVE) {
        return;
    }

    $isHttps = isset($_SERVER['HTTPS'])
        && $_SERVER['HTTPS'] !== ''
        && $_SERVER['HTTPS'] !== 'off';

    ini_set('session.use_strict_mode', '1');

    session_set_cookie_params([
        'lifetime' => 0,
        'path' => '/',
        'secure' => $isHttps,
        'httponly' => true,
        'samesite' => 'Lax',
    ]);

    session_start();
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

function csrfToken(): string
{
    if (
        !isset($_SESSION['csrf_token'])
        || !is_string($_SESSION['csrf_token'])
    ) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }

    return $_SESSION['csrf_token'];
}

function isValidCsrfToken(mixed $submittedToken): bool
{
    $storedToken = $_SESSION['csrf_token'] ?? null;

    return is_string($submittedToken)
        && is_string($storedToken)
        && hash_equals($storedToken, $submittedToken);
}

function postText(string $key): string
{
    $value = $_POST[$key] ?? null;

    return is_string($value) ? trim($value) : '';
}

function positiveId(mixed $value): ?int
{
    if (!is_string($value) && !is_int($value)) {
        return null;
    }

    $id = filter_var(
        $value,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1]]
    );

    return $id === false ? null : $id;
}
~~~

Configure HTTPS detection correctly if a trusted reverse proxy sits in front of the PHP application. Do not trust arbitrary forwarded headers supplied by a visitor.

## 5. Validate task fields

### File: src/task-validation.php

~~~php
<?php
declare(strict_types=1);

/**
 * @param array{title: string, description: string, status: string} $values
 * @return array<string, string>
 */
function validateTask(array $values): array
{
    $errors = [];

    if ($values['title'] === '') {
        $errors['title'] = 'Enter a task title.';
    } elseif (
        !mb_check_encoding($values['title'], 'UTF-8')
        || mb_strlen($values['title'], 'UTF-8') > 200
    ) {
        $errors['title'] = 'Title must be valid UTF-8 and no more than 200 characters.';
    }

    if (
        !mb_check_encoding($values['description'], 'UTF-8')
        || mb_strlen($values['description'], 'UTF-8') > 2000
    ) {
        $errors['description'] =
            'Description must be valid UTF-8 and no more than 2000 characters.';
    }

    if (!in_array($values['status'], ['open', 'done'], true)) {
        $errors['status'] = 'Choose an allowed task status.';
    }

    return $errors;
}
~~~

The HTML form will also use required, maxlength, and select options. Those attributes help the visitor; this PHP function remains the enforcement point.

## 6. Keep SQL in repository functions

### File: src/task-repository.php

~~~php
<?php
declare(strict_types=1);

/** @return list<array<string, mixed>> */
function allTasks(PDO $pdo): array
{
    $statement = $pdo->query(
        "SELECT id, title, description, status, created_at, updated_at
         FROM tasks
         ORDER BY CASE status WHEN 'open' THEN 0 ELSE 1 END, id DESC"
    );

    return $statement->fetchAll();
}

/** @return array<string, mixed>|null */
function findTask(PDO $pdo, int $id): ?array
{
    $statement = $pdo->prepare(
        'SELECT id, title, description, status, created_at, updated_at
         FROM tasks
         WHERE id = :id'
    );
    $statement->execute(['id' => $id]);

    $task = $statement->fetch();

    return $task === false ? null : $task;
}

/**
 * @param array{title: string, description: string, status: string} $values
 */
function createTask(PDO $pdo, array $values): void
{
    $statement = $pdo->prepare(
        'INSERT INTO tasks (title, description, status)
         VALUES (:title, :description, :status)'
    );
    $statement->execute([
        'title' => $values['title'],
        'description' => $values['description'],
        'status' => $values['status'],
    ]);
}

/**
 * @param array{title: string, description: string, status: string} $values
 */
function updateTask(PDO $pdo, int $id, array $values): void
{
    $statement = $pdo->prepare(
        'UPDATE tasks
         SET title = :title,
             description = :description,
             status = :status,
             updated_at = CURRENT_TIMESTAMP
         WHERE id = :id'
    );
    $statement->execute([
        'title' => $values['title'],
        'description' => $values['description'],
        'status' => $values['status'],
        'id' => $id,
    ]);
}

function deleteTask(PDO $pdo, int $id): bool
{
    $statement = $pdo->prepare('DELETE FROM tasks WHERE id = :id');
    $statement->execute(['id' => $id]);

    return $statement->rowCount() === 1;
}
~~~

All user-provided values are parameters. The table and column names are fixed in the SQL. The update operation does not rely on rowCount to decide whether a record existed; the request handler checks the record first.

## 7. Handle requests and render the page

### File: public/index.php

~~~php
<?php
declare(strict_types=1);

require_once __DIR__ . '/../src/bootstrap.php';
require_once __DIR__ . '/../src/task-validation.php';
require_once __DIR__ . '/../src/task-repository.php';

startApplicationSession();
$token = csrfToken();

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
if (!in_array($method, ['GET', 'POST'], true)) {
    header('Allow: GET, POST');
    http_response_code(405);
}

$flashMessage = $_SESSION['flash_message'] ?? null;
unset($_SESSION['flash_message']);
$flashMessage = is_string($flashMessage) ? $flashMessage : null;

$values = [
    'title' => '',
    'description' => '',
    'status' => 'open',
];
$errors = [];
$formMode = 'create';
$editingId = null;
$tasks = [];
$databaseError = false;

try {
    $pdo = databaseConnection();

    if ($method === 'POST') {
        $action = postText('action');
        $submittedToken = $_POST['csrf_token'] ?? null;

        if (!isValidCsrfToken($submittedToken)) {
            $errors['form'] = 'The request could not be verified. Reload the page and try again.';
        } elseif ($action === 'create' || $action === 'update') {
            $values = [
                'title' => postText('title'),
                'description' => postText('description'),
                'status' => postText('status'),
            ];
            $errors = validateTask($values);

            if ($action === 'update') {
                $formMode = 'update';
                $editingId = positiveId($_POST['id'] ?? null);

                if ($editingId === null) {
                    $errors['form'] = 'Choose a valid task to update.';
                } elseif (findTask($pdo, $editingId) === null) {
                    $errors['form'] = 'That task no longer exists.';
                }
            }

            if ($errors === []) {
                if ($action === 'create') {
                    createTask($pdo, $values);
                    $_SESSION['flash_message'] = 'Task created.';
                } else {
                    updateTask($pdo, $editingId, $values);
                    $_SESSION['flash_message'] = 'Task updated.';
                }

                $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
                header('Location: index.php', true, 303);
                exit;
            }
        } elseif ($action === 'delete') {
            $taskId = positiveId($_POST['id'] ?? null);

            if ($taskId === null) {
                $errors['form'] = 'Choose a valid task to delete.';
            } elseif (findTask($pdo, $taskId) === null) {
                $errors['form'] = 'That task no longer exists.';
            } elseif (!deleteTask($pdo, $taskId)) {
                $errors['form'] = 'The task could not be deleted.';
            } else {
                $_SESSION['flash_message'] = 'Task deleted.';
                $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
                header('Location: index.php', true, 303);
                exit;
            }
        } else {
            $errors['form'] = 'Choose a valid task action.';
        }
    } elseif ($method === 'GET' && isset($_GET['edit'])) {
        $editingId = positiveId($_GET['edit']);

        if ($editingId === null) {
            $errors['form'] = 'Choose a valid task to edit.';
        } else {
            $task = findTask($pdo, $editingId);

            if ($task === null) {
                $errors['form'] = 'That task no longer exists.';
            } else {
                $formMode = 'update';
                $values = [
                    'title' => (string) $task['title'],
                    'description' => (string) $task['description'],
                    'status' => (string) $task['status'],
                ];
            }
        }
    }

    $tasks = allTasks($pdo);
} catch (PDOException | RuntimeException $exception) {
    error_log('Task manager database operation failed: ' . $exception->getMessage());
    $databaseError = true;
    $tasks = [];
}

if ($databaseError) {
    http_response_code(500);
}
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="A small PHP and SQLite task manager.">
    <title>Task manager</title>
    <link rel="stylesheet" href="assets/styles.css">
</head>
<body>
    <main class="page">
        <header class="page-header">
            <p class="eyebrow">PHP project</p>
            <h1>Task manager</h1>
            <p>Create tasks, keep them in SQLite, and update them as work changes.</p>
        </header>

        <?php if ($databaseError): ?>
            <div class="notice notice-error" role="alert">
                The task database is unavailable. Check the server setup and try again.
            </div>
        <?php endif; ?>

        <?php if ($flashMessage !== null): ?>
            <p class="notice notice-success" role="status">
                <?= escapeHtml($flashMessage) ?>
            </p>
        <?php endif; ?>

        <?php if ($errors !== []): ?>
            <div class="notice notice-error" role="alert" tabindex="-1">
                <p>Please review the following:</p>
                <ul>
                    <?php foreach ($errors as $error): ?>
                        <li><?= escapeHtml($error) ?></li>
                    <?php endforeach; ?>
                </ul>
            </div>
        <?php endif; ?>

        <section class="panel" aria-labelledby="form-heading">
            <h2 id="form-heading">
                <?= $formMode === 'update' ? 'Edit task' : 'Add a task' ?>
            </h2>

            <form method="post" action="index.php">
                <input type="hidden" name="csrf_token" value="<?= escapeHtml($token) ?>">
                <input type="hidden" name="action" value="<?= escapeHtml($formMode) ?>">

                <?php if ($formMode === 'update' && $editingId !== null): ?>
                    <input type="hidden" name="id" value="<?= (string) $editingId ?>">
                <?php endif; ?>

                <div class="field">
                    <label for="title">Title</label>
                    <input
                        id="title"
                        name="title"
                        type="text"
                        value="<?= escapeHtml($values['title']) ?>"
                        maxlength="200"
                        required
                        aria-invalid="<?= isset($errors['title']) ? 'true' : 'false' ?>"
                        aria-describedby="title-help<?= isset($errors['title']) ? ' title-error' : '' ?>"
                    >
                    <p class="field-help" id="title-help">Maximum 200 characters.</p>
                    <?php if (isset($errors['title'])): ?>
                        <p class="field-error" id="title-error">
                            <?= escapeHtml($errors['title']) ?>
                        </p>
                    <?php endif; ?>
                </div>

                <div class="field">
                    <label for="description">Description</label>
                    <textarea
                        id="description"
                        name="description"
                        rows="4"
                        maxlength="2000"
                        aria-invalid="<?= isset($errors['description']) ? 'true' : 'false' ?>"
                        aria-describedby="description-help<?= isset($errors['description']) ? ' description-error' : '' ?>"
                    ><?= escapeHtml($values['description']) ?></textarea>
                    <p class="field-help" id="description-help">
                        Optional. Maximum 2000 characters.
                    </p>
                    <?php if (isset($errors['description'])): ?>
                        <p class="field-error" id="description-error">
                            <?= escapeHtml($errors['description']) ?>
                        </p>
                    <?php endif; ?>
                </div>

                <div class="field">
                    <label for="status">Status</label>
                    <select
                        id="status"
                        name="status"
                        aria-invalid="<?= isset($errors['status']) ? 'true' : 'false' ?>"
                        aria-describedby="<?= isset($errors['status']) ? 'status-error' : 'status-help' ?>"
                    >
                        <option value="open" <?= $values['status'] === 'open' ? 'selected' : '' ?>>
                            Open
                        </option>
                        <option value="done" <?= $values['status'] === 'done' ? 'selected' : '' ?>>
                            Done
                        </option>
                    </select>
                    <p class="field-help" id="status-help">Choose the task's current status.</p>
                    <?php if (isset($errors['status'])): ?>
                        <p class="field-error" id="status-error">
                            <?= escapeHtml($errors['status']) ?>
                        </p>
                    <?php endif; ?>
                </div>

                <div class="form-actions">
                    <button type="submit">
                        <?= $formMode === 'update' ? 'Save changes' : 'Create task' ?>
                    </button>
                    <?php if ($formMode === 'update'): ?>
                        <a href="index.php">Cancel</a>
                    <?php endif; ?>
                </div>
            </form>
        </section>

        <section class="task-section" aria-labelledby="tasks-heading">
            <div class="section-heading">
                <h2 id="tasks-heading">Tasks</h2>
                <p><?= count($tasks) ?> total</p>
            </div>

            <?php if ($tasks === []): ?>
                <p class="empty-state">No tasks yet. Add one above to get started.</p>
            <?php else: ?>
                <ul class="task-list">
                    <?php foreach ($tasks as $task): ?>
                        <?php $taskId = (int) $task['id']; ?>
                        <li class="task-card">
                            <div class="task-content">
                                <div class="task-title-row">
                                    <h3><?= escapeHtml((string) $task['title']) ?></h3>
                                    <span class="status status-<?= escapeHtml((string) $task['status']) ?>">
                                        <?= escapeHtml(ucfirst((string) $task['status'])) ?>
                                    </span>
                                </div>

                                <?php if ((string) $task['description'] !== ''): ?>
                                    <p><?= escapeHtml((string) $task['description']) ?></p>
                                <?php else: ?>
                                    <p class="muted">No description.</p>
                                <?php endif; ?>

                                <p class="timestamp">
                                    Updated <?= escapeHtml((string) $task['updated_at']) ?>
                                </p>
                            </div>

                            <div class="task-actions">
                                <a href="index.php?edit=<?= $taskId ?>">Edit</a>

                                <form method="post" action="index.php">
                                    <input
                                        type="hidden"
                                        name="csrf_token"
                                        value="<?= escapeHtml($token) ?>"
                                    >
                                    <input type="hidden" name="action" value="delete">
                                    <input type="hidden" name="id" value="<?= $taskId ?>">
                                    <button class="button-delete" type="submit">
                                        Delete
                                    </button>
                                </form>
                            </div>
                        </li>
                    <?php endforeach; ?>
                </ul>
            <?php endif; ?>
        </section>
    </main>
</body>
</html>
~~~

## 8. Style the page

### File: public/assets/styles.css

~~~css
:root {
    font-family: system-ui, sans-serif;
    color: #1c2733;
    background: #f4f7fa;
    line-height: 1.55;
}

*,
*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
}

.page {
    width: min(100% - 2rem, 54rem);
    margin-inline: auto;
    padding-block: 2rem 5rem;
}

.page-header {
    margin-bottom: 2rem;
}

.eyebrow,
.field-help,
.timestamp,
.muted {
    color: #526476;
}

.eyebrow {
    margin-bottom: 0.25rem;
    font-size: 0.85rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
}

h1,
h2,
h3,
p {
    margin-top: 0;
}

h1 {
    margin-bottom: 0.5rem;
    font-size: clamp(2.25rem, 7vw, 3.5rem);
    line-height: 1.1;
}

.panel,
.task-card {
    padding: 1.25rem;
    border: 1px solid #ccd5de;
    border-radius: 0.6rem;
    background: #fff;
}

.panel {
    margin-bottom: 2.5rem;
}

.field {
    margin-bottom: 1.25rem;
}

label {
    display: block;
    margin-bottom: 0.35rem;
    font-weight: 700;
}

input,
textarea,
select {
    display: block;
    width: 100%;
    padding: 0.7rem;
    border: 1px solid #8293a3;
    border-radius: 0.35rem;
    color: inherit;
    background: #fff;
    font: inherit;
}

textarea {
    resize: vertical;
}

a {
    color: #075e9b;
}

a:focus-visible,
button:focus-visible,
input:focus-visible,
textarea:focus-visible,
select:focus-visible {
    outline: 3px solid #bd4d19;
    outline-offset: 3px;
}

.field-help,
.field-error {
    margin: 0.3rem 0 0;
    font-size: 0.93rem;
}

.field-error {
    color: #8b1e1e;
    font-weight: 650;
}

[aria-invalid="true"] {
    border-color: #a12424;
}

.form-actions,
.task-actions {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.8rem;
}

button {
    padding: 0.65rem 0.95rem;
    border: 0;
    border-radius: 0.35rem;
    color: #fff;
    background: #075e9b;
    font: inherit;
    font-weight: 700;
    cursor: pointer;
}

button:hover {
    background: #043c64;
}

.button-delete {
    color: #801d1d;
    background: #fff0f0;
    border: 1px solid #d7a5a5;
}

.button-delete:hover {
    color: #fff;
    background: #801d1d;
}

.notice {
    margin-bottom: 1.25rem;
    padding: 1rem;
    border: 1px solid;
    border-radius: 0.35rem;
}

.notice p:last-child,
.notice ul:last-child {
    margin-bottom: 0;
}

.notice-success {
    border-color: #28633d;
    color: #174326;
    background: #eef9f0;
}

.notice-error {
    border-color: #a12424;
    color: #681818;
    background: #fff0f0;
}

.section-heading,
.task-title-row {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: 1rem;
}

.section-heading p {
    color: #526476;
}

.task-list {
    display: grid;
    gap: 1rem;
    margin: 0;
    padding: 0;
    list-style: none;
}

.task-card {
    display: flex;
    justify-content: space-between;
    gap: 1.5rem;
}

.task-content {
    min-width: 0;
}

.task-title-row {
    justify-content: flex-start;
    flex-wrap: wrap;
}

.task-title-row h3 {
    margin-bottom: 0;
}

.status {
    padding: 0.15rem 0.6rem;
    border-radius: 99rem;
    background: #e8eef4;
    font-size: 0.85rem;
    font-weight: 700;
}

.status-done {
    color: #174326;
    background: #e3f3e7;
}

.timestamp {
    margin-bottom: 0;
    font-size: 0.85rem;
}

.task-actions form {
    margin: 0;
}

.empty-state {
    padding: 1.25rem;
    border: 1px dashed #8293a3;
    border-radius: 0.5rem;
    color: #526476;
    background: #fff;
}

@media (max-width: 42rem) {
    .task-card {
        flex-direction: column;
    }

    .task-actions {
        padding-top: 0.75rem;
        border-top: 1px solid #e0e6ec;
    }
}
~~~

## 9. Run the project

From the project root:

~~~sh
php bin/init-db.php
php -S localhost:8000 -t public
~~~

Open the local address in a browser. The PHP development server is for local development, not production hosting.

## Expected behavior

| Action | Result |
| --- | --- |
| First visit after initialization | The page shows an empty task list and a create form |
| Create a valid task | The row is inserted with a prepared statement and appears in the list |
| Submit an empty title | The page shows a validation error and does not write a task |
| Edit a task | The form is filled with its current values; saving updates its timestamp |
| Delete a task | A CSRF-protected POST removes the selected row |
| Refresh after a successful change | The browser reloads the list with a status message instead of repeating the POST |
| Database file is missing | The page displays a generic setup error; diagnostic detail goes to the server log |

## How to extend this project safely

- Add authentication before making it available to multiple people.
- Add an owner or account ID to each row and include it in every read, update, and delete authorization check.
- Add a migration tool and migration history before evolving the schema.
- Add tests for repository operations and request validation.
- Add backups and verify that a restore works.
- Keep the SQLite database and all secrets outside the public document root.
- For a public application, configure HTTPS, production logging, database permissions, and error display settings.

## Important scope limit

This example intentionally has no authentication. Anyone who can reach it can view, create, edit, and delete every task. CSRF protection helps validate the origin of a browser form submission; it does not identify the user or authorize access to a record.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [149 - Project: create a validated contact form](../lessons/149-project-create-a-validated-contact-form.md) | [Lesson](../lessons/150-project-build-a-database-backed-crud-application.md) | [Q&A](../qnas/150-project-build-a-database-backed-crud-application.md) | [151 - Project: build a JSON API](../lessons/151-project-build-a-json-api.md) |
