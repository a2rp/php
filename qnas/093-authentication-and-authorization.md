# 093 - Authentication and authorization: Q&A

## Multiple-choice questions

### 1. What does authentication answer?

- **A.** Which records may this user edit?
- **B.** Who is making this request?
- **C.** Which database table should be queried?
- **D.** Is the HTML button visible?

**Answer: B.** Authentication verifies an identity or account. Authorization separately decides what that identity may access or do.

### 2. What does authorization answer?

- **A.** Whether a submitted password matches a stored hash
- **B.** Whether the visitor has JavaScript enabled
- **C.** Whether this identity may perform an action on a resource
- **D.** Which session ID PHP generated

**Answer: C.** Authorization checks the requested action and resource against the user's permissions, role, ownership, membership, and relevant policy.

### 3. Does a successful login grant access to every record?

- **A.** Yes, because the user has a session
- **B.** No, each protected resource and action still needs an authorization decision
- **C.** Yes, if the user knows the record ID
- **D.** Only if the browser hides admin links

**Answer: B.** Authentication establishes identity for the session. It does not grant unlimited access to records or features.

### 4. Where must the decisive authorization check run?

- **A.** Only in client-side JavaScript
- **B.** Only when the dashboard first loads
- **C.** On the server for each protected request
- **D.** Only in the stylesheet

**Answer: C.** A user can bypass a hidden button or call an endpoint directly. The server must enforce access on each protected request.

### 5. What does “deny by default” mean?

- **A.** Permit access unless an error occurs
- **B.** Deny when no explicit rule grants the requested access
- **C.** Give every authenticated user the admin role
- **D.** Allow access if a resource ID looks random

**Answer: B.** Start from denial and grant only permissions the application can justify for that user, action, and resource.

### 6. What should a successful password login do before saving authenticated state in the session?

- **A.** Regenerate the session ID
- **B.** Print the password hash
- **C.** Copy the user's role from a request parameter
- **D.** Put the session ID in the redirect URL

**Answer: A.** Rotate the session ID after verifying credentials and before storing the authenticated user state to reduce session-fixation risk.

### 7. What is an IDOR-style authorization bug?

- **A.** A slow SQL query caused by a missing index
- **B.** A user changes an object identifier and accesses a resource they are not permitted to use
- **C.** A hash contains a random salt
- **D.** A browser refuses a cookie

**Answer: B.** Knowing or guessing an object's identifier must not grant access. Load the object and check the current user's permission for that specific object.

### 8. Why use the same public login failure for an unknown account and a wrong password?

- **A.** To hide whether an account exists
- **B.** To make the database query unnecessary
- **C.** To enable an account to skip password verification
- **D.** To replace HTTPS

**Answer: A.** Different messages or response behavior can help attackers enumerate valid account identifiers.

### 9. What is the difference between `401` and `403` in a typical web flow?

- **A.** 401 means access denied to an authenticated user; 403 means sign in first
- **B.** 401 is typically for missing/invalid authentication; 403 is for an authenticated identity without permission
- **C.** They always mean “not found”
- **D.** Neither is an HTTP status

**Answer: B.** Applications may choose their own concealment policy for sensitive resources, but the distinction is a useful default model.

### 10. What does a CSRF token check establish?

- **A.** That the user is an administrator
- **B.** That the user owns the database record
- **C.** That a cookie-carrying state-changing form request contains the expected application token
- **D.** That a password hash is current

**Answer: C.** CSRF protection helps prevent a browser from being tricked into submitting a forged request. It does not replace authentication or authorization.

## Short-answer questions

### 11. Can a visitor be authorized without being authenticated?

**Answer:** Yes. Public resources may be available to everyone without identifying the visitor.

**Explanation:** Authentication is not always required for access. Authorization still decides whether a request is allowed, including requests to public content.

### 12. What is least privilege?

**Answer:** Give a user or service only the minimum access needed for its task.

**Explanation:** Smaller permissions limit the impact of mistakes, compromised accounts, and unintended operations.

### 13. What is the difference between role-based access and resource ownership checks?

**Answer:** A role grants a set of general capabilities, while an ownership check relates a particular user to a particular resource.

**Explanation:** An editor role may allow editing articles generally, but the application may still restrict an editor to articles in their own workspace or authored by them.

### 14. Why should the server load a user's role from trusted storage instead of trusting a hidden field?

**Answer:** Form fields, URL parameters, and ordinary cookies are controlled by the client and can be changed.

**Explanation:** Use authenticated server-side identity and current permission data when making an authorization decision.

### 15. What is a generic login failure message?

**Answer:** A response such as “Invalid email or password” that does not say whether the account exists, is disabled, or has a different password.

**Explanation:** Keep the public response shape and status consistent enough to avoid obvious account enumeration. Apply throttling and monitoring as well.

### 16. Why should rate limiting accompany password authentication?

**Answer:** Password hashing slows offline guessing, but it does not stop an attacker from making many online login attempts.

**Explanation:** Add controls such as per-account and broader abuse limits, monitoring, and appropriate step-up measures without creating an easy denial-of-service path against a victim's account.

### 17. What information should not appear in authentication logs?

**Answer:** Passwords, session IDs, reset tokens, and other secrets or complete credential payloads.

**Explanation:** Logs should support investigation without becoming another place where stolen credentials can be recovered.

### 18. Why is account recovery part of authentication security?

**Answer:** A weak recovery flow can let an attacker take over an account even if the normal password login is strong.

**Explanation:** Recovery tokens should be unpredictable, short-lived, single-use, and handled without leaking them through logs or URLs.

## Code and behavior questions

### 19. Why is this check insufficient?

```php
if ($user['role'] === 'admin') {
    echo '<a href="/admin.php">Admin</a>';
}
```

**Answer:** Hiding the link only changes the interface. A user can request `/admin.php` directly.

**Correct approach:** Check the authenticated user's current permission on the server inside the protected admin request itself.

### 20. Why is this resource lookup unsafe by itself?

```php
$article = findArticle($_GET['id']);
showArticle($article);
```

**Answer:** It loads an object based on a client-supplied identifier without checking whether the current user may view that object.

**Safer flow:**

```php
$article = findArticle($id);

if ($article === null || !canViewArticle($currentUser, $article)) {
    http_response_code(404);
    exit('Not found');
}

showArticle($article);
```

**Explanation:** The application must authorize access to the specific record after loading it. Some systems intentionally return 404 rather than 403 to conceal a protected resource's existence.

### 21. What is missing from this login success branch?

```php
if (password_verify($password, $user['password_hash'])) {
    $_SESSION['user_id'] = $user['id'];
}
```

**Answer:** The session ID should be regenerated before saving authenticated state, and the flow also needs account-status checks, CSRF protection for the form, and safe failure handling.

**Improved core sequence:**

```php
if (password_verify($password, $user['password_hash'])) {
    if (!session_regenerate_id()) {
        http_response_code(500);
        exit;
    }

    $_SESSION['user_id'] = $user['id'];
}
```

### 22. Does a valid CSRF token mean the user can delete an article?

**Answer:** No. The CSRF token helps establish that the form submission came through the expected application flow; the server must still check that the user is authorized to delete that article.

### 23. Is this permission check reliable?

```php
if ($_POST['role'] === 'admin') {
    deleteUser($targetId);
}
```

**Answer:** No. The request supplies the role value and can change it.

**Safer approach:** Load the current user from the authenticated session, load current permissions from trusted storage, and check whether that user may delete the specific target account.

### 24. What must happen after an authorization check fails?

**Answer:** Stop the protected operation and return a suitable denial response without performing the write.

**Explanation:** Check permissions before changing data. Log useful security context without exposing credentials, tokens, or session IDs.

## Key takeaways

- Authentication verifies identity; authorization decides access to each action and resource.
- Rotate the session ID after successful authentication and before saving authenticated state.
- Deny by default and enforce checks on the server for every protected request.
- Check ownership or membership for individual records; a hard-to-guess ID is not authorization.
- Use generic authentication failures, rate limits, and secure recovery flows.
- CSRF protection, authentication, and authorization solve different problems and may all be needed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [092 - Password hashing and verification](../lessons/092-password-hashing-and-verification.md) | [Lesson](../lessons/093-authentication-and-authorization.md) | [Example](../examples/093-authentication-and-authorization.md) | [094 - Handling file uploads](../lessons/094-handling-file-uploads.md) |