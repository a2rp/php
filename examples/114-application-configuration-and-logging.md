# 114 - Application configuration and logging: Example

## Goal

Create a small PHP command that validates environment configuration at startup, injects a PSR-3 logger into an application service, and writes a structured log record to a private log file.

## Question

How can an application read environment-specific settings once, reject invalid values early, and record useful events without hard-coding a logging vendor into the use case?

## Assumptions

- PHP 8.1 or newer and Composer are available.
- Commands are run from the project root.
- This example uses Monolog 3 and writes logs to `var/log/app.log`.
- The command reads variables supplied by its process; plain PHP does not automatically parse `.env` files.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
config-log-demo/
├── bin/
│   └── demo.php
├── src/
│   ├── Application/
│   │   └── ReceiptProcessor.php
│   ├── Config/
│   │   └── AppConfig.php
│   └── Infrastructure/
│       └── Logging/
│           └── LoggerFactory.php
├── var/
│   └── log/                  created at runtime; ignored by Git
├── .gitignore
└── composer.json
~~~

The configuration object validates the process settings. The logging factory selects and configures Monolog. The application service receives only the logger interface it needs.

## 2. Configure Composer and ignored files

Create `composer.json`:

~~~json
{
    "name": "example/config-log-demo",
    "type": "project",
    "description": "A small PHP example of validated configuration and logging",
    "require": {
        "php": "^8.1",
        "monolog/monolog": "^3.0",
        "psr/log": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
~~~

Create `.gitignore`:

~~~gitignore
/vendor/
/var/log/
.env.local
~~~

From the project root, install the dependency and generate Composer's autoloader:

~~~sh
composer install
~~~

The Composer lock file records the selected dependency versions. Do not commit `vendor/` or generated runtime logs.

## 3. Read and validate settings once

Create `src/Config/AppConfig.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Config;

use InvalidArgumentException;

final class AppConfig
{
    private const ENVIRONMENTS = ['development', 'test', 'production'];

    private const LOG_LEVELS = [
        'emergency',
        'alert',
        'critical',
        'error',
        'warning',
        'notice',
        'info',
        'debug',
    ];

    private function __construct(
        public readonly string $environment,
        public readonly bool $debug,
        public readonly string $logLevel,
    ) {
    }

    public static function fromEnvironment(): self
    {
        $environment = getenv('APP_ENV');
        $environment = $environment === false || $environment === ''
            ? 'production'
            : strtolower($environment);

        if (!in_array($environment, self::ENVIRONMENTS, true)) {
            throw new InvalidArgumentException('APP_ENV is not supported.');
        }

        $rawDebug = getenv('APP_DEBUG');
        $debug = false;

        if ($rawDebug !== false && $rawDebug !== '') {
            $parsedDebug = filter_var(
                $rawDebug,
                FILTER_VALIDATE_BOOLEAN,
                FILTER_NULL_ON_FAILURE,
            );

            if ($parsedDebug === null) {
                throw new InvalidArgumentException('APP_DEBUG must be a boolean value.');
            }

            $debug = $parsedDebug;
        }

        $logLevel = getenv('LOG_LEVEL');
        $logLevel = $logLevel === false || $logLevel === ''
            ? 'warning'
            : strtolower($logLevel);

        if (!in_array($logLevel, self::LOG_LEVELS, true)) {
            throw new InvalidArgumentException('LOG_LEVEL is not a supported PSR-3 level.');
        }

        return new self($environment, $debug, $logLevel);
    }
}
~~~

This class converts environment strings into typed settings. It defaults to production behavior, keeps debug output off, and rejects an unknown environment, boolean, or log level rather than silently guessing.

## 4. Configure a PSR-3 logger implementation

Create `src/Infrastructure/Logging/LoggerFactory.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Infrastructure\Logging;

use App\Config\AppConfig;
use Monolog\Handler\StreamHandler;
use Monolog\Level;
use Monolog\Logger;
use Psr\Log\LoggerInterface;
use RuntimeException;

final class LoggerFactory
{
    public function create(AppConfig $config, string $projectRoot): LoggerInterface
    {
        $logDirectory = $projectRoot . '/var/log';

        if (
            !is_dir($logDirectory)
            && !mkdir($logDirectory, 0770, true)
            && !is_dir($logDirectory)
        ) {
            throw new RuntimeException('The application log directory could not be created.');
        }

        $level = match ($config->logLevel) {
            'debug' => Level::Debug,
            'info' => Level::Info,
            'notice' => Level::Notice,
            'warning' => Level::Warning,
            'error' => Level::Error,
            'critical' => Level::Critical,
            'alert' => Level::Alert,
            'emergency' => Level::Emergency,
        };

        $logger = new Logger('config-log-demo');
        $logger->pushHandler(
            new StreamHandler($logDirectory . '/app.log', $level),
        );

        return $logger;
    }
}
~~~

The factory maps the validated PSR-3 level to Monolog's level enum and writes records to a file outside the public document root. In a real deployment, make sure the process has appropriate directory permissions and configure log rotation or collection.

## 5. Inject the logger into an application service

Create `src/Application/ReceiptProcessor.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Application;

use InvalidArgumentException;
use Psr\Log\LoggerInterface;

final class ReceiptProcessor
{
    public function __construct(
        private readonly LoggerInterface $logger,
    ) {
    }

    public function prepare(string $orderNumber, int $totalInCents): string
    {
        if (trim($orderNumber) === '') {
            throw new InvalidArgumentException('An order number is required.');
        }

        if ($totalInCents < 0) {
            throw new InvalidArgumentException('The order total cannot be negative.');
        }

        $receipt = sprintf(
            'Order %s total: $%d.%02d',
            $orderNumber,
            intdiv($totalInCents, 100),
            $totalInCents % 100,
        );

        $this->logger->info(
            'Receipt prepared',
            [
                'order_number' => $orderNumber,
                'total_cents' => $totalInCents,
            ],
        );

        return $receipt;
    }
}
~~~

The use case depends on the PSR-3 interface, not Monolog's concrete logger. It records a stable event name and structured fields. This example uses a sample order number and total; do not add secrets or unnecessary personal data to log context.

## 6. Build the application at the command entry point

Create `bin/demo.php`:

~~~php
<?php

declare(strict_types=1);

use App\Application\ReceiptProcessor;
use App\Config\AppConfig;
use App\Infrastructure\Logging\LoggerFactory;

$projectRoot = dirname(__DIR__);
require $projectRoot . '/vendor/autoload.php';

$config = AppConfig::fromEnvironment();
$logger = (new LoggerFactory())->create($config, $projectRoot);
$processor = new ReceiptProcessor($logger);
$orderNumber = 'ORD-1001';

try {
    $receipt = $processor->prepare($orderNumber, 3000);
    fwrite(STDOUT, $receipt . PHP_EOL);
} catch (Throwable $exception) {
    $logger->error(
        'Receipt preparation failed',
        [
            'order_number' => $orderNumber,
            'exception' => $exception,
        ],
    );

    $message = $config->debug
        ? $exception->getMessage()
        : 'Receipt preparation failed.';
    fwrite(STDERR, $message . PHP_EOL);
    exit(1);
}
~~~

The command entry point is the composition root: it loads settings, creates the configured logger, and injects that logger into the application service. The catch block records the exception at this boundary and writes a safe message to standard error unless local debug mode is enabled.

A web application must never send stack traces or internal exception messages to a public response, even when it logs them privately.

## 7. Supply environment variables and run

In PowerShell, set variables for the current shell and run the command:

~~~powershell
$env:APP_ENV = 'development'
$env:APP_DEBUG = 'true'
$env:LOG_LEVEL = 'info'
php bin/demo.php
~~~

In a POSIX shell, the equivalent command can be:

~~~sh
APP_ENV=development APP_DEBUG=true LOG_LEVEL=info php bin/demo.php
~~~

These commands set process environment variables directly. They do not require or imply that PHP reads a `.env` file.

## Expected output

The command prints:

~~~text
Order ORD-1001 total: $30.00
~~~

The file `var/log/app.log` contains an INFO entry with the `Receipt prepared` message and context fields for `order_number` and `total_cents`. Its timestamp and exact line formatting depend on the logger formatter and runtime.

If `LOG_LEVEL=error`, the INFO event is below the configured threshold and is not written by this handler. If a setting is malformed, startup fails with a configuration error before the receipt operation runs.

## 8. Try invalid configuration

For example, set an unsupported log level:

~~~powershell
$env:LOG_LEVEL = 'verbose'
php bin/demo.php
~~~

`AppConfig` rejects the value instead of quietly treating it as another level. Fix the setting to one of the supported names before starting the application.

## What this example demonstrates

1. Runtime settings are read once and converted into typed values.
2. Missing settings use only deliberate safe defaults.
3. Invalid environment names, booleans, and log levels fail early.
4. The composition root selects Monolog and its output destination.
5. Application code depends on `LoggerInterface`, not a vendor-specific logger class.
6. Log events use stable messages and structured context.
7. Sensitive values do not need to be copied into logs to identify the sample operation.
8. `var/log/` and Composer's `vendor/` directory are ignored by Git.

## Important notes

- Do not commit real secrets in `.env`, `.env.local`, or another configuration file. This sample uses shell variables and no secrets.
- Plain PHP does not automatically load `.env`; configure a dotenv library if the project intentionally uses one.
- Restrict access to log files and configure rotation, retention, and monitoring in a real deployment.
- Log context should be minimized and reviewed for secrets and personal data.
- Keep debug output disabled in production, and never put exception details in a public HTTP response.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [113 - Dependency injection and service design](../lessons/113-dependency-injection-and-service-design.md) | [Q&A](../qnas/114-application-configuration-and-logging.md) | [Lesson](../lessons/114-application-configuration-and-logging.md) | [115 - Caching data and application output](../lessons/115-caching-data-and-application-output.md) |
