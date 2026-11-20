# 060 - Error Reporting, Handlers, and Assertions: Example

## Question

How do you configure diagnostic reporting for a script, handle one selected warning safely, validate actual input, and use an assertion only for an internal expectation?

## Complete example

This example is for PHP 8.0 and later. Save it as `error-reporting-handlers.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

// Select all diagnostic levels for this run.
// This does not decide whether the visitor sees an error or where it is logged.
$previousReportingLevel = error_reporting(E_ALL);

$selectedLevels = E_WARNING | E_USER_WARNING;

echo 'E_WARNING selected: '
    . (($selectedLevels & E_WARNING) !== 0 ? 'yes' : 'no')
    . PHP_EOL;

// Keep the example output deterministic by recording the selected diagnostic.
// In an application, send the record to a protected logger instead of showing
// diagnostic details to a visitor.
$handledDiagnostics = [];

set_error_handler(
    static function (
        int $severity,
        string $message,
        string $file,
        int $line
    ) use (&$handledDiagnostics): bool {
        if ((error_reporting() & $severity) === 0) {
            return false;
        }

        $handledDiagnostics[] = [
            'severity' => $severity,
            'message' => $message,
            'file' => $file,
            'line' => $line,
        ];

        return true;
    },
    E_USER_WARNING
);

try {
    trigger_error('Optional profile value is missing.', E_USER_WARNING);
} finally {
    restore_error_handler();
}

echo 'Handled diagnostic: '
    . $handledDiagnostics[0]['message']
    . PHP_EOL;

// The expression is a pure internal invariant, not input validation.
// If assertions are disabled, program correctness does not depend on it.
$total = array_sum([12, 18, 30]);
assert($total === 60, 'The calculated total should equal 60.');

echo 'Calculated total: ' . $total . PHP_EOL;

function validatePageSize(mixed $value): int
{
    if (!is_int($value) || $value < 1 || $value > 100) {
        throw new InvalidArgumentException(
            'Page size must be an integer from 1 to 100.'
        );
    }

    return $value;
}

try {
    $pageSize = validatePageSize(25);
    echo 'Validated page size: ' . $pageSize . PHP_EOL;

    validatePageSize(0);
} catch (InvalidArgumentException $exception) {
    echo 'Rejected invalid input: ' . $exception->getMessage() . PHP_EOL;
} finally {
    // error_reporting() returns the level that was active before the change.
    error_reporting($previousReportingLevel);
}
~~~

## Expected output

~~~text
E_WARNING selected: yes
Handled diagnostic: Optional profile value is missing.
Calculated total: 60
Validated page size: 25
Rejected invalid input: Page size must be an integer from 1 to 100.
~~~

## Explanation

### 1. Reporting is separate from display and logging

The call to `error_reporting(E_ALL)` selects diagnostic levels for this execution. It does not turn on public error display and does not choose a log destination.

A local PHP configuration can use settings like these:

~~~ini
error_reporting = E_ALL
display_errors = Off
log_errors = On
~~~

Use configuration for the PHP SAPI that serves the application. Do not expose diagnostic details to visitors; configure a protected log destination for operators.

### 2. The custom handler is narrow and temporary

The handler is registered only for `E_USER_WARNING`. It checks the active `error_reporting()` mask, records the diagnostic for this command-line demonstration, and returns `true` because it handled the selected warning.

The `finally` block restores the previous handler. In a real application, replace the in-memory array with the application's protected logging service. Do not print raw file paths, request data, or exception details into a public response.

If a handler should let PHP's normal handler continue, return `false`. The handler's severity mask and the global reporting mask are separate controls.

### 3. Assertions are not validation

The assertion checks a pure internal calculation. Nothing required for the program to function happens inside the assertion expression. If assertion code is omitted by configuration, the calculation and output still work.

The page-size check uses an ordinary condition and throws `InvalidArgumentException` for invalid input. That validation always runs, regardless of assertion settings.

### 4. Restore settings after a temporary change

`error_reporting()` returns the previous mask when a new one is supplied. Restoring it in `finally` avoids leaving a temporary setting active if the protected code exits by throwing.

A short-lived CLI script can also end without restoration, but scoped restoration is a useful habit in reusable libraries and long-running processes.

## Optional uncaught-exception boundary

Register a global uncaught-exception handler once at an application boundary, such as a framework bootstrap or front controller. It should record details privately and provide only a safe response. This is separate from the diagnostic handler above.

~~~php
set_exception_handler(
    static function (Throwable $exception): void {
        error_log(sprintf(
            'Uncaught %s: %s',
            $exception::class,
            $exception->getMessage()
        ));

        if (PHP_SAPI !== 'cli' && !headers_sent()) {
            http_response_code(500);
            echo 'An unexpected error occurred.';
        }
    }
);
~~~

A global exception handler is a last resort for a `Throwable` that was not caught. It does not resume execution after the failure. Catch an exception closer to the operation when the application can recover or choose a more specific response.

## Try it yourself

1. Add `E_USER_NOTICE` to the handler's accepted-level mask, then trigger that level and observe what changes.
2. Change `validatePageSize(25)` to `validatePageSize(150)` and inspect the caught validation error.
3. Change the total to an incorrect value. If assertions are active, the failed invariant raises an assertion failure; if disabled, the program continues, which is why assertions must not guard required behavior.
4. Temporarily make the custom handler return `false` and observe PHP's standard handling for the warning.
5. Move handler registration into a helper and add a `finally` block there so it cannot leak into later application code.

## Key takeaways

- `error_reporting`, `display_errors`, and `log_errors` answer different questions.
- A custom diagnostic handler handles only the selected diagnostic levels.
- Restore temporary handlers and runtime settings in a `finally` block.
- Use normal conditions and exceptions to validate user input.
- Assertions document internal expectations; their execution can be disabled or omitted.
- Keep diagnostic details in protected logs and return generic public error responses.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [059 - PHP errors, exceptions, and error categories](../lessons/059-php-errors-exceptions-and-error-categories.md) | [Q&A](../qnas/060-error-reporting-handlers-and-assertions.md) | [Lesson](../lessons/060-error-reporting-handlers-and-assertions.md) | [061 - Debugging, diagnostics, and application logging](../lessons/061-debugging-diagnostics-and-application-logging.md) |
