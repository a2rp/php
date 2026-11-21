# 106 - Environment configuration and secrets

## Goals

After this lesson, you should be able to:

- Separate application configuration from PHP source code.
- Read environment values in PHP and handle missing values explicitly.
- Validate configuration values before connecting to services.
- Keep credentials and other secrets out of source control, output, and logs.
- Understand why the CLI and web runtimes may receive different environment values.
- Configure local development and deployment environments deliberately.
- Rotate a secret without making old credentials valid forever.

## Configuration and secrets

An application needs values that differ between environments. Examples include:

- Database host, port, database name, and user.
- Public application URL.
- Runtime environment name such as development, test, or production.
- Feature toggles and resource limits.
- Database passwords, API tokens, encryption keys, and signing keys.

Configuration describes how an application should run. A secret is configuration that grants access or protects data. A database host is usually not secret; the database password is.

Keep these values outside application source code. The same code can then run locally, in tests, and in production with different runtime configuration.

Do not treat every configuration value as secret. Protect secret values carefully, and keep ordinary settings visible to the people who need to operate the application.

## The Twelve-Factor-style environment model

A common deployment pattern passes configuration to the application through its process environment. A process manager, container runtime, hosting platform, or secret manager supplies the values when starting the PHP process.

The PHP code then reads the names it needs:

~~~php
$databaseHost = getenv('DB_HOST');
$databaseName = getenv('DB_NAME');
~~~

Environment variables are a delivery mechanism, not a vault by themselves. A secret in an environment variable can still be exposed through process inspection, debugging output, crash reports, child processes, or misconfigured logs. Restrict who can configure and inspect the runtime, and use a dedicated secret-management facility when the platform provides one.

Avoid putting credentials in source code, command-line arguments, repository history, screenshots, support tickets, or deployment logs.

## Reading an environment value with getenv()

getenv() returns the requested string or false when the variable is unavailable.

~~~php
$databaseHost = getenv('DB_HOST');

if ($databaseHost === false || $databaseHost === '') {
    throw new RuntimeException('DB_HOST is required.');
}
~~~

Check the value explicitly. Do not use a truthiness check when values such as '0' may be valid, and do not silently substitute a production default for a missing credential.

A small helper can keep required-value handling consistent:

~~~php
function requiredEnvironmentValue(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException(
            'Required application configuration is missing.'
        );
    }

    return $value;
}
~~~

The message intentionally does not include the value. Never include a password or token in an exception message.

Do not trim every value automatically. Whitespace may be meaningful in a secret. Validate a setting according to its domain: trim a host name or environment label if that is the defined policy, but preserve the exact bytes of a password or key unless the secret format says otherwise.

## getenv(), $_ENV, and $_SERVER

PHP offers several ways to access environment and server data:

- getenv('NAME') reads one environment value.
- $_ENV is the environment superglobal, if populated under the runtime's configuration.
- $_SERVER contains server and execution information and may contain values supplied by a SAPI or web server.

Do not assume $_ENV is populated in every environment. Its availability depends on PHP configuration, including variables_order. A value may be available through getenv() even when the $_ENV array is empty.

Use one clear source for each setting. Mixing getenv(), $_ENV, $_SERVER, a local file, and hard-coded fallbacks creates ambiguous precedence. If multiple sources are deliberately supported, document the order and test it.

A safe lookup does not enumerate and print every environment variable. The environment may contain unrelated credentials supplied to the process.

## Validate and convert values

Environment values arrive as strings. Convert and validate them before passing them to an API.

### Boolean settings

Do not assume that the string 'false' is false in PHP. Nonempty strings are generally truthy. Parse an explicit set of accepted values:

~~~php
$debugValue = requiredEnvironmentValue('APP_DEBUG');

$debug = filter_var(
    $debugValue,
    FILTER_VALIDATE_BOOLEAN,
    FILTER_NULL_ON_FAILURE
);

if ($debug === null) {
    throw new RuntimeException(
        'APP_DEBUG must be true or false.'
    );
}
~~~

This accepts recognized Boolean spellings and rejects unknown text. Define which spellings are allowed by your application.

### Integer settings

Validate both the integer syntax and the permitted range:

~~~php
$portValue = requiredEnvironmentValue('DB_PORT');
$port = filter_var($portValue, FILTER_VALIDATE_INT);

if ($port === false || $port < 1 || $port > 65535) {
    throw new RuntimeException('DB_PORT is invalid.');
}
~~~

The valid range depends on the setting. A timeout, page size, port, and retry count each have different constraints.

### URLs and names

Validate a URL with the rule the application requires, and validate identifiers such as a database name against a documented allowlist. Do not build a connection string from arbitrary user input.

Successful parsing does not mean a value is operationally correct. A host can be syntactically valid but unreachable; a key can be present but expired. Check connectivity through the application’s normal startup or health process without disclosing the secret.

## Fail fast and avoid unsafe defaults

Required configuration should be checked before the application accepts work. Failing during startup is easier to diagnose than returning partial responses or failing later in a database query.

Do not silently choose a credential, signing key, encryption key, or production endpoint when a required value is missing.

A default can be useful for a non-sensitive development setting when it is explicit and safe. For example, a local feature flag may default to false. Never use a fake or shared default secret in production.

A boot-time configuration object can hold validated values in one place. Keep access centralized so the application does not repeatedly reinterpret raw environment strings.

## Local development and .env files

PHP does not automatically parse a file named .env. A library or framework may load such a file, but that behavior belongs to that dependency, not to the PHP language itself.

A common local-development arrangement is:

- Commit a .env.example containing variable names and safe placeholders.
- Create a real local .env file with private values.
- Add the real .env file to .gitignore before creating it.
- Load it with a documented development-only tool or configure variables in the local runtime.
- Do not copy local secret values into production.

Example .env.example:

~~~text
APP_ENV=development
APP_DEBUG=false
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=php_app
DB_USER=php_app
DB_PASSWORD=
~~~

The blank password is a placeholder, not a working credential. Never commit a real .env file. Check the staged file list and repository history when investigating accidental secret exposure.

A .gitignore rule only affects future untracked files. If a secret file was already committed, adding it to .gitignore does not remove it from Git history or revoke the credential.

## Runtime environments and process managers

Configuration must reach the PHP process that handles the work.

CLI PHP and a web-server SAPI may run as different users, use different executables, load different php.ini files, and inherit different environments. A value exported in an interactive shell is not necessarily available to a service that was started earlier or by another manager.

PHP-FPM adds a specific detail: its pool setting clear_env defaults to yes. FPM clears the worker environment before adding variables explicitly configured for the pool. A variable visible in a terminal may therefore be absent from a web request.

For PHP-FPM, configure only the required values through the deployment mechanism or the pool's explicit environment configuration. Do not disable environment clearing broadly just to make a missing variable appear. Review access to pool configuration because it can contain secrets.

After changing service configuration, reload or restart the relevant service as required by that runtime. Verify configuration using a safe diagnostic that reports only whether required values are present, never their contents.

## Secret handling practices

### Limit access

Give each service its own credentials and the least privilege it needs. A PHP application's database account should not normally be a database administrator.

Restrict who can read deployment settings, secret-manager entries, local environment files, backups, and diagnostic logs.

### Never disclose a secret

Do not:

- echo or var_dump the full configuration array;
- print a password in an exception or error page;
- include a token in a log line, URL, metric label, or support message;
- expose phpinfo() publicly;
- put credentials in command-line arguments or committed scripts;
- include real values in example files or screenshots.

When debugging, report the setting name and whether it is configured. Do not print the value.

### Rotate and revoke

A secret should have a process for rotation:

1. Create or issue a replacement credential.
2. Configure the application to use the replacement.
3. Reload processes and verify the new credential works.
4. Revoke the old credential.
5. Review logs and access records if exposure is suspected.

For systems requiring zero downtime, plan a temporary overlap where both credentials work. Remove the old credential promptly after all processes have moved.

### Respond to exposure

If a credential was committed or disclosed, assume it may have been copied. Remove it from active use and rotate or revoke it. Removing the visible file does not undo a leaked credential or erase all repository history copies.

## Error handling and safe diagnostics

Give operators enough information to identify the missing setting or failed dependency without exposing its value.

Good diagnostic:

~~~text
Application startup failed: DB_HOST is not configured.
~~~

Unsafe diagnostic:

~~~text
Database connection failed with password: example-secret
~~~

For a database failure, log a carefully reviewed error code or sanitized context. Avoid serializing the entire configuration or blindly logging exception data that may contain sensitive connection details.

The public response should remain generic. Detailed diagnostics belong in an access-controlled log with an appropriate retention policy.

## Configuration lifecycle

Configuration has a lifecycle separate from source code:

- Developers use local values that cannot access production data.
- Tests use disposable or isolated services and credentials.
- CI jobs receive only the credentials required for that job.
- Staging uses its own endpoints and secrets.
- Production uses production-specific credentials with limited access.
- Rotation and revocation are part of operations, not a source-code edit.

Never reuse production credentials for tests or examples. Avoid copying production data into local development unless it has been properly sanitized and approved.

## Common mistakes

### Assuming $_ENV always contains the environment

The E entry in variables_order affects whether PHP populates $_ENV. Use a documented access method and verify the actual runtime.

### Assuming shell exports reach PHP-FPM

FPM workers may have a cleared environment. Configure required pool values or inject them through the service's supported deployment mechanism.

### Treating .env as native PHP behavior

PHP does not load .env files by itself. A dependency or framework must parse the file, and its loading rules must be understood.

### Committing .env.example with real credentials

Example files are usually tracked. Put only safe placeholders in them.

### Using a default production password

A default credential is not a safe recovery mechanism. Fail startup when required secrets are missing.

### Treating 'false' as a Boolean false

The string 'false' is nonempty and can be truthy. Parse Boolean values explicitly and reject unrecognized text.

### Printing secrets while debugging

Configuration dumps, phpinfo(), stack traces, and logs can leak credentials. Log presence and safe metadata, not secret values.

### Believing .gitignore removes a committed secret

Ignore rules do not erase existing Git history. Revoke or rotate a disclosed credential and clean repository history when appropriate.

### Sharing one credential across unrelated services

A shared credential increases the impact of compromise and makes rotation disruptive. Use separate credentials with limited privileges.

## Key takeaways

- Keep runtime configuration outside source code and read required values explicitly.
- PHP environment values are strings; validate and convert them before use.
- getenv() can work when $_ENV is empty because of runtime configuration.
- CLI and web PHP may receive different environments; PHP-FPM clears worker environments by default.
- PHP does not parse .env files without a library or framework.
- Commit only safe placeholders; keep real local and production secrets out of Git.
- Environment variables help deliver settings but are not a secret vault by themselves.
- Fail fast when required settings are missing; do not use silent production defaults.
- Use least-privilege credentials, safe diagnostics, and a practiced rotation process.
- If a secret is exposed, revoke or rotate it; deleting a file alone is not enough.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [105 - Command-line applications and arguments](105-command-line-applications-and-arguments.md) | [Q&A](../qnas/106-environment-configuration-and-secrets.md) | [Example](../examples/106-environment-configuration-and-secrets.md) | [107 - Composer and dependency management](107-composer-and-dependency-management.md) |
