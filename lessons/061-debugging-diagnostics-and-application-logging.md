# 061 - Debugging, Diagnostics, and Application Logging

Debugging is a repeatable way to find the cause of a problem. Application logging records important events so that failures can be understood after they happen. PHP diagnostics, exception traces, temporary inspection, and application logs each provide different evidence.

## Learning goals

By the end of this chapter, you should be able to:

- Follow a repeatable process for reproducing and narrowing down a bug.
- Inspect PHP values and types during local development.
- Use command-line tools to check syntax and runtime configuration.
- Record useful application events with structured context.
- Log exceptions without exposing internal details to visitors.
- Avoid leaking secrets or personal data into logs and debug output.

## A repeatable debugging process

When something fails, use a small, evidence-driven loop:

1. **Reproduce the issue.** Record the input, action, environment, and expected behavior.
2. **Read the complete diagnostic.** Note the message, source file, line, and any exception trace.
3. **Confirm the runtime.** Check the PHP version, SAPI, loaded configuration, and required extensions.
4. **Narrow the failing area.** Reduce the case while preserving the failure.
5. **Inspect the relevant values.** Check both type and value immediately before the failing operation.
6. **Form one hypothesis.** Change one condition at a time and observe what changes.
7. **Fix the cause.** Avoid hiding a diagnostic without understanding the failure.
8. **Repeat the original scenario.** Confirm that the failure is resolved and nearby behavior still makes sense.
9. **Remove temporary debug output.** Keep only intentional, safe logs.

A small reproducible case is usually easier to reason about than a large application trace. Keep a note of the original input so the issue can be reproduced after a change.

## Read diagnostics and traces carefully

A PHP diagnostic usually identifies a severity, message, file, and line. A thrown `Throwable` can include a stack trace showing the path through functions and files that led to the failure.

Start with the first meaningful failure. Later messages can be consequences of an earlier problem. For example, a missing configuration value may cause a warning and then a database connection failure.

A trace can contain source paths, function arguments, and object details. Treat it as internal diagnostic data. Do not return it directly to a visitor.

## Temporary inspection during local development

### var_dump()

`var_dump()` prints the type and value of a variable, recursively for arrays and objects:

~~~php
var_dump($user, $requestData);
~~~

Use it only in a development context. In a web application, direct output may alter the response, expose sensitive values, or corrupt JSON. Remove temporary dumps once the issue is understood.

### print_r() and var_export()

`print_r()` is a readable representation, while `var_export()` produces a parsable representation for many PHP values:

~~~php
print_r($settings);
var_export($settings);
~~~

These are inspection tools, not production logging policies. Both can expose secrets if used on a complete request or configuration object.

### Breakpoints and IDE debugging

An IDE debugger can pause execution at a breakpoint so you can inspect a selected variable, evaluate an expression, and step through calls. This can be more precise than adding output statements. Debugger extensions and settings should be restricted to local or controlled development environments; do not expose an unauthenticated debugger on a public server.

## Command-line checks

Use the same PHP installation and extensions that the application uses whenever possible.

~~~sh
php -v
php --ini
php -m
php -l path/to/file.php
~~~

- `php -v` displays the CLI PHP version.
- `php --ini` shows the CLI configuration files.
- `php -m` lists extensions loaded by the CLI runtime.
- `php -l` checks a PHP file for syntax errors without executing it.

The CLI configuration can differ from PHP-FPM, Apache, a container, or a hosting provider. If a problem happens only on the server, compare the server runtime and configuration instead of assuming that the local CLI matches it.

### Inspecting configuration

For a focused value, inspect a directive without dumping the whole environment:

~~~php
var_dump(ini_get('error_reporting'));
var_dump(ini_get('log_errors'));
var_dump(ini_get('error_log'));
~~~

Avoid publishing a page that calls `phpinfo()`. It can reveal environment variables, request details, filesystem paths, extensions, and configuration. If you temporarily need it for local investigation, remove or protect it immediately afterward.

## PHP diagnostics and application events

PHP's diagnostic log records runtime messages according to the active configuration. Application logs record events that matter to the program, such as:

- A job started, completed, or failed.
- A payment provider timed out.
- A user action was rejected by a business rule.
- A retry was scheduled.
- A request exceeded an expected duration.

A useful application log entry answers: **what happened, when, and which operation was involved?** Include enough context to correlate related events, but not so much that the record exposes secrets or sensitive personal data.

## Write structured log records

A structured record has named fields rather than an unlabelled sentence. JSON is a common format because log tools can parse its fields:

~~~php
$record = [
    'time' => gmdate(DATE_ATOM),
    'level' => 'warning',
    'event' => 'profile.lookup.missing',
    'request_id' => $requestId,
    'user_id' => $userId,
];

$json = json_encode(
    $record,
    JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
);

error_log($json);
~~~

By default, `error_log()` sends a message to PHP's configured system logger, file, or SAPI logging handler. Configure the destination for the environment. Do not assume the same destination across local development, containers, and production.

Many applications use a logging service or library rather than calling `error_log()` throughout business logic. A shared logging layer makes it easier to set formats, destinations, context, redaction, and retention consistently.

### Include useful context

Context can make a log actionable:

- A stable event name.
- A severity or level.
- A request or job identifier.
- A relevant internal object identifier.
- A safe reason or exception class.
- A timestamp, usually in a consistent timezone such as UTC.

A request identifier can be generated at the application boundary:

~~~php
$requestId = bin2hex(random_bytes(16));
~~~

Generate it once for the request and pass it to the services that need to log. If a client supplies an identifier, validate its format and length before using it. Never let arbitrary request data become an unbounded log message.

### Avoid duplicate logging

Log an exception at the boundary that has enough context to decide what the record should say. If every layer logs and rethrows the same exception, one failure can create many nearly identical entries.

When a layer can recover, handle the exception there. When it cannot, let it propagate to a boundary that records the failure once and returns an appropriate response.

## Protect logs and debug data

Logs often contain information that is not intended for users. Apply these practices:

- Do not log passwords, access tokens, session identifiers, private keys, full payment details, or raw authorization headers.
- Avoid logging full request bodies by default; they can contain credentials and personal data.
- Redact or omit sensitive fields before recording context.
- Sanitize untrusted values so newline characters cannot create forged log entries.
- Restrict access to log files and logging services.
- Use rotation, retention, and deletion policies that fit the data and operational needs.
- Keep detailed exception messages and traces out of public responses.
- Avoid logging the same failure repeatedly at every call layer.

When a user needs a reference to report an issue, show a request ID or incident code instead of the internal trace.

## Exception logging and safe responses

A web boundary can log a failure with internal context, then return a generic response:

~~~php
try {
    $result = processOrder($order);
} catch (Throwable $exception) {
    error_log(sprintf(
        'order.process.failed request_id=%s exception=%s file=%s line=%d',
        $requestId,
        $exception::class,
        $exception->getFile(),
        $exception->getLine()
    ));

    http_response_code(500);
    echo 'The order could not be processed. Reference: '
        . htmlspecialchars($requestId, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
~~~

This is an illustrative pattern, not a complete web framework. A real application should use its established response and logging layers. Be deliberate about logging exception messages: they may contain user input, SQL details, paths, or secrets. If logged, sanitize and redact them according to the application's policy.

Catch `Throwable` only at a boundary that can provide a safe final response. Catch narrower exception types closer to the code when the application can recover or respond differently.

## Last-error inspection

`error_get_last()` returns information about the most recent PHP diagnostic, or `null` if there is none:

~~~php
$lastError = error_get_last();

if ($lastError !== null) {
    error_log(sprintf(
        'Last PHP diagnostic type=%d file=%s line=%d',
        $lastError['type'],
        $lastError['file'],
        $lastError['line']
    ));
}
~~~

This is useful for a limited shutdown-reporting pattern, but it is not a general debugging history. It only describes the last diagnostic, and a handled error may not be available there. Inspect the error type before treating it as fatal. A shutdown callback cannot resume a terminated script or repair source that failed before execution.

## Common mistakes

- Reading only the final error message while ignoring the first failure in a sequence.
- Debugging with a different PHP version or configuration than the failing environment.
- Leaving `var_dump()`, `phpinfo()`, or detailed traces on a public page.
- Logging entire request objects, headers, or configuration arrays.
- Putting untrusted newlines into a log message without sanitizing them.
- Recording the same exception at every layer as it propagates.
- Returning a raw exception message or stack trace to the user.
- Treating `error_get_last()` as a complete diagnostic history.
- Hiding or suppressing an error before identifying its cause.

## Key takeaways

- Reproduce a failure, read the first meaningful diagnostic, and narrow it down with evidence.
- Use `var_dump()`, breakpoints, and CLI checks only in a controlled development environment.
- CLI and web PHP may use different versions, extensions, and configuration files.
- Application logs should use useful event names and enough safe context to correlate failures.
- Keep secrets and unnecessary personal data out of logs.
- Log once at a boundary that has the right context, then show users a safe message and reference ID.
- `error_get_last()` is a limited last-error lookup, not a replacement for structured logging.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [060 - Error reporting, handlers, and assertions](060-error-reporting-handlers-and-assertions.md) | [Open Q&A](../qnas/061-debugging-diagnostics-and-application-logging.md) | [Open example](../examples/061-debugging-diagnostics-and-application-logging.md) | [062 - Classes, objects, and object-oriented PHP](062-classes-objects-and-object-oriented-php.md) |
