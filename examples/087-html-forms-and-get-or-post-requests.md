# 087 - HTML forms and GET or POST requests: Example

This small PHP example includes a read-only search form that uses GET and a contact form that uses POST. The contact flow validates submitted values, checks a session CSRF token, and uses a 303 redirect after accepting the submission. The demo does not store or send contact messages; a real application must add its intended persistence or delivery step.

## File structure

```text
project/
└── public/
    ├── search.php
    ├── contact.php
    └── thank-you.php
```

## 1. GET search form: `public/search.php`

```php
<?php

declare(strict_types=1);

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

function sendBadRequest(string $message): void
{
    http_response_code(400);
    header('Content-Type: text/plain; charset=UTF-8');
    exit($message);
}

if (($_SERVER['REQUEST_METHOD'] ?? '') !== 'GET') {
    http_response_code(405);
    header('Allow: GET');
    header('Content-Type: text/plain; charset=UTF-8');
    exit('This search page only accepts GET requests.');
}

$searchTerm = $_GET['q'] ?? '';

if (!is_string($searchTerm)) {
    sendBadRequest('The q parameter must be a single string.');
}

$searchTerm = trim($searchTerm);

if (strlen($searchTerm) > 100) {
    sendBadRequest('The search term must not exceed 100 bytes.');
}

$catalog = [
    'PHP forms',
    'PHP request handling',
    'HTML forms',
    'CSS selectors',
];

$results = array_values(array_filter(
    $catalog,
    static fn (string $title): bool => $searchTerm === ''
        || stripos($title, $searchTerm) !== false
));

header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Tutorial search</title>
</head>
<body>
    <main>
        <h1>Tutorial search</h1>

        <form action="/search.php" method="get">
            <label for="q">Search tutorials</label>
            <input id="q" name="q" type="search" value="<?= escapeHtml($searchTerm) ?>">
            <button type="submit">Search</button>
        </form>

        <h2>Results</h2>
        <?php if ($results === []): ?>
            <p>No matching tutorials were found.</p>
        <?php else: ?>
            <ul>
                <?php foreach ($results as $title): ?>
                    <li><?= escapeHtml($title) ?></li>
                <?php endforeach; ?>
            </ul>
        <?php endif; ?>
    </main>
</body>
</html>
```

Submitting `PHP` sends a request like `/search.php?q=PHP`. The search reads the value from `$_GET`; the URL can be bookmarked or shared. It only filters the sample list and does not change application state.

## 2. POST contact form: `public/contact.php`

```php
<?php

declare(strict_types=1);

session_start();

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

$method = $_SERVER['REQUEST_METHOD'] ?? '';

if (!in_array($method, ['GET', 'POST'], true)) {
    http_response_code(405);
    header('Allow: GET, POST');
    header('Content-Type: text/plain; charset=UTF-8');
    exit('Method not allowed.');
}

$name = '';
$message = '';
$error = '';

if ($method === 'POST') {
    $submittedToken = $_POST['csrf_token'] ?? null;
    $sessionToken = $_SESSION['contact_csrf_token'] ?? null;

    if (
        !is_string($submittedToken)
        || !is_string($sessionToken)
        || !hash_equals($sessionToken, $submittedToken)
    ) {
        http_response_code(403);
        header('Content-Type: text/plain; charset=UTF-8');
        exit('The form session is invalid. Reload the form and try again.');
    }

    $nameInput = $_POST['name'] ?? '';
    $messageInput = $_POST['message'] ?? '';

    if (!is_string($nameInput) || !is_string($messageInput)) {
        http_response_code(400);
        header('Content-Type: text/plain; charset=UTF-8');
        exit('Name and message must each be a single text value.');
    }

    $name = trim($nameInput);
    $message = trim($messageInput);

    if ($name === '' || $message === '') {
        $error = 'Enter both your name and a message.';
    } elseif (strlen($name) > 100 || strlen($message) > 2000) {
        $error = 'Name must be at most 100 bytes and message at most 2000 bytes.';
    }

    if ($error === '') {
        // A real application would persist or deliver the validated message here.
        $_SESSION['contact_flash'] =
            'Thanks, ' . $name . '. The demo accepted your message but does not store or send it.';
        unset($_SESSION['contact_csrf_token']);

        header('Location: /thank-you.php', true, 303);
        exit;
    }

    http_response_code(400);
}

if (!isset($_SESSION['contact_csrf_token'])) {
    $_SESSION['contact_csrf_token'] = bin2hex(random_bytes(32));
}

$csrfToken = $_SESSION['contact_csrf_token'];
header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Contact</title>
</head>
<body>
    <main>
        <h1>Contact</h1>

        <?php if ($error !== ''): ?>
            <p role="alert"><?= escapeHtml($error) ?></p>
        <?php endif; ?>

        <form action="/contact.php" method="post">
            <input type="hidden" name="csrf_token" value="<?= escapeHtml($csrfToken) ?>">

            <label for="name">Name</label>
            <input
                id="name"
                name="name"
                autocomplete="name"
                maxlength="100"
                required
                value="<?= escapeHtml($name) ?>"
            >

            <label for="message">Message</label>
            <textarea
                id="message"
                name="message"
                maxlength="2000"
                required
            ><?= escapeHtml($message) ?></textarea>

            <button type="submit">Send message</button>
        </form>
    </main>
</body>
</html>
```

The form sends its values in the POST body. PHP exposes these URL-encoded form fields in `$_POST`. The handler checks the method, token, types, and required values on the server; browser attributes such as `required` and `maxlength` are only helpful client-side constraints.

A valid submission stores a one-time confirmation message in the session, then sends a `303 See Other` redirect. The browser follows the redirect with a GET request. The CSRF token prevents this demo's state-changing form from accepting a cross-site submission without the session-bound token; production applications should use their framework's established CSRF support where available.

## 3. Redirect destination: `public/thank-you.php`

```php
<?php

declare(strict_types=1);

session_start();

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

$confirmation = $_SESSION['contact_flash'] ?? null;
unset($_SESSION['contact_flash']);

if (!is_string($confirmation)) {
    http_response_code(404);
    header('Content-Type: text/plain; charset=UTF-8');
    exit('There is no recent form submission to confirm.');
}

header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Thank you</title>
</head>
<body>
    <main>
        <h1>Thank you</h1>
        <p><?= escapeHtml($confirmation) ?></p>
        <a href="/contact.php">Send another message</a>
    </main>
</body>
</html>
```

The confirmation is removed from the session after it is read, so refreshing the thank-you page does not repeat the success message. The name included in the confirmation came from the visitor and is escaped again when rendered.

## Run the example

From the project directory, start PHP's built-in development server:

```bash
php -S localhost:8000 -t public
```

Open `/search.php`, search for `PHP`, and observe the query string in the address bar. Open `/contact.php`, submit a name and message, and observe that the browser lands on `/thank-you.php` after the POST.

## Expected behavior

For the search request:

```text
/search.php?q=PHP
```

The page keeps `PHP` in the search field and shows:

```text
Results
- PHP forms
- PHP request handling
```

For a valid contact submission:

1. The browser sends a POST request to `/contact.php`.
2. PHP validates the form values and CSRF token.
3. The example sets a session confirmation and returns a 303 redirect.
4. The browser requests `/thank-you.php` using GET and displays the confirmation.

Invalid or missing form values return the form with an error. A request using another method receives status 405. A missing or incorrect CSRF token receives status 403.

## Important notes

- The sample checks lengths in bytes with `strlen()`. If the product requirement is expressed in Unicode characters, use a Unicode-aware length check and align it with the chosen encoding.
- `session_start()` and `header()` must run before output. Avoid whitespace or HTML before those calls.
- A CSRF token does not replace authentication, authorization, validation, or HTTPS.
- The demo intentionally does not save or email contact messages. Add the real application operation only after validation and authorization.
- For file uploads, use `method="post"`, `enctype="multipart/form-data"`, and validate uploaded files through `$_FILES` with dedicated safeguards.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [086 - Reading query strings and request data](../lessons/086-reading-query-strings-and-request-data.md) | [Q&A](../qnas/087-html-forms-and-get-or-post-requests.md) | Example | [088 - Input validation and normalization](../lessons/088-input-validation-and-normalization.md) |