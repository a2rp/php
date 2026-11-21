# 091 - Sessions and server-side state: Example

## Goal

Build a small PHP page that counts visits in `$_SESSION`. The browser carries a session ID cookie between requests, while PHP keeps the counter and first-visit timestamp in the configured server-side session store.

## Question

How can the page remember a visitor across requests without placing the counter itself in a browser cookie?

## Complete example: `public/session-demo.php`

```php
<?php

declare(strict_types=1);

// Enable strict ID handling before starting the session.
if (ini_set('session.use_strict_mode', '1') === false) {
    http_response_code(500);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Could not enable strict session ID handling.\n");
}

// This check assumes PHP can see the direct HTTPS connection.
// Production applications behind a proxy should use trusted proxy-aware configuration.
$isHttps = isset($_SERVER['HTTPS'])
    && $_SERVER['HTTPS'] !== ''
    && $_SERVER['HTTPS'] !== 'off';

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => $isHttps,
    'httponly' => true,
    'samesite' => 'Lax',
]);

if (!session_start()) {
    http_response_code(500);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("The session could not be started.\n");
}

$pageLoads = $_SESSION['page_loads'] ?? 0;
if (!is_int($pageLoads) || $pageLoads < 0) {
    $pageLoads = 0;
}
$pageLoads++;
$_SESSION['page_loads'] = $pageLoads;

$firstSeen = $_SESSION['first_seen'] ?? null;
if (!is_int($firstSeen) || $firstSeen < 0) {
    $firstSeen = time();
    $_SESSION['first_seen'] = $firstSeen;
}

$firstSeenText = date(DATE_ATOM, $firstSeen);

// Save changes and release the session before rendering the response.
session_write_close();

header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PHP session demo</title>
</head>
<body>
    <main>
        <h1>PHP session demo</h1>
        <p>Page loads in this session: <strong><?= $pageLoads ?></strong></p>
        <p>First visit recorded by this session: <time><?= htmlspecialchars($firstSeenText, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8') ?></time></p>
        <p>Refresh the page to see the counter increase.</p>
        <p>The browser carries the session ID. The counter and timestamp are stored by PHP's configured session handler.</p>
    </main>
</body>
</html>
```

## How it works

1. **Strict ID handling:** The example enables `session.use_strict_mode` before opening a session. Production deployments should configure session security centrally and confirm that any custom save handler validates IDs.
2. **Cookie settings:** The session cookie uses a browser-session lifetime, site-wide path, `HttpOnly`, and `SameSite=Lax`. `Secure` is enabled when PHP sees a direct HTTPS connection.
3. **Session startup:** `session_start()` reads the session ID from the request and loads the associated server-side data into `$_SESSION`.
4. **State update:** The page validates the stored counter's type, increments it, and records the first visit time if needed.
5. **Early close:** `session_write_close()` saves the updates and releases the active session before rendering HTML.
6. **Refresh:** The browser sends its session ID on the next request, allowing PHP to load and increment the same counter.

This demo tracks anonymous page views only. It does not authenticate a visitor or authorize access to anything. A real login flow must verify credentials, regenerate the session ID before recording authentication, and enforce authorization on protected operations.

## Run it

From the project root, start PHP's built-in development server:

```bash
php -S localhost:8000 -t public
```

Open the page in a browser and refresh it several times:

```text
http://localhost:8000/session-demo.php
```

The page-load count should increase while the browser sends the same session cookie. On a plain-HTTP local server, the example's `Secure` setting is false so the browser can return the cookie. Use HTTPS in production and ensure the session cookie is marked `Secure`. Behind a TLS-terminating proxy, use trusted deployment configuration rather than trusting arbitrary forwarding headers.

## Inspect requests with `curl.exe`

Run these commands in a second terminal while the server is running. The first request saves the session cookie; later requests send it back:

```bash
curl.exe -i -c session-jar.txt http://localhost:8000/session-demo.php
curl.exe -i -b session-jar.txt -c session-jar.txt http://localhost:8000/session-demo.php
curl.exe -i -b session-jar.txt -c session-jar.txt http://localhost:8000/session-demo.php
```

The first response should include a `Set-Cookie` header for the session ID. Later requests should include a `Cookie` request header, and the HTML counter should advance. The ID value is sensitive; do not share terminal output containing it.

## Expected results

| Request | Session state | Visible page result |
| --- | --- | --- |
| First request without an existing session cookie | PHP creates a session and returns a session ID cookie | Page loads: `1`; first-visit timestamp is recorded |
| Second request with that cookie | PHP loads the same server-side session | Page loads: `2`; first-visit timestamp remains the same |
| Third request with the same cookie | PHP continues the same session | Page loads: `3`; first-visit timestamp remains the same |
| Request after the browser no longer sends that session ID | PHP starts a different session | Page loads: `1` for the new session |

The ID is not shown in the HTML. PHP's configured save handler controls where session data is stored and when obsolete data is cleaned up.

## Important notes

- A session ID is a bearer credential. Protect it with HTTPS and appropriate cookie settings; do not place it in a URL or reveal it in page output.
- The `Secure` detection shown assumes a direct TLS connection. Configure trusted proxy behavior explicitly in proxied deployments.
- `session_write_close()` saves and closes the active session. Do not expect later `$_SESSION` changes in the same request to be saved unless the session is started again.
- Browser cookie lifetime and server-side session cleanup are separate. Add an application-level timeout for authenticated sessions.
- In a real authentication flow, regenerate the session ID after successful login and before saving authenticated state. Handle concurrent requests and old-session invalidation deliberately.
- This visitor counter is only a learning demo; it is not an access-control or authentication design.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [090 - Cookies and browser state](../lessons/090-cookies-and-browser-state.md) | [Q&A](../qnas/091-sessions-and-server-side-state.md) | Example | [092 - Password hashing and verification](../lessons/092-password-hashing-and-verification.md) |