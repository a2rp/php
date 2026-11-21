# 127 - Authentication, authorization, and access control

Authentication and authorization answer different questions. Authentication verifies who a person or service is. Authorization decides what that authenticated identity may do to a particular resource. A secure application performs both checks on the server, on every protected request.

This chapter covers safe password verification, authenticated sessions, permission checks, ownership checks, and common access-control mistakes in PHP applications.

## Learning goals

By the end of this chapter, you should be able to:

- Distinguish identification, authentication, authorization, and access control.
- Store and verify passwords with PHP's password APIs.
- Establish an authenticated session without keeping a reusable password in it.
- Check permissions and resource ownership on every protected request.
- Recognize insecure direct object references and client-controlled roles.
- Choose appropriate responses for unauthenticated and unauthorized requests.
- Test both allowed and denied access paths.

## Four related terms

- **Identification** is the claim a person makes about an identity, such as an email address or account name.
- **Authentication** verifies evidence for that claim. A password, passkey, or second factor can provide that evidence.
- **Authorization** evaluates whether an authenticated identity may perform an action on a resource.
- **Access control** is the set of application rules and checks that enforce authorization.

A successful login establishes an identity for a request. It does not grant permission to every page, record, or action. For example, a signed-in member may view their own document but not another member's private document.

## Store passwords as password hashes

Never store a user's plaintext password. Do not encrypt passwords for later recovery, and do not hash them with a fast general-purpose digest such as MD5 or SHA-256. Passwords can be guessed offline if a database is exposed, so password storage needs a password-hashing algorithm designed to make each guess expensive.

PHP provides password-specific APIs. At registration, hash the submitted password and store the resulting string:

~~~php
$passwordHash = password_hash($password, PASSWORD_DEFAULT);

$statement = $pdo->prepare(
    'INSERT INTO users (email, password_hash)
     VALUES (:email, :password_hash)'
);

$statement->execute([
    'email' => $email,
    'password_hash' => $passwordHash,
]);
~~~

Use a password_hash database column large enough for future algorithms and parameter changes. A VARCHAR(255) is a practical choice. The generated hash includes the information PHP needs to verify it; do not create or store a separate salt yourself.

PASSWORD_DEFAULT lets PHP choose its current default password algorithm. If your deployment has a specific algorithm or compliance requirement, choose a supported option deliberately and verify that the production PHP build supports it. Keep the application and PHP runtime current.

## Verify a password during login

Load the account using a parameterized database query, then pass the stored hash to password_verify():

~~~php
$statement = $pdo->prepare(
    'SELECT id, email, password_hash
     FROM users
     WHERE email = :email'
);

$statement->execute(['email' => $email]);
$user = $statement->fetch(PDO::FETCH_ASSOC);

if (
    $user === false
    || !password_verify($submittedPassword, $user['password_hash'])
) {
    http_response_code(401);
    $message = 'The email address or password is incorrect.';
} else {
    // Continue with successful-login handling.
}
~~~

Use the same public failure message when the account does not exist and when the password is wrong. A response such as “email not found” can help an attacker enumerate accounts. Keep useful diagnostics in protected server logs without logging submitted passwords, password hashes, or session IDs.

Rate-limit login attempts and password-reset requests. Add multi-factor authentication where the risk warrants it, especially for administrators and sensitive actions. Use a secure, expiring, single-use reset flow rather than emailing a current password.

### Upgrade hashes when needed

Password algorithms and costs may change. After a password has been verified successfully, check whether PHP recommends a new hash and update the stored value:

~~~php
if (password_needs_rehash(
    $user['password_hash'],
    PASSWORD_DEFAULT
)) {
    $newHash = password_hash($submittedPassword, PASSWORD_DEFAULT);

    $update = $pdo->prepare(
        'UPDATE users
         SET password_hash = :password_hash
         WHERE id = :id'
    );

    $update->execute([
        'password_hash' => $newHash,
        'id' => $user['id'],
    ]);
}
~~~

Only rehash after verification succeeds. Never replace a stored hash using a password that has not been authenticated.

## Establish an authenticated session

A browser session lets later requests refer to a successfully authenticated user without resending the password. Configure session cookies before starting the session. For an HTTPS-only production site, a typical baseline is:

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

Secure requires HTTPS. If HTTPS ends at a trusted reverse proxy, use trusted deployment configuration to determine the setting; do not trust an arbitrary client-supplied forwarding header. HttpOnly prevents ordinary client-side JavaScript from reading the cookie. SameSite can reduce some cross-site cookie sending, but it does not replace CSRF protection for state-changing actions.

After the password is verified, rotate the session identifier before marking the session authenticated:

~~~php
session_regenerate_id(true);

$_SESSION['user_id'] = (int) $user['id'];
~~~

Rotating the identifier helps prevent session fixation, where an attacker tries to make a victim use a session ID the attacker already knows. Regenerate it after login and after other privilege changes. Applications with concurrent requests or long-running sessions should use a session lifecycle strategy appropriate to their framework and deployment.

Store only the minimum session state needed to look up the identity. Treat the session ID as a bearer secret: anyone who obtains it may be able to act as that session. Do not put session IDs in URLs, HTML, logs, or analytics. Expire inactive sessions according to the application's risk, and provide a logout route that invalidates the server-side session and expires its cookie. Because logout changes state, protect it from cross-site request forgery as well.

For each protected request, load the current user from the server-side session identity and trusted application data. Do not take an authenticated user ID, role, or permission from a query parameter, form field, or other client-controlled value.

## Check authorization on every request

A navigation menu that hides an Admin link is not an authorization check. An attacker can request a route directly, call an API without the page, or change an object identifier. Enforce the policy in the server-side handler before reading or changing protected data.

A reliable authorization policy follows these principles:

1. **Deny by default.** If a policy does not explicitly allow an action, reject it.
2. **Use least privilege.** Grant only the actions and data a person needs.
3. **Check every request.** Do not rely on a previous page, a hidden button, or a client-side check.
4. **Check the specific action and resource.** Permission to view one record does not imply permission to edit or delete it.
5. **Use trusted identity data.** Obtain the principal and permissions from the authenticated server-side context, not from the request.

Roles can help group permissions, such as editor or admin, but a broad role check is not always enough. A role may allow document editing generally while the user should still edit only documents in their own organization. Combine role or permission checks with resource-specific rules such as ownership, tenant membership, or the current workflow state.

## Prevent insecure direct object references

An insecure direct object reference (IDOR) occurs when an application accepts an object identifier but fails to check whether the current user may access that object. Sequential IDs make guessing easier, but changing an ID is not the underlying bug: the missing authorization check is.

Avoid loading a record globally and assuming that knowing its ID grants access:

~~~php
// This lookup alone does not authorize the current user.
$statement = $pdo->prepare(
    'SELECT id, owner_id, title
     FROM documents
     WHERE id = :id'
);
~~~

For a basic owner-only policy, scope the query to the authenticated user:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, body
     FROM documents
     WHERE id = :document_id
       AND owner_id = :user_id'
);

$statement->execute([
    'document_id' => $documentId,
    'user_id' => $currentUser['id'],
]);

$document = $statement->fetch(PDO::FETCH_ASSOC);

if ($document === false) {
    http_response_code(404);
    exit('Document not found.');
}
~~~

Here, $currentUser must come from the authenticated server-side context, and $documentId must be validated as an identifier. The query makes it harder to accidentally use a record before checking its owner. If administrators have broader access, express that as a separate, explicit permission and apply it consistently. Do not grant it because a submitted form says role=admin.

Some applications return 404 Not Found for both a missing resource and a resource the user may not view, to avoid revealing whether another user's record exists. Others return 403 Forbidden for a known but disallowed resource. Choose a consistent policy for the application; either response must leave the protected data and state untouched.

## Authentication and authorization failures

Use 401 Unauthorized when a request has no valid authenticated identity and authentication is required. A browser application may redirect to its login page instead. Use 403 Forbidden when the identity is known but the action is disallowed. A deliberate 404 policy may be appropriate when revealing resource existence is undesirable.

Do not confuse status codes with enforcement. The essential requirement is that the protected operation is not performed unless its policy check succeeds.

## Test access-control rules

Write down who may perform each action on each resource, then test both sides of each rule. For a document feature, test at least:

| Identity | Own document | Another member's document | Admin-only action |
| --- | --- | --- | --- |
| Anonymous visitor | Denied | Denied | Denied |
| Member Alice | Allowed if policy permits | Denied | Denied |
| Member Bob | Allowed if policy permits | Denied | Denied |
| Administrator | According to explicit admin policy | According to explicit admin policy | Allowed only if granted |

Also test direct URLs and API requests, changed object IDs, different tenant IDs, expired sessions, removed permissions, and requests with client-supplied role or user fields. Confirm denied requests do not leak sensitive data or partially perform the action.

## Common mistakes

- Treating a successful login as permission to access every record.
- Storing plaintext passwords or using a fast general-purpose hash.
- Giving different login messages for unknown accounts and incorrect passwords.
- Keeping a session ID unchanged after authentication or privilege elevation.
- Accepting user IDs, roles, or permissions supplied by the browser.
- Checking permissions only in the interface or only on the first page.
- Loading an object by ID without checking ownership, tenant, and action.
- Assuming random or hard-to-guess IDs replace authorization.
- Giving every staff account the same broad administrator role.
- Returning an error after the protected side effect has already occurred.

## Key takeaways

- Authentication establishes an identity; authorization decides what it may do.
- Use password_hash(), password_verify(), and password_needs_rehash() for password storage and verification.
- Rotate the session ID after login and privilege changes; keep session credentials out of URLs and logs.
- Check permissions on the server for every request and every protected resource.
- Scope data access by the authenticated identity, ownership, tenant, and action.
- Deny by default, use least privilege, and test unauthorized paths as carefully as allowed paths.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [126 - Preventing cross-site request forgery](126-preventing-cross-site-request-forgery.md) | [Q&A](../qnas/127-authentication-authorization-and-access-control.md) | [Example](../examples/127-authentication-authorization-and-access-control.md) | [128 - Session and cookie security](128-session-and-cookie-security.md) |