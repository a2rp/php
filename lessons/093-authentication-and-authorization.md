# 093 - Authentication and authorization

## Learning goals

By the end of this lesson, you will be able to:

- Distinguish authentication from authorization.
- Build a basic sign-in flow around password verification and server-side sessions.
- Return generic login failures and protect sign-in from automated guessing.
- Check permissions on the server for every protected action and resource.
- Apply least privilege, deny-by-default rules, role checks, and ownership checks.
- Recognize why hiding a button or changing an object ID is not access control.
- Separate authorization checks from CSRF protection and session security.

## 1. Authentication and authorization answer different questions

**Authentication** checks who is making a request. A password, passkey, or other authenticator helps establish an identity.

**Authorization** decides whether that authenticated identity may perform a particular action on a particular resource.

```text
Authentication: Is this request associated with user 42?
Authorization: May user 42 edit article 81 in this workspace?
```

A user can be authenticated but not authorized to read every record, visit an admin page, or perform every operation. Public pages may be available without authentication at all.

## 2. A basic password sign-in flow

A typical password login flow is:

1. Receive the account identifier and password over HTTPS.
2. Look up the account using a parameterized query.
3. Check that the account is allowed to sign in.
4. Verify the submitted password against the stored hash with `password_verify()`.
5. On success, regenerate the session ID before recording the authenticated user in the session.
6. Enforce authorization separately on each protected request.

A simplified flow after validating input and fetching `$user` from the database looks like this:

```php
<?php

if ($user === null || !password_verify($submittedPassword, $user['password_hash'])) {
    // Use the same public failure message for an unknown account or bad password.
    $loginError = 'Invalid email or password.';
} elseif (!$user['is_active']) {
    // Apply the account policy without revealing its state to an unauthenticated visitor.
    $loginError = 'Invalid email or password.';
} elseif (!session_regenerate_id()) {
    http_response_code(500);
    exit('Could not establish a secure session.');
} else {
    // The new session ID is established before authenticated state is stored.
    $_SESSION['user_id'] = $user['id'];
    $_SESSION['authenticated_at'] = time();
}
```

This is a teaching outline; a real handler also needs request validation, CSRF protection for form submissions, safe session configuration, login throttling, logging, and clear error handling. If the account is missing, production code may also perform a dummy password verification to reduce obvious timing differences. Do not disclose account existence through different messages or response behavior.

## 3. Authentication factors and account lifecycle

Passwords are one authentication factor. Depending on the risk and product, authentication may also use a passkey, a security key, an authenticator application, or another factor. Multi-factor authentication requires evidence from distinct factor categories; two passwords are not two different factors.

Registration, email verification, password changes, recovery, account lockout, and session revocation are part of the authentication lifecycle. Password recovery should not provide an easier way to take over an account than normal login. Keep recovery tokens random, short-lived, single-use, and out of logs and URLs where possible.

Use HTTPS for authentication pages and requests. Never log passwords, full authentication payloads, session IDs, or recovery tokens. Apply rate limits and monitoring to login and recovery endpoints to reduce automated guessing and credential stuffing.

## 4. Start a session only after authentication succeeds

A session can carry a server-side identity between requests. Configure session cookies and start the session before output. After credentials have been verified, rotate the session ID, then place the user ID in the session:

```php
if (password_verify($submittedPassword, $user['password_hash'])) {
    if (!session_regenerate_id()) {
        http_response_code(500);
        exit;
    }

    $_SESSION['user_id'] = $user['id'];
    $_SESSION['authenticated_at'] = time();
}
```

On later requests, load the account from trusted server-side storage using the session's user ID. Check current account status and permissions when needed; do not accept a role or user ID supplied in a form, URL, or ordinary cookie as proof of identity.

A session proves that the application associated the request with an authenticated session. It does not automatically grant access to every operation.

## 5. Authorization: decide for the action and resource

Authorization rules should answer questions like:

- Can this user view this page?
- Can this user edit this particular article?
- Can this user manage members of this workspace?
- Is this user allowed to delete this record now?

A role check can protect a broad area:

```php
if ($user['role'] !== 'admin') {
    http_response_code(403);
    exit('Forbidden');
}
```

A resource ownership rule can protect an individual record:

```php
$mayEdit = $user['role'] === 'admin'
    || $article['owner_id'] === $user['id'];

if (!$mayEdit) {
    http_response_code(403);
    exit('Forbidden');
}
```

Real policies may combine roles, permissions, ownership, organization membership, resource status, and other attributes. Write those rules explicitly and keep them close to the protected operation.

## 6. Deny by default and check every request

Start from denial, then grant only the actions a user needs. For every protected request, check the user, action, and resource on the server. A check on the dashboard does not protect a separate API endpoint or a direct request to a PHP file.

The interface may hide a button from users who cannot use the action, but that only improves the interface. The server must still deny the operation if the user sends the request directly.

Never rely on an object identifier being hard to guess. If a request includes `article_id=81`, fetch the record and verify that the current user can act on that specific article. Otherwise, changing `81` to another ID could expose or change somebody else's data, a class of bug often called insecure direct object reference or IDOR.

## 7. Roles, permissions, and least privilege

A role groups permissions, such as `editor` or `admin`. A permission describes a specific allowed action, such as `article:publish`. Ownership and membership are relationships between a user and a resource. Many applications need a combination rather than a single role comparison.

Give users only the permissions they need. Avoid making every authenticated user an administrator, and avoid assuming that a familiar route or UI element is safe. When no rule explicitly grants access, deny the request.

## 8. Authorization is not CSRF protection

Authorization asks whether the signed-in user is allowed to perform an action. CSRF protection checks whether a browser request that carries the user's cookies was intentionally initiated by the application flow.

For state-changing forms, use an appropriate CSRF defense, such as a random server-generated token stored in the session and checked on submission. Do not change server state through GET requests. `SameSite` cookies can add protection but do not replace a CSRF token for sensitive actions.

Also validate and normalize request data, use prepared database statements, and encode output. These controls solve different problems and do not substitute for authorization.

## 9. Handle denial safely

- Return `401 Unauthorized` when the request lacks valid authentication credentials, or redirect to sign-in where that is the application convention.
- Return `403 Forbidden` when the identity is known but the requested action is not allowed.
- Some applications return `404 Not Found` for a resource the user is not allowed to discover. Choose a consistent policy that does not leak sensitive existence information.
- Stop the protected operation after a failed check. Do not perform part of the write before checking permission.
- Log security-relevant denials with enough context to investigate, while excluding passwords, session IDs, and secrets.

## Common mistakes

### Treating login as full access

A valid login proves an identity for the session. It does not imply that the user can view every record or use every admin operation.

### Checking permissions only in the interface

A hidden button can be bypassed by calling the endpoint directly. Enforce the rule on the server for every request.

### Trusting a role from request data

Do not authorize based on `?role=admin`, a hidden form field, or a client-controlled cookie. Load identity and permission data from trusted server-side storage.

### Checking a role but ignoring record ownership

A normal user may be allowed to edit their own articles but not another user's article. Check the specific resource relationship after loading the record.

### Revealing whether an account exists

Use a generic login failure and consistent response behavior for an unknown account, incorrect password, or unavailable account. Add throttling and monitoring as well.

### Confusing CSRF with authorization

A valid CSRF token does not grant permission to edit a record. A user who is authorized can still be tricked into sending a request without CSRF protection. Apply both checks where appropriate.

## Key takeaways

- Authentication verifies identity; authorization grants or denies actions on resources.
- Verify credentials, rotate the session ID, and only then record authenticated state.
- Load permissions from trusted server-side data and check them for every protected request.
- Deny by default, follow least privilege, and check ownership or membership for specific records.
- Hide controls for usability, but enforce access on the server.
- Use CSRF defenses, parameterized queries, rate limits, and secure sessions alongside authorization.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [092 - Password hashing and verification](092-password-hashing-and-verification.md) | [Q&A](../qnas/093-authentication-and-authorization.md) | [Example](../examples/093-authentication-and-authorization.md) | [094 - Handling file uploads](094-handling-file-uploads.md) |