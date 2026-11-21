# Secure coding and threat modeling

## Question

How can a team threat-model a profile update and implement safeguards against unauthorized record changes and SQL injection?

This example maps the feature's assets, data flow, and trust boundary, reviews threats with STRIDE, then uses a prepared statement and a server-derived user ID to update only the signed-in user's profile.

## Scope and assumptions

The example focuses on one profile-update operation. A real HTTP route must also authenticate the request, validate its CSRF token, use secure session settings, enforce request-size limits, and encode profile values when rendering HTML. Those controls surround this database function and are listed in the threat review.

The code below uses an in-memory SQLite database. PHP must have PDO and PDO_SQLITE enabled. The code and representative output were not executed here.

## 1. Map the feature

### Assets

- The user's profile data and account integrity.
- The authenticated session that identifies the current user.
- The role field, which must not be writable through this feature.

### Actors and data flow

~~~text
Signed-in user and browser
          |
          | session cookie and profile form
          v
PHP route
  - verifies the session
  - validates the CSRF token
  - reads the authenticated user ID from the session
          |
          | authenticated user ID and display name
          v
Profile update function
          |
          | parameterized SQL values
          v
Users table
~~~

The browser is outside the server's trust boundary. A user can change form fields or send a request without using the normal page, so the route must not trust a user ID supplied in the request body.

## 2. Review threats with STRIDE

| Threat prompt | Example risk | Safeguard to require |
| --- | --- | --- |
| Spoofing | A stolen or forged session is used to act as another user. | Use secure session handling, HTTPS, appropriate cookie attributes, and session lifecycle controls. |
| Tampering | Crafted profile data changes SQL structure or protected fields. | Validate the display name, bind values with a prepared statement, and update only the allowed column. |
| Repudiation | An important profile change cannot be investigated. | Record an audit event with the actor, action, target, and time without logging secrets. |
| Information disclosure | Private profile data or stored text is exposed. | Enforce read authorization, minimize returned data, and encode values for the output context. |
| Denial of service | Oversized or repeated requests consume resources. | Set request and field limits, use bounded queries, and apply suitable rate controls. |
| Elevation of privilege | A user changes another user's profile or submits a role field. | Derive the target user ID from the verified session and never accept role changes in this operation. |

This table is a starting review, not a complete security assessment. Prioritize threats using the feature's data, exposure, impact, and existing controls.

## 3. Implement the update with explicit boundaries

### File: src/ProfileUpdater.php

~~~php
<?php

declare(strict_types=1);

function updateOwnDisplayName(
    PDO $pdo,
    int $authenticatedUserId,
    string $displayName
): void {
    if ($authenticatedUserId < 1) {
        throw new InvalidArgumentException('A signed-in user is required.');
    }

    $displayName = trim($displayName);

    if (preg_match('/^.{1,80}$/uD', $displayName) !== 1) {
        throw new InvalidArgumentException(
            'Display name must contain 1 to 80 valid characters.'
        );
    }

    $statement = $pdo->prepare(
        'UPDATE users SET display_name = :display_name WHERE id = :user_id'
    );

    $statement->execute([
        'display_name' => $displayName,
        'user_id' => $authenticatedUserId,
    ]);
}
~~~

The function does not accept a target user ID from the form. Its caller must pass the identity established by the server's authenticated session. The query updates only display_name, and PDO binds the values separately from SQL syntax.

The length check limits this field to 80 Unicode code points after trimming. A production system should choose limits that match the product's needs and request-size policy.

## 4. Run a small demonstration

### File: demo.php

~~~php
<?php

declare(strict_types=1);

require_once __DIR__ . '/src/ProfileUpdater.php';

$pdo = new PDO('sqlite::memory:');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);

$pdo->exec(
    'CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        display_name TEXT NOT NULL,
        role TEXT NOT NULL
    )'
);

$insert = $pdo->prepare(
    'INSERT INTO users (id, display_name, role)
     VALUES (:id, :display_name, :role)'
);

$insert->execute([
    'id' => 1,
    'display_name' => 'Alice',
    'role' => 'member',
]);

$insert->execute([
    'id' => 2,
    'display_name' => 'Bob',
    'role' => 'member',
]);

// In a web route, get this value from the verified server-side session.
// Never take the authenticated identity from the request body.
$authenticatedUserId = 1;

// This string resembles an attempt to change SQL structure.
// A prepared statement stores it as a display-name value.
$submittedDisplayName = "Alice', role = 'admin' -- x";

updateOwnDisplayName($pdo, $authenticatedUserId, $submittedDisplayName);

$readUser = $pdo->prepare(
    'SELECT display_name, role FROM users WHERE id = :user_id'
);

$readUser->execute(['user_id' => 1]);
$alice = $readUser->fetch();

$readUser->execute(['user_id' => 2]);
$bob = $readUser->fetch();

echo 'Alice display name: ', $alice['display_name'], PHP_EOL;
echo 'Alice role: ', $alice['role'], PHP_EOL;
echo 'Bob display name: ', $bob['display_name'], PHP_EOL;
echo 'Bob role: ', $bob['role'], PHP_EOL;
~~~

Run it from the project root:

~~~sh
php demo.php
~~~

### Expected output

~~~text
Alice display name: Alice', role = 'admin' -- x
Alice role: member
Bob display name: Bob
Bob role: member
~~~

The submitted text is stored as a value; it does not become part of the SQL command. The query has no target-user parameter from the browser, so this function can update only the identity provided by its authenticated caller.

## 5. Verify the mitigation in the application

Add tests and review checks for these requirements:

1. A user can update their own display name.
2. A request cannot select another user's ID as the update target.
3. A SQL-shaped display name remains plain data and cannot change the role column.
4. A blank or oversized display name is rejected.
5. The route rejects an invalid or missing CSRF token.
6. Rendering a stored display name encodes it for its output context.
7. Audit events identify the actor and action without recording session tokens or secrets.

A database demonstration checks only part of the threat model. The route, session configuration, output rendering, deployment limits, and audit behavior need their own review and tests.

## What this example demonstrates

- Threat modeling begins with assets, data flow, and trust boundaries.
- STRIDE helps organize threat questions for the feature.
- The authenticated identity comes from the server-side session, not a form field.
- A prepared statement keeps data separate from SQL syntax.
- The database function updates only the field that this feature owns.
- Threat responses become explicit requirements that can be tested.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [122 - Formatting, linting, and code quality tools](../lessons/122-formatting-linting-and-code-quality-tools.md) | [Q&A](../qnas/123-secure-coding-and-threat-modeling.md) | [Lesson](../lessons/123-secure-coding-and-threat-modeling.md) | [124 - Preventing SQL injection](../lessons/124-preventing-sql-injection.md) |