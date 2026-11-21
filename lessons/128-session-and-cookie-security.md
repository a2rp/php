# 128 - Session and cookie security

A PHP session lets an application recognize related requests from the same browser. The browser usually sends an opaque session ID in a cookie, while the application keeps the session data on the server. That ID is a bearer credential: someone who steals it may be able to act as the signed-in user.

Secure session handling requires more than setting a cookie flag. Configure PHP to accept session IDs only through cookies, reject unknown IDs, use HTTPS for the whole session, rotate identifiers at privilege changes, enforce server-side expiration, and invalidate sessions completely at logout.

## Learning goals

By the end of this chapter, you should be able to:

- Explain how a session cookie identifies server-side session data.
- Configure PHP session handling and cookie attributes before starting a session.
- Use strict session mode and cookie-only session IDs.
- Reduce session fixation and session theft risks.
- Enforce idle and absolute session timeouts on the server.
- End a session and expire its cookie during logout.
- Recognize why browser-close behavior and PHP garbage collection do not provide reliable expiration.
- Test session lifecycle and cookie behavior.

## What a session does

HTTP requests are independent. A session gives the application a server-side way to associate later requests with state created earlier. In a typical PHP application:

1. PHP creates a random session ID and associates server-side session data with it.
2. The response gives the browser a cookie containing that ID.
3. The browser sends the cookie on later requests to the cookie's host and path.
4. PHP uses the ID to retrieve the corresponding session data.

The cookie should contain an opaque identifier, not the user's password, personal profile, permissions, or other application state. Keep the data on the server and use the session ID only to look up the session.

A session ID is a credential, not just a label. Anyone who obtains a valid ID may be able to use the session. High randomness helps prevent guessing, but does not protect an ID that leaks through an insecure connection, URL, log, browser extension, or compromised device.

## Configure sessions before starting them

Session settings that affect ID acceptance and cookies must be in place before session_start(). Set stable production defaults in the PHP configuration when possible. A small application can configure them at the start of its request bootstrap:

~~~php
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
~~~

This example is for a site served over HTTPS. The Secure flag must remain enabled in production. If TLS ends at a reverse proxy, use trusted deployment configuration to determine HTTPS; do not enable Secure based on an arbitrary client-supplied forwarding header.

- session.use_strict_mode tells PHP to reject an uninitialized session ID rather than creating a session around an ID supplied by a client.
- session.use_only_cookies prevents session IDs from being accepted in URLs or request bodies through PHP's session ID mechanisms.
- Secure tells the browser to send the cookie only over HTTPS.
- HttpOnly prevents ordinary browser scripts from reading the cookie through document.cookie.
- SameSite controls whether the browser sends the cookie in some cross-site request contexts.
- A lifetime of zero requests a browser-session cookie; it does not enforce a server-side timeout.

Use session_start() before reading or writing $_SESSION and before sending output. Otherwise, PHP may be unable to send the session cookie or other response headers.

## Choose cookie scope deliberately

Prefer a host-only cookie by omitting the Domain attribute unless the application has a specific need to share the session across subdomains. A broad Domain can expose the cookie to more hosts than the application requires. Keep the Path and other cookie options consistent when refreshing or deleting the cookie.

Path is a routing scope, not a complete security boundary. Applications hosted at different paths on the same host should not assume that Path prevents all forms of cross-application interaction.

Where PHP and the deployment support it, a cookie name using the __Host- prefix can add browser-enforced constraints: the cookie must be Secure, use Path=/, and omit Domain. Use it only when those constraints match the application. Otherwise, a host-only cookie with explicit attributes is still a useful baseline.

Choose SameSite intentionally:

- Lax is often a practical default for browser applications. It restricts many cross-site cookie sends while permitting some top-level safe navigations.
- Strict blocks more cross-site cookie contexts, but may interfere with legitimate links into the site or external sign-in flows.
- None permits cross-site cookie sending and must be paired with Secure. Use it only when the application actually requires cross-site cookies.

SameSite is defense in depth. It does not replace CSRF tokens on state-changing requests that use browser-attached session cookies.

## Use HTTPS for the entire session

Protect the session ID from the login page through logout. Do not send it over HTTP and then switch to HTTPS; an attacker who sees it on an unencrypted connection may reuse it. Redirect HTTP requests to HTTPS before issuing an authenticated session cookie, and configure the application and hosting layer so protected routes stay on HTTPS.

The Secure cookie flag helps ensure that browsers do not send the session cookie over HTTP. It is not a substitute for HTTPS: HTTPS protects the full connection, while Secure controls whether the browser sends this particular cookie.

## Prevent session fixation

Session fixation is an attack where someone tries to make a victim use a session ID the attacker already knows. After verifying a login, rotate the identifier before storing the authenticated identity:

~~~php
if ($credentialsAreValid) {
    session_regenerate_id(true);

    $_SESSION['user_id'] = (int) $user['id'];
    $_SESSION['created_at'] = time();
    $_SESSION['last_activity'] = time();
}
~~~

Regenerate the ID after authentication and after meaningful privilege changes, such as completing multi-factor authentication or entering an administrator mode. Do not accept a user ID or authenticated flag from the request.

Session rotation has lifecycle details. Requests can overlap, and a user may have an unstable network or several tabs open. Deleting an old session immediately can disrupt concurrent requests in some applications. Use the framework's session handling when available, and follow its guidance for invalidating old IDs and handling concurrent requests. Do not regenerate the ID blindly on every page view.

## Enforce server-side expiration

A cookie with lifetime zero does not guarantee logout when the browser closes. Session restore features may keep browser-session cookies. Likewise, PHP's garbage collector removes old storage opportunistically; its cleanup schedule is not a security timeout and it may not run on each request.

Enforce an idle timeout and an absolute maximum lifetime in application logic. At login, store server-side timestamps such as created_at and last_activity. On every authenticated request:

1. Check that the timestamps exist and have valid types.
2. End the session if the idle limit has passed.
3. End the session if the absolute lifetime has passed.
4. Otherwise update last_activity and continue.

For example, an application might choose a 20-minute idle limit and an 8-hour absolute limit, then adjust them to the sensitivity of its data and its user needs:

~~~php
$idleLimit = 20 * 60;
$absoluteLimit = 8 * 60 * 60;
$now = time();

$createdAt = $_SESSION['created_at'] ?? null;
$lastActivity = $_SESSION['last_activity'] ?? null;

$invalidTimestamps = !is_int($createdAt) || !is_int($lastActivity);
$idleExpired = !$invalidTimestamps
    && ($now - $lastActivity) > $idleLimit;
$absoluteExpired = !$invalidTimestamps
    && ($now - $createdAt) > $absoluteLimit;

if ($invalidTimestamps || $idleExpired || $absoluteExpired) {
    // Invalidate the session and require authentication again.
} else {
    $_SESSION['last_activity'] = $now;
}
~~~

The limits above are examples, not universal values. Sensitive applications may require shorter limits or step-up authentication for high-risk actions. Do not rely on a timestamp supplied by the browser.

A server-side timeout should be checked before returning protected data or performing an action. Do not treat a JavaScript countdown as enforcement; client-side code can be disabled or altered.

## Invalidate a session on logout

A complete logout clears session data, expires the browser's session cookie using matching cookie scope, and invalidates the server-side session:

~~~php
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
~~~

The cookie deletion must use the same name, path, and domain scope as the cookie being removed. session_destroy() removes server-side data for the current session, but by itself it does not clear the $_SESSION array or expire the browser cookie.

Make logout a state-changing POST route and validate its CSRF token. A cross-site page should not be able to force logout simply by causing a browser request. After logout, redirect to a public page and send no sensitive cached response.

If the product offers “log out all devices,” track the user's active sessions in a server-side store so they can all be revoked. Deleting one browser's cookie does not invalidate copies of the session ID held elsewhere.

## Protect the session store and responses

Keep session storage outside the public web root, restrict access to the PHP worker account, and use a supported session handler appropriate to the deployment. If the application uses several web servers, configure a shared session store or another consistent session mechanism. Avoid custom session handlers unless they validate IDs and safely handle concurrent writes.

Do not put session IDs in URLs, hidden fields, logs, analytics, or error messages. Avoid logging full cookie headers. Validate the shape of session values before using them; session data should be written only by trusted server-side code, but deployments and old data can still produce unexpected values.

Sensitive authenticated pages should use an appropriate cache policy, commonly Cache-Control: no-store. This helps prevent private responses from being stored by browser and intermediary caches. It does not erase screenshots, downloads, or copies a user has already made.

HttpOnly prevents ordinary JavaScript from reading a session cookie, but an XSS flaw may still let injected code issue requests as the user. Continue to escape output and prevent XSS. Cookie flags also do not replace authorization checks or CSRF defenses.

## Common mistakes

- Starting the session after output has already been sent.
- Leaving strict mode disabled or accepting session IDs in URLs.
- Setting HttpOnly but forgetting Secure on an HTTPS site.
- Setting cookie attributes after session_start().
- Using a broad cookie Domain without a real need.
- Treating SameSite as a complete CSRF solution.
- Assuming the browser closing or PHP garbage collection ends a session on time.
- Rotating an ID on every request or deleting active session state without considering concurrent requests.
- Calling session_destroy() and assuming that it also clears the cookie and $_SESSION.
- Deleting a cookie with a different path or domain from the one used to create it.
- Logging session IDs or exposing them in URLs.
- Assuming a cookie flag prevents XSS, stolen-device access, or unauthorized actions.

## Key takeaways

- Treat every session ID as a secret bearer credential.
- Enable strict mode and cookie-only session IDs before session_start().
- Use HTTPS and explicit Secure, HttpOnly, and SameSite cookie attributes.
- Rotate IDs after login and privilege changes with a lifecycle strategy that accounts for concurrent requests.
- Enforce idle and absolute expiration on the server; browser-close behavior and garbage collection are not enough.
- Logout must clear data, invalidate the server-side session, and expire the correctly scoped cookie.
- Keep CSRF protection, XSS prevention, and authorization checks in place alongside cookie protections.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [127 - Authentication, authorization, and access control](127-authentication-authorization-and-access-control.md) | [Q&A](../qnas/128-session-and-cookie-security.md) | [Example](../examples/128-session-and-cookie-security.md) | [129 - Path traversal and filesystem security](129-path-traversal-and-filesystem-security.md) |