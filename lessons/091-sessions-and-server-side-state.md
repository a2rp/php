# 091 - Sessions and server-side state

## Learning goals

By the end of this lesson, you will be able to:

- Explain how PHP connects a browser request to server-side session data.
- Start a session and read or update values in `$_SESSION`.
- Configure session cookies before starting a session.
- Explain why a session ID is a sensitive bearer credential.
- Regenerate the session ID when authentication or privileges change.
- Clear session data, expire the browser cookie, and understand the limits of `session_destroy()`.
- Close a session early when the request no longer needs to read or write it.

## 1. What a session is

A session lets an application keep state between HTTP requests. HTTP requests are independent, so PHP needs a way to recognize that later requests belong to the same session.

With PHP's usual cookie-based setup, the browser stores a session ID cookie, commonly named `PHPSESSID`. It sends that ID with later matching requests. PHP uses the ID to find the associated session data through the configured session handler.

```text
Browser                         PHP application and session storage
   |                                      |
   | request: no session cookie           |
   |------------------------------------->|
   |                                      | create session data
   | response: Set-Cookie: PHPSESSID=...  |
   |<-------------------------------------|
   |                                      |
   | next request: Cookie: PHPSESSID=...  |
   |------------------------------------->| load associated session data
```

The session ID is the link between the browser and the server-side state. Anyone who obtains a valid session ID may be able to act as that session's user. Protect it like a credential.

Session data is handled by the configured session save handler. PHP's default handler stores it on the server, while applications can configure other handlers. The browser should normally receive only the opaque ID, not the session's private data.

## 2. Start the session before using `$_SESSION`

Call `session_start()` before output. It sends or reads session-related headers, then loads existing data into `$_SESSION` or creates a new session.

```php
<?php

session_start();

$_SESSION['display_name'] = 'Amina';
$_SESSION['cart_count'] = 2;
```

On a later request that sends the same valid session ID, PHP can read those values again:

```php
<?php

session_start();

$displayName = $_SESSION['display_name'] ?? 'Guest';
$cartCount = $_SESSION['cart_count'] ?? 0;
```

As with any input, check types and expected shapes before using values. Keep session entries small and purposeful; store a database identifier or a few necessary flags rather than copying an entire database record into every session.

## 3. Configure cookie-based sessions

Configure session behavior before calling `session_start()`. For a production site served over HTTPS, a baseline configuration commonly includes:

```ini
session.use_strict_mode = 1
session.use_only_cookies = 1
session.cookie_lifetime = 0
session.cookie_path = /
session.cookie_secure = 1
session.cookie_httponly = 1
session.cookie_samesite = Lax
```

Set `session.cookie_secure = 1` only when the application is accessed over HTTPS. If TLS terminates at a reverse proxy, configure the application to recognize HTTPS through trusted deployment settings; do not trust arbitrary client-supplied forwarding headers.

The same cookie options can be set in application bootstrap before the first session starts:

```php
<?php

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

session_start();
```

This connection check assumes PHP sees a direct HTTPS connection. Behind a proxy, use the hosting environment's trusted HTTPS configuration. A local plain-HTTP development server may need `secure` to be `false` so the browser sends the cookie back; keep it enabled in production HTTPS deployments.

`session.use_strict_mode` tells PHP to reject uninitialized session IDs instead of adopting an arbitrary ID supplied by a client. Use cookie-based session IDs; do not put session IDs in URLs, where they can leak through copied links, browser history, or logs.

## 4. Keep authentication state on the server

A session ID is a bearer credential: possession can grant access to the session. Use HTTPS, protect the session cookie with appropriate attributes, and never print the ID into a page or log it unnecessarily.

After a user successfully authenticates or receives a privilege increase, regenerate the ID before saving the new authentication state:

```php
<?php

// The session has already been started and credentials were verified.
if (!session_regenerate_id()) {
    http_response_code(500);
    exit('Could not refresh the session.');
}

$_SESSION['user_id'] = $verifiedUserId;
$_SESSION['authenticated_at'] = time();
```

Regeneration reduces session-fixation risk by moving the active session to a new ID. Real applications with parallel requests or unreliable connections need a session lifecycle strategy: deleting old session data immediately can cause races or log out legitimate in-flight requests. Follow the session handler's guidance and ensure obsolete authenticated state is not accepted.

`session.use_strict_mode` and ID regeneration address different parts of session security. Neither replaces authentication checks, authorization checks, HTTPS, or protection from cross-site scripting and cross-site request forgery.

## 5. Enforce application timeouts

A browser cookie lifetime and the lifetime of server-side session data are separate. A session cookie may be removed by the browser while old session data remains on the server until cleanup. Conversely, a user may still hold a session ID after the application should consider the login inactive.

Do not rely only on garbage-collection timing to enforce an authentication timeout. Track an application timestamp and check it on requests that require authentication:

```php
$now = time();
$idleLimit = 30 * 60;
$lastActivity = $_SESSION['last_activity'] ?? null;

if (is_int($lastActivity) && ($now - $lastActivity) > $idleLimit) {
    unset($_SESSION['user_id'], $_SESSION['authenticated_at']);
}

$_SESSION['last_activity'] = $now;
```

The example illustrates an inactivity check, not a complete logout or session-revocation design. A production application should decide how to invalidate expired authentication state and coordinate that policy across requests and devices.

## 6. End or clear a session correctly

`session_destroy()` removes data associated with the current session on the server, but it does not clear `$_SESSION` in the current script or remove the browser's session cookie. A logout flow may need to clear all three deliberately.

```php
<?php

session_start();
$_SESSION = [];

if (ini_get('session.use_cookies')) {
    $cookie = session_get_cookie_params();

    setcookie(session_name(), '', [
        'expires' => time() - 42000,
        'path' => $cookie['path'],
        'domain' => $cookie['domain'],
        'secure' => $cookie['secure'],
        'httponly' => $cookie['httponly'],
        'samesite' => $cookie['samesite'] ?? 'Lax',
    ]);
}

session_destroy();
```

This sends an expired cookie with the same name, path, and domain scope. The code is a basic single-session logout pattern; applications with concurrent requests, custom handlers, or account-wide revocation need a deliberate invalidation strategy. Do not treat a missing browser cookie as proof that server-side state was immediately erased.

For ordinary updates, clearing selected `$_SESSION` keys is often preferable to destroying the whole session. Reserve full destruction for a lifecycle action such as logout, and ensure the application also removes authenticated state.

## 7. Close the session when finished

The default file-based session handler locks a session while a request has it open. A second request using the same session may wait while the first request holds that lock.

When the request has finished changing session data, call `session_write_close()` before slow work or long responses:

```php
<?php

session_start();
$_SESSION['last_seen'] = time();
session_write_close();

// Slow report generation can continue without holding the session open.
```

PHP writes the session changes and ends the active session. Do not modify `$_SESSION` after closing it unless you deliberately start the session again. For a request that only needs to read session data, `session_start(['read_and_close' => true])` can avoid holding the session open for the rest of the request.

## 8. Common mistakes

### Starting a session after output

```php
<?php

echo '<h1>Welcome</h1>';
session_start(); // May fail because output has begun.
```

Start the session in the request bootstrap before rendering or sending whitespace.

### Treating the session ID as harmless

The ID is the credential that connects a request to server-side state. Do not expose it in URLs or HTML, and use HTTPS with protected cookie settings.

### Storing authorization decisions only in a browser value

Keep identity and permissions in server-controlled state. Validate the user and permission for each protected operation; a session entry is not a substitute for authorization logic.

### Assuming `session_destroy()` expires the cookie

It does not remove the browser cookie or clear the current `$_SESSION` array. Explicitly clear the data, expire the cookie with its matching scope, and destroy server-side data when the application's logout policy requires it.

### Holding a session lock during slow work

Close the session after the required reads and writes. This can let other requests from the same browser continue without waiting for a long-running operation to release the session lock.

## Key takeaways

- `session_start()` loads or creates session state and must run before output.
- The browser normally stores a session ID; the configured server-side handler stores the session data.
- Treat the session ID as a credential: use HTTPS, cookie protections, strict mode, and regenerate it when authentication privileges change.
- Enforce application timeouts explicitly; cookie lifetime and server-side data cleanup are separate.
- `session_destroy()` alone does not clear `$_SESSION` or remove the browser cookie.
- Call `session_write_close()` once the request no longer needs the session open.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [090 - Cookies and browser state](090-cookies-and-browser-state.md) | [Q&A](../qnas/091-sessions-and-server-side-state.md) | [Example](../examples/091-sessions-and-server-side-state.md) | [092 - Password hashing and verification](092-password-hashing-and-verification.md) |