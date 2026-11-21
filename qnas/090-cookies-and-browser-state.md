# 090 - Cookies and browser state: Q&A

## Multiple-choice questions

### 1. How does a server normally ask a browser to store or replace a cookie?

- **A.** By adding a `Cookie` request header
- **B.** By sending a `Set-Cookie` response header
- **C.** By writing directly to the browser's file system
- **D.** By adding a value to the HTML document title

**Answer: B.** The server sends `Set-Cookie` in a response. The browser decides whether to store it and sends its name and value in a later matching request.

### 2. When should PHP call `setcookie()`?

- **A.** After the complete response body has been printed
- **B.** Before response output begins
- **C.** Only after calling `session_destroy()`
- **D.** After the browser has submitted another request

**Answer: B.** `setcookie()` sends a response header, so it must run before output such as HTML or whitespace is sent.

### 3. When can a cookie set by a response normally be read from `$_COOKIE`?

- **A.** Immediately after calling `setcookie()` in the same request
- **B.** On a later request when the browser sends it back
- **C.** Only after restarting PHP
- **D.** Only when JavaScript reads it

**Answer: B.** `$_COOKIE` reflects cookies received with the current request. The browser receives `Set-Cookie` in the response, then may send the value on a later matching request.

### 4. What does the `Secure` attribute do?

- **A.** Encrypts the cookie value in the browser's storage
- **B.** Sends the cookie only over HTTPS connections
- **C.** Prevents the user from deleting the cookie
- **D.** Makes the cookie inaccessible to PHP

**Answer: B.** `Secure` restricts transmission to secure HTTPS connections. It does not encrypt the stored value or protect it from being read in the browser's storage tools.

### 5. What does `HttpOnly` primarily restrict?

- **A.** Normal JavaScript access to the cookie value
- **B.** PHP access to `$_COOKIE`
- **C.** The cookie's expiration date
- **D.** Requests to the site's API

**Answer: A.** `HttpOnly` prevents normal script APIs from reading the cookie value. It does not stop an injected script from performing other actions as the visitor.

### 6. Which statement about `SameSite=None` is correct?

- **A.** It means the browser never sends the cookie
- **B.** It must be paired with `Secure` in modern browser handling
- **C.** It makes a cookie readable by JavaScript
- **D.** It is the same as `Path=/`

**Answer: B.** Cookies using `SameSite=None` are intended for cross-site use and require `Secure`; otherwise browsers generally reject them.

### 7. What is the typical effect of omitting the cookie `domain` option?

- **A.** The cookie is scoped to the host that set it
- **B.** The cookie is shared with every website
- **C.** The cookie is deleted immediately
- **D.** The browser sends it only to PHP files

**Answer: A.** Omitting `domain` creates a host-only cookie. Set a broader domain only when subdomain sharing is intentional.

### 8. How should an application delete a cookie?

- **A.** Set the same name and scope with an expiration in the past
- **B.** Remove it from the current request's `$_COOKIE` array only
- **C.** Call `unset()` on a PHP variable and do nothing else
- **D.** Send an empty HTML response

**Answer: A.** The browser stores cookies by name and scope. Send an expired cookie with the same path and domain so the browser can remove the intended cookie.

## Short-answer questions

### 9. What is the difference between the `Set-Cookie` and `Cookie` headers?

**Answer:** `Set-Cookie` is sent by a server to ask a browser to store or update a cookie. `Cookie` is sent later by the browser with matching requests and contains cookie name-value pairs.

**Explanation:** Cookie attributes such as `Secure` and `SameSite` are part of the `Set-Cookie` response, not normally repeated in the later `Cookie` request header.

### 10. Is `$_COOKIE` trusted because the application previously set the value?

**Answer:** No. The browser controls the next request and a visitor can edit, omit, replay, or forge cookie values.

**Explanation:** Validate the type and allowed values every time. Do not use a preference cookie as proof of identity, permission, or authorization.

### 11. What does the `expires` option represent in PHP's `setcookie()` options array?

**Answer:** It is a Unix timestamp in seconds that tells the browser when the persistent cookie should expire.

**Explanation:** For example, `time() + 3600` requests an expiration one hour from the server's current time. The browser manages whether and how long it stores the cookie.

### 12. What is the difference between a cookie with no persistent expiration and one with an `expires` timestamp?

**Answer:** The first has no persistent expiration requested and is commonly called a session cookie; the second asks the browser to retain it until the specified expiration, subject to browser policy.

**Explanation:** Do not promise that a session cookie always disappears when a browser window closes. Browsers may restore sessions or apply their own storage behavior.

### 13. What does `SameSite=Lax` do in broad terms?

**Answer:** It restricts the cookie in many cross-site request contexts while allowing some top-level navigations to the site.

**Explanation:** `Strict` is more restrictive, while `None` is for cross-site cookie use and requires `Secure`. Exact behavior is controlled by browsers, and `SameSite` is not a complete CSRF defense.

### 14. Is cookie `Path` an authorization boundary?

**Answer:** No. `Path` tells the browser which URL paths should receive a cookie, but applications must enforce access control on the server.

### 15. What can `HttpOnly` not guarantee?

**Answer:** It cannot guarantee that a site is safe from cross-site scripting. It restricts JavaScript from reading the cookie value, but malicious script may still perform actions within the page.

### 16. Why might an application omit the `domain` option?

**Answer:** Omitting it keeps the cookie host-only, limiting which hosts receive it.

**Explanation:** A parent-domain cookie may be sent to multiple subdomains. Broader sharing should be deliberate and limited to trusted hosts.

## Code and behavior questions

### 17. What value does this code normally see immediately after `setcookie()`?

```php
setcookie('theme', 'dark', ['path' => '/']);
$currentTheme = $_COOKIE['theme'] ?? 'light';
```

**Answer:** It reads the value sent in the current request, or `light` if no `theme` cookie was sent. It does not normally read `dark` just because `setcookie()` was called.

**Explanation:** The browser receives the response header and can send the new value on the next matching request. A redirect is one way to cause that next request.

### 18. Why is this cookie value check incomplete?

```php
$theme = $_COOKIE['theme'] ?? 'light';
```

**Answer:** It does not verify that the value is a string or one of the themes the application supports.

**Safer version:**

```php
$theme = $_COOKIE['theme'] ?? 'light';
$allowedThemes = ['light', 'dark'];

if (!is_string($theme) || !in_array($theme, $allowedThemes, true)) {
    $theme = 'light';
}
```

**Explanation:** A request such as `theme[]=dark` can make the value an array. Strict allowlist validation prevents unexpected input from being used as a theme.

### 19. Why may this cookie fail on a local HTTP development server?

```php
setcookie('theme', 'dark', [
    'secure' => true,
    'path' => '/',
]);
```

**Answer:** A `Secure` cookie is restricted to HTTPS, while a plain local HTTP server does not provide an HTTPS connection.

**Explanation:** Production sites served over HTTPS should use `Secure`. For a local-only HTTP demonstration, the setting may need to reflect that local setup; do not weaken the production setting.

### 20. What requirements apply to a cookie named with the `__Host-` prefix?

**Answer:** It must be set with `Secure`, use `Path=/`, and omit the `Domain` attribute.

**Explanation:** Supporting browsers enforce these requirements to prevent a cookie with that prefix from being scoped to a parent domain or narrower path.

### 21. Does `setcookie()` returning `true` prove the browser stored the cookie?

**Answer:** No. It indicates PHP successfully prepared the response header. The client may still reject, block, or later remove the cookie.

### 22. Why should deleting a cookie use the original path and domain scope?

**Answer:** Cookies with different paths or domains are separate browser entries, even if they share a name. The deletion response must target the same scope as the cookie being removed.

## Key takeaways

- Servers send `Set-Cookie`; browsers return matching name-value pairs in later `Cookie` request headers.
- Cookies are client-controlled input, so validate them and keep sensitive authority on the server.
- `Secure`, `HttpOnly`, and `SameSite` address different browser behaviors and are not interchangeable.
- `setcookie()` must run before output, and a new cookie is normally available to PHP on a later request.
- Delete a cookie by expiring it with the matching path and domain scope.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [089 - Response headers, status codes, and redirects](../lessons/089-response-headers-status-codes-and-redirects.md) | [Lesson](../lessons/090-cookies-and-browser-state.md) | [Example](../examples/090-cookies-and-browser-state.md) | [091 - Sessions and server-side state](../lessons/091-sessions-and-server-side-state.md) |