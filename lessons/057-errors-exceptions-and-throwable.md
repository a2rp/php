# 057 - Errors, Exceptions, and Throwable

Errors are part of development and production operation. PHP reports diagnostic errors such as warnings and notices, and it can throw objects when an operation fails. Good handling makes failures visible to maintainers, gives users safe feedback, and lets the program recover only when recovery is valid.

## Two kinds of failure signals

PHP has two related but distinct mechanisms:

1. **Diagnostic errors** have severity levels such as E_WARNING, E_NOTICE, and E_DEPRECATED. PHP can display or log them, and an application can install a custom handler for many of them.
2. **Throwable objects** are thrown with the throw keyword and handled with try/catch. This group includes both application exceptions and engine errors.

The central hierarchy is:

~~~text
Throwable (interface)
├── Exception (class)
│   ├── RuntimeException
│   ├── InvalidArgumentException
│   └── application-specific exceptions
└── Error (class)
    ├── TypeError
    ├── ValueError
    ├── ParseError
    └── other engine errors
~~~

The built-in Exception and Error branches both implement Throwable. Application-defined exceptions normally extend Exception or one of its subclasses; user classes cannot implement Throwable directly.

A diagnostic warning is not automatically an exception. For example, a failed file operation may emit a warning and return false. The program must inspect the return value or deliberately convert the warning to an ErrorException.

## Report errors during development

Use E_ALL while developing so notices, deprecations, warnings, and other reported diagnostics are not silently missed:

~~~php
<?php

error_reporting(E_ALL);
~~~

Configure runtime behavior deliberately:

- In development, display diagnostics so problems are easy to find.
- In production web applications, keep display_errors disabled so paths, queries, configuration, and stack details are not sent to visitors.
- Enable log_errors and send diagnostics to a protected log destination.
- Keep logs access-controlled, rotate them, and avoid recording passwords, tokens, or unnecessary personal data.
- Configure startup diagnostics in the PHP configuration. Setting display_errors at runtime cannot show a fatal error that prevented the script from reaching that setting.

error_reporting() selects which diagnostic levels are reported. It does not turn a warning into a Throwable. The E_ALL constant is preferable to hard-coded numeric masks.

## Throw and catch exceptions

Throw an exception when a function cannot complete its documented task and the caller can make a meaningful decision:

~~~php
<?php

function requirePositiveQuantity(int $quantity): int
{
    if ($quantity <= 0) {
        throw new InvalidArgumentException(
            'Quantity must be greater than zero.'
        );
    }

    return $quantity;
}

try {
    $quantity = requirePositiveQuantity(0);
    echo "Quantity: $quantity\n";
} catch (InvalidArgumentException $exception) {
    echo 'The quantity is invalid. Please enter a positive number.';
}
~~~

When throw runs, the rest of the current path in the try block is skipped. PHP searches for the first matching catch block. If none matches in the current function, the Throwable bubbles up the call stack until a matching catch is found or the request reaches the global boundary.

Catch specific types that the current layer knows how to handle. Put more specific classes before their parents:

~~~php
try {
    performOperation();
} catch (InvalidArgumentException $exception) {
    // Report invalid user input in a useful way.
} catch (RuntimeException $exception) {
    // Handle an expected operational failure.
}
~~~

Catching a parent first would also catch its child types, making later child catches unreachable. As of PHP 7.1, a single catch can name unrelated types separated by a pipe when they share the same recovery behavior. As of PHP 8.0, the caught variable may be omitted if the block does not need the object.

Do not catch every Throwable just to hide the failure. Catch where the code can recover, translate the error into a useful domain-level result, or add context and rethrow. Catching Throwable can also catch programming errors such as TypeError. It is useful at a deliberate application boundary for logging and safe shutdown, but it should not make a broken operation appear successful.

## Finally blocks and cleanup

A finally block runs after the try and catch path while the stack is unwinding, whether the operation succeeds or a Throwable is caught, rethrown, or left uncaught:

~~~php
<?php

$handle = fopen('input.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the input file.');
}

try {
    processStream($handle);
} finally {
    fclose($handle);
}
~~~

Use finally for cleanup that must happen on both success and failure, such as closing a resource or releasing a lock. A finally block does not run if the process terminates abruptly, for example through exit() or a fatal condition that stops execution. Avoid returning or throwing from finally because it can replace the result or failure already in flight.

## Define useful application exceptions

A custom exception can express a failure that callers need to distinguish:

~~~php
<?php

final class ConfigurationException extends RuntimeException
{
}

function loadRequiredConfiguration(?string $value): string
{
    if ($value === null || $value === '') {
        throw new ConfigurationException(
            'A required application setting is missing.'
        );
    }

    return $value;
}
~~~

Choose a standard exception class when its meaning fits. Use custom classes for stable domain distinctions, not for every individual message.

When translating a lower-level exception, preserve it as the previous exception:

~~~php
<?php

try {
    $value = readConfigurationFile();
} catch (RuntimeException $exception) {
    throw new ConfigurationException(
        'The application configuration could not be loaded.',
        0,
        $exception
    );
}
~~~

The original exception remains available through getPrevious() for protected logs and debugging. Avoid sending raw exception messages or stack traces to visitors.

## Engine errors and the Throwable interface

PHP engine errors include problems such as TypeError, ValueError, and ParseError. They can be caught because they implement Throwable, but they often indicate a programming defect or invalid call rather than an expected business condition.

Use a catch for Error only when there is a deliberate recovery policy. Do not use broad catches to continue with partially valid state. Correct the underlying defect when recovery would conceal a broken invariant.

The throw keyword requires an object implementing Throwable. In modern PHP, throw is an expression and can appear in expression contexts, but an ordinary if statement is often clearer for beginners.

## Convert selected diagnostics to exceptions

A custom error handler can turn selected diagnostics into ErrorException objects. This helps code that prefers exception-based control flow, but should be applied narrowly and predictably:

~~~php
<?php

set_error_handler(
    function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        // Since PHP 8, a custom handler can run even for a diagnostic
        // suppressed by @. Respect the current reporting mask.
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
    E_WARNING | E_USER_WARNING
);

try {
    $contents = file_get_contents('missing.txt');

    if ($contents === false) {
        throw new RuntimeException('The file could not be read.');
    }
} catch (ErrorException $exception) {
    error_log($exception->getMessage());
    echo 'The requested file is not available.';
} finally {
    restore_error_handler();
}
~~~

The handler checks the reporting mask because a custom handler may still be called when the @ operator suppresses a diagnostic. Returning false delegates the diagnostic to PHP's standard handler. The example converts only warning levels; it does not install a handler for every possible diagnostic.

set_error_handler() cannot handle every error. In particular, E_ERROR, E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, and E_COMPILE_WARNING cannot be handled by a user-defined error handler. Some errors also occur before the script has registered its handler. Do not rely on a custom handler as a replacement for fixing code or configuring logs.

Avoid using @ as normal error handling. It hides useful signals and can make a failed operation look like a valid result. Prefer checking documented return values or converting a selected diagnostic with a deliberate handler.

## Handle uncaught exceptions at the application boundary

set_exception_handler() registers a final handler for uncaught Throwable objects:

~~~php
<?php

set_exception_handler(
    function (Throwable $throwable): void {
        error_log((string) $throwable);

        http_response_code(500);
        echo 'An unexpected error occurred.';
    }
);
~~~

This is a last-resort boundary, not a way to resume the interrupted operation. It should log enough protected context for maintainers and return only a generic response to users. Do not expose stack traces, file paths, SQL, or secrets in a public response.

## Fatal diagnostics at shutdown

A shutdown callback can inspect error_get_last() for some fatal diagnostics that stopped execution:

~~~php
<?php

register_shutdown_function(
    function (): void {
        $lastError = error_get_last();

        if ($lastError === null) {
            return;
        }

        $fatalTypes = [
            E_ERROR,
            E_PARSE,
            E_CORE_ERROR,
            E_COMPILE_ERROR,
            E_USER_ERROR,
            E_RECOVERABLE_ERROR,
        ];

        if (in_array($lastError['type'], $fatalTypes, true)) {
            error_log(
                $lastError['message']
                . ' in '
                . $lastError['file']
                . ':'
                . $lastError['line']
            );
        }
    }
);
~~~

This is supplementary diagnostics, not ordinary recovery. It cannot handle every startup or compile failure, and the application cannot safely continue after many fatal errors. Avoid printing internal fatal details to the user.

## Choose a response that matches the failure

| Situation | Typical response |
| --- | --- |
| Invalid user input | Validate near the boundary and return a clear field-level message |
| Expected missing record | Return an explicit not-found result or throw a domain exception, according to the API contract |
| Network or storage unavailable | Retry only when safe, otherwise report an operational failure and log context |
| Programming defect such as TypeError | Fix the defect; let the request fail safely and record it |
| Diagnostic warning from a legacy API | Check its return value or convert a selected warning deliberately |
| Uncaught failure at the web boundary | Log privately and send a generic error response |

Do not use exceptions for ordinary branching such as checking whether a value exists when a simple conditional is clearer. Exceptions are for failures that interrupt the normal operation and need to travel to a layer that can handle them.

## Common mistakes

- Treating every warning or notice as an Exception automatically.
- Catching Exception when the code must also handle every Throwable, or catching Throwable without a recovery policy.
- Catching a parent class before its more specific child class.
- Swallowing an exception and continuing as if the operation succeeded.
- Returning from finally and unintentionally replacing an earlier result.
- Displaying exception messages, paths, SQL, or traces to public users.
- Disabling all reporting instead of fixing diagnostics.
- Using @ instead of checking return values or handling errors.
- Assuming set_error_handler() catches parse and fatal errors.
- Assuming set_exception_handler() resumes the failed operation.
- Treating a shutdown callback as a way to recover from every fatal condition.
- Logging credentials, tokens, or private data along with the error.

## Try it yourself

1. Throw InvalidArgumentException for a negative quantity and catch it without catching all Throwable objects.
2. Add a second catch for RuntimeException and place the more specific catch first.
3. Throw an exception inside a helper function and catch it in its caller.
4. Add a finally block that closes a resource after both success and failure.
5. Create a custom exception that extends RuntimeException and preserves a previous exception.
6. Convert only E_WARNING to ErrorException, then restore the prior handler in finally.
7. Check how an @-suppressed warning interacts with your handler, and confirm that you honor error_reporting().
8. Configure development to show E_ALL and production to log errors without displaying internals.

## Key takeaways

- PHP diagnostics and Throwable objects are related but distinct failure mechanisms.
- Exception and Error both implement Throwable; catch only what the current layer can handle.
- Use finally for cleanup and avoid overriding an in-flight result or exception.
- Convert diagnostics to exceptions only when a deliberate handler makes the flow clearer.
- User error handlers cannot intercept every parse, compile, or fatal error.
- Use protected logs for details and generic public responses in production.
- Uncaught and shutdown handlers are final diagnostics boundaries, not recovery mechanisms.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [056 - Serialization and safe data exchange](056-serialization-and-safe-data-exchange.md) | [Open Q&A](../qnas/057-errors-exceptions-and-throwable.md) | [Open example](../examples/057-errors-exceptions-and-throwable.md) | [058 - Namespaces and autoloading](058-namespaces-and-autoloading.md) |
