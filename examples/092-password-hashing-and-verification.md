# 092 - Password hashing and verification: Example

## Goal

Follow a password from registration through hash storage and login verification. The runnable script uses an in-memory variable to simulate retrieving a hash from a database, so it does not create a user account or store any credentials.

## Question

How can PHP hash a password once, verify a later candidate, and replace an outdated hash without ever comparing newly generated hash strings?

## Complete runnable example: `password-workflow.php`

Save this code as `password-workflow.php` and run it from a terminal with `php password-workflow.php`.

```php
<?php

declare(strict_types=1);

// Tutorial-only sample input. A real application receives this from a form.
$registrationPassword = 'Demo-only-long-passphrase-42';

// Registration: create the value that would be stored in users.password_hash.
$storedHash = password_hash($registrationPassword, PASSWORD_DEFAULT);
echo "Registration: password hash created.\n";

// In a real login request, load this complete string from the database.
$hashLoadedFromStorage = $storedHash;

$correctCandidate = 'Demo-only-long-passphrase-42';
$incorrectCandidate = 'Not-the-password';

$correctLogin = password_verify($correctCandidate, $hashLoadedFromStorage);
$incorrectLogin = password_verify($incorrectCandidate, $hashLoadedFromStorage);

echo 'Correct candidate: ' . ($correctLogin ? "accepted\n" : "rejected\n");
echo 'Incorrect candidate: ' . ($incorrectLogin ? "accepted\n" : "rejected\n");

// Check the current policy only after a candidate has verified successfully.
$algorithm = PASSWORD_DEFAULT;
$options = [];

if ($correctLogin && password_needs_rehash($hashLoadedFromStorage, $algorithm, $options)) {
    $hashLoadedFromStorage = password_hash($correctCandidate, $algorithm, $options);
    echo "Hash policy changed: stored hash refreshed.\n";
} else {
    echo "Hash policy changed: no refresh needed.\n";
}
```

## Expected output

```text
Registration: password hash created.
Correct candidate: accepted
Incorrect candidate: rejected
Hash policy changed: no refresh needed.
```

The hash itself is deliberately not printed. Its salt is random, so the exact hash string changes between runs. In this script the generated value is kept in `$storedHash`; that assignment simulates the database write followed by a later database read.

## What each step demonstrates

1. **Registration:** `password_hash()` creates a salted password hash using PHP's current default policy.
2. **Storage boundary:** A real application stores the complete returned hash, not the password, and later loads that hash for login.
3. **Verification:** `password_verify()` accepts the correct candidate and rejects the wrong one.
4. **Rehash check:** `password_needs_rehash()` checks the stored algorithm and options against the current policy. The new hash is generated only after successful verification and only when needed.

The example intentionally uses a fake tutorial password in source code so the output is reproducible. Never hard-code a real user's password, print it, or log it.

## Mapping this flow to a database

A user table needs a column wide enough for hashes created by current and future algorithms. For example:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    email VARCHAR(254) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL
);
```

After validating registration fields and creating `$hash`, store it with a prepared statement:

```php
$statement = $pdo->prepare(
    'INSERT INTO users (email, password_hash) VALUES (:email, :password_hash)'
);
$statement->execute([
    'email' => $email,
    'password_hash' => $hash,
]);
```

For login, fetch the hash for the requested account using a prepared query, then verify the candidate:

```php
$statement = $pdo->prepare(
    'SELECT id, password_hash FROM users WHERE email = :email'
);
$statement->execute(['email' => $email]);
$user = $statement->fetch(PDO::FETCH_ASSOC);

if ($user !== false && password_verify($candidate, $user['password_hash'])) {
    // If needed, create and save a replacement hash before continuing login.
    // Then continue the authentication flow.
} else {
    // Return the same generic failure for an unknown email or wrong password.
}
```

These database snippets assume `$pdo` is already configured. The upcoming database chapters cover PDO setup and database operations. Keep the user lookup parameterized, and do not return a different login message for “unknown email” and “wrong password.”

## Important notes

- `PASSWORD_DEFAULT` may change. Keep the whole hash in a sufficiently wide field such as `VARCHAR(255)`.
- Do not generate or pass a manual salt; the password API handles it.
- Do not replace `password_verify()` with a newly generated hash comparison.
- If selecting `PASSWORD_BCRYPT` explicitly, remember its 72-byte input limit; do not silently truncate long passwords.
- Hashing slows offline guessing but does not prevent online login attempts. Add rate limiting and monitoring.
- This example is not a complete registration, login, reset, or authorization system. A real application also needs validation, safe database writes, secure sessions, and an account recovery process.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [091 - Sessions and server-side state](../lessons/091-sessions-and-server-side-state.md) | [Q&A](../qnas/092-password-hashing-and-verification.md) | Example | [093 - Authentication and authorization](../lessons/093-authentication-and-authorization.md) |