# 126 - Preventing cross-site request forgery: Example

## Question

Create a PHP profile-preferences page that changes a value only when a POST request includes a valid CSRF token associated with the user's session. Show what happens for a normal request and for a missing or incorrect token.

## Complete example

Save this as a PHP file and serve it through an HTTPS-enabled PHP web server. The example stores the display name in the session so it does not require a database.

~~~php
<?php

declare(strict_types=1);

$isHttps = isset($_SERVER['HTTPS'])
    && $_SERVER['HTTPS'] !== ''
    && strtolower((string) $_SERVER['HTTPS']) !== 'off';

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => $isHttps,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

if (
    !isset($_SESSION['csrf_token'])
    || !is_string($_SESSION['csrf_token'])
) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$csrfToken = $_SESSION['csrf_token'];
$message = '';
$requestMethod = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($requestMethod === 'POST') {
    $submittedToken = $_POST['csrf_token'] ?? null;

    if (
        !is_string($submittedToken)
        || !hash_equals($csrfToken, $submittedToken)
    ) {
        http_response_code(403);
        $message = 'Request rejected. Reload the page and try again.';
    } else {
        $submittedName = $_POST['display_name'] ?? null;

        if (
            !is_string($submittedName)
            || trim($submittedName) === ''
            || strlen($submittedName) > 80
        ) {
            http_response_code(400);
            $message = 'Enter a name of 1 to 80 bytes.';
        } else {
            $_SESSION['display_name'] = trim($submittedName);
            $message = 'Preferences saved.';
        }
    }
} elseif ($requestMethod !== 'GET') {
    http_response_code(405);
    $message = 'Method not allowed.';
}

$currentName = $_SESSION['display_name'] ?? 'Guest';

if (!is_string($currentName)) {
    $currentName = 'Guest';
}
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Profile preferences</title>
</head>
<body>
    <main>
        <h1>Profile preferences</h1>

        <?php if ($message !== ''): ?>
            <p role="status"><?= escapeHtml($message) ?></p>
        <?php endif; ?>

        <p>
            Current display name:
            <strong><?= escapeHtml($currentName) ?></strong>
        </p>

        <form method="post" action="">
            <input
                type="hidden"
                name="csrf_token"
                value="<?= escapeHtml($csrfToken) ?>"
            >

            <label for="display-name">New display name</label>
            <input
                id="display-name"
                name="display_name"
                type="text"

                required
            >

            <button type="submit">Save preferences</button>
        </form>
    </main>
</body>
</html>
~~~

For an HTTPS deployment behind a reverse proxy, configure the Secure cookie setting from trusted deployment configuration. Do not infer that the original request used HTTPS from an untrusted forwarded header.

## Expected behavior

### First page view

The server creates a session-bound token and displays the form. The token is included in a hidden field; its value is a random hexadecimal string and will differ between sessions.

The current display name is Guest.

### Submit with the form

The browser submits the hidden token with the POST request. The server checks it against the value in the session before updating the preference.

~~~text
HTTP status: 200
Message: Preferences saved.
Current display name: the submitted name
~~~

### Submit without a token or with an incorrect token

The request is rejected before the display name is changed.

~~~text
HTTP status: 403
Message: Request rejected. Reload the page and try again.
Current display name for the submitted session: unchanged
~~~

### Try to change the name with GET

A GET request only displays the page. It does not update the session value.

## Explanation

### The token is unpredictable and tied to the session

The example creates 32 random bytes with random_bytes(), then uses bin2hex() to make a form-friendly token. The server stores the token in the session and renders it into the form.

The token is not placed in the URL. A cross-site page can try to submit a request, but it cannot normally read this page and discover the session token.

### The server validates before the update

The POST handler checks that the submitted token is a string and compares it with the session token using hash_equals(). Only the valid-token branch changes the display name.

The request method is checked separately. GET never updates the value, and methods other than GET or POST are rejected.

### Cookie attributes are another layer

The session cookie is configured with HttpOnly, SameSite=Lax, and Secure when the server reports a direct HTTPS connection. These options help protect the session, but the token check remains the main defense in this example.

For production behind a TLS-terminating proxy, use trusted deployment configuration to decide whether Secure is enabled. Configure the cookie options before session_start() on every request that starts the session.

### Real features need their normal checks too

This example validates a small display-name field and stores it in the session only to keep the demonstration self-contained. A real profile update should require authentication, verify that the user may edit that profile, validate the full set of business rules, and persist the change safely.

## What this example demonstrates

- A state-changing form uses POST and includes a hidden CSRF token.
- The server creates a strong token and stores it in the user's session.
- The server rejects a missing or incorrect token before the action.
- GET does not change application state.
- Cookie flags, input validation, and authorization complement CSRF validation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [125 - Preventing cross-site scripting](../lessons/125-preventing-cross-site-scripting.md) | [Q&A](../qnas/126-preventing-cross-site-request-forgery.md) | [Lesson](../lessons/126-preventing-cross-site-request-forgery.md) | [127 - Authentication, authorization, and access control](../lessons/127-authentication-authorization-and-access-control.md) |