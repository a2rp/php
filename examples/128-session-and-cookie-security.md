# 128 - Session and cookie security: Example

## Question

Create a protected PHP account page that uses secure session-cookie settings, rejects missing or expired authentication state, enforces both idle and absolute timeouts, and supports CSRF-protected logout. Show the expected page and failure responses.

## Complete example

Save this as `account.php` and serve it through an HTTPS-enabled PHP web server. This is a protected page: the application's login handler must already have verified credentials, rotated the session ID, and stored an integer user_id plus integer created_at and last_activity timestamps in the server-side session.

~~~php
<?php

declare(strict_types=1);

const IDLE_TIMEOUT_SECONDS = 20 * 60;
const ABSOLUTE_TIMEOUT_SECONDS = 8 * 60 * 60;

ini_set('session.use_strict_mode', '1');
ini_set('session.use_only_cookies', '1');

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();

header('Content-Type: text/html; charset=UTF-8');
header('Cache-Control: no-store');

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

function endCurrentSession(): void
{
    $_SESSION = [];

    if (ini_get('session.use_cookies')) {
        $cookie = session_get_cookie_params();

        $options = [
            'expires' => time() - 42000,
            'path' => $cookie['path'],
            'secure' => $cookie['secure'],
            'httponly' => $cookie['httponly'],
            'samesite' => $cookie['samesite'] ?? 'Lax',
        ];

        if ($cookie['domain'] !== '') {
            $options['domain'] = $cookie['domain'];
        }

        setcookie(session_name(), '', $options);
    }

    session_destroy();
}

$userId = $_SESSION['user_id'] ?? null;

if (!is_int($userId)) {
    http_response_code(401);
    exit('Sign in is required.');
}

$now = time();
$createdAt = $_SESSION['created_at'] ?? null;
$lastActivity = $_SESSION['last_activity'] ?? null;

$invalidTimestamps = !is_int($createdAt)
    || !is_int($lastActivity)
    || $createdAt > $now
    || $lastActivity > $now
    || $lastActivity < $createdAt;

$idleExpired = !$invalidTimestamps
    && ($now - $lastActivity) > IDLE_TIMEOUT_SECONDS;

$absoluteExpired = !$invalidTimestamps
    && ($now - $createdAt) > ABSOLUTE_TIMEOUT_SECONDS;

if ($invalidTimestamps || $idleExpired || $absoluteExpired) {
    endCurrentSession();
    http_response_code(401);
    exit('Session expired. Sign in again.');
}

$_SESSION['last_activity'] = $now;

if (
    !isset($_SESSION['csrf_token'])
    || !is_string($_SESSION['csrf_token'])
) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$requestMethod = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($requestMethod === 'POST') {
    $submittedToken = $_POST['csrf_token'] ?? null;

    if (
        !is_string($submittedToken)
        || !hash_equals($_SESSION['csrf_token'], $submittedToken)
    ) {
        http_response_code(403);
        exit('Request rejected.');
    }

    endCurrentSession();
    exit('You have been signed out.');
}

if ($requestMethod !== 'GET') {
    header('Allow: GET, POST');
    http_response_code(405);
    exit('Method not allowed.');
}

$csrfToken = escapeHtml($_SESSION['csrf_token']);

echo '<!doctype html>';
echo '<html lang="en">';
echo '<head>';
echo '<meta charset="utf-8">';
echo '<meta name="viewport" content="width=device-width, initial-scale=1">';
echo '<title>Account session</title>';
echo '</head>';
echo '<body>';
echo '<main>';
echo '<h1>Account</h1>';
echo '<p>Signed in account number: ' . $userId . '</p>';
echo '<p>This session ends after 20 minutes without activity or 8 hours total.</p>';
echo '<form method="post" action="">';
echo '<input type="hidden" name="csrf_token" value="' . $csrfToken . '">';
echo '<button type="submit">Sign out</button>';
echo '</form>';
echo '</main>';
echo '</body>';
echo '</html>';
~~~

In the real login handler, set the session state only after credentials are verified and the ID is rotated:

~~~php
session_regenerate_id(true);

$_SESSION['user_id'] = (int) $authenticatedUser['id'];
$_SESSION['created_at'] = time();
$_SESSION['last_activity'] = time();
$_SESSION['csrf_token'] = bin2hex(random_bytes(32));
~~~

For an application with concurrent requests or several tabs, follow the framework's guidance for rotating and retiring session IDs. Configure the session cookie and strict-mode settings before session_start() in the login handler too.

## Expected output

With a valid session, the browser displays:

~~~text
Account
Signed in account number: 42
This session ends after 20 minutes without activity or 8 hours total.
[Sign out]
~~~

If there is no authenticated user ID, the response is HTTP 401 with:

~~~text
Sign in is required.
~~~

If either timeout has passed or its timestamp is invalid, the server invalidates the session and responds HTTP 401 with:

~~~text
Session expired. Sign in again.
~~~

Submitting the logout form with its valid token clears server-side session data, expires the session cookie, and displays:

~~~text
You have been signed out.
~~~

A missing or incorrect logout token receives HTTP 403 and does not sign out the session.

## Explanation

### Cookie settings are applied before PHP reads the session

The example enables strict session ID handling and cookie-only IDs before session_start(). Its cookie is scoped to the host, has Path=/, and uses Secure, HttpOnly, and SameSite=Lax. Secure requires HTTPS; do not turn it off for a production site.

HttpOnly stops ordinary JavaScript from reading the cookie, while SameSite can reduce some cross-site cookie sends. Neither attribute prevents every XSS impact or replaces CSRF protection.

### Expiration is checked by the server on every request

The login flow stores the session creation and last-activity times on the server. The account page validates them before showing account data. The idle timeout ends an inactive session; the absolute timeout ends it even if requests keep arriving.

The cookie lifetime of zero asks for a browser-session cookie, but browser restore may preserve it. PHP's garbage collection is also only storage cleanup, not an authentication timeout. The timestamp checks enforce the limits.

### Logout clears both the server session and its cookie

endCurrentSession() empties the current session data, expires the cookie with matching path and domain attributes, and destroys the server-side session. Removing only the browser cookie would leave a copied session ID usable until the server rejected it.

The logout form uses POST and carries a random session-bound token. The handler checks that token before invalidating the session, so a cross-site request cannot simply force the action.

### Sensitive HTML is not stored by caches

The response sends Cache-Control: no-store before output. This helps keep a browser or intermediary from storing the protected page for later display after logout. It cannot remove screenshots or copies a user has already made.

The account number shown here comes from the authenticated server-side session and is escaped or constrained before output. A real page should load the current user's permitted data and still enforce the authorization rules from the previous chapter.

## What this example demonstrates

- PHP is configured for strict, cookie-only session IDs before session_start().
- The session cookie uses Secure, HttpOnly, and SameSite attributes over HTTPS.
- Missing or malformed session state is rejected before protected content is returned.
- Idle and absolute timeouts are checked on the server.
- Logout validates a CSRF token, invalidates session data, and expires the cookie.
- Sensitive authenticated HTML receives a no-store cache policy.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [127 - Authentication, authorization, and access control](../lessons/127-authentication-authorization-and-access-control.md) | [Q&A](../qnas/128-session-and-cookie-security.md) | [Lesson](../lessons/128-session-and-cookie-security.md) | [129 - Path traversal and filesystem security](../lessons/129-path-traversal-and-filesystem-security.md) |