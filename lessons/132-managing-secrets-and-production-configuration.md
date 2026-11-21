# 132 - Managing secrets and production configuration

PHP applications need configuration such as environment names, database connection details, API credentials, encryption keys, and debug settings. Keep ordinary configuration separate from secrets, provide each environment with only the values and permissions it needs, and make the application fail safely when required settings are missing.

A secret is any value that grants access or authority: passwords, API tokens, private keys, signing keys, encryption keys, and database credentials. A value does not stop being a secret because it is stored in a configuration file, environment variable, container, build system, or database.

## Learning goals

By the end of this chapter, you should be able to:

- Distinguish ordinary configuration from secrets.
- Keep credentials and private keys out of source control and public responses.
- Load deployment-provided settings safely in PHP.
- Use local environment files only as a protected development convenience.
- Fail fast when required settings are missing or invalid.
- Use distinct, least-privileged credentials for development, testing, and production.
- Rotate and revoke secrets after exposure.
- Avoid leaking secrets through logs, errors, process arguments, or debugging output.

## Configuration and secrets

Configuration describes how an application should run in a particular environment. Examples include the environment name, a feature flag, a public application URL, or a non-sensitive timeout. Secrets prove identity or grant access, such as a database password, API token, session-signing key, or encryption key.

Keep configuration names in code, but supply environment-specific values at deployment time. Never put real credentials, keys, or production endpoints containing credentials into a tracked PHP file.

A source-controlled example file can document required variable names with unmistakable placeholders:

~~~text
APP_ENV=development
APP_DEBUG=false
APP_DATABASE_DSN=mysql:host=127.0.0.1;dbname=app;charset=utf8mb4
APP_DATABASE_USER=app_user
APP_DATABASE_PASSWORD=replace-with-a-local-only-value
~~~

This is a template, not a place to paste real credentials. The real local file or deployment settings must remain outside version control.

## Choose a secret delivery method for the environment

For production, prefer the platform's managed secret store or identity-based secret retrieval when available. A platform may inject a secret at runtime, provide a protected mounted file, or grant the application identity permission to retrieve a secret. Choose the supported method that limits who and what can read it.

Environment variables are a common delivery mechanism, and PHP can read them with `getenv()`. They are not automatically a secure vault: process inspection, crash dumps, diagnostics, deployment settings, or overly broad access may expose them. Protect the application process and deployment controls, and avoid assuming one delivery method fits every platform.

For local development, a dotenv file can be convenient if a framework or library loads it. Keep the real file out of version control, restrict access on shared machines, and provide a safe example file with placeholders. PHP does not load a `.env` file by itself.

## Load and validate required settings

Read the configuration once during application startup. Reject missing or malformed values instead of silently using a real credential, an unsafe default, or a production debug setting.

~~~php
<?php

declare(strict_types=1);

function requiredEnvironmentValue(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException(
            'Missing required configuration: ' . $name
        );
    }

    return $value;
}

function booleanEnvironmentValue(string $name, bool $default): bool
{
    $value = getenv($name);

    if ($value === false) {
        return $default;
    }

    return match (strtolower($value)) {
        '1', 'true', 'yes', 'on' => true,
        '0', 'false', 'no', 'off' => false,
        default => throw new RuntimeException(
            'Invalid boolean configuration: ' . $name
        ),
    };
}

function loadApplicationConfig(): array
{
    $environment = requiredEnvironmentValue('APP_ENV');

    if (!in_array($environment, ['development', 'test', 'production'], true)) {
        throw new RuntimeException('Unsupported application environment.');
    }

    $debug = booleanEnvironmentValue('APP_DEBUG', false);

    if ($environment === 'production' && $debug) {
        throw new RuntimeException('Debug mode must be disabled in production.');
    }

    return [
        'environment' => $environment,
        'debug' => $debug,
        'databaseDsn' => requiredEnvironmentValue('APP_DATABASE_DSN'),
        'databaseUser' => requiredEnvironmentValue('APP_DATABASE_USER'),
        'databasePassword' => requiredEnvironmentValue('APP_DATABASE_PASSWORD'),
    ];
}

$config = loadApplicationConfig();
~~~

`getenv()` returns `false` when a named environment variable is unavailable, so check for that explicitly. A value that is present can still be invalid: validate enums, booleans, lengths, formats, and deployment-specific constraints. Do not print the returned array; it contains credentials.

This example loads settings and enforces a production debug rule. A real application should also validate the database driver and connection requirements, and should avoid showing raw configuration values in exceptions.

## Give each environment a separate identity

Use separate credentials for local development, automated tests, staging, and production. A test process should not be able to access production data. A local developer account should not have production administrator privileges.

Grant each credential only the operations and resources the application needs. Prefer short-lived credentials where the platform supports them. Restrict who can read and update production secret values, and audit those actions.

Do not let configuration choose an unapproved security policy. For example, an environment flag can enable a development-only feature, but a request parameter must never decide that the production application disables authentication or TLS verification.

## Keep secrets out of files and version control

- Do not commit credentials, private keys, tokens, production `.env` files, or database dumps.
- Keep local secret files outside the repository where practical; otherwise ignore the exact local files and restrict their permissions.
- Commit a placeholder-only template such as `.env.example`.
- Remember that `.gitignore` does not remove a file already tracked by Git.
- Do not assume deleting a leaked secret from the latest commit removes it from history, forks, logs, or clones.
- Use repository and CI secret scanning as a detection layer, not as permission to commit secrets.

If a secret reaches a repository, treat it as exposed: revoke or rotate it immediately, inspect its use, and then clean repository history if appropriate. Rewriting history alone does not invalidate a credential.

## Avoid accidental disclosure

Never put a secret in browser-delivered HTML, JavaScript, CSS, source maps, public API responses, or error pages. Anything sent to a visitor can be copied. Keep private API credentials on the server and authorize each operation that uses them.

Do not print secrets in application logs, exception messages, command-line arguments, URLs, analytics events, crash reports, or support bundles. Avoid dumping `$_ENV`, `$_SERVER`, configuration arrays, or `phpinfo()` output in production. Configure production errors to be logged to a protected destination without displaying stack traces or internal paths to visitors.

Logs should record useful events without credentials or full secret-bearing payloads. Redact sensitive fields before logging, limit access and retention, and ensure that observability tools and backups follow the same access policy.

## Passwords are handled differently

A password used to authenticate a person should normally be stored as a one-way password hash, not as a recoverable application secret. In PHP, use `password_hash()` to create the stored hash and `password_verify()` to check a submitted password. Do not encrypt user passwords for later display, and do not use a general-purpose fast hash such as `md5()` or `sha1()` for password storage.

Application credentials such as a database password are different: the application must recover them to connect, so they need controlled secret storage and access rather than password hashing.

## Rotation and incident response

Secrets have a lifecycle: create, distribute, use, rotate, revoke, and remove. Have a plan for changing a credential without unnecessarily interrupting service. Where possible, support overlapping old and new credentials during a controlled rotation, verify the new value works, and revoke the old value promptly.

When exposure is suspected:

1. Revoke or rotate the affected credential first.
2. Check audit logs and access records for suspicious use.
3. Identify every service, environment, build, and backup that received the value.
4. Update deployment configuration and restart or reload affected processes as required.
5. Remove the value from repository history and other retained artifacts where appropriate.
6. Review how it was exposed and add prevention or detection controls.

Do not wait for repository cleanup before revoking a live credential.

## Production configuration checklist

- Load production configuration from deployment-controlled settings or a managed secret system.
- Keep real local and production values out of tracked files and build artifacts.
- Use unique credentials for each environment and service.
- Grant each application identity the minimum permissions it needs.
- Fail startup when required secrets are missing; do not fall back to example or development credentials.
- Validate security-sensitive flags and force debug output off in production.
- Do not expose secrets to the browser, logs, error pages, command lines, or diagnostics.
- Restrict secret access, monitor it, and rotate or revoke values on a defined schedule or after exposure.
- Use password hashing APIs for human passwords instead of treating them as retrievable service secrets.
- Test that deployment and incident procedures work without printing credential values.

## Key takeaways

- Keep values that grant access out of source code and version control.
- Treat environment variables as a delivery mechanism that still needs access controls.
- Load configuration at startup, validate it, and fail safely when required values are missing.
- Use separate least-privileged credentials for every environment.
- Never send server secrets to the browser or write them to logs.
- Rotate exposed credentials immediately; deleting a commit does not revoke a secret.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [131 - Command injection and unsafe deserialization](131-command-injection-and-unsafe-deserialization.md) | [Q&A](../qnas/132-managing-secrets-and-production-configuration.md) | [Example](../examples/132-managing-secrets-and-production-configuration.md) | [133 - Dependency security and software supply chains](133-dependency-security-and-software-supply-chains.md) |