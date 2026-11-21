# Capstone: plan, build, test, and document an application — Example

Build a small personal reading tracker. It stores books in SQLite and lets one local user list, create, edit, and delete records.

The project demonstrates a complete workflow: a clear scope, database schema, routing, server-side validation, prepared statements, CSRF checks, escaped HTML, manual acceptance checks, and setup documentation.

This is a local single-user learning app. It has no login and must not be exposed as a public multi-user service.

## Requirements

- PHP 8.0 or later.
- PDO with the PDO_SQLite driver enabled.
- A browser and terminal.
- PowerShell for the commands below, or equivalent commands for your shell.

Check installed extensions with:

~~~powershell
php -m
~~~

The output should include PDO and pdo_sqlite.

## Scope and acceptance criteria

Version one includes:

- Add a book with a title, author, status, and optional notes.
- List saved books.
- Edit a book.
- Delete a book using a POST form.
- Keep records in SQLite between requests.
- Reject invalid fields and show useful errors.
- Include a CSRF token in every state-changing form.
- Escape dynamic text when rendering HTML.

It does not include accounts, multiple users, file uploads, ratings, recommendations, or public deployment.

## Project structure

~~~text
reading-tracker/
├── .gitignore
├── README.md
├── bin/
│   └── init-db.php
├── database/
│   └── schema.sql
├── public/
│   ├── index.php
│   ├── router.php
│   └── style.css
├── src/
│   ├── book-repository.php
│   ├── book-validation.php
│   └── bootstrap.php
├── templates/
│   ├── book-form.php
│   ├── book-list.php
│   ├── error.php
│   └── layout.php
└── var/
    └── .gitkeep
~~~

The web server exposes only public. The database is stored in var, outside the document root.

## 1. Ignore generated data

Create **.gitignore**:

~~~gitignore
/var/*.sqlite
/var/*.sqlite-*
!/var/.gitkeep
~~~

Create an empty **var/.gitkeep** so Git retains the directory. The SQLite database itself is created locally and ignored.

## 2. Create the schema

Create **database/schema.sql**:

~~~sql
CREATE TABLE IF NOT EXISTS books (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL CHECK (length(title) BETWEEN 1 AND 160),
    author TEXT NOT NULL CHECK (length(author) BETWEEN 1 AND 120),
    status TEXT NOT NULL DEFAULT 'planned'
        CHECK (status IN ('planned', 'reading', 'finished')),
    notes TEXT NOT NULL DEFAULT '' CHECK (length(notes) <= 2000),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
~~~

The database constraints protect stored data even if a future code path forgets a validation check.

## 3. Create shared setup and security helpers

Create **src/bootstrap.php**:

~~~php
<?php

declare(strict_types=1);

if (PHP_SAPI !== 'cli') {
    ini_set('session.use_strict_mode', '1');

    session_set_cookie_params([
        'lifetime' => 0,
        'path' => '/',
        'secure' => isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] !== 'off',
        'httponly' => true,
        'samesite' => 'Lax',
    ]);

    session_start();
}

function database(): PDO
{
    $directory = dirname(__DIR__) . '/var';

    if (
        !is_dir($directory)
        && !mkdir($directory, 0775, true)
        && !is_dir($directory)
    ) {
        throw new RuntimeException('Could not create the data directory.');
    }

    $pdo = new PDO('sqlite:' . $directory . '/reading-tracker.sqlite');
    $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
    $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    $pdo->exec('PRAGMA foreign_keys = ON');
    $pdo->exec('PRAGMA busy_timeout = 5000');

    return $pdo;
}

function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

function csrfToken(): string
{
    if (!isset($_SESSION['csrf_token'])) {
        $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
    }

    return $_SESSION['csrf_token'];
}

function verifyCsrf(): void
{
    $submitted = $_POST['csrf_token'] ?? null;
    $stored = $_SESSION['csrf_token'] ?? null;

    if (
        !is_string($submitted)
        || !is_string($stored)
        || !hash_equals($stored, $submitted)
    ) {
        http_response_code(403);
        render('error', [
            'pageTitle' => 'Request rejected',
            'message' => 'The form token is missing or invalid. Reload the page and try again.',
        ]);
        exit;
    }
}

function setFlash(string $message): void
{
    $_SESSION['flash'] = $message;
}

function redirect(string $path): void
{
    http_response_code(303);
    header('Location: ' . $path);
    exit;
}

function render(string $view, array $data = []): void
{
    extract($data, EXTR_SKIP);

    $pageTitle = $pageTitle ?? 'Reading tracker';
    $flash = $_SESSION['flash'] ?? null;
    unset($_SESSION['flash']);

    ob_start();
    require dirname(__DIR__) . '/templates/' . $view . '.php';
    $content = ob_get_clean();

    require dirname(__DIR__) . '/templates/layout.php';
}
~~~

The view name is selected by application code, never by a request parameter. The e() helper is for HTML output. It does not replace SQL parameters or validation.

## 4. Validate submitted book fields

Create **src/book-validation.php**:

~~~php
<?php

declare(strict_types=1);

function validateBook(array $input): array
{
    $allowed = ['title', 'author', 'status', 'notes', 'csrf_token'];
    $errors = [];
    $values = [];

    if (array_diff(array_keys($input), $allowed) !== []) {
        $errors['_form'] = 'The form contains an unsupported field.';
    }

    foreach (['title', 'author', 'status', 'notes'] as $field) {
        $raw = $input[$field] ?? '';

        if (!is_string($raw)) {
            $errors[$field] = 'Enter this value as text.';
            $values[$field] = '';
            continue;
        }

        $values[$field] = trim($raw);
    }

    if ($values['title'] === '') {
        $errors['title'] = 'Enter a book title.';
    } elseif (mb_strlen($values['title'], 'UTF-8') > 160) {
        $errors['title'] = 'The title must be no longer than 160 characters.';
    }

    if ($values['author'] === '') {
        $errors['author'] = 'Enter an author.';
    } elseif (mb_strlen($values['author'], 'UTF-8') > 120) {
        $errors['author'] = 'The author must be no longer than 120 characters.';
    }

    if (!in_array($values['status'], ['planned', 'reading', 'finished'], true)) {
        $errors['status'] = 'Choose planned, reading, or finished.';
    }

    if (mb_strlen($values['notes'], 'UTF-8') > 2000) {
        $errors['notes'] = 'Notes must be no longer than 2,000 characters.';
    }

    return [
        'values' => $values,
        'errors' => $errors,
    ];
}
~~~

The function checks types before using string operations, trims values, enforces limits, and accepts only documented status values. It returns both cleaned values and errors so the form can be redisplayed without losing the user’s input.

## 5. Put database queries in repository functions

Create **src/book-repository.php**:

~~~php
<?php

declare(strict_types=1);

function formatBook(array $book): array
{
    $book['id'] = (int) $book['id'];

    return $book;
}

function listBooks(PDO $pdo): array
{
    $statement = $pdo->query(
        'SELECT id, title, author, status, notes, created_at
         FROM books
         ORDER BY id DESC'
    );

    return array_map('formatBook', $statement->fetchAll());
}

function findBook(PDO $pdo, int $id): ?array
{
    $statement = $pdo->prepare(
        'SELECT id, title, author, status, notes, created_at
         FROM books
         WHERE id = :id'
    );
    $statement->execute(['id' => $id]);
    $book = $statement->fetch();

    return $book === false ? null : formatBook($book);
}

function createBook(PDO $pdo, array $book): void
{
    $statement = $pdo->prepare(
        'INSERT INTO books (title, author, status, notes)
         VALUES (:title, :author, :status, :notes)'
    );
    $statement->execute([
        'title' => $book['title'],
        'author' => $book['author'],
        'status' => $book['status'],
        'notes' => $book['notes'],
    ]);
}

function updateBook(PDO $pdo, int $id, array $book): void
{
    $statement = $pdo->prepare(
        'UPDATE books
         SET title = :title, author = :author,
             status = :status, notes = :notes
         WHERE id = :id'
    );
    $statement->execute([
        'title' => $book['title'],
        'author' => $book['author'],
        'status' => $book['status'],
        'notes' => $book['notes'],
        'id' => $id,
    ]);
}

function deleteBook(PDO $pdo, int $id): bool
{
    $statement = $pdo->prepare('DELETE FROM books WHERE id = :id');
    $statement->execute(['id' => $id]);

    return $statement->rowCount() > 0;
}
~~~

Every query uses a fixed SQL statement. Values from the form are bound as parameters. The ID is still validated by the route handler and the record is checked before it is changed.

## 6. Create the shared page layout

Create **templates/layout.php**:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= e($pageTitle) ?> | Reading tracker</title>
    <link rel="stylesheet" href="/style.css">
</head>
<body>
    <header class="site-header">
        <a class="brand" href="/books">Reading tracker</a>
        <nav aria-label="Main navigation">
            <a href="/books">My books</a>
            <a href="/books/new">Add a book</a>
        </nav>
    </header>

    <main class="container">
        <?php if (is_string($flash)): ?>
            <p class="notice" role="status"><?= e($flash) ?></p>
        <?php endif; ?>

        <h1><?= e($pageTitle) ?></h1>
        <?= $content ?>
    </main>
</body>
</html>
~~~

The content variable contains output from a trusted local template. Dynamic values inside each template still need escaping.

## 7. Render the book list

Create **templates/book-list.php**:

~~~php
<p class="intro">Keep a small list of books you plan to read, are reading, or have finished.</p>

<?php if ($books === []): ?>
    <section class="empty-state">
        <h2>No books yet</h2>
        <p>Add your first book to start your reading list.</p>
        <a class="button" href="/books/new">Add a book</a>
    </section>
<?php else: ?>
    <ul class="book-list">
        <?php foreach ($books as $book): ?>
            <li class="book-card">
                <div class="book-details">
                    <h2><?= e($book['title']) ?></h2>
                    <p class="byline">By <?= e($book['author']) ?></p>
                    <p><span class="status"><?= e($book['status']) ?></span></p>

                    <?php if ($book['notes'] !== ''): ?>
                        <p class="notes"><?= nl2br(e($book['notes'])) ?></p>
                    <?php endif; ?>

                    <p class="date">Added <?= e($book['created_at']) ?></p>
                </div>

                <div class="book-actions">
                    <a href="/books/<?= e((string) $book['id']) ?>/edit">Edit</a>

                    <form
                        method="post"
                        action="/books/<?= e((string) $book['id']) ?>/delete"
                    >
                        <input
                            type="hidden"
                            name="csrf_token"
                            value="<?= e(csrfToken()) ?>"
                        >
                        <button class="button-danger" type="submit">
                            Delete
                        </button>
                    </form>
                </div>
            </li>
        <?php endforeach; ?>
    </ul>
<?php endif; ?>
~~~

Titles, authors, notes, timestamps, and IDs are escaped at the output point. Notes are escaped before line breaks are converted to HTML.

## 8. Render create and edit forms

Create **templates/book-form.php**:

~~~php
<p class="intro">Fields marked required must be completed.</p>

<?php if (isset($errors['_form'])): ?>
    <p class="error-summary" role="alert"><?= e($errors['_form']) ?></p>
<?php endif; ?>

<form class="book-form" method="post" action="<?= e($formAction) ?>">
    <input
        type="hidden"
        name="csrf_token"
        value="<?= e(csrfToken()) ?>"
    >

    <div class="field">
        <label for="title">Title <span>(required)</span></label>
        <input
            id="title"
            name="title"
            type="text"
            maxlength="160"
            required
            value="<?= e($values['title']) ?>"
            <?= isset($errors['title']) ? 'aria-invalid="true" aria-describedby="title-error"' : '' ?>
        >
        <?php if (isset($errors['title'])): ?>
            <p class="field-error" id="title-error"><?= e($errors['title']) ?></p>
        <?php endif; ?>
    </div>

    <div class="field">
        <label for="author">Author <span>(required)</span></label>
        <input
            id="author"
            name="author"
            type="text"
            maxlength="120"
            required
            value="<?= e($values['author']) ?>"
            <?= isset($errors['author']) ? 'aria-invalid="true" aria-describedby="author-error"' : '' ?>
        >
        <?php if (isset($errors['author'])): ?>
            <p class="field-error" id="author-error"><?= e($errors['author']) ?></p>
        <?php endif; ?>
    </div>

    <div class="field">
        <label for="status">Reading status</label>
        <select
            id="status"
            name="status"
            <?= isset($errors['status']) ? 'aria-invalid="true" aria-describedby="status-error"' : '' ?>
        >
            <?php foreach (['planned', 'reading', 'finished'] as $status): ?>
                <option
                    value="<?= e($status) ?>"
                    <?= $values['status'] === $status ? 'selected' : '' ?>
                >
                    <?= e(ucfirst($status)) ?>
                </option>
            <?php endforeach; ?>
        </select>
        <?php if (isset($errors['status'])): ?>
            <p class="field-error" id="status-error"><?= e($errors['status']) ?></p>
        <?php endif; ?>
    </div>

    <div class="field">
        <label for="notes">Notes</label>
        <textarea
            id="notes"
            name="notes"
            rows="5"
            maxlength="2000"
            <?= isset($errors['notes']) ? 'aria-invalid="true" aria-describedby="notes-error"' : '' ?>
        ><?= e($values['notes']) ?></textarea>
        <?php if (isset($errors['notes'])): ?>
            <p class="field-error" id="notes-error"><?= e($errors['notes']) ?></p>
        <?php endif; ?>
    </div>

    <div class="form-actions">
        <button class="button" type="submit"><?= e($submitLabel) ?></button>
        <a href="/books">Cancel</a>
    </div>
</form>
~~~

Labels are connected to controls, errors are associated with fields, and the submitted values remain available after a validation failure.

## 9. Create an error page

Create **templates/error.php**:

~~~php
<p class="error-summary" role="alert"><?= e($message) ?></p>
<p><a href="/books">Return to the book list</a></p>
~~~

The caller sets the HTTP status and supplies a safe message. Do not print exception messages or SQL details here.

## 10. Route requests through the front controller

Create **public/router.php**:

~~~php
<?php

declare(strict_types=1);

$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);

if ($path === '/style.css') {
    return false;
}

require __DIR__ . '/index.php';
return true;
~~~

This router is for PHP’s local development server. It serves the stylesheet as a static file and sends other paths through the application. Production servers need equivalent routing rules.

Create **public/index.php**:

~~~php
<?php

declare(strict_types=1);

require_once dirname(__DIR__) . '/src/bootstrap.php';
require_once dirname(__DIR__) . '/src/book-validation.php';
require_once dirname(__DIR__) . '/src/book-repository.php';

function failPage(int $status, string $title, string $message): void
{
    http_response_code($status);
    render('error', [
        'pageTitle' => $title,
        'message' => $message,
    ]);
    exit;
}

function methodNotAllowed(array $methods): void
{
    header('Allow: ' . implode(', ', $methods));
    failPage(405, 'Method not allowed', 'This method is not available for this page.');
}

function bookIdFrom(string $value): int
{
    $id = filter_var(
        $value,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1]]
    );

    if ($id === false) {
        failPage(404, 'Book not found', 'The requested book was not found.');
    }

    return $id;
}

function formValues(array $book): array
{
    return [
        'title' => (string) $book['title'],
        'author' => (string) $book['author'],
        'status' => (string) $book['status'],
        'notes' => (string) $book['notes'],
    ];
}

$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);
$method = strtoupper($_SERVER['REQUEST_METHOD'] ?? 'GET');

if (!is_string($path)) {
    failPage(400, 'Invalid request', 'The request path is invalid.');
}

$path = $path === '/' ? '/' : rtrim($path, '/');

try {
    $pdo = database();

    if ($path === '/') {
        if ($method !== 'GET') {
            methodNotAllowed(['GET']);
        }

        redirect('/books');
    }

    if ($path === '/books') {
        if ($method === 'GET') {
            render('book-list', [
                'pageTitle' => 'My books',
                'books' => listBooks($pdo),
            ]);
            exit;
        }

        if ($method !== 'POST') {
            methodNotAllowed(['GET', 'POST']);
        }

        verifyCsrf();
        $result = validateBook($_POST);

        if ($result['errors'] !== []) {
            http_response_code(422);
            render('book-form', [
                'pageTitle' => 'Add a book',
                'formAction' => '/books',
                'submitLabel' => 'Save book',
                'values' => $result['values'],
                'errors' => $result['errors'],
            ]);
            exit;
        }

        createBook($pdo, $result['values']);
        setFlash('Book added to your reading list.');
        redirect('/books');
    }

    if ($path === '/books/new') {
        if ($method !== 'GET') {
            methodNotAllowed(['GET']);
        }

        render('book-form', [
            'pageTitle' => 'Add a book',
            'formAction' => '/books',
            'submitLabel' => 'Save book',
            'values' => [
                'title' => '',
                'author' => '',
                'status' => 'planned',
                'notes' => '',
            ],
            'errors' => [],
        ]);
        exit;
    }

    if (preg_match('~\A/books/([0-9]+)/edit\z~D', $path, $matches)) {
        if ($method !== 'GET') {
            methodNotAllowed(['GET']);
        }

        $id = bookIdFrom($matches[1]);
        $book = findBook($pdo, $id);

        if ($book === null) {
            failPage(404, 'Book not found', 'The requested book was not found.');
        }

        render('book-form', [
            'pageTitle' => 'Edit book',
            'formAction' => '/books/' . $id,
            'submitLabel' => 'Save changes',
            'values' => formValues($book),
            'errors' => [],
        ]);
        exit;
    }

    if (preg_match('~\A/books/([0-9]+)/delete\z~D', $path, $matches)) {
        if ($method !== 'POST') {
            methodNotAllowed(['POST']);
        }

        $id = bookIdFrom($matches[1]);
        verifyCsrf();

        if (!deleteBook($pdo, $id)) {
            failPage(404, 'Book not found', 'The requested book was not found.');
        }

        setFlash('Book deleted from your reading list.');
        redirect('/books');
    }

    if (preg_match('~\A/books/([0-9]+)\z~D', $path, $matches)) {
        if ($method !== 'POST') {
            methodNotAllowed(['POST']);
        }

        $id = bookIdFrom($matches[1]);
        $book = findBook($pdo, $id);

        if ($book === null) {
            failPage(404, 'Book not found', 'The requested book was not found.');
        }

        verifyCsrf();
        $result = validateBook($_POST);

        if ($result['errors'] !== []) {
            http_response_code(422);
            render('book-form', [
                'pageTitle' => 'Edit book',
                'formAction' => '/books/' . $id,
                'submitLabel' => 'Save changes',
                'values' => $result['values'],
                'errors' => $result['errors'],
            ]);
            exit;
        }

        updateBook($pdo, $id, $result['values']);
        setFlash('Book details updated.');
        redirect('/books');
    }

    failPage(404, 'Page not found', 'The requested page was not found.');
} catch (Throwable $exception) {
    error_log('[reading-tracker] ' . $exception);
    failPage(
        500,
        'Something went wrong',
        'The application could not complete that request. Try again later.'
    );
}
~~~

The front controller checks routes and methods, validates IDs, verifies CSRF on every POST, and redirects after successful changes. Unexpected details go to the server log while visitors see a generic error.

## 11. Add simple styling

Create **public/style.css**:

~~~css
:root {
    color-scheme: light;
    font-family: system-ui, sans-serif;
    line-height: 1.5;
    color: #172033;
    background: #f4f6fa;
}

*,
*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
}

.site-header {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    justify-content: space-between;
    gap: 1rem;
    padding: 1rem max(1rem, calc((100% - 56rem) / 2));
    color: white;
    background: #172033;
}

.site-header a {
    color: white;
}

.brand {
    font-size: 1.2rem;
    font-weight: 700;
    text-decoration: none;
}

nav {
    display: flex;
    flex-wrap: wrap;
    gap: 1rem;
}

.container {
    width: min(56rem, calc(100% - 2rem));
    margin: 2rem auto;
}

.intro,
.date {
    color: #526078;
}

.book-list {
    display: grid;
    gap: 1rem;
    margin: 1.5rem 0;
    padding: 0;
    list-style: none;
}

.book-card,
.empty-state,
.book-form {
    padding: 1.25rem;
    border: 1px solid #ccd3df;
    border-radius: 0.75rem;
    background: white;
}

.book-card {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 1rem;
}

.book-details h2 {
    margin-top: 0;
}

.byline {
    margin-top: -0.75rem;
}

.status {
    display: inline-block;
    padding: 0.15rem 0.55rem;
    border-radius: 999px;
    background: #e8edf6;
    text-transform: capitalize;
}

.notes {
    white-space: normal;
}

.book-actions,
.form-actions {
    display: flex;
    flex-wrap: wrap;
    align-items: center;
    gap: 0.8rem;
}

.book-form {
    display: grid;
    gap: 1rem;
    max-width: 42rem;
}

.field {
    display: grid;
    gap: 0.35rem;
}

.field label {
    font-weight: 650;
}

.field label span {
    font-weight: 400;
    color: #526078;
}

input,
select,
textarea,
button,
.button {
    font: inherit;
}

input,
select,
textarea {
    width: 100%;
    padding: 0.65rem 0.75rem;
    border: 1px solid #9aa6b8;
    border-radius: 0.35rem;
}

textarea {
    resize: vertical;
}

button,
.button {
    display: inline-block;
    width: fit-content;
    padding: 0.6rem 0.9rem;
    border: 0;
    border-radius: 0.4rem;
    color: white;
    background: #2457c5;
    cursor: pointer;
    text-decoration: none;
}

.button-danger {
    background: #a12535;
}

.notice {
    padding: 0.75rem 1rem;
    border-left: 0.3rem solid #247245;
    background: #e5f4eb;
}

.field-error,
.error-summary {
    color: #8d1828;
}

:focus-visible {
    outline: 3px solid #e4a900;
    outline-offset: 2px;
}

@media (max-width: 38rem) {
    .site-header {
        align-items: flex-start;
        flex-direction: column;
    }
}
~~~

The focus indicator remains visible for keyboard users. Status is written as text, so color is not the only way to understand it.

## 12. Initialize and run the app

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

echo "Reading tracker database initialized." . PHP_EOL;
~~~

From the project directory, initialize the database and start the local server:

~~~powershell
php bin/init-db.php
php -S 127.0.0.1:8000 -t public public/router.php
~~~

Open http://127.0.0.1:8000/books in your browser. The server listens only on the local computer.

## 13. Verify the acceptance criteria manually

Use a separate browser session and check each workflow:

| Check | Action | Expected result |
| --- | --- | --- |
| Empty list | Open /books before adding records | Empty-state message and Add a book link |
| Create | Add a title, author, status, and notes | Redirect to list and success message |
| Persistence | Restart the PHP server and reopen /books | Created book remains |
| Required fields | Submit a blank title or author | Form shows a field error and saves nothing |
| Allowed status | Change a submitted status to an unknown value | Validation rejects it |
| Edit | Change one book and save | Updated values appear on the list |
| Delete | Delete a book from its form | Redirect and success message; record is gone |
| Missing record | Open /books/999999/edit | 404 page |
| CSRF | Submit a POST without the hidden token | 403 page; data is not changed |
| Escaping | Save a title containing angle brackets | It displays as text, not active markup |
| Keyboard use | Navigate forms and links without a mouse | Focus is visible and controls are reachable |

For a larger application, add automated unit, integration, and HTTP tests using an isolated test database. The checklist is a repeatable manual baseline, not a substitute for a maintained test suite.

You can also check PHP syntax without starting the app:

~~~powershell
php -l public/index.php
php -l src/bootstrap.php
php -l src/book-validation.php
php -l src/book-repository.php
php -l templates/book-form.php
php -l templates/book-list.php
~~~

## 14. Document setup and decisions

Create **README.md**:

~~~markdown
# Reading tracker

A small, single-user PHP application for tracking books you plan to read, are reading, or have finished.

## Requirements

- PHP 8.0 or later
- PDO and pdo_sqlite
- mbstring

## Setup

From the project directory, initialize the SQLite database:

~~~powershell
php bin/init-db.php
~~~

Start the local development server:

~~~powershell
php -S 127.0.0.1:8000 -t public public/router.php
~~~

Open http://127.0.0.1:8000/books.

The database is stored at var/reading-tracker.sqlite and is excluded from Git.

## Main routes

| Method | Path | Purpose |
| --- | --- | --- |
| GET | /books | List books |
| GET | /books/new | Show the create form |
| POST | /books | Validate and create a book |
| GET | /books/{id}/edit | Show the edit form |
| POST | /books/{id} | Validate and update a book |
| POST | /books/{id}/delete | Delete a book |

## Verification checklist

- Add a valid book and confirm it appears in the list.
- Restart the server and confirm the record remains.
- Submit a blank title and confirm the form reports an error without saving.
- Edit a record and confirm its values change.
- Delete a record and confirm it is removed.
- Try a missing record ID and confirm the app returns a 404 page.
- Submit a state-changing request without a CSRF token and confirm it is rejected.
- Save markup in a title and confirm it is displayed as text.

## Decisions and limitations

- SQLite keeps local setup simple for a single-user learning project.
- The database stays outside the public document root.
- The app uses server-side validation, PDO prepared statements, CSRF tokens, and HTML escaping.
- There is no authentication, multi-user authorization, automated test suite, migration system, or production deployment configuration.
- Do not expose this demo publicly. A deployed multi-user app needs authentication, per-record authorization, HTTPS, protected logs, backups, migrations, and operational monitoring.

## Future improvements

- Add automated unit, database integration, and HTTP tests.
- Add migrations and a documented backup and restore process.
- Add accounts and authorization before supporting multiple users.
- Add pagination if the list grows.
~~~

This README gives another developer a repeatable setup path, the main routes, checks to perform, and the decisions that should not be mistaken for production guarantees.

## Expected result

After setup, the user can create a book, see it on the list, edit it, and delete it. Invalid submissions show field errors and do not change the database. Refreshing after a successful POST does not submit the form again because the app redirects.

## Limitations and next steps

This capstone is intentionally small. It does not include account management, user authorization, automated tests, database migrations, pagination, logging infrastructure, or production web-server configuration. The database belongs to one local user; do not deploy it as a shared service without designing those protections.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [151 - Project: build a JSON API](../lessons/151-project-build-a-json-api.md) | [Q&A](../qnas/152-capstone-project-plan-build-test-and-document-an-application.md) | [Lesson](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) | [153 - PHP quick reference and glossary](../lessons/153-php-quick-reference-and-glossary.md) |
