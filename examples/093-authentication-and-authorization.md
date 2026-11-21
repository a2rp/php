# 093 - Authentication and authorization: Example

## Goal

Run a small PHP demo with one editor account. Sign in, view an article owned by that editor, then try the admin page and another editor's article. The example shows that authentication identifies the user, while authorization checks each requested resource and action.

## Question

How should a PHP application authenticate a user, protect session state, and deny access to an admin page or another user's article?

## Complete example: `public/auth-demo.php`

```php
<?php

declare(strict_types=1);

// Configure session handling before session_start().
if (
    ini_set('session.use_strict_mode', '1') === false
    || ini_set('session.use_only_cookies', '1') === false
) {
    http_response_code(500);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Could not apply the session security settings.\n");
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
    exit("Could not start the session.\n");
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

function csrfTokenIsValid(): bool
{
    $submittedToken = $_POST['_csrf'] ?? null;
    $expectedToken = $_SESSION['csrf_token'] ?? null;

    return is_string($submittedToken)
        && is_string($expectedToken)
        && hash_equals($expectedToken, $submittedToken);
}

if (!isset($_SESSION['csrf_token']) || !is_string($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$csrfToken = $_SESSION['csrf_token'];

// Public fixture credentials for this local tutorial only.
$demoUser = [
    'id' => 7,
    'email' => 'editor@example.test',
    'role' => 'editor',
    // This public sample hash matches the demo password: rasmuslerdorf.
    'password_hash' => '$2y$12$4Umg0rCJwMswRw/l.SwHvuQV01coP0eWmGzd61QH2RvAOMANUBGC.',
];
$usersByEmail = [$demoUser['email'] => $demoUser];
$usersById = [$demoUser['id'] => $demoUser];

$articles = [
    '42' => [
        'id' => 42,
        'owner_id' => 7,
        'title' => 'The editor’s own draft',
        'body' => 'This draft belongs to the signed-in editor.',
    ],
    '43' => [
        'id' => 43,
        'owner_id' => 99,
        'title' => 'Another editor’s draft',
        'body' => 'This draft belongs to a different user.',
    ],
];

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
if (!in_array($method, ['GET', 'POST'], true)) {
    http_response_code(405);
    header('Allow: GET, POST');
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Method not allowed.\n");
}

$page = $_GET['page'] ?? 'home';
if (!is_string($page)) {
    http_response_code(400);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Invalid page.\n");
}

$loginError = null;

if ($method === 'POST') {
    if (!csrfTokenIsValid()) {
        http_response_code(403);
        header('Content-Type: text/plain; charset=UTF-8');
        exit("The form token is invalid. Reload the page and try again.\n");
    }

    if ($page === 'login') {
        $emailInput = $_POST['email'] ?? null;
        $passwordInput = $_POST['password'] ?? null;
        $email = is_string($emailInput) ? strtolower(trim($emailInput)) : '';
        $user = $usersByEmail[$email] ?? null;

        // Verify against a stored hash even when the email is unknown.
        $hashToCheck = is_array($user)
            ? $user['password_hash']
            : $demoUser['password_hash'];
        $passwordIsValid = is_string($passwordInput)
            && password_verify($passwordInput, $hashToCheck)
            && is_array($user);

        if (!$passwordIsValid) {
            http_response_code(401);
            $loginError = 'Invalid email or password.';
        } elseif (!session_regenerate_id()) {
            http_response_code(500);
            header('Content-Type: text/plain; charset=UTF-8');
            exit("Could not refresh the session ID.\n");
        } else {
            // Rotate the form token and store only server-side identity state.
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
            $_SESSION['user_id'] = $user['id'];
            $_SESSION['authenticated_at'] = time();

            header('Location: /auth-demo.php?page=dashboard', true, 303);
            exit;
        }
    } elseif ($page === 'logout') {
        // Remove server-side state and expire the browser's session cookie.
        $_SESSION = [];
        $cookie = session_get_cookie_params();
        setcookie(session_name(), '', [
            'expires' => time() - 42000,
            'path' => $cookie['path'],
            'domain' => $cookie['domain'],
            'secure' => $cookie['secure'],
            'httponly' => $cookie['httponly'],
            'samesite' => $cookie['samesite'] ?? 'Lax',
        ]);
        session_destroy();

        header('Location: /auth-demo.php', true, 303);
        exit;
    } else {
        http_response_code(405);
        header('Allow: GET');
        header('Content-Type: text/plain; charset=UTF-8');
        exit("This page does not accept POST requests.\n");
    }
}

$sessionUserId = $_SESSION['user_id'] ?? null;
$currentUser = is_int($sessionUserId)
    ? ($usersById[$sessionUserId] ?? null)
    : null;

if ($currentUser === null) {
    unset($_SESSION['user_id'], $_SESSION['authenticated_at']);
}

$status = http_response_code() ?: 200;
$title = 'Authentication and authorization demo';
$content = '';

if ($page === 'home') {
    $content = '<h1>Authentication and authorization demo</h1>'
        . '<p>This demo distinguishes signing in from permission checks.</p>'
        . '<p><a href="/auth-demo.php?page=login">Sign in</a></p>';
} elseif ($page === 'login') {
    $title = 'Sign in';
    $content = '<h1>Sign in</h1>';

    if ($loginError !== null) {
        $content .= '<p role="alert">' . escapeHtml($loginError) . '</p>';
    }

    $content .= '<form action="/auth-demo.php?page=login" method="post">'
        . '<input type="hidden" name="_csrf" value="' . escapeHtml($csrfToken) . '">'
        . '<label for="email">Email</label>'
        . '<input id="email" name="email" type="email" autocomplete="username" required>'
        . '<label for="password">Password</label>'
        . '<input id="password" name="password" type="password" autocomplete="current-password" required>'
        . '<button type="submit">Sign in</button>'
        . '</form>'
        . '<p>Local demo credentials: <code>editor@example.test</code> / <code>rasmuslerdorf</code></p>'
        . '<p>This public sample credential is for local learning only.</p>';
} elseif ($page === 'dashboard') {
    $title = 'Dashboard';

    if ($currentUser === null) {
        http_response_code(401);
        $content = '<h1>Sign in required</h1>'
            . '<p><a href="/auth-demo.php?page=login">Sign in to continue</a></p>';
    } else {
        $content = '<h1>Dashboard</h1>'
            . '<p>Signed in as <strong>' . escapeHtml($currentUser['email']) . '</strong>.</p>'
            . '<p>Role loaded by the server: <code>' . escapeHtml($currentUser['role']) . '</code></p>'
            . '<ul>'
            . '<li><a href="/auth-demo.php?page=article&amp;id=42">View your own article (42)</a></li>'
            . '<li><a href="/auth-demo.php?page=article&amp;id=43">Try another editor’s article (43)</a></li>'
            . '<li><a href="/auth-demo.php?page=admin">Try the admin page</a></li>'
            . '</ul>'
            . '<form action="/auth-demo.php?page=logout" method="post">'
            . '<input type="hidden" name="_csrf" value="' . escapeHtml($csrfToken) . '">'
            . '<button type="submit">Sign out</button>'
            . '</form>';
    }
} elseif ($page === 'admin') {
    $title = 'Admin page';

    if ($currentUser === null) {
        http_response_code(401);
        $content = '<h1>Sign in required</h1><p>Sign in before requesting this page.</p>';
    } elseif ($currentUser['role'] !== 'admin') {
        http_response_code(403);
        $content = '<h1>Forbidden</h1><p>You are signed in, but this demo account is not an administrator.</p>';
    } else {
        $content = '<h1>Admin page</h1><p>Administrator access granted.</p>';
    }
} elseif ($page === 'article') {
    $title = 'Article';
    $requestedId = $_GET['id'] ?? null;
    $article = is_string($requestedId) ? ($articles[$requestedId] ?? null) : null;

    if ($currentUser === null) {
        http_response_code(401);
        $content = '<h1>Sign in required</h1><p>Sign in before requesting an article.</p>';
    } elseif ($article === null) {
        http_response_code(404);
        $content = '<h1>Not found</h1><p>This article does not exist in the demo.</p>';
    } elseif ($currentUser['role'] !== 'admin' && $article['owner_id'] !== $currentUser['id']) {
        http_response_code(403);
        $content = '<h1>Forbidden</h1><p>You cannot view this article because it belongs to another user.</p>';
    } else {
        $content = '<h1>' . escapeHtml($article['title']) . '</h1>'
            . '<p>' . escapeHtml($article['body']) . '</p>'
            . '<p>Authorization passed for article ' . $article['id'] . '.</p>';
    }
} else {
    http_response_code(404);
    $title = 'Not found';
    $content = '<h1>Not found</h1><p>Unknown page.</p>';
}

$status = http_response_code() ?: 200;
http_response_code($status);
header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= escapeHtml($title) ?></title>
</head>
<body>
    <main>
        <?= $content ?>
        <p><a href="/auth-demo.php">Home</a></p>
    </main>
</body>
</html>
```

## How it works

1. **Session setup:** The file enables strict ID handling and cookie-only session IDs before `session_start()`. The cookie is `HttpOnly`, uses `SameSite=Lax`, and is marked `Secure` when PHP sees a direct HTTPS connection.
2. **CSRF token:** The server creates a random token in the session. The login and logout forms include it, and POST requests must return the matching token.
3. **Authentication:** The demo verifies the submitted password against the editor's stored hash. Wrong credentials get the same generic message.
4. **Session rotation:** On successful login, PHP regenerates the session ID before storing the user ID and authentication time.
5. **Server-side identity:** The current user record and role are loaded from the server-side demo data using the session user ID. The request cannot submit a role to become an administrator.
6. **Authorization:** The admin route checks the role. The article route checks both the role and the particular article's owner ID.
7. **Logout:** A CSRF-checked POST clears session data, expires the session cookie using its configured scope, destroys the current session, and redirects home.
8. **Escaping:** User and record text is escaped before being inserted into HTML.

## Run it

From the project root, start PHP's built-in development server:

```bash
php -S localhost:8000 -t public
```

Open the home page:

```text
http://localhost:8000/auth-demo.php
```

Sign in with the local demo credentials shown on the form. From the dashboard, try both article links, the admin link, and the sign-out button.

## Expected behavior

| Action | Expected result |
| --- | --- |
| Open dashboard without signing in | `401` and a sign-in prompt |
| Sign in with the displayed demo credentials | `303` redirect to the dashboard; the session ID is regenerated |
| Sign in with an incorrect email or password | `401` with the same generic “Invalid email or password” message |
| Open article 42 as the editor | `200`; the article is owned by the signed-in user |
| Open article 43 as the editor | `403`; the article belongs to another user |
| Open the admin page as the editor | `403`; authentication succeeded, but the role check fails |
| Submit login or logout without the matching CSRF token | `403` and no state-changing action is performed |
| Sign out through the form | `303` redirect home; session data is cleared and the cookie is expired |

The account hash and password are public tutorial fixtures. They are not suitable for a real application. In production, load users and permissions from a database, use real account credentials, add rate limiting and monitoring, and configure HTTPS and session settings for the deployment.

## Important notes

- The demo's hard-coded user array replaces a database only for a self-contained learning example. Never hard-code production users or credentials.
- Its sample password is public by design; never use it outside this local tutorial.
- The HTTPS check assumes a direct connection. Behind a TLS proxy, use trusted proxy-aware configuration rather than trusting arbitrary forwarding headers.
- A CSRF token does not prove that the user may access an article. The server still checks authentication and ownership for each request.
- A hidden link is not an authorization rule. Try the protected URLs directly to see the server-side checks.
- The example returns `403` for a known but unauthorized article. Some applications intentionally return `404` to conceal whether a protected resource exists.
- This is a learning demo, not a production authentication system. Production systems also need safe account recovery, rate limiting, abuse monitoring, robust session lifecycle handling, and deployment-specific security review.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [092 - Password hashing and verification](../lessons/092-password-hashing-and-verification.md) | [Q&A](../qnas/093-authentication-and-authorization.md) | Example | [094 - Handling file uploads](../lessons/094-handling-file-uploads.md) |