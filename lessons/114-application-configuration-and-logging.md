# 114 - Application configuration and logging

## Goals

After this lesson, you should be able to:

- Separate application configuration from business logic.
- Load deployment-specific values from the process environment or a configuration system.
- Validate and convert string configuration values deliberately.
- Keep credentials and other secrets out of source control and logs.
- Use log levels and structured context to record useful application events.
- Inject a logger into the code that needs it instead of using global output.
- Configure log destinations, access, and retention for the environment.
- Distinguish diagnostic logs from user responses and formal audit records.

## Configuration describes the running environment

Configuration contains values that can vary without changing the application's business rules. Examples include:

- The active environment, such as development, test, or production.
- Database connection details and API endpoints.
- Request timeouts and retry limits.
- Feature switches and allowed origins.
- The minimum log level and log destination.

Keep configuration separate from application behavior. A service should receive the few values it needs; it should not read the whole process environment or parse a configuration file every time it performs an action.

### Common configuration sources

A project may combine several sources:

1. Safe defaults that make sense for local development.
2. Checked-in configuration files for non-sensitive shared settings.
3. Process environment variables supplied by the shell, web server, container, or deployment platform.
4. A secret-management system for credentials and sensitive keys.

The exact precedence is application- and framework-specific. Document which source wins. A common approach is for deployment-provided values to override safe defaults. Do not silently let a checked-in development value replace a production secret.

A plain PHP process does not automatically parse a `.env` file. A framework or dotenv package may load one. Without such a loader, use environment variables supplied by the runtime, or explicitly add and configure a loader.

A checked-in `.env.example` can document required variable names and harmless sample values. A local `.env`, `.env.local`, or deployment file containing actual credentials must not be committed. Add machine-specific files and generated secret files to `.gitignore` as appropriate.

## Read and validate configuration at the boundary

Environment variables are strings. A missing variable can return `false` from `getenv()`. Converting values by a loose cast can turn a typo into a misleading default. Read values once during application startup, validate them, and create a typed configuration object.

For a required value, fail early with a clear configuration error:

~~~php
<?php

declare(strict_types=1);

function requiredEnvironmentValue(string $name): string
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        throw new RuntimeException("Required configuration {$name} is missing.");
    }

    return $value;
}
~~~

For a boolean, accept recognized values and reject an unrecognized spelling instead of treating every non-empty string as true:

~~~php
<?php

declare(strict_types=1);

function environmentBoolean(string $name, bool $default): bool
{
    $value = getenv($name);

    if ($value === false || $value === '') {
        return $default;
    }

    $parsed = filter_var(
        $value,
        FILTER_VALIDATE_BOOLEAN,
        FILTER_NULL_ON_FAILURE,
    );

    if ($parsed === null) {
        throw new RuntimeException("Configuration {$name} must be a boolean value.");
    }

    return $parsed;
}
~~~

Use the same idea for integers, URLs, enumerated values, and durations: parse into the expected type, validate allowed ranges or choices, and report a startup error when a required value is invalid. Set safe defaults only where absence is genuinely supported.

### Use a configuration object

A configuration object can collect validated settings and make them available to the relevant parts of the application. Keep it focused; avoid turning it into a global bag that every class can inspect.

~~~php
<?php

declare(strict_types=1);

final class ApplicationSettings
{
    public function __construct(
        public readonly string $environment,
        public readonly bool $debug,
        public readonly string $logLevel,
    ) {
    }
}
~~~

Build the object once at startup. Pass only the needed setting to a service when that makes its dependency clearer. Injecting the entire settings object everywhere can be as opaque as reading globals if classes can access unrelated values they do not need.

### Keep secrets private

Treat database passwords, API keys, signing keys, session secrets, and private certificates as secrets. Do not place real secret values in:

- Source code, example configuration, or a public repository.
- Error messages, screenshots, issue reports, or command history that may be shared.
- Log messages, structured log context, HTTP responses, or debug pages.

Supply secrets through the deployment platform or an approved secret store, restrict who can read them, and rotate a value if it may have been exposed. Do not dump all environment variables or `phpinfo()` output in a public application; those can reveal sensitive values.

## What application logs are for

Logs are records that help operators and developers understand events and diagnose failures after the fact. They should provide enough context to answer questions such as:

- Which operation failed?
- When did it happen and at what severity?
- Which request or job was involved?
- Which safe resource identifier can help find related records?
- What exception class or failure category occurred?

Logs are not a substitute for a clear response to the caller. A client should receive an appropriate status and a safe message; the log can hold protected diagnostic context for an operator.

## Use severity levels consistently

PSR-3 describes eight common levels, from most severe to most detailed:

| Level | Typical use |
| --- | --- |
| `emergency` | The system is unusable |
| `alert` | Immediate action is required |
| `critical` | A critical component or condition has failed |
| `error` | An operation failed and needs investigation |
| `warning` | An unusual condition or recoverable problem occurred |
| `notice` | A normal but significant event occurred |
| `info` | A useful operational event occurred |
| `debug` | Detailed diagnostic information, often disabled in production |

Use levels consistently so operators can filter and alert on the events that need attention. A log level is not a replacement for a clear event name or useful context. Avoid marking every ordinary action as an error.

## Prefer structured messages and context

Use a stable message and put variable data in a context array. This keeps events easier to search, format, and process:

~~~php
$logger->info(
    'Order receipt prepared for {order_number}',
    ['order_number' => $order->number()],
);
~~~

PSR-3 permits brace-delimited placeholders whose names correspond to keys in the context array. A logger implementation may interpolate those placeholders. Context should use stable, descriptive keys and values that are safe to retain.

For exceptions, pass the exception object under the `exception` context key when the logger supports PSR-3. This gives a logger implementation a standard way to format exception details:

~~~php
try {
    $receiptService->prepare($orderNumber);
} catch (Throwable $exception) {
    $logger->error(
        'Receipt preparation failed for {order_number}',
        [
            'order_number' => $orderNumber,
            'exception' => $exception,
        ],
    );

    throw $exception;
}
~~~

Log an exception at a boundary that can add useful operation context. Avoid logging the same exception at every layer as it propagates; duplicate entries make failures harder to diagnose.

## Select a logger and destination

Application code can type-hint a logger interface such as PSR-3's `LoggerInterface`. A concrete implementation decides how to format, filter, and send records. A logging library may provide handlers for files, the PHP error log, syslog, or an external logging service.

Choose a destination that matches the runtime:

- A local command may write to standard error or a local file.
- A web process or container may write to standard error for collection by the hosting platform.
- A managed environment may forward records to a centralized logging service.

Do not assume a log directory is writable in every deployment. Ensure the runtime has the correct permissions and that logs are rotated or retained according to operational and privacy requirements. Avoid creating an unbounded log file in a public web directory.

A small script can use PHP's `error_log()` directly. A larger application often benefits from an injected PSR-3 logger so code does not depend on one destination or vendor-specific API.

## Protect privacy in logs

Log only data needed for operations. User-submitted values may contain secrets, personal information, control characters, or content crafted to confuse a log viewer. Avoid logging passwords, authentication headers, cookies, access tokens, complete request bodies, payment details, or database connection strings.

Prefer a safe record identifier and a request or job identifier over copying an entire object into context. If sensitive information is required for a specific operational need, apply explicit access controls, minimization, masking, and retention policies.

Logs can contain untrusted strings. Use structured logging and a logger's formatter rather than assembling multiline output by concatenating raw request data. Treat log storage as sensitive operational data.

## Debug behavior belongs to the environment

Detailed error output helps during local development but can reveal internal paths, SQL, configuration, stack traces, and secrets. Keep debug output disabled in production. Return safe error messages to web clients and put appropriate diagnostics in protected logs.

Debug settings and log levels are related but different:

- Debug mode changes how much diagnostic behavior the application enables.
- Log level controls which severities a logger sends to its configured handlers.

Do not use a production `debug` log level as a substitute for fixing a failing configuration, and do not leave verbose diagnostics enabled indefinitely.

## Logs are not automatically an audit trail

Operational logs help diagnose and monitor software. A regulated or security-sensitive audit trail may need stronger guarantees about completeness, integrity, access, retention, and who performed an action. Do not assume that an ordinary application log satisfies those requirements. Design an audit record explicitly when the product or policy requires one.

## Common mistakes

### Reading environment variables throughout the application

Repeated reads make configuration behavior inconsistent and hide a service's requirements. Load and validate settings at startup, then pass what each component needs.

### Casting configuration without validation

A malformed timeout or feature flag can silently become an unintended value. Parse it deliberately and reject invalid input early.

### Treating a `.env` file as built-in PHP behavior

PHP does not automatically parse `.env` files. Use the framework or dotenv loader configured for the project, or provide values through the runtime environment.

### Committing real credentials

Example files should contain placeholders only. Keep local secrets ignored and use a secret store or deployment configuration for real values.

### Logging secrets to diagnose a failure

Logs may be retained and sent to systems with broad operator access. Record a safe identifier and failure category rather than credentials or complete request data.

### Logging everything at `error`

This makes alerts noisy and hides real failures. Select levels according to the event's meaning.

### Logging and rethrowing at every layer

This creates duplicate stack traces for a single failure. Add context and log at an appropriate boundary, then let centralized error handling produce the client response.

### Writing debug details into an HTTP response

A public response is visible to the caller and may be cached or copied. Keep detailed diagnostics in protected logs and return a safe error response.

### Ignoring log rotation and access

A log file can fill storage or retain sensitive data indefinitely. Set permissions, rotation, retention, and monitoring according to the deployment's operational needs.

## A practical configuration and logging checklist

1. Identify which values vary between local, test, and production environments.
2. Keep safe defaults separate from required deployment values and secrets.
3. Read settings once during startup and convert them to validated types.
4. Fail early when a required value is missing or malformed.
5. Pass only the relevant settings to each service.
6. Inject a logger through a PSR-3 interface or the framework's equivalent.
7. Choose a destination and minimum severity appropriate for the runtime.
8. Record stable event names with safe structured context.
9. Avoid secrets and unnecessary personal data in logs.
10. Configure permissions, collection, retention, and rotation.
11. Keep debug output disabled in production.
12. Return safe responses and log unexpected failures once at a useful boundary.

## Key takeaways

- Configuration changes how the application runs; it should not be scattered through business logic.
- Environment variables are strings, and `getenv()` returns `false` when a named value is unavailable.
- Parse and validate values once at startup, then expose typed settings.
- A plain PHP process does not automatically load a `.env` file.
- Never commit real credentials or write them to logs or public error responses.
- Use consistent log levels and structured context through an injected logger.
- Pass exceptions in the PSR-3 `exception` context key when appropriate.
- Protect log destinations with suitable access, retention, and rotation.
- Diagnostic logs and formal audit trails have different requirements.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [113 - Dependency injection and service design](113-dependency-injection-and-service-design.md) | [Q&A](../qnas/114-application-configuration-and-logging.md) | [Example](../examples/114-application-configuration-and-logging.md) | [115 - Caching data and application output](115-caching-data-and-application-output.md) |
