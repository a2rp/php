# Code review, maintenance, and upgrades — Example

## Scenario

A pull request says: “Allow customers to update their email address.”

The proposed change builds an SQL statement by concatenating the email into the query. Review the change for security, correctness, maintainability, tests, and release impact. Then work through a safe PHP runtime and dependency maintenance plan.

## Proposed change to review

~~~php
function updateCustomerEmail(PDO $pdo, int $customerId, string $email): void
{
    $sql = "UPDATE customers SET email = '$email' WHERE id = $customerId";
    $pdo->exec($sql);
}
~~~

## Review findings

### Finding 1: The query mixes data with SQL code

The email is inserted into the SQL text. An input containing quotes or SQL syntax can change what the database executes. Use a prepared statement and bind values as parameters.

### Finding 2: The input contract is incomplete

A native string type says the value is a string; it does not establish that it has an acceptable email format. Validate the value and define the application's normalization policy.

### Finding 3: Authorization is not shown

A database update method does not prove that the current user may update this customer. The calling application service or controller must check authentication and authorization before calling it. Never rely on the customer ID supplied by a request as proof of permission.

### Finding 4: The change needs focused evidence

The pull request should show that a valid address is saved and that invalid input is rejected. A security review should also confirm that an input containing SQL syntax is handled as ordinary data and cannot change the query structure.

## Revised implementation

This complete script uses an in-memory SQLite database so the example can demonstrate the update without a separate database server. It requires PHP 8.0 or later and the PDO SQLite extension.

~~~php
<?php
declare(strict_types=1);

/**
 * Update a customer's email address after validating the input.
 *
 * The caller is responsible for authenticating the user and checking
 * authorization before calling this database operation.
 *
 * @throws InvalidArgumentException If the customer ID or email is invalid.
 * @throws PDOException If the database operation fails.
 */
function updateCustomerEmail(PDO $pdo, int $customerId, string $email): void
{
    if ($customerId < 1) {
        throw new InvalidArgumentException('Customer ID must be positive.');
    }

    $normalizedEmail = trim($email);

    if (filter_var($normalizedEmail, FILTER_VALIDATE_EMAIL) === false) {
        throw new InvalidArgumentException('Enter a valid email address.');
    }

    $statement = $pdo->prepare(
        'UPDATE customers SET email = :email WHERE id = :id'
    );

    $statement->execute([
        'email' => $normalizedEmail,
        'id' => $customerId,
    ]);
}

$pdo = new PDO('sqlite::memory:');
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$pdo->exec(
    'CREATE TABLE customers (
        id INTEGER PRIMARY KEY,
        email TEXT NOT NULL
    )'
);
$pdo->exec(
    "INSERT INTO customers (id, email) VALUES (1, 'old@example.test')"
);

try {
    updateCustomerEmail($pdo, 1, '  new@example.test  ');

    $statement = $pdo->query(
        'SELECT email FROM customers WHERE id = 1'
    );

    printf("Saved email: %s\n", $statement->fetchColumn());

    updateCustomerEmail($pdo, 1, "attacker@example.test' OR 1=1 --");
} catch (InvalidArgumentException $exception) {
    printf("Input rejected: %s\n", $exception->getMessage());
} catch (PDOException $exception) {
    echo "Database update failed.\n";
}
~~~

## Expected output

~~~text
Saved email: new@example.test
Input rejected: Enter a valid email address.
~~~

## Why the revised version is safer

- The prepared statement keeps SQL structure separate from email and ID values.
- The input is trimmed and validated before it reaches the database.
- The integer parameter type and positive-ID check express separate parts of the input contract.
- Database failures are allowed to surface as PDO exceptions; the example reports a generic message rather than showing database details to an end user.
- The docblock explains the operation and its exception behavior. It also makes the authorization responsibility explicit.
- The validation here is an example policy. Real applications should define address rules that fit their product; syntax validation does not prove that an inbox exists or that a user owns it.

## Review evidence to request

| Check | Example evidence |
| --- | --- |
| Valid update | A valid address is stored after the operation |
| Normalization | Leading and trailing whitespace is removed according to the chosen policy |
| Invalid input | Malformed addresses are rejected before an update |
| SQL safety | Quote characters are passed as parameter values and cannot alter the SQL statement |
| Authorization | The caller checks that the current user may update this customer |
| Database failure | Failures are handled without exposing SQL, credentials, or internal details |
| Compatibility | The target PHP runtime and required PDO SQLite extension are available in development and CI |

A reviewer can ask for automated tests around these behaviors. The in-memory database makes a focused integration test possible without modifying a developer's local database.

## Dependency maintenance example

Before updating a package in an application, inspect the direct dependencies and current advisories:

~~~sh
composer outdated --direct
composer audit
~~~

If one package has a reviewed update, update it deliberately:

~~~sh
composer update vendor/package
git diff -- composer.json composer.lock
~~~

Replace **vendor/package** with the actual package name. Review the lock-file changes and the package's release notes. Then run the project checks and important flows. Do not commit a regenerated lock file until the dependency changes are understood.

## PHP runtime upgrade example

Use this plan when moving the application to a newer PHP release:

1. Record the PHP version and extensions used by the developer CLI, CI, web runtime, workers, and scheduled jobs.
2. Check the PHP support schedule and select a supported target that the deployment platform can run.
3. Review migration notes for every version between the current runtime and the target.
4. Check framework and package requirements, then update the declared PHP constraint only when the code and tooling support it.
5. Resolve incompatibilities in small, focused changes and inspect deprecation and static-analysis reports.
6. Run the project checks and critical application flows on the target runtime.
7. Deploy to a representative environment and confirm that web requests, CLI commands, workers, and scheduled tasks use the same intended runtime.
8. Watch logs and health signals during rollout. Keep a known-good application artifact and compatible dependency/runtime set available.
9. Review database changes separately. A code rollback will not restore data removed by an irreversible migration.

## Example review summary

A useful review conclusion could be:

> **Blocking:** Please replace the interpolated SQL with a prepared statement. The email value currently changes the SQL text. Also reject invalid email input before saving it.
>
> **Follow-up:** Confirm the caller checks authorization for this customer ID, and add coverage for valid input, invalid input, and SQL-shaped input.
>
> **Release note:** This patch does not change the database schema. Verify its PHP and PDO requirements in CI before release.

This summary names required corrections, requests concrete evidence, and identifies the release check without mixing unrelated refactoring into the patch.

## Key takeaways

- Review the behavior and risk behind a diff, not only its formatting.
- Parameterize database values and validate input at the application boundary.
- Keep authentication and authorization explicit in the calling flow.
- Make maintenance changes small enough to understand and verify.
- Inspect dependency and lock-file changes before updating an application.
- Test runtime upgrades across web, CLI, workers, schedulers, and CI.
- Plan recovery around data changes as well as code.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [146 - PHPDoc, API documentation, and type annotations](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) | [Lesson](../lessons/147-code-review-maintenance-and-upgrades.md) | [Q&A](../qnas/147-code-review-maintenance-and-upgrades.md) | [148 - Project: build a personal portfolio page](../lessons/148-project-build-a-personal-portfolio-page.md) |
