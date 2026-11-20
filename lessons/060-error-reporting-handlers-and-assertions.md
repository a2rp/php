# 060 - Error Reporting, Handlers, and Assertions

PHP provides several controls for deciding which diagnostics are reported, where they are sent, and how selected failures are handled. It also provides assertions for checking assumptions inside a program. These tools have different purposes: error reporting is operational configuration, handlers are custom response hooks, and assertions are checks for internal expectations.

## Learning goals

By the end of this chapter, you should be able to:

- Set an appropriate diagnostic reporting level with named constants.
- Explain the difference between reporting, displaying, and logging an error.
- Register and restore a custom diagnostic error handler safely.
- Distinguish an error handler from an uncaught-exception handler.
- Use assertions for internal expectations without depending on them for application behavior.
- Account for assertion configuration changes introduced in PHP 8.3.

## Error reporting is a bitmask

The `error_reporting()` function controls which diagnostic levels PHP reports during the current request or script. Its value is a bitmask made from error-level constants.

For development, a common setting is:

~~~php
error_reporting(E_ALL);
~~~

Use named constants instead of numeric values. PHP may add error levels over time, so hard-coded numbers are difficult to maintain.

To omit one level from a mask, use bitwise AND with the bitwise NOT operator:

~~~php
error_reporting(E_ALL & ~E_DEPRECATED);
~~~

This demonstrates how masks work, but hiding deprecations is usually a poor long-term fix. Deprecation messages are signals that code may need updating before a future PHP version removes or changes behavior.

Calling `error_reporting()` without an argument returns the active mask. Calling it with a new mask changes the setting for the rest of the current execution unless it is changed again:

~~~php
$previousLevel = error_reporting(E_ALL);

// Application code runs with E_ALL reporting.

error_reporting($previousLevel);
~~~

The function returns the previous reporting level when it changes the mask.

## Reporting, displaying, and logging are separate

Three settings answer different questions:

| Setting | Question it answers | Typical use |
| --- | --- | --- |
| `error_reporting` | Which diagnostic levels are enabled for reporting? | Use `E_ALL` while developing |
| `display_errors` | Should PHP show diagnostic details in the response? | On locally; off for public production responses |
| `log_errors` | Should PHP send diagnostics to an error log? | On where logs are configured and protected |
| `error_log` | Where should PHP send logged messages? | Set by PHP or hosting configuration |

A production system can report all useful diagnostic levels while keeping `display_errors` off and `log_errors` on. Visitors should not see filesystem paths, SQL details, stack traces, secrets, or internal configuration.

For local experiments, runtime settings may look like this:

~~~php
error_reporting(E_ALL);
ini_set('display_errors', '1');
ini_set('log_errors', '1');
~~~

These calls are not a substitute for correct server configuration. Some settings cannot be changed at runtime, a host may restrict changes, and errors that occur before the script starts cannot be displayed by code that never ran. Configure production behavior in the PHP, web-server, container, or hosting configuration used by the deployment.

The command-line SAPI can help you inspect configuration:

~~~sh
php --ini
php -i
~~~

The configuration used by a command-line process may differ from the configuration used by PHP-FPM or another web server SAPI. Check the environment that actually runs the application.

## Custom diagnostic error handlers

Use `set_error_handler()` to register a callback for selected diagnostic levels. The callback receives the severity, message, file, and line. PHP keeps the previous handler on a stack; `restore_error_handler()` restores it.

~~~php
$previousHandler = set_error_handler(
    static function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        if ((error_reporting() & $severity) === 0) {
            return false;
        }

        error_log(sprintf(
            'PHP diagnostic [%d] %s in %s:%d',
            $severity,
            $message,
            $file,
            $line
        ));

        return true;
    },
    E_WARNING | E_USER_WARNING
);

try {
    // Code whose selected warnings should use this handler.
} finally {
    restore_error_handler();
}
~~~

The second argument to `set_error_handler()` is a mask deciding which diagnostic levels invoke the callback. This callback mask is distinct from `error_reporting()`: PHP can call a registered handler even when the level is excluded from the reporting mask. Check the current mask inside the callback if suppressed diagnostics should be ignored.

Return `false` when PHP's normal error handler should continue handling that diagnostic. Return `true` when the custom handler has handled it. Do not print raw internal error details to a public response. Logging should use a protected destination, and any user-controlled values included in logs should be sanitized to prevent forged log entries.

The `try/finally` block restores the previous handler whether the protected code completes or throws. Avoid registering a process-wide handler deep inside a helper and forgetting to restore it.

### What a diagnostic handler cannot handle

A user-defined error handler does not receive every PHP problem. It cannot handle several engine-level failures, including parse and compile errors that prevent the relevant code from running. The exact error categories have changed across PHP versions.

A diagnostic handler also does not replace `try/catch` for thrown objects. A `TypeError` or `InvalidArgumentException` is a `Throwable`, not an `E_WARNING` diagnostic.

## Uncaught-exception handlers are different

`set_exception_handler()` registers a callback for a `Throwable` that escapes all matching `try/catch` blocks. It is a last-resort boundary for an uncaught exception; it does not resume normal execution after the exception.

~~~php
set_exception_handler(
    static function (Throwable $exception): void {
        error_log(sprintf(
            'Uncaught %s: %s',
            $exception::class,
            $exception->getMessage()
        ));

        // A web application should send a generic response from its
        // framework or front controller, not reveal the exception details.
    }
);
~~~

Use ordinary `try/catch` where the application can recover or choose a useful response. Keep an uncaught-exception handler focused on final reporting and a safe response. Avoid assuming it will catch PHP diagnostics.

For an application, the framework's front controller or established logging layer is often the best place to register this boundary. A handler should not expose secrets or detailed traces to visitors.

## Shutdown functions and fatal-error inspection

`register_shutdown_function()` schedules a callback for the end of script execution. `error_get_last()` can return information about the most recent diagnostic. Some applications use these functions to record selected fatal errors that prevented ordinary handling.

This is a limited last-resort technique, not a replacement for proper logging or exception handling:

- Shutdown callbacks cannot repair code that failed to parse before execution.
- `error_get_last()` describes the most recent error, which may not be the failure you expected.
- Do not treat every last error as fatal; inspect its type.
- Keep shutdown reporting minimal and protect log contents.
- A shutdown callback cannot make a terminated request continue normally.

For routine application failures, use explicit exceptions and a normal error-reporting path.

## Assertions check internal expectations

An assertion states that a condition should be true at a particular point in the program. If it is false while assertions are active, PHP reports an assertion failure; in current PHP the default failure behavior is an `AssertionError`.

~~~php
$total = array_sum([4, 6, 10]);

assert($total === 20, 'The calculated total should equal 20.');
~~~

Assertions are useful for documenting and checking invariants during development, testing, or in codebases that intentionally use them. They are not a substitute for validation or normal error handling.

Do not put required side effects inside an assertion:

~~~php
// Incorrect: the function call may be skipped when assertions are disabled.
assert(saveRecord($record));
~~~

Call the operation explicitly, then handle its result:

~~~php
$saved = saveRecord($record);

if ($saved === false) {
    throw new RuntimeException('The record could not be saved.');
}

assert($saved === true, 'A successful save should return true.');
~~~

Likewise, validate user input with normal conditions and exceptions. An assertion might be disabled in a deployment, so it cannot be the only protection for a request, payment, permission check, or database write.

## The zend.assertions setting

The `zend.assertions` configuration controls whether assertion code is generated and executed:

| Value | Behavior |
| --- | --- |
| `1` | Generate and execute assertion code |
| `0` | Generate assertion code but skip it at runtime |
| `-1` | Do not generate assertion code |

When PHP starts in production mode with `zend.assertions=-1`, assertion code is omitted and cannot be enabled later at runtime. Configure the setting in the PHP startup configuration used by the process.

As of PHP 8.3, the `assert.*` INI settings, including `assert.active`, `assert.exception`, `assert.warning`, `assert.bail`, and `assert.callback`, are deprecated. Prefer the supported `zend.assertions` behavior and do not build new code around the deprecated settings.

Since PHP 8.0, a string passed as the assertion expression is not evaluated as PHP code. Pass a boolean expression instead:

~~~php
// Correct
assert($count >= 0);

// Do not pass PHP source as a string
// assert('$count >= 0');
~~~

## Choosing the right tool

| Situation | Prefer |
| --- | --- |
| A user submits invalid input | Validate it and return a clear response or throw a suitable exception |
| A recoverable operation fails | Check its result and handle the failure explicitly |
| A warning needs centralized recording | A narrowly scoped diagnostic handler |
| A thrown failure reaches the application boundary | A global uncaught-exception handler |
| An internal invariant should hold during development | An assertion |
| A public production request fails | Protected logs plus a generic response |

## Common mistakes

- Treating `error_reporting` as if it controls whether details are displayed.
- Turning off all reporting in production instead of logging problems privately.
- Assuming `set_error_handler()` catches parse errors, every fatal error, or thrown exceptions.
- Forgetting that the handler's severity mask is separate from `error_reporting()`.
- Returning `false` from a handler after it already logged or transformed the diagnostic, causing duplicate handling.
- Registering a custom handler and not restoring the previous one.
- Using assertions for input validation, authorization, or any work required for correctness.
- Putting function calls or state changes inside an assertion expression.
- Depending on deprecated `assert.*` settings on PHP 8.3 or later.
- Showing stack traces, source paths, credentials, or SQL details to visitors.

## Key takeaways

- `error_reporting`, `display_errors`, and `log_errors` control different parts of error handling.
- Use `E_ALL` during development, and configure production to log details without displaying them publicly.
- A custom diagnostic handler handles only selected diagnostic levels; it is not an exception handler.
- Return `false` to defer a diagnostic to PHP's standard handler, and restore custom handlers when their scope ends.
- Use an uncaught-exception handler only as a final boundary for unhandled `Throwable` objects.
- Use assertions for internal expectations, never for required runtime validation or side effects.
- PHP 8.3 deprecated the `assert.*` INI settings; understand `zend.assertions` for the PHP process you deploy.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [059 - PHP errors, exceptions, and error categories](059-php-errors-exceptions-and-error-categories.md) | [Open Q&A](../qnas/060-error-reporting-handlers-and-assertions.md) | [Open example](../examples/060-error-reporting-handlers-and-assertions.md) | [061 - Debugging, diagnostics, and application logging](061-debugging-diagnostics-and-application-logging.md) |
