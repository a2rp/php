# 110 - PHP-FIG standards and PSR recommendations: Example

## Goal

Build a small PHP application service that depends on the PSR-3 logger interface, then provide Monolog as the concrete implementation from the application's bootstrap.

## Question

How can business code use a shared logging contract without constructing or depending directly on one logging implementation?

## Assumptions

- PHP 8.1 or newer and Composer are available.
- Commands are run from the project root.
- The application uses PSR-3 and Monolog 3.
- This example demonstrates one interface boundary; it does not require every project to use every PSR.

## 1. Create the project manifest

Create a directory named `psr-logger-demo` and add this `composer.json`:

~~~json
{
    "name": "example/psr-logger-demo",
    "type": "project",
    "description": "A small application using the PSR-3 logger interface",
    "require": {
        "php": "^8.1",
        "psr/log": "^3.0",
        "monolog/monolog": "^3.0"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
~~~

The application directly requires `psr/log` because its source code uses `LoggerInterface`. Monolog is the selected implementation. The `App\` prefix maps application classes to `src/`.

Create `.gitignore`:

~~~gitignore
/vendor/
~~~

## 2. Write a service against the PSR interface

Create `src/Order/OrderProcessor.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Order;

use Psr\Log\LoggerInterface;

final class OrderProcessor
{
    public function __construct(
        private LoggerInterface $logger,
    ) {
    }

    public function process(string $orderId): void
    {
        $this->logger->info(
            'Order processed',
            ['order_id' => $orderId],
        );
    }
}
~~~

`OrderProcessor` knows the logger contract and how to report an order. It does not know whether a logger writes to a file, a monitoring service, or another destination.

## 3. Configure a concrete logger in the application

Create `app.php` in the project root:

~~~php
<?php

declare(strict_types=1);

use App\Order\OrderProcessor;
use Monolog\Handler\StreamHandler;
use Monolog\Level;
use Monolog\Logger;

require __DIR__ . '/vendor/autoload.php';

$logger = new Logger('orders');
$logger->pushHandler(
    new StreamHandler('php://stderr', Level::Info),
);

$processor = new OrderProcessor($logger);
$processor->process('ORD-1042');
~~~

Monolog implements the PSR-3 logger interface. The bootstrap creates and configures it, then passes it into the service. If the application later selects another compatible implementation, `OrderProcessor` can keep the same constructor and logging call.

## 4. Install and inspect the packages

From the project root, run:

~~~sh
composer install
composer validate
composer show --direct
composer check-platform-reqs
php app.php
~~~

Because this new project has no lock file, Composer resolves package versions that satisfy the manifest, writes `composer.lock`, installs packages under `vendor/`, and generates the autoloader. Review and commit `composer.json` and `composer.lock`; keep generated `vendor/` ignored in this workflow.

## Expected output

Monolog writes to standard error. It includes a timestamp, so that part changes on each run:

~~~text
[timestamp] orders.INFO: Order processed {"order_id":"ORD-1042"} []
~~~

The exact formatting can vary by Monolog configuration. The important parts are the `orders` channel, the `INFO` level, the message, and its structured context.

## 5. Swap the implementation without changing the service

For a simple no-output implementation, PSR-Log also provides `NullLogger`. The bootstrap can choose it without modifying `OrderProcessor`:

~~~php
use App\Order\OrderProcessor;
use Psr\Log\NullLogger;

require __DIR__ . '/vendor/autoload.php';

$processor = new OrderProcessor(new NullLogger());
$processor->process('ORD-1043');
~~~

This is useful in a test or a command where logging should intentionally be discarded. In a real application, select a logger that meets the operational needs. The service stays coupled to the interface, while the bootstrap chooses the behavior.

## 6. Relate the example to other PSRs

The same interface-versus-implementation approach can be useful in other parts of an application:

| Need | Possible PSR boundary | What the application still chooses |
| --- | --- | --- |
| Store or retrieve cached data | PSR-6 or PSR-16 | Cache backend, key policy, and expiration policy |
| Process HTTP messages | PSR-7 | Concrete message implementation and framework integration |
| Handle server requests | PSR-15 | Router, middleware stack, and handler behavior |
| Send HTTP requests | PSR-18 | Concrete client, transport, and retry policy |
| Obtain current time | PSR-20 | System clock or a controlled test clock |

Do not implement these interfaces just to make an example look standards-compliant. Add a boundary when it helps with interoperability, substitution, or testing.

## 7. Keep style decisions explicit

Use one documented style guide and apply it consistently. This sample follows familiar PSR-12-style layout conventions for imports, braces, indentation, and method declarations. PHP-FIG's newer PER Coding Style extends PSR-12 for more recent PHP features; a project should choose and enforce the guide that matches its codebase and supported PHP versions.

A code formatter or linter can apply a style guide. Formatting is a separate concern from whether a service depends on an interface.

## Example source-control state

~~~text
psr-logger-demo/
├── .gitignore
├── app.php
├── composer.json
├── composer.lock
├── src/
│   └── Order/
│       └── OrderProcessor.php
└── vendor/                 generated; ignored by Git
~~~

Commit the manifest, lock file, and source code. A fresh checkout can install the locked dependencies with `composer install`.

## What this example demonstrates

1. A service can type-hint a PSR interface rather than a concrete implementation.
2. The application declares `psr/log` directly because its code uses that interface.
3. Monolog supplies the configured logging behavior through the shared contract.
4. The bootstrap selects and configures the implementation.
5. Structured context keeps order data separate from the log message text.
6. A compatible no-output logger can replace the concrete logger without changing the service.
7. Composer's lock file records the selected package versions for repeatable installs.
8. A PSR is useful where it provides a real interoperability or substitution boundary.

## Important notes

- Review the selected implementation's maintenance, license, PHP requirement, and configuration before using it in a real application.
- Never put credentials or sensitive customer data in log context without an explicit data-handling policy.
- The sample writes logs to standard error; production log collection and retention are separate deployment decisions.
- Keep `vendor/` ignored and commit the application lock file.
- The PHP source in this example is provided for study and was not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [109 - Version constraints, lock files, and packages](../lessons/109-version-constraints-lockfiles-and-packages.md) | [Q&A](../qnas/110-php-fig-standards-and-psr-recommendations.md) | [Lesson](../lessons/110-php-fig-standards-and-psr-recommendations.md) | [111 - Project structure, namespaces, and application layers](../lessons/111-project-structure-namespaces-and-application-layers.md) |
