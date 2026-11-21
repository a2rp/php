# 132 - Managing secrets and production configuration: Questions and answers

Use these questions to review where PHP applications should get configuration, how to protect credentials, and what to do when a secret is exposed.

## Multiple-choice questions

### 1. Which value is a secret?

- A. A feature flag that is already public.
- B. A database password that grants access to application data.
- C. A heading displayed on the home page.
- D. The name of a CSS class.

**Answer:** B. **Explanation:** A database password grants access. Treat API tokens, private keys, signing keys, and similar credentials the same way.

### 2. What belongs in a source-controlled `.env.example` file?

- A. The live production database password.
- B. A real API token with limited permissions.
- C. Variable names and unmistakable placeholder values.
- D. A copy of the production environment.

**Answer:** C. **Explanation:** An example file documents required settings without exposing credentials. The real local file must remain untracked and access-controlled.

### 3. What does `getenv('APP_KEY')` return if the named variable is unavailable?

- A. Always an empty string.
- B. `false`.
- C. The previous request's value.
- D. A generated key.

**Answer:** B. **Explanation:** Check explicitly for `false` and reject missing required settings. Do not silently substitute a known development or example secret.

### 4. Which is a sound production default for debug output?

- A. Enable it because production has more users.
- B. Disable it and log errors to a protected destination.
- C. Show it only when the request has a debug parameter.
- D. Print the full configuration when startup fails.

**Answer:** B. **Explanation:** Visitors should not receive stack traces, internal paths, configuration values, or secret-bearing details.

### 5. Why should development, test, and production use separate credentials?

- A. To make password strings longer in production.
- B. So compromise of one environment does not automatically grant access to another.
- C. Because PHP requires different variable names in each environment.
- D. To avoid validating configuration.

**Answer:** B. **Explanation:** Separate identities limit the scope of a leak or compromise. Test credentials should not be able to access production data.

### 6. What should the team do first if a live API token is pushed to a repository?

- A. Delete the latest commit and assume the token is safe.
- B. Rename the token variable.
- C. Revoke or rotate the token, then investigate and clean up retained copies.
- D. Make the repository private and take no further action.

**Answer:** C. **Explanation:** The value may remain in history, clones, logs, or caches. Repository cleanup cannot revoke a credential.

### 7. What is the correct storage strategy for a user's login password?

- A. Store it as an environment variable.
- B. Encrypt it so support staff can read it.
- C. Store a password hash created with PHP's password hashing API.
- D. Store it in a tracked configuration file.

**Answer:** C. **Explanation:** A user password should generally be verified from a one-way password hash using `password_hash()` and `password_verify()`, not recovered like a database credential.

### 8. Which statement about environment variables is most accurate?

- A. They are always secret and cannot be inspected.
- B. They are a possible secret delivery method, but process and deployment access still need protection.
- C. They automatically rotate database credentials.
- D. They are safe to print because they are not files.

**Answer:** B. **Explanation:** Process inspection, diagnostics, crash dumps, or broad deployment access may expose them. Use the platform's supported controls and secret-delivery model.

## Short-answer questions

### 9. What is the difference between configuration and a secret?

**Answer:** Configuration describes how an application should run, such as its environment name or a timeout. A secret proves identity or grants authority, such as a credential, token, signing key, or private key. **Explanation:** Some configuration values may also be sensitive, so classify them by what an attacker could do with them.

### 10. Why should a required production setting fail startup when it is absent?

**Answer:** Failing early prevents the application from silently running with an unsafe default, a development credential, or an unintended security setting. **Explanation:** A clear startup failure is easier to detect and safer than an application that partially works with unknown configuration.

### 11. Does `.gitignore` protect a secret file that is already tracked?

**Answer:** No. Git continues tracking a file already in the index even after an ignore rule is added. **Explanation:** Remove it from tracking, rotate any exposed secret, and review the repository history and other copies.

### 12. Why is deleting a credential from the latest commit not enough?

**Answer:** The credential may remain in earlier commits, forks, local clones, CI logs, artifacts, backups, or monitoring systems. **Explanation:** Revoke or rotate the credential first. Then investigate and remove retained copies where practical.

### 13. Why should an application not log its full configuration array?

**Answer:** The array may contain database credentials, API tokens, signing keys, internal hosts, or other sensitive values. **Explanation:** Log safe event metadata and redact sensitive fields rather than dumping runtime state.

### 14. Why is a production secret manager or identity-based retrieval useful?

**Answer:** It can centralize access control, auditing, and rotation while letting a workload retrieve only the secrets its identity needs. **Explanation:** The precise protection depends on the platform and permissions; the application still needs least privilege and safe logging.

### 15. What should a placeholder in a sample configuration look like?

**Answer:** It should be unmistakably fake, such as `replace-with-a-local-only-value`, and must not grant access anywhere. **Explanation:** Plausible-looking sample credentials are easy to accidentally deploy or confuse with a real secret.

## Code review questions

### 16. What is wrong with this configuration loader?

~~~php
$password = getenv('APP_DATABASE_PASSWORD') ?: 'admin123';
~~~

**Answer:** Missing or empty production configuration silently falls back to a weak, known credential. **Explanation:** Check whether the variable is missing and fail startup; never use a public or development default for production credentials.

### 17. What is wrong with this production error handling?

~~~php
catch (Throwable $exception) {
    echo $exception;
    echo json_encode($_ENV);
}
~~~

**Answer:** It exposes internal exception details and may reveal environment credentials to the visitor. **Explanation:** Return a generic error response, log a redacted diagnostic to a protected destination, and never dump environment data.

### 18. Is this ignore rule sufficient to secure a tracked local environment file?

~~~gitignore
.env
~~~

**Answer:** No. It prevents new untracked `.env` files from being added by ordinary matching, but it does not remove a file that Git already tracks. **Explanation:** Check tracking state, stop tracking the real file, add a safe `.env.example`, and rotate any value already exposed.

### 19. What is wrong with embedding an API token in browser JavaScript?

~~~javascript
const token = 'live-service-token';
~~~

**Answer:** The browser receives the source, so every visitor can inspect and copy the token. **Explanation:** Keep private credentials on the server and expose only an authorized application endpoint.

### 20. Why is this database connection example dangerous?

~~~php
$dsn = getenv('APP_DATABASE_DSN') ?: 'mysql:host=prod-db.example;dbname=app';
$user = getenv('APP_DATABASE_USER') ?: 'root';
$password = getenv('APP_DATABASE_PASSWORD') ?: 'password';
~~~

**Answer:** It silently falls back to production-like host details and a privileged, predictable account if deployment settings are missing. **Explanation:** Require explicit settings, validate them, and use a dedicated least-privileged database identity.

## Applied scenarios

### 21. A developer committed a real private key yesterday and removed it in a new commit. What should the team do?

**Answer:** Revoke or replace the key immediately, identify what it protected, review access logs and downstream systems, and then remove it from repository history and retained artifacts where appropriate. **Explanation:** Every clone or cached copy may still contain the old key; deleting the newest copy does not invalidate it.

### 22. A PHP project uses a dotenv loader in local development. What should be committed?

**Answer:** Commit the loader configuration if appropriate and a placeholder-only `.env.example`. Ignore the real local `.env` file and never copy production credentials into it. **Explanation:** PHP itself does not automatically load dotenv files; the project must use a known library or framework behavior.

### 23. A staging application currently shares the production database password. What is the safer design?

**Answer:** Create a separate staging identity with only the staging database permissions it needs, update staging deployment settings, verify the application, and revoke the shared credential if it is no longer required. **Explanation:** Separate credentials prevent a staging compromise from automatically reaching production.

### 24. An application needs to display an error to a visitor and diagnose the failure. How should it handle the exception?

**Answer:** Return a generic status and message to the visitor. Record a correlation ID and a redacted diagnostic in a protected log, excluding credentials and sensitive request fields. **Explanation:** This gives operators a way to investigate without exposing implementation details or secrets publicly.

### 25. The deployment platform can inject environment variables or mount secret files. Which should the team choose?

**Answer:** Choose the method the platform supports that gives the application only necessary access and keeps values out of source control, images, build output, logs, and broad diagnostics. Prefer managed or identity-based secret delivery where available. **Explanation:** Neither environment variables nor files are automatically safe; access controls, process isolation, permissions, and lifecycle management matter.

## Coding exercise

### 26. Write a function that reads a required environment variable, rejects missing values, and never prints the secret. What should the caller print to confirm configuration loaded?

**Answer:**

~~~php
<?php

declare(strict_types=1);

function requiredSecret(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException(
            'Missing required configuration: ' . $name
        );
    }

    return $value;
}

$databasePassword = requiredSecret('APP_DATABASE_PASSWORD');

// Use the value in the server-side database connection.
// Never echo it, log it, or include it in an exception.
echo "Database credential: configured", PHP_EOL;
~~~

**Explanation:** The code checks for both a missing and empty value and does not expose the credential. A real application should also validate the rest of its configuration, restrict the credential's permissions, and use a protected deployment mechanism to provide it.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [131 - Command injection and unsafe deserialization](../lessons/131-command-injection-and-unsafe-deserialization.md) | [Lesson](../lessons/132-managing-secrets-and-production-configuration.md) | [Example](../examples/132-managing-secrets-and-production-configuration.md) | [133 - Dependency security and software supply chains](../lessons/133-dependency-security-and-software-supply-chains.md) |