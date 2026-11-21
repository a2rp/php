# 106 - Environment configuration and secrets: Example

## Goal

Load database settings from the PHP process environment, validate them, connect with PDO, and report success without printing credentials.

## Question

How can a PHP application require its database settings, reject invalid debug or port values, keep a password out of source and output, and work when CLI and PHP-FPM receive different environments?

## Assumptions

- PHP 8.0 or newer with PDO MySQL enabled.
- A local MySQL database and a least-privilege application account already exist.
- The process environment is configured before the script starts.
- The password used in local practice is not a production credential.
- The example reads process environment values with getenv(). It does not parse .env files.

## Project layout

~~~text
config-demo/
├── .env.example
├── .gitignore
└── database-check.php
~~~

## 1. Document the required settings

Create .env.example:

~~~text
APP_ENV=development
APP_DEBUG=false
DB_HOST=127.0.0.1
DB_PORT=3306
DB_NAME=php_config_demo
DB_USER=php_app
DB_PASSWORD=
~~~

This file documents variable names and safe example values. It is not itself loaded by PHP and contains no working password.

Create or update .gitignore:

~~~gitignore
.env
.env.*
!.env.example
~~~

This ignores real local .env files while allowing the safe example file to be committed. If a secret file was already tracked, this rule alone does not remove it from Git history; revoke or rotate any exposed credential.

## 2. Create the PHP configuration check

Create database-check.php:

~~~php
<?php

declare(strict_types=1);

function requiredEnvironmentValue(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException(
            $name . ' is required but is not configured.'
        );
    }

    return $value;
}

function loadConfiguration(): array
{
    $environment = requiredEnvironmentValue('APP_ENV');
    $allowedEnvironments = [
        'development',
        'test',
        'staging',
        'production',
    ];

    if (!in_array($environment, $allowedEnvironments, true)) {
        throw new RuntimeException(
            'APP_ENV must name a supported environment.'
        );
    }

    $debugText = requiredEnvironmentValue('APP_DEBUG');
    $debug = filter_var(
        $debugText,
        FILTER_VALIDATE_BOOLEAN,
        FILTER_NULL_ON_FAILURE
    );

    if ($debug === null) {
        throw new RuntimeException(
            'APP_DEBUG must be a recognized Boolean value.'
        );
    }

    if ($environment === 'production' && $debug) {
        throw new RuntimeException(
            'APP_DEBUG must be false in production.'
        );
    }

    $host = requiredEnvironmentValue('DB_HOST');

    if (!preg_match('/\A[a-zA-Z0-9.-]+\z/', $host)) {
        throw new RuntimeException(
            'DB_HOST contains unsupported characters.'
        );
    }

    $portText = requiredEnvironmentValue('DB_PORT');
    $port = filter_var($portText, FILTER_VALIDATE_INT);

    if ($port === false || $port < 1 || $port > 65535) {
        throw new RuntimeException(
            'DB_PORT must be an integer from 1 through 65535.'
        );
    }

    $database = requiredEnvironmentValue('DB_NAME');

    if (!preg_match('/\A[a-zA-Z0-9_]+\z/', $database)) {
        throw new RuntimeException(
            'DB_NAME contains unsupported characters.'
        );
    }

    $username = requiredEnvironmentValue('DB_USER');
    $password = requiredEnvironmentValue('DB_PASSWORD');

    return [
        'environment' => $environment,
        'debug' => $debug,
        'host' => $host,
        'port' => $port,
        'database' => $database,
        'username' => $username,
        'password' => $password,
    ];
}

function connectToDatabase(array $config): PDO
{
    $dsn = 'mysql:host=' . $config['host']
        . ';port=' . $config['port']
        . ';dbname=' . $config['database']
        . ';charset=utf8mb4';

    return new PDO(
        $dsn,
        $config['username'],
        $config['password'],
        [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
            PDO::ATTR_EMULATE_PREPARES => false,
        ]
    );
}

try {
    $config = loadConfiguration();
    $pdo = connectToDatabase($config);
    $result = $pdo->query('SELECT 1')->fetchColumn();

    if ((string) $result !== '1') {
        throw new RuntimeException(
            'The database health query returned an unexpected result.'
        );
    }

    fwrite(
        STDOUT,
        'Configuration loaded for '
        . $config['environment']
        . '.' . PHP_EOL
    );
    fwrite(STDOUT, 'Debug mode: ' . ($config['debug'] ? 'on' : 'off') . '.' . PHP_EOL);
    fwrite(STDOUT, "Database connection: ready.\n");
} catch (PDOException $exception) {
    error_log(
        'Database connection failed. SQLSTATE: '
        . (string) $exception->getCode()
    );

    fwrite(
        STDERR,
        "Database connection failed. Check the service configuration and protected logs.\n"
    );

    exit(1);
} catch (RuntimeException $exception) {
    fwrite(STDERR, 'Configuration check failed: ' . $exception->getMessage() . PHP_EOL);
    exit(1);
} catch (Throwable $exception) {
    error_log(
        'Unexpected startup failure. Exception type: '
        . get_class($exception)
    );

    fwrite(STDERR, "Application startup failed.\n");
    exit(1);
}
~~~

## 3. Supply local settings in PowerShell

First set the non-secret settings for a local database. The PHP process must also be able to reach this database, and the user must have permission to connect to it.

~~~powershell
$env:APP_ENV = 'development'
$env:APP_DEBUG = 'false'
$env:DB_HOST = '127.0.0.1'
$env:DB_PORT = '3306'
$env:DB_NAME = 'php_config_demo'
$env:DB_USER = 'php_app'
~~~

For a local practice run, prompt for the database password rather than writing it in a command:

~~~powershell
$securePassword = Read-Host -Prompt 'Local demo database password' -AsSecureString
$env:DB_PASSWORD = [System.Net.NetworkCredential]::new('', $securePassword).Password
php database-check.php
Remove-Item Env:DB_PASSWORD
~~~

The value is converted to a normal process environment string so PHP can read it. This is only a local demonstration; environment variables are not a vault. Avoid reusing a production credential, and use the platform's protected secret injection for deployed services.

Expected output when the local database settings are valid:

~~~text
Configuration loaded for development.
Debug mode: off.
Database connection: ready.
~~~

The output reports no password, username, DSN, or full configuration array.

## 4. Configure a service environment

The same variables must be provided to the process that handles the real work:

- For a CLI task, configure the environment for the scheduler or job runner.
- For a web request using PHP-FPM, configure the required values through the deployment mechanism or the FPM pool.
- For a container or managed platform, use its supported configuration and secret facilities.
- After changing service-level values, reload or restart the process as required and verify it safely.

PHP-FPM's clear_env pool setting defaults to yes. If a value works in a terminal but not in a web request, check how the FPM worker receives it. Explicitly pass only required values; do not broadly expose an entire host environment to every pool.

The .env.example file is documentation for developers. Since this PHP script does not load .env, the settings must be injected into the process or loaded by a separately configured dependency.

## 5. Review failure behavior

If DB_PASSWORD is missing, the script prints an error naming the missing setting and returns status 1. It does not substitute a default password.

If APP_DEBUG is set to text that PHP cannot interpret as a Boolean, startup fails. The script also refuses debug mode in production.

If the database connection fails, a short SQLSTATE code is written to the protected server log and a generic message goes to STDERR. The password and full DSN are not printed.

In production, make sure logs are access-controlled and that the runtime does not expose stack traces or phpinfo() output to visitors.

## How it works

1. getenv() reads one required value at a time.
2. Missing and empty settings fail immediately.
3. APP_ENV is checked against an allowlist.
4. APP_DEBUG is parsed as a Boolean instead of relying on PHP string truthiness.
5. Production debug mode is rejected.
6. The port is converted to an integer and range-checked.
7. Host and database names are constrained before being used in the DSN.
8. Username and password are passed as PDO constructor parameters.
9. A simple SELECT 1 confirms the connection.
10. Output reports the environment and connection status without revealing credentials.
11. Database errors are logged with a limited code, and the command returns a nonzero status.

## Important notes

- Keep the database account least-privileged and separate from administrative credentials.
- The host validation shown is intentionally limited to DNS-style hostnames and IPv4 text; extend it deliberately if the deployment needs another valid host format.
- A secret in an environment variable may still be accessible to processes or diagnostics with sufficient access. Environment configuration is not a substitute for host permissions or a dedicated secret manager.
- Do not put a real password in .env.example, a shell command, a screenshot, source control, or a log.
- If a real secret was committed, rotate or revoke it. Removing a file or adding an ignore rule does not undo exposure.
- This example source is displayed for study and was not run against a database here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [105 - Command-line applications and arguments](../lessons/105-command-line-applications-and-arguments.md) | [Q&A](../qnas/106-environment-configuration-and-secrets.md) | [Lesson](../lessons/106-environment-configuration-and-secrets.md) | [107 - Composer and dependency management](../lessons/107-composer-and-dependency-management.md) |
