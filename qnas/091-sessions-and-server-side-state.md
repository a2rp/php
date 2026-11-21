# 091 - Sessions and server-side state: Q&A

## Multiple-choice questions

### 1. Why does a web application use a session?

- **A.** To make every HTTP request automatically remember prior requests
- **B.** To associate later requests with application state
- **C.** To store the complete server database in a browser cookie
- **D.** To replace authorization checks

**Answer: B.** HTTP requests are independent. A session identifier lets PHP associate a later request with server-side state.

### 2. Which function starts or resumes a PHP session?

- **A.** `session_start()`
- **B.** `session_create()`
- **C.** `session_resume()`
- **D.** `start_session_data()`

**Answer: A.** `session_start()` reads the session identifier, asks the configured handler to load the corresponding data, and makes it available through `$_SESSION`.

### 3. With the usual cookie-based setup, what does the browser normally store?

- **A.** All values from `$_SESSION`
- **B.** The session identifier cookie
- **C.** PHP's session files
- **D.** The user's authorization rules

**Answer: B.** The browser usually stores the session ID. PHP's configured save handler stores session data on the server.

### 4. When should `session_start()` run?

- **A.** After the page's HTML has been printed
- **B.** Before response output begins
- **C.** After `session_destroy()` on every request
- **D.** Only in JavaScript

**Answer: B.** Session startup may send or read headers, so start it before HTML, whitespace, or other response output.

### 5. What does `session.use_strict_mode` help PHP do?

- **A.** Reject uninitialized session IDs supplied by a client
- **B.** Encrypt every value in `$_SESSION`
- **C.** Keep a session alive forever
- **D.** Make all session values available to JavaScript

**Answer: A.** Strict mode helps PHP reject session IDs that were not initialized by the configured session handler. A custom handler must support session ID validation for strict mode to be effective.

### 6. When should an application normally regenerate a session ID?

- **A.** After successful authentication or a privilege change
- **B.** Every time it renders a paragraph
- **C.** Only after logging out
- **D.** Never; session IDs should be permanent

**Answer: A.** Regenerate the ID at authentication and privilege boundaries to reduce session-fixation risk. Save the authenticated state only after the ID has been refreshed.

### 7. Which is true about `session_destroy()`?

- **A.** It automatically removes the browser's session cookie and clears `$_SESSION`
- **B.** It removes data associated with the current session on the server, but does not itself clear `$_SESSION` or remove the cookie
- **C.** It permanently deletes every session for every user
- **D.** It only clears one `$_SESSION` key

**Answer: B.** A logout flow may also need to clear the current array and expire the session cookie with its matching scope.

### 8. What does `session_write_close()` do?

- **A.** Writes current session changes and ends the active session
- **B.** Deletes all session records from the server
- **C.** Removes the browser cookie without a response
- **D.** Starts a new PHP process

**Answer: A.** It saves session data and releases the active session. It is useful after the request is done reading or writing session values.

### 9. What is the main risk of putting a session ID in a URL?

- **A.** The URL becomes invalid HTML
- **B.** The ID can leak through copied links, browser history, or logs
- **C.** PHP cannot use the session handler anymore
- **D.** The browser turns the ID into a password hash

**Answer: B.** URLs are copied and recorded in many places. Keep session IDs in protected cookies rather than URLs.

### 10. Is `session.gc_maxlifetime` by itself a reliable login inactivity timeout?

- **A.** Yes, it always logs out a user at an exact time
- **B.** No, enforce application timeouts using tracked timestamps and checks
- **C.** Yes, provided the browser is open
- **D.** No, because PHP sessions cannot expire

**Answer: B.** Garbage collection removes old stored data according to handler and configuration behavior; it is not an exact application-level authentication timeout.

## Short-answer questions

### 11. What is the relationship between a session ID and `$_SESSION`?

**Answer:** The browser sends a session ID, and PHP uses it with the configured session handler to load the associated data into `$_SESSION`.

**Explanation:** The ID is not the session data itself. Treat the ID as a credential because someone who obtains a valid ID may gain access to the session.

### 12. Where is session data stored?

**Answer:** It is stored by the configured session save handler. PHP's default handler is server-side, but an application can configure another handler.

### 13. Why should session configuration be set before `session_start()`?

**Answer:** PHP needs the configuration when it sends the session cookie and opens the session. Cookie parameters set after startup do not configure that already-started session response.

### 14. What does `session.use_only_cookies` help prevent?

**Answer:** It keeps PHP from accepting session IDs through URL or form parameters for session propagation.

**Explanation:** URL-based session IDs can leak through browser history, copied links, and logs. Use cookie-based IDs with appropriate protections.

### 15. Does `HttpOnly` make a session completely safe from cross-site scripting?

**Answer:** No. It prevents ordinary JavaScript cookie APIs from reading the session cookie, but injected code may still perform actions within the user's page.

### 16. Why should an application store only a small amount of data in a session?

**Answer:** Keeping session state small reduces storage and serialization work and makes its purpose easier to manage. Store only values needed across requests, such as a user ID or a small workflow flag.

## Code and behavior questions

### 17. Why may this code fail to start the session?

```php
<?php

echo '<h1>Dashboard</h1>';
session_start();
```

**Answer:** Output may already have started before PHP tries to send or process session headers.

**Safer order:**

```php
<?php

session_start();
echo '<h1>Dashboard</h1>';
```

### 18. What value is shown on a later request after this code runs?

```php
session_start();
$_SESSION['cart_count'] = 3;
```

**Answer:** A later request carrying the same valid session ID can read `3` from `$_SESSION['cart_count']`.

**Explanation:** The browser carries the ID, while the configured handler retrieves the associated server-side data.

### 19. What is wrong with this login order?

```php
$_SESSION['user_id'] = $verifiedUserId;
session_regenerate_id();
```

**Answer:** Authentication state is saved before the ID is regenerated. Regenerate the ID first, then store the verified user information.

**Safer order:**

```php
if (!session_regenerate_id()) {
    http_response_code(500);
    exit;
}

$_SESSION['user_id'] = $verifiedUserId;
```

### 20. Why is this logout code incomplete?

```php
session_destroy();
```

**Answer:** It does not clear the current `$_SESSION` array or remove the browser's session cookie.

**Explanation:** A logout flow should clear relevant session values, expire the cookie using the same scope, and destroy the server-side session when the application's lifecycle policy requires it. Be mindful of concurrent requests and custom handlers.

### 21. What is the benefit of closing the session before a slow operation?

```php
session_start();
$_SESSION['last_seen'] = time();
session_write_close();

buildLargeReport();
```

**Answer:** PHP writes the session changes and releases the active session before the slow work begins, allowing other requests using the same session to proceed.

**Explanation:** The default file-based handler locks the session while it is open. Do not change `$_SESSION` after closing it unless the session is deliberately started again.

### 22. What is missing from this idle-time check?

```php
if (time() - $_SESSION['last_activity'] > 1800) {
    echo 'Expired';
}
```

**Answer:** It assumes the key exists and has the right type, and it merely prints a message without clearing or rejecting the authenticated state.

**Explanation:** Check that the timestamp exists and is an integer, enforce the timeout on protected requests, and implement the application's intended invalidation behavior.

### 23. What should this read-only request consider?

```php
session_start();
$userId = $_SESSION['user_id'] ?? null;
```

**Answer:** If the request will not modify session data, it can close the session after reading or use `session_start(['read_and_close' => true])`.

**Explanation:** This avoids holding a session lock longer than needed. Do not use read-and-close if the request must update `$_SESSION`.

### 24. Does regenerating an ID mean the old session data should always be deleted immediately?

**Answer:** No. Applications with concurrent requests or unstable connections may encounter races if old session data is removed immediately.

**Explanation:** Design how obsolete authenticated state is rejected and eventually cleaned up. Do not apply `session_regenerate_id(true)` blindly to every production session lifecycle.

## Key takeaways

- The browser normally carries a session ID; the configured handler retrieves server-side data into `$_SESSION`.
- Start the session before output and configure its cookie before startup.
- Protect the session ID as a credential; regenerate it before recording authentication after a privilege change.
- Session cleanup, cookie removal, and clearing `$_SESSION` are separate tasks.
- Track application timeouts explicitly and release session locks when no longer needed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [090 - Cookies and browser state](../lessons/090-cookies-and-browser-state.md) | [Lesson](../lessons/091-sessions-and-server-side-state.md) | [Example](../examples/091-sessions-and-server-side-state.md) | [092 - Password hashing and verification](../lessons/092-password-hashing-and-verification.md) |