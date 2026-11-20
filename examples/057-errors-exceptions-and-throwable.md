# 057 - Errors, Exceptions, and Throwable: Example

## Question

How can a PHP application validate input with a specific exception, preserve a lower-level cause, run cleanup on success and failure, and convert a selected warning into an exception?

## Complete example

Save this as a PHP file and run it with PHP 8.0 or later. The warning conversion is scoped to one diagnostic level and the previous handler is restored afterward.

~~~php
<?php

declare(strict_types=1);

error_reporting(E_ALL);

final class QuantityException extends InvalidArgumentException
{
}

final class SettingsLoadException extends RuntimeException
{
}

function requirePositiveQuantity(int $quantity): int
{
    if ($quantity <= 0) {
        throw new QuantityException(
            'Quantity must be greater than zero.'
        );
    }

    return $quantity;
}

function runQuantityCheck(int $input): void
{
    try {
        $quantity = requirePositiveQuantity($input);
        echo 'Accepted quantity: ' . $quantity . "\n";
    } catch (QuantityException $exception) {
        echo 'Rejected quantity: ' . $exception->getMessage() . "\n";
    } finally {
        // This runs after a success or a handled exception.
        echo "Quantity check cleanup completed.\n";
    }
}

function loadSettings(): array
{
    try {
        // Simulate a lower-level storage failure.
        throw new RuntimeException('The settings store is unavailable.');
    } catch (RuntimeException $exception) {
        throw new SettingsLoadException(
            'The application settings could not be loaded.',
            0,
            $exception
        );
    }
}

runQuantityCheck(3);
runQuantityCheck(-2);

try {
    loadSettings();
} catch (SettingsLoadException $exception) {
    echo 'Settings failure: ' . $exception->getMessage() . "\n";
    echo 'Original cause preserved: '
        . ($exception->getPrevious() instanceof RuntimeException ? 'yes' : 'no')
        . "\n";
}

// Convert only E_USER_WARNING into ErrorException in this local scope.
set_error_handler(
    static function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        // A custom handler may run even when @ suppresses a diagnostic.
        // Respect the current error_reporting mask.
        if ((error_reporting() & $severity) === 0) {
            return false;
        }

        throw new ErrorException(
            $message,
            0,
            $severity,
            $file,
            $line
        );
    },
    E_USER_WARNING
);

try {
    trigger_error(
        'Optional profile value is missing.',
        E_USER_WARNING
    );
} catch (ErrorException $exception) {
    echo 'Converted diagnostic: ' . $exception->getMessage() . "\n";
} finally {
    // Restore the handler that was active before this example section.
    restore_error_handler();
}
~~~

## Expected output

~~~text
Accepted quantity: 3
Quantity check cleanup completed.
Rejected quantity: Quantity must be greater than zero.
Quantity check cleanup completed.
Settings failure: The application settings could not be loaded.
Original cause preserved: yes
Converted diagnostic: Optional profile value is missing.
~~~

## How it works

- QuantityException extends InvalidArgumentException to give callers a specific way to handle invalid input.
- runQuantityCheck() catches only the exception it can handle. Its finally block runs after both the valid and invalid cases.
- loadSettings() translates a lower-level RuntimeException into SettingsLoadException and passes the original as the previous exception. The caller can inspect that cause with getPrevious().
- set_error_handler() is limited to E_USER_WARNING. The callback uses the provided severity, message, file, and line to create an ErrorException.
- The reporting-mask check respects suppression state. Returning false would delegate a diagnostic to PHP's standard handler if this callback did not convert it.
- restore_error_handler() runs in finally so the prior handler is restored even if the warning conversion or catch logic changes.
- error_reporting(E_ALL) is useful in development. For production web applications, configure PHP to log errors privately and avoid displaying internal diagnostics to users.

## Application-boundary pattern

A top-level uncaught exception handler is a last-resort logging and response boundary. Register it during application bootstrap, before handling requests:

~~~php
<?php

set_exception_handler(
    static function (Throwable $throwable): void {
        // Keep detailed context in a protected server-side log.
        error_log((string) $throwable);

        http_response_code(500);
        echo 'An unexpected error occurred.';
    }
);
~~~

This callback does not resume the code that failed. It should not reveal a stack trace, file path, query, or secret in a public response.

## Try it yourself

1. Run the quantity check with zero and with a positive integer.
2. Remove the catch block and observe that an uncaught Throwable reaches the application boundary.
3. Change the lower-level failure to another RuntimeException and confirm the previous cause is still retained.
4. Change the custom handler mask to E_WARNING and use an operation that emits a warning; verify it handles only the configured level.
5. Remove the reporting-mask check and consider what happens if the expression is suppressed with @.
6. Move restore_error_handler() out of finally and explain why an exception could leave the custom handler installed for later code.

## Common mistakes

- Catching Throwable everywhere and continuing with invalid or partial state.
- Catching a broad parent before a specific exception type.
- Losing the original exception when translating a lower-level failure.
- Returning from finally and replacing a result that was already in flight.
- Installing a process-wide error handler and never restoring the previous handler.
- Converting every diagnostic to an exception without considering the APIs and recovery policy.
- Assuming an error handler catches parse errors and every fatal condition.
- Displaying exception details or stack traces to website visitors.
- Treating a shutdown callback or uncaught handler as a way to resume a failed operation.

## Key takeaways

- Use specific exception types for failures the caller can handle.
- Catch at the layer that can recover or provide a safe response.
- Use finally to restore handlers and release resources.
- Preserve lower-level causes when translating exceptions.
- Convert diagnostics selectively and restore the prior error handler.
- Log internal details privately and show users a generic failure message.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [056 - Serialization and safe data exchange](056-serialization-and-safe-data-exchange.md) | [Open Q&A](../qnas/057-errors-exceptions-and-throwable.md) | [Read lesson](../lessons/057-errors-exceptions-and-throwable.md) | [058 - Namespaces and autoloading](058-namespaces-and-autoloading.md) |
