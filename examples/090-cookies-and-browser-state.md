# 090 - Cookies and browser state: Example

## Goal

Create a small PHP page that remembers a visitor's light or dark theme preference in a cookie. The page will set or delete the cookie, redirect, and then read the value from the browser's next request.

## Question

How can a PHP page remember a small browser preference without assuming that a cookie is trusted or immediately available in the request that sets it?

## Complete example: `public/cookie-demo.php`

```php
<?php

declare(strict_types=1);

function respondWithError(int $status, string $message): void
{
    http_response_code($status);
    header('Content-Type: text/plain; charset=UTF-8');
    echo $message;
    exit;
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if (!in_array($method, ['GET', 'POST'], true)) {
    http_response_code(405);
    header('Allow: GET, POST');
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Method not allowed. Use GET or POST.\n");
}

$cookieName = 'demo_theme';
$allowedThemes = ['light', 'dark'];

// This check assumes PHP can see the direct HTTPS connection.
// Production applications behind a proxy should use trusted proxy-aware configuration.
$isHttps = isset($_SERVER['HTTPS'])
    && $_SERVER['HTTPS'] !== ''
    && $_SERVER['HTTPS'] !== 'off';

$cookieOptions = [
    'path' => '/',
    'secure' => $isHttps,
    'httponly' => true,
    'samesite' => 'Lax',
];

if ($method === 'POST') {
    $action = $_POST['action'] ?? null;

    if ($action === 'set-theme') {
        $requestedTheme = $_POST['theme'] ?? null;

        if (
            !is_string($requestedTheme)
            || !in_array($requestedTheme, $allowedThemes, true)
        ) {
            respondWithError(400, "Choose either the light or dark theme.\n");
        }

        $options = $cookieOptions;
        $options['expires'] = time() + (30 * 24 * 60 * 60);

        if (!setcookie($cookieName, $requestedTheme, $options)) {
            respondWithError(500, "The preference cookie could not be sent.\n");
        }
    } elseif ($action === 'clear-theme') {
        $options = $cookieOptions;
        $options['expires'] = time() - 3600;

        if (!setcookie($cookieName, '', $options)) {
            respondWithError(500, "The preference cookie could not be cleared.\n");
        }
    } else {
        respondWithError(400, "Unknown action.\n");
    }

    // The next GET request will contain the updated cookie, if the browser accepts it.
    header('Location: /cookie-demo.php', true, 303);
    exit;
}

$theme = $_COOKIE[$cookieName] ?? 'light';

if (!is_string($theme) || !in_array($theme, $allowedThemes, true)) {
    $theme = 'light';
}

$themeLabel = $theme === 'dark' ? 'Dark' : 'Light';
header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Cookie preference demo</title>
</head>
<body data-theme="<?= escapeHtml($theme) ?>">
    <main>
        <h1>Cookie preference demo</h1>
        <p>Theme read from this request: <strong><?= escapeHtml($themeLabel) ?></strong></p>
        <p>Validated cookie value: <code><?= escapeHtml($theme) ?></code></p>

        <form action="/cookie-demo.php" method="post">
            <input type="hidden" name="action" value="set-theme">
            <button type="submit" name="theme" value="light">Use light theme</button>
            <button type="submit" name="theme" value="dark">Use dark theme</button>
        </form>

        <form action="/cookie-demo.php" method="post">
            <input type="hidden" name="action" value="clear-theme">
            <button type="submit">Delete preference cookie</button>
        </form>

        <p>The cookie stores only a display preference. It is not used for identity or access control.</p>
    </main>
</body>
</html>
```

## How it works

1. **Method handling:** The page allows `GET` and `POST`. Other methods receive `405` and an `Allow` header.
2. **Cookie options:** The preference is scoped to `/`, is `HttpOnly`, and uses `SameSite=Lax`. The demo sets `Secure` when PHP sees a direct HTTPS connection.
3. **Input validation:** The POST handler accepts only the two known theme values. It does not copy arbitrary form input into the cookie.
4. **Set:** `setcookie()` sends a 30-day preference cookie. The response then redirects with `303` so the browser makes a new GET request.
5. **Read:** The GET request reads the cookie through `$_COOKIE`. Missing or unexpected values fall back to `light`.
6. **Delete:** The delete action expires the cookie using the same name and path scope. It redirects so the next request reflects the browser's updated state.
7. **Output:** The value is validated and escaped before it is placed in the HTML.

The forms change only a low-impact display preference. In a real application, protect sensitive state-changing actions with CSRF defenses and authorization. Do not reuse this preference cookie as an authentication token.

## Run it

From the project root, start PHP's built-in development server:

```bash
php -S localhost:8000 -t public
```

Open this page in a browser:

```text
http://localhost:8000/cookie-demo.php
```

Select **Use dark theme**, then select **Delete preference cookie**. The page redirects after each action and displays the cookie value received by the following request. On a local plain-HTTP server, the demo's `Secure` option is false so the browser can send the cookie back. Use HTTPS in production and ensure the application sets `Secure` there. If TLS ends at a reverse proxy, configure the application to recognize HTTPS only through trusted proxy settings.

## Inspect the cookie with `curl.exe`

Run these commands in a second terminal while the server is running. The cookie jar stores cookies between requests:

```bash
curl.exe -i -c cookie-jar.txt -b cookie-jar.txt -L -d "action=set-theme&theme=dark" http://localhost:8000/cookie-demo.php
curl.exe -i -b cookie-jar.txt http://localhost:8000/cookie-demo.php
curl.exe -i -c cookie-jar.txt -b cookie-jar.txt -L -d "action=clear-theme" http://localhost:8000/cookie-demo.php
```

The first response includes a `Set-Cookie` header and a `303` redirect. The later GET sends the cookie back; after deletion, the page falls back to `light`.

## Expected results

| Action | Response behavior | Page after the redirect |
| --- | --- | --- |
| Open the page for the first time | `200 OK`; no preference cookie is required | Theme read as `Light` (`light`) |
| Choose **Use dark theme** | `Set-Cookie` requests a 30-day value; response is `303 See Other` | Theme read as `Dark` (`dark`) |
| Choose **Use light theme** | The cookie value is replaced; response is `303 See Other` | Theme read as `Light` (`light`) |
| Choose **Delete preference cookie** | An expired cookie with the same path is sent; response is `303 See Other` | No preference cookie is received, so the page reads `Light` (`light`) |
| Submit an unsupported theme value | `400 Bad Request` | A plain-text validation message |
| Send a `PUT` request | `405 Method Not Allowed` with `Allow: GET, POST` | A plain-text method message |

On HTTPS, the response cookie includes `Secure`; on the local HTTP server it does not. `HttpOnly` means browser JavaScript cannot read this cookie through the usual cookie API, while PHP can still read it from the next request.

## Important notes

- A cookie is returned on a later matching request; `setcookie()` does not change `$_COOKIE` in the current request.
- The value is still client-controlled. The allowlist check is required even though the server originally issued the cookie.
- This example omits `Domain`, so the cookie is host-only. The delete action uses the same path and omits `Domain` as well.
- `SameSite=Lax` is useful defense in depth, not a replacement for CSRF protections on important actions.
- The HTTPS detection shown assumes a direct TLS connection. Behind a proxy, use trusted deployment configuration instead of trusting arbitrary forwarding headers.
- Cookie behavior and storage can vary by browser. The server can request attributes, but cannot force a browser to accept or retain a cookie.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [089 - Response headers, status codes, and redirects](../lessons/089-response-headers-status-codes-and-redirects.md) | [Q&A](../qnas/090-cookies-and-browser-state.md) | Example | [091 - Sessions and server-side state](../lessons/091-sessions-and-server-side-state.md) |