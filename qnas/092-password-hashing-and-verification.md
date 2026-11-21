# 092 - Password hashing and verification: Q&A

## Multiple-choice questions

### 1. What should an application store for a user's password?

- **A.** The password as plaintext
- **B.** A reversible encrypted version so it can be recovered
- **C.** A purpose-built password hash
- **D.** A plain MD5 digest

**Answer: C.** Store a password hash made with PHP's password API. The application should verify a submitted password without recovering the original one.

### 2. Which PHP function creates a password hash?

- **A.** `password_hash()`
- **B.** `password_verify()`
- **C.** `hash_file()`
- **D.** `password_encrypt()`

**Answer: A.** `password_hash()` creates a one-way password hash with an algorithm and options. `password_verify()` is used later to check a candidate password.

### 3. Which function should check a submitted password against a stored password hash?

- **A.** `password_hash()`
- **B.** `password_verify()`
- **C.** `strcmp()`
- **D.** `md5()`

**Answer: B.** `password_verify($candidate, $storedHash)` uses the algorithm and salt information encoded in the stored hash.

### 4. Should an application generate and store a separate manual salt for `password_hash()`?

- **A.** Yes, use the username as the salt
- **B.** Yes, use the same random salt for every account
- **C.** No, let `password_hash()` create a random salt automatically
- **D.** No, remove the salt from the returned hash

**Answer: C.** PHP generates a suitable random salt and includes the information needed for verification in the hash result. Do not provide a custom salt.

### 5. Why do two calls to `password_hash()` for the same password usually return different strings?

- **A.** The function randomly changes the password itself
- **B.** Each hash normally uses a new random salt
- **C.** PHP cannot store repeated values
- **D.** The algorithm always adds the current timestamp

**Answer: B.** Random salts make identical passwords produce distinct stored hashes. Verify with `password_verify()` instead of comparing a newly generated hash string.

### 6. What does `PASSWORD_DEFAULT` mean?

- **A.** A fixed algorithm that will never change
- **B.** PHP's current default password algorithm, which may change in a future release
- **C.** No hashing algorithm
- **D.** A reversible encryption mode

**Answer: B.** The identifier lets PHP's default evolve. Store hashes in a field wide enough for future output, such as `VARCHAR(255)`.

### 7. When should an application call `password_needs_rehash()`?

- **A.** Before a password has been checked
- **B.** After `password_verify()` succeeds
- **C.** Only when registering a new account
- **D.** Every time a public page loads

**Answer: B.** After successful verification, the submitted password is available for creating a replacement hash using the current algorithm and options.

### 8. What is `password_verify()` designed to return?

- **A.** The original password
- **B.** A boolean indicating whether the candidate matches the hash
- **C.** The database row containing the hash
- **D.** A newly generated salt

**Answer: B.** It returns `true` when the candidate matches and `false` otherwise. It is designed to avoid timing-attack-prone manual comparisons.

### 9. Why is MD5 not suitable for password storage?

- **A.** It is a fast general-purpose hash, so guesses can be tested quickly
- **B.** It requires a database column of 255 bytes
- **C.** PHP cannot calculate it
- **D.** It changes passwords into HTML

**Answer: A.** Password hashing should be deliberately expensive to slow offline guessing. Use PHP's purpose-built password API rather than MD5 or SHA-1.

### 10. What limit should be remembered if an application explicitly uses `PASSWORD_BCRYPT`?

- **A.** Bcrypt processes at most 72 bytes of a password
- **B.** Bcrypt accepts only numeric passwords
- **C.** Bcrypt stores passwords in a cookie
- **D.** Bcrypt always truncates passwords to 8 characters

**Answer: A.** The documented limit is 72 bytes, which may be fewer than 72 visible characters for multibyte text. Do not silently truncate input.

## Short-answer questions

### 11. Why is password hashing different from encryption?

**Answer:** Hashing is designed to be one-way, while encryption is designed to be reversible with a key.

**Explanation:** A login checks a candidate against a stored hash. The application should not decrypt or recover the user's password.

### 12. What information should be stored from `password_hash()`?

**Answer:** Store the complete returned hash string.

**Explanation:** The output contains the algorithm identifier, cost/options, salt, and hash data needed by `password_verify()`. Do not strip parts out or store a separate manually generated salt.

### 13. Why should the password-hash column be wide enough for future algorithms?

**Answer:** The output length can change as `PASSWORD_DEFAULT` changes or an algorithm stores different metadata.

**Explanation:** A common practical field size is `VARCHAR(255)`, which avoids truncating future hashes.

### 14. Why should an application not silently trim or truncate a password?

**Answer:** It changes the credential the user entered and can make different inputs behave as if they were the same password.

**Explanation:** Define a clear maximum input policy to bound work, apply it consistently, and reject over-limit values instead of changing them silently.

### 15. Why is rehashing performed only after a successful verification?

**Answer:** Rehashing requires the submitted plaintext candidate, and the application should replace the stored hash only after proving that the candidate matches the existing one.

### 16. What does a password hash not protect against?

**Answer:** It does not prevent online password guessing, phishing, stolen credentials, SQL injection, or insecure account recovery.

**Explanation:** Use rate limits and monitoring for login attempts, parameterized queries for database operations, and a secure authentication and recovery design.

### 17. Why should login errors be generic?

**Answer:** A generic message such as “Invalid email or password” avoids telling an attacker whether an account exists.

### 18. What should be considered before selecting Argon2id?

**Answer:** Confirm that the PHP build supports it and tune its memory and time costs on the actual deployment hardware.

**Explanation:** Password hashing consumes resources by design. A configuration should be strong enough to slow guesses while remaining practical for the application and its capacity.

## Code and behavior questions

### 19. Why is this registration code unsafe?

```php
$users[$email]['password'] = $submittedPassword;
```

**Answer:** It stores the original password in plaintext.

**Safer approach:**

```php
$storedHash = password_hash($submittedPassword, PASSWORD_DEFAULT);
```

Store `$storedHash` using a prepared database query, not the plaintext value.

### 20. Why is this password check incorrect?

```php
if (password_hash($candidate, PASSWORD_DEFAULT) === $storedHash) {
    // Login succeeds.
}
```

**Answer:** The newly generated hash normally uses a different random salt, so it will not match the stored string even when the password is correct.

**Correct check:**

```php
if (password_verify($candidate, $storedHash)) {
    // Login succeeds.
}
```

### 21. What does this login flow do when the hash uses an older policy?

```php
if (password_verify($candidate, $storedHash)) {
    if (password_needs_rehash($storedHash, PASSWORD_DEFAULT)) {
        $storedHash = password_hash($candidate, PASSWORD_DEFAULT);
        // Save the new hash.
    }
}
```

**Answer:** It verifies the candidate and, only if verification succeeds, replaces an outdated hash with one generated under the current default policy.

**Explanation:** Save the new hash to the same account record using a prepared update. Do not log or retain the candidate password after the request.

### 22. What is wrong with manually passing a constant salt?

```php
$hash = password_hash($password, PASSWORD_DEFAULT, [
    'salt' => 'shared-salt',
]);
```

**Answer:** Every user would share the same predictable salt, and custom salt handling is unnecessary and unsupported in modern PHP versions.

**Correct approach:**

```php
$hash = password_hash($password, PASSWORD_DEFAULT);
```

### 23. Does a successful `password_hash()` call prove that an account was securely registered?

**Answer:** No. It only creates the hash. The application still needs validated account data, a safe prepared database write, appropriate duplicate-account handling, and any required verification workflow.

### 24. What should the application do if `password_verify()` returns `false`?

**Answer:** Reject the login with a generic failure response, avoid creating an authenticated session, and apply the application's rate-limiting and monitoring policy.

## Key takeaways

- Hash with `password_hash()` and verify with `password_verify()`; do not invent a password scheme.
- Let PHP create the salt and store the whole hash string.
- Rehash only after successful verification when `password_needs_rehash()` indicates the policy changed.
- Use a sufficiently wide hash column and reject oversized passwords rather than silently truncating them.
- Password hashing is only one part of a secure registration and login system.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [091 - Sessions and server-side state](../lessons/091-sessions-and-server-side-state.md) | [Lesson](../lessons/092-password-hashing-and-verification.md) | [Example](../examples/092-password-hashing-and-verification.md) | [093 - Authentication and authorization](../lessons/093-authentication-and-authorization.md) |