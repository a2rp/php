# 127 - Authentication, authorization, and access control: Example

## Question

Create a small PHP demonstration that verifies a password, establishes a trusted user identity, and allows access to a document only when the user owns it or has an explicit administrator permission. Show successful login, denied access to another member's document, and the expected output.

## Complete example

Save this as `authentication-access-control.php` and run it from a terminal with PHP CLI:

~~~sh
php authentication-access-control.php
~~~

This self-contained demonstration uses in-memory fixture data so it can run without a database. The sample passwords and records are for this demonstration only; never hard-code real user passwords or use in-memory fixtures as production storage.

~~~php
<?php

declare(strict_types=1);

$users = [
    [
        'id' => 1,
        'email' => 'alice@example.test',
        'role' => 'member',
        'password_hash' => password_hash(
            'alice-demo-password',
            PASSWORD_DEFAULT
        ),
    ],
    [
        'id' => 2,
        'email' => 'bob@example.test',
        'role' => 'member',
        'password_hash' => password_hash(
            'bob-demo-password',
            PASSWORD_DEFAULT
        ),
    ],
    [
        'id' => 3,
        'email' => 'admin@example.test',
        'role' => 'admin',
        'password_hash' => password_hash(
            'admin-demo-password',
            PASSWORD_DEFAULT
        ),
    ],
];

$documents = [
    [
        'id' => 101,
        'owner_id' => 1,
        'title' => 'Alice private notes',
    ],
    [
        'id' => 202,
        'owner_id' => 2,
        'title' => 'Bob private notes',
    ],
];

function authenticate(
    string $email,
    string $submittedPassword,
    array $users
): ?array {
    foreach ($users as $user) {
        if ($user['email'] !== $email) {
            continue;
        }

        if (!password_verify($submittedPassword, $user['password_hash'])) {
            return null;
        }

        // Return only trusted identity fields needed by the policy.
        return [
            'id' => $user['id'],
            'email' => $user['email'],
            'role' => $user['role'],
        ];
    }

    return null;
}

function findViewableDocument(
    int $documentId,
    array $currentUser,
    array $documents
): ?array {
    foreach ($documents as $document) {
        if ($document['id'] !== $documentId) {
            continue;
        }

        $isOwner = $document['owner_id'] === $currentUser['id'];
        $hasViewAllPermission = $currentUser['role'] === 'admin';

        if ($isOwner || $hasViewAllPermission) {
            return $document;
        }

        // Do not reveal whether another user's document exists.
        return null;
    }

    return null;
}

function reportDocumentAccess(
    string $label,
    int $documentId,
    array $currentUser,
    array $documents
): void {
    $document = findViewableDocument(
        $documentId,
        $currentUser,
        $documents
    );

    if ($document === null) {
        echo $label . ': denied (not found or not permitted).' . PHP_EOL;
        return;
    }

    echo $label . ': allowed (' . $document['title'] . ').' . PHP_EOL;
}

$alice = authenticate(
    'alice@example.test',
    'alice-demo-password',
    $users
);

if ($alice === null) {
    echo 'Alice: authentication failed.' . PHP_EOL;
    exit(1);
}

echo 'Alice: login succeeded.' . PHP_EOL;

reportDocumentAccess(
    'Alice opening her document',
    101,
    $alice,
    $documents
);

reportDocumentAccess(
    'Alice opening Bob’s document',
    202,
    $alice,
    $documents
);

$invalidLogin = authenticate(
    'alice@example.test',
    'incorrect-password',
    $users
);

echo $invalidLogin === null
    ? 'Incorrect password: authentication failed.' . PHP_EOL
    : 'Incorrect password: login succeeded.' . PHP_EOL;

$admin = authenticate(
    'admin@example.test',
    'admin-demo-password',
    $users
);

if ($admin !== null) {
    reportDocumentAccess(
        'Admin opening Bob’s document',
        202,
        $admin,
        $documents
    );
}
~~~

## Expected output

~~~text
Alice: login succeeded.
Alice opening her document: allowed (Alice private notes).
Alice opening Bob’s document: denied (not found or not permitted).
Incorrect password: authentication failed.
Admin opening Bob’s document: allowed (Bob private notes).
~~~

## Explanation

### Passwords are verified, not compared as plain text

The fixture hashes each demonstration password with password_hash(). During login, authenticate() uses password_verify() against the stored hash. The function returns only the user's ID, email, and role after successful verification; it never returns the password or password hash as the authenticated identity.

In a real application, retrieve the user from a database with a prepared query. Store a password hash in a column wide enough for future algorithms, and use password_needs_rehash() after successful verification when the hash policy changes. Return the same public failure message for an unknown account and an incorrect password, and rate-limit login attempts.

### The current identity comes from trusted authentication

The access checks receive the identity returned by authenticate(). A web application should establish this identity in its server-side session after successful login, and load it from that trusted session on each protected request. Never replace it with a user ID or role submitted by the browser.

A web login handler should rotate the session ID before setting the authenticated user ID:

~~~php
session_regenerate_id(true);
$_SESSION['user_id'] = (int) $authenticatedUser['id'];
~~~

Configure the session cookie before session_start(). For an HTTPS deployment, use Secure, HttpOnly, and an intentional SameSite setting, and enable strict session mode. Applications with concurrent requests should follow the session lifecycle approach recommended for their framework and deployment.

### Ownership and administrator permission are separate policy paths

findViewableDocument() allows a member to see a document they own. It also allows the example's administrator role to view every document. The role is read from the trusted fixture record, not a request parameter. The administrator rule is explicit so it can be reviewed and tested.

Production applications should represent permissions in their trusted account or authorization data. A role may map to a permission such as documents.view_all. Keep tenant membership, resource ownership, and the requested action in the policy too; having a role called admin should not silently bypass every policy.

For a database-backed owner-only lookup, include both the document ID and authenticated owner ID in the query:

~~~sql
SELECT id, title, body
FROM documents
WHERE id = :document_id
  AND owner_id = :user_id
~~~

This scoped lookup prevents a member from receiving another member's row just by changing an identifier. Use parameterized queries, validate identifiers, and check authorization before performing updates or other side effects.

### Denials do not disclose another member's record

When Alice tries to open document 202, the function returns no document. The output intentionally says “not found or not permitted” instead of confirming that Bob's private record exists. This is one possible response policy; an application may use 403 for a known but disallowed resource if that matches its design.

The important property is that the sensitive record is not returned and no action occurs. An unguessable document ID or a hidden interface link would not be a substitute for this server-side check.

## What this example demonstrates

- A password hash is created and checked with PHP's password APIs.
- Login returns a limited identity only after password verification succeeds.
- The identity used for authorization comes from trusted server-side state.
- A member can view their own document but is denied another member's document.
- A separate, explicit administrator policy can allow broader access.
- A denial avoids disclosing private document contents or confirming their existence.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [126 - Preventing cross-site request forgery](../lessons/126-preventing-cross-site-request-forgery.md) | [Q&A](../qnas/127-authentication-authorization-and-access-control.md) | [Lesson](../lessons/127-authentication-authorization-and-access-control.md) | [128 - Session and cookie security](../lessons/128-session-and-cookie-security.md) |