# 090 - Cookies and browser state

## Learning goals

By the end of this lesson, you will be able to:

- Explain how a server sets a cookie and how a browser sends it on a later request.
- Set cookie expiration and scope with PHP's `setcookie()` options.
- Read and validate cookie values from `$_COOKIE`.
- Update and delete cookies correctly.
- Choose `Secure`, `HttpOnly`, and `SameSite` attributes for an appropriate purpose.
- Explain why cookie data is client-controlled and should not be treated as trusted storage.

## 1. What a cookie does

A cookie is a small name-value pair that a browser stores for a website. A server asks the browser to save or replace one by sending a `Set-Cookie` response header:

```http
Set-Cookie: theme=dark; Path=/; Secure; HttpOnly; SameSite=Lax
```

On a later request that matches the cookie's scope and rules, the browser sends the name and value back:

```http
Cookie: theme=dark
```

The browser does not send the cookie's attributes back in the `Cookie` request header. Attributes tell the browser when and where to store and send the cookie.

Cookies let a site remember small pieces of browser state, such as a display preference or a session identifier. They are sent with matching requests, so avoid placing large or sensitive data in them.

## 2. Set a cookie with `setcookie()`

PHP's `setcookie()` sends a `Set-Cookie` header. It must run before any response output, just like other response headers.

The options-array form has been available since PHP 7.3:

```php
<?php

$expiresAt = time() + (30 * 24 * 60 * 60);

setcookie('theme', 'dark', [
    'expires' => $expiresAt,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);
```

The `expires` value is a Unix timestamp in seconds. In this example, the browser is asked to keep the preference for 30 days. Omitting a persistent expiration creates a cookie without a persistent expiry; browser behavior when a session ends can vary.

The `path` controls which URL paths receive the cookie. `path => '/'` makes it available throughout the site. A path is useful for scoping, but it is not an authorization or security boundary.

Omitting `domain` keeps the cookie host-only. Set a `domain` only when subdomains really need to share the cookie; a broader domain exposes it to more hosts.

`setcookie()` returns `true` when PHP successfully prepares the header and `false` when it cannot. A `true` result does not prove that the browser accepted or stored the cookie.

## 3. Read and validate a cookie

PHP exposes cookies received with the current request through `$_COOKIE`:

```php
$allowedThemes = ['light', 'dark'];
$theme = $_COOKIE['theme'] ?? 'light';

if (!is_string($theme) || !in_array($theme, $allowedThemes, true)) {
    $theme = 'light';
}
```

The browser controls the request. A visitor can edit, omit, replay, or forge cookie values, and a request such as `theme[]=dark` can produce an array rather than a string. Validate the shape and allowed values before using cookie data.

Setting a cookie does not update `$_COOKIE` in the current request. The browser receives the `Set-Cookie` response and sends the cookie on a later matching request. A redirect after setting a cookie is a common way to load that next request immediately.

## 4. Cookie attributes

| Attribute | What it tells the browser | Practical guidance |
| --- | --- | --- |
| `Expires` / `expires` | When a persistent cookie should expire | Use only the lifetime the feature needs. Expiration is evaluated by the browser using the supplied timestamp. |
| `Path` / `path` | Which URL paths may receive the cookie | Use `/` when the whole site needs it. Do not use path as access control. |
| `Domain` / `domain` | Which host or hosts may receive it | Usually omit it for a host-only cookie. A parent domain shares it with eligible subdomains. |
| `Secure` / `secure` | Send the cookie only over HTTPS | Enable it in production sites served over HTTPS. A secure cookie will not work over plain HTTP. |
| `HttpOnly` / `httponly` | Keep JavaScript cookie APIs from reading the cookie | Use it for cookies that client-side JavaScript does not need. It does not prevent all effects of cross-site scripting. |
| `SameSite` / `samesite` | Restrict sending the cookie in some cross-site contexts | Set an explicit policy such as `Lax` or `Strict`. `None` is for cross-site use and requires `Secure`. |

`SameSite=Lax` permits some top-level cross-site navigations, while `Strict` is more restrictive. The right policy depends on how the site is used. `SameSite` can reduce some cross-site request risks, but it is not a complete replacement for CSRF protections on state-changing actions.

For a cookie that should be restricted to one HTTPS host, the `__Host-` name prefix has additional browser requirements: it must be `Secure`, use `Path=/`, and omit `Domain`. This can help enforce a narrow scope when those requirements fit the application.

## 5. Update or delete a cookie

Setting the same cookie name and scope again replaces its value. To delete a cookie, send it again with an expiration in the past and the same path and domain scope used when it was created:

```php
setcookie('theme', '', [
    'expires' => time() - 3600,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);
```

If the original cookie used a `domain`, use that same `domain` when deleting it. A cookie with a different path or domain is a different cookie scope and may remain stored.

## 6. Cookies and server-side sessions

A cookie is stored by the browser and returned with matching requests. A server-side session stores application data on the server and commonly uses a cookie to carry only a session identifier. The identifier is still a sensitive credential: protect it with HTTPS and appropriate cookie attributes, and manage the session on the server.

The next lesson covers session data and session identifiers in more detail. Do not put a password, private profile, payment information, or other secret directly into a cookie. Encoding or signing a value does not make it confidential. If an application needs to store a client-side value, treat it as untrusted input and design for its exposure and modification.

## 7. Common mistakes

### Sending output before `setcookie()`

This can prevent PHP from sending the `Set-Cookie` response header:

```php
<?php

echo '<p>Page starts too early</p>';
setcookie('theme', 'dark'); // May fail because output already began.
```

Choose the cookie before rendering the page, and check the return value when failure needs to be handled.

### Trusting a cookie because the server set it

The value comes back from the browser and can be modified. Validate it each time and never use a preference cookie as proof of identity or authorization.

### Assuming `HttpOnly` prevents every script attack

`HttpOnly` prevents normal JavaScript access to the cookie value. It does not stop injected script from performing actions as the visitor or reading other page data. Prevent cross-site scripting separately.

### Treating `SameSite` as the only CSRF defense

`SameSite` is one browser control. Sensitive state-changing actions still need an appropriate CSRF defense and server-side authorization.

### Using broad scope without a reason

A parent-domain cookie may be sent to multiple subdomains. Omit `domain` unless sharing is needed and every eligible subdomain is trusted to receive the cookie.

## Key takeaways

- `setcookie()` sends a response header, so call it before output.
- Cookies set in a response normally appear in `$_COOKIE` only on a later matching request.
- Validate cookie values because browser-supplied data is untrusted.
- Use a short, intentional lifetime, narrow scope, HTTPS, and appropriate `HttpOnly` and `SameSite` settings.
- Delete cookies with an expired value and matching path/domain scope.
- Keep secrets and authorization decisions on the server; cookies are browser state, not trusted storage.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [089 - Response headers, status codes, and redirects](089-response-headers-status-codes-and-redirects.md) | [Q&A](../qnas/090-cookies-and-browser-state.md) | [Example](../examples/090-cookies-and-browser-state.md) | [091 - Sessions and server-side state](091-sessions-and-server-side-state.md) |