# 132 - Managing secrets and production configuration: Example

## Question

Create a runnable PHP CLI example that reads environment-specific configuration and database credentials, rejects missing or invalid values, and confirms that configuration loaded without printing any secret.

The example must not connect to a real database or contain a real credential.

## Complete example

Save this file as `safe-production-config.php`:

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

try {
    $config = loadApplicationConfig();
} catch (Throwable $exception) {
    // The exception contains a setting name, never the setting value.
    fwrite(STDERR, 'Configuration error: ' . $exception->getMessage() . PHP_EOL);
    exit(1);
}

echo 'Environment: ', $config['environment'], PHP_EOL;
echo 'Debug mode: ', $config['debug'] ? 'enabled' : 'disabled', PHP_EOL;
echo 'Database configuration: loaded; credential values are hidden.', PHP_EOL;
~~~

## Run the local demonstration

The following PowerShell commands use fake, local-only values. The script only validates configuration; it does not connect to a database.

~~~powershell
$env:APP_ENV = 'development'
$env:APP_DEBUG = 'false'
$env:APP_DATABASE_DSN = 'mysql:host=127.0.0.1;dbname=demo;charset=utf8mb4'
$env:APP_DATABASE_USER = 'demo_user'
$env:APP_DATABASE_PASSWORD = 'demo-only-not-a-real-secret'

php .safe-production-config.php
~~~

Expected output:

~~~text
Environment: development
Debug mode: disabled
Database configuration: loaded; credential values are hidden.
~~~

For Bash or a similar POSIX shell, set equivalent demo values in the process environment before running the script:

~~~sh
APP_ENV=development APP_DEBUG=false APP_DATABASE_DSN='mysql:host=127.0.0.1;dbname=demo;charset=utf8mb4' APP_DATABASE_USER='demo_user' APP_DATABASE_PASSWORD='demo-only-not-a-real-secret' php safe-production-config.php
~~~

These commands demonstrate local process configuration only. Do not type production credentials into shared terminals, scripts, shell history, or documentation. Use the deployment platform's protected secret settings or secret manager for real credentials.

## Check safe failure behavior

Set production debug mode to true and run the script again:

~~~powershell
$env:APP_ENV = 'production'
$env:APP_DEBUG = 'true'
php .safe-production-config.php
~~~

Expected error on STDERR:

~~~text
Configuration error: Debug mode must be disabled in production.
~~~

The program exits with a failure status instead of silently enabling production debug output.

To check missing-secret handling, remove the fake password and run the script:

~~~powershell
Remove-Item Env:APP_DATABASE_PASSWORD
php .safe-production-config.php
~~~

Expected error on STDERR:

~~~text
Configuration error: Missing required configuration: APP_DATABASE_PASSWORD
~~~

The diagnostic identifies which setting is missing but does not print the value of any configured credential.

## Explanation

- Configuration is loaded once at startup, then validated before the application proceeds.
- The environment name is restricted to known values.
- Boolean settings accept a small explicit vocabulary; an unexpected value is an error.
- Production debug mode is rejected rather than silently enabled.
- Required database settings do not fall back to sample or default credentials.
- The code stores the password only in server-side memory for the connection code that would need it. The demonstration never prints or logs the password.
- The same loader can be used with values supplied by a protected local development setup, a production deployment system, or a managed secret provider.

A real web application must not send this CLI error text directly to visitors. Return a generic server error to the client, and send a redacted diagnostic to a protected operational log. Restrict access to deployment variables, process diagnostics, crash dumps, and logs.

## Keep local values out of Git

A repository can include this ignore pattern when the project uses a local dotenv file:

~~~gitignore
.env
.env.*
!.env.example
~~~

Commit only a placeholder-only `.env.example`. If a real file is already tracked, the ignore rule does not untrack it; remove it from Git's index and rotate any credential it contained.

## Do not copy these patterns

Do not hardcode a production credential in PHP:

~~~php
$databasePassword = 'real-production-password';
~~~

Do not expose runtime configuration in an error page:

~~~php
echo json_encode($_ENV);
~~~

Do not put a private service token in a browser asset:

~~~javascript
const apiToken = 'private-server-token';
~~~

Use deployment-controlled configuration, keep credentials on the server, and show only safe status messages to users.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [131 - Command injection and unsafe deserialization](../lessons/131-command-injection-and-unsafe-deserialization.md) | [Lesson](../lessons/132-managing-secrets-and-production-configuration.md) | [Q&A](../qnas/132-managing-secrets-and-production-configuration.md) | [133 - Dependency security and software supply chains](../lessons/133-dependency-security-and-software-supply-chains.md) |