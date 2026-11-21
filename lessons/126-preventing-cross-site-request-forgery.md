# 126 - Preventing cross-site request forgery

Cross-site request forgery (CSRF) tricks a browser into sending an unwanted request to a site where the user is already signed in. Because browsers may attach that site's session cookie automatically, the server can mistake the forged request for an action the user intended.

Prevent CSRF by requiring a request-specific proof that an outside site cannot normally supply, and by designing state-changing routes carefully. This chapter uses PHP sessions and the synchronizer-token pattern, then explains cookie and origin checks as additional defenses.

## Learning goals

By the end of this chapter, you should be able to:

- Explain why authenticated browser requests can be forged cross-site.
- Keep GET requests free of state changes.
- Generate, render, and validate a session-bound CSRF token in PHP.
- Reject missing or invalid tokens before performing an action.
- Configure SameSite and other session cookie attributes as defense in depth.
- Explain the limits of CORS, Origin checks, and CSRF tokens.
- Apply CSRF protection to forms and cookie-authenticated APIs.

## How CSRF works

Suppose a browser is signed in to an application and holds its session cookie. A malicious page may cause that browser to submit a request to the application. The browser can include the application's cookie even though the user did not intentionally use the application form.

If the server only checks that the session is authenticated, it may accept the request as the user's action. A request to change an email address, update account settings, or make a purchase could then be forged.

CSRF relies on the browser's automatic credentials, such as cookies. The attacker may be able to make the browser send a request but generally cannot read the protected page's response because of the browser's same-origin policy. Preventing cross-origin reads does not by itself prevent the request from being sent.

## Keep safe methods safe

GET, HEAD, and other safe methods should retrieve information without changing application state. Do not use a link or GET route to delete a record, change an email address, log a user out, or perform another state-changing action.

Use a state-changing method such as POST for an HTML form that modifies state, and still validate a CSRF token. The HTTP method alone does not prove that the user intended the action.

Some clients and APIs also use PUT, PATCH, or DELETE. Apply the same CSRF policy to every state-changing route that relies on browser-attached credentials.

## Synchronizer-token pattern

A synchronizer token is a secret, unpredictable value associated with the user's server-side session:

1. The server creates a token using a cryptographically secure random source.
2. The server stores it in the session and includes it in a form or same-origin request header.
3. The browser submits the token with a state-changing request.
4. The server compares the submitted value with the session value.
5. The server rejects the request before the action if the token is missing or invalid.

An unrelated site can cause a browser to send a request, but it should not be able to read the application's page and obtain the session-bound token.

## Generate and render a token in PHP

Start the session before outputting HTML. Create a token for the session if one is not already present:

~~~php
session_start();

if (
    !isset($_SESSION['csrf_token'])
    || !is_string($_SESSION['csrf_token'])
) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}

$csrfToken = $_SESSION['csrf_token'];
~~~

random_bytes() returns cryptographically secure random bytes. Encoding the bytes as hexadecimal gives the form a printable string. Do not use rand(), mt_rand(), timestamps, or a predictable user value to create a security token.

Include the token as a hidden field in a POST form:

~~~php
<form method="post" action="/profile/update">
    <input
        type="hidden"
        name="csrf_token"
        value="<?= htmlspecialchars(
            $csrfToken,
            ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
            'UTF-8'
        ) ?>"
    >

    <label for="display-name">Display name</label>
    <input id="display-name" name="display_name" type="text">

    <button type="submit">Save profile</button>
</form>
~~~

The value is escaped for its HTML attribute context. A hexadecimal token contains a restricted character set, but ordinary output handling is still a good template habit.

Keep a per-session token stable for the session in a simple application. Creating a new token on every page request can invalidate forms open in other tabs or pages reached with the Back button. Per-request tokens can have tradeoffs; use a well-reviewed framework approach if an application needs them.

## Validate before changing state

Check the HTTP method and token before performing the protected action:

~~~php
if ($_SERVER['REQUEST_METHOD'] !== 'POST') {
    http_response_code(405);
    exit('Method not allowed.');
}

$sessionToken = $_SESSION['csrf_token'] ?? null;
$submittedToken = $_POST['csrf_token'] ?? null;

if (
    !is_string($sessionToken)
    || !is_string($submittedToken)
    || !hash_equals($sessionToken, $submittedToken)
) {
    http_response_code(403);
    exit('Request rejected.');
}

// Validate the form fields, authorize the action, and only then update data.
~~~

hash_equals() compares the known session token with the user-supplied token in a timing-safe manner. Pass the known value first and the submitted value second.

Do not perform part of the action before the token check. Missing, malformed, or mismatched values must fail closed: reject the request and do not update data.

CSRF validation does not replace normal validation or authorization. After the token passes, validate the submitted values and confirm that the signed-in user is allowed to perform the action.

## Session cookie protections

Configure session cookies deliberately. For an HTTPS application, PHP can set common protections before starting the session:

~~~php
session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();
~~~

Call session_set_cookie_params() before session_start() on each request where the settings are required. The secure option means browsers send the cookie only over HTTPS. HttpOnly prevents script from reading the cookie directly, and SameSite asks browsers to limit some cross-site cookie sending.

SameSite=Lax or Strict can reduce CSRF exposure, but cookie behavior has limits and compatibility tradeoffs. Lax can still send a cookie on some cross-site top-level navigations using safe methods. That is another reason GET routes must never change state. SameSite also does not defend against every deployment arrangement or client-side request-forgery flaw. Keep a CSRF token for protected cookie-authenticated actions.

HttpOnly is useful protection against direct cookie theft, but it does not stop a script running in the page from making same-origin requests. Fix XSS separately.

## Origin and Referer checks

For additional defense, a server can compare the request's Origin header with the application's configured origin. When Origin is absent, some applications also check Referer according to a documented policy.

Use a trusted configured origin, especially when the application is behind a reverse proxy. Do not build a trusted origin from an unverified Host or forwarded header. Compare the complete scheme, host, and port; a string-prefix check can accept a hostile lookalike domain.

Headers can be missing for legitimate privacy or browser reasons. Decide explicitly how the application handles such requests. For a protected action, use a valid CSRF token as the main check or a deliberate fallback; do not treat a missing header or the value null as proof of a same-origin request.

Origin checks complement the token rather than replacing it in a typical session-based form application.

## AJAX and APIs

For same-origin JavaScript requests, a token can be sent in a custom request header. The server must still validate it. Restrict the client to trusted application endpoints and do not attach a token indiscriminately to requests sent to arbitrary URLs.

Do not place tokens in query strings. URLs can be stored in browser history, server logs, analytics, screenshots, and referrer data.

CORS is not a general CSRF defense. It controls whether browser JavaScript can read or make certain cross-origin requests under specific conditions; simple cross-origin form submissions can still be sent. If an API uses cookies for authentication, apply a CSRF defense and configure CORS with explicit trusted origins. Do not combine credentialed requests with a permissive origin policy.

A service that uses an explicit bearer token in an Authorization header, rather than automatically attached browser cookies, has a different CSRF risk profile. Review how credentials are stored and attached instead of assuming every API is protected.

## Token patterns and common mistakes

### Do not put the token in a URL

Use a hidden form field or a protected same-origin request header. A URL may be copied, logged, saved in history, or sent in a referrer.

### Do not trust a token supplied only in a cookie

In the synchronizer-token pattern, compare a submitted token with the value stored in the server-side session. A cookie-only double-submit design is a different pattern and must be implemented with its own integrity and origin protections; do not casually treat it as equivalent.

### Do not accept missing tokens as an exception

A missing or invalid token should stop the protected action. Silently skipping validation for one route, browser, or request format can create a bypass.

### Do not rely on a token to fix XSS

A same-origin script injected through XSS may be able to read a token from the page or make a request that includes it. CSRF defenses and XSS defenses address different problems.

## A practical review checklist

For each authenticated state-changing feature:

1. Make sure GET and other safe methods do not change state.
2. Identify whether the browser automatically attaches the authentication credential.
3. Generate an unpredictable token on the server and bind it to the user's session.
4. Include it in each relevant form or same-origin request header.
5. Validate the token before performing the action; reject failures.
6. Validate the submitted fields and authorize the action separately.
7. Set session cookies with Secure, HttpOnly, and an intentional SameSite value.
8. Consider exact Origin or Referer checks as additional defenses.
9. Review AJAX destinations, CORS settings, and all alternate routes to the action.
10. Test missing, invalid, and valid tokens, including requests from another origin.

## Key takeaways

- CSRF abuses browser-attached credentials to forge a user's request.
- Keep GET and other safe methods free of state changes.
- Use a cryptographically random, session-bound token for protected actions.
- Compare the session token to the submitted value before changing state.
- SameSite cookies and origin checks are defense in depth; CORS alone is not a CSRF defense.
- CSRF tokens do not replace input validation, authorization, or XSS prevention.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [125 - Preventing cross-site scripting](125-preventing-cross-site-scripting.md) | [Q&A](../qnas/126-preventing-cross-site-request-forgery.md) | [Example](../examples/126-preventing-cross-site-request-forgery.md) | [127 - Authentication, authorization, and access control](127-authentication-authorization-and-access-control.md) |