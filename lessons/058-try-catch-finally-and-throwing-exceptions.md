# 058 - Try, Catch, Finally, and Throwing Exceptions

This chapter focuses on exception control flow: how to signal a failure, how PHP searches for a matching handler, how an exception travels between functions, and how to guarantee cleanup. The goal is to handle failures at the layer that can make a useful decision without hiding or corrupting the original failure.

## The basic flow

A throwable object is raised with throw. PHP immediately stops the current path and looks for a matching catch:

~~~php
<?php

try {
    echo "Step 1\n";
    throw new RuntimeException('The operation failed.');
    echo "Step 2\n";
} catch (RuntimeException $exception) {
    echo 'Handled: ' . $exception->getMessage() . "\n";
}

echo "Continue after catch\n";
~~~

Output:

~~~text
Step 1
Handled: The operation failed.
Continue after catch
~~~

Step 2 does not run because throw changes the control flow. Once a matching catch finishes, execution continues after the try/catch structure.

A try block must have at least one catch or finally block. A catch handles a failure; finally is generally used for cleanup.

## Throw a useful exception

Throw an exception when the current operation cannot complete and the caller should decide what to do:

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
~~~

The thrown value must be an object implementing Throwable. A string or arbitrary object cannot be thrown. PHP's standard exceptions include useful types such as InvalidArgumentException and RuntimeException.

Choose the type based on the problem:

- Use InvalidArgumentException when a caller supplies a value that violates a function's documented input contract.
- Use LogicException when the program's state or usage violates a logical contract.
- Use RuntimeException when an operation fails at runtime and the caller may have a recovery decision.
- Define a domain-specific exception when callers need to distinguish a stable application failure.

Do not use an exception for ordinary branching. If a condition is expected and easy to test, a conditional or explicit result may be clearer.

## Match catches by type

PHP tests catch blocks from top to bottom. The first compatible type handles the thrown object:

~~~php
try {
    performOrderOperation();
} catch (InvalidArgumentException $exception) {
    // Handle invalid caller input.
} catch (RuntimeException $exception) {
    // Handle an expected operational failure.
}
~~~

Put more specific subclasses before their parents. If a RuntimeException catch comes first, it also matches an InvalidArgumentException because InvalidArgumentException extends LogicException, not RuntimeException; choose the hierarchy carefully. A safer example of parent and child ordering is:

~~~php
try {
    performOperation();
} catch (LengthException $exception) {
    // Handle this specific RuntimeException subtype first.
} catch (RuntimeException $exception) {
    // Handle other runtime failures.
}
~~~

The first matching catch runs; later catches are not also executed. If no catch matches in the current function, the exception bubbles to the caller. That continues until a matching catch is found or execution reaches the global boundary.

As of PHP 7.1, one catch may list multiple unrelated exception types with the pipe character when the same recovery action applies:

~~~php
try {
    readApplicationData();
} catch (JsonException | RuntimeException $exception) {
    reportUnreadableData($exception);
}
~~~

Avoid redundant union types where one listed type is already a parent of another.

Since PHP 8.0, a catch variable can be omitted when the handler does not need the exception object:

~~~php
try {
    performOptionalOperation();
} catch (OptionalOperationException) {
    echo "The optional operation was skipped.\n";
}
~~~

Keep the variable when you need the message, code, previous exception, or other diagnostic context.

## Exceptions travel through function calls

An exception can be thrown deep in a helper function and caught by a caller. This lets low-level code describe what failed while a higher layer decides how to respond:

~~~php
<?php

function readRequiredSetting(array $settings, string $name): string
{
    if (!isset($settings[$name]) || !is_string($settings[$name])) {
        throw new InvalidArgumentException(
            "Missing or invalid setting: $name"
        );
    }

    return $settings[$name];
}

try {
    $region = readRequiredSetting([], 'region');
} catch (InvalidArgumentException $exception) {
    echo 'Configuration problem: ' . $exception->getMessage();
}
~~~

Do not catch an exception just because it occurred in the current function. Catch it where the code can recover, translate it into an application-level response, or add context and rethrow it.

## Rethrow or wrap while preserving the cause

If a layer cannot recover but can add useful context, wrap the lower-level exception and pass it as the previous exception:

~~~php
<?php

final class OrderProcessingException extends RuntimeException
{
}

function loadOrderRecord(): array
{
    try {
        // Imagine this call is to a storage adapter.
        throw new RuntimeException('Storage connection timed out.');
    } catch (RuntimeException $exception) {
        throw new OrderProcessingException(
            'The order could not be loaded.',
            0,
            $exception
        );
    }
}

try {
    loadOrderRecord();
} catch (OrderProcessingException $exception) {
    echo $exception->getMessage() . "\n";
    echo 'Cause: ' . $exception->getPrevious()->getMessage() . "\n";
}
~~~

Passing the original exception as the third constructor argument preserves the causal chain. Do not replace the detailed cause with a generic exception and discard the original.

If no additional context is needed, rethrow the same object:

~~~php
try {
    performOperation();
} catch (RuntimeException $exception) {
    logFailure($exception);
    throw $exception;
}
~~~

A throw inside a catch is not handled by a sibling catch from that same try. The new or rethrown exception travels outward to an enclosing try or the caller.

## Use finally for reliable cleanup

A finally block runs after the try/catch path whether the operation succeeds, a catch handles the exception, or the exception continues outward:

~~~php
<?php

$handle = fopen('records.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the records file.');
}

try {
    processRecords($handle);
} finally {
    fclose($handle);
}
~~~

The resource is opened before the try, and the failure case is handled before cleanup is needed. Once the handle exists, finally closes it on both normal and exceptional paths.

A try may use finally without a catch. In that form, cleanup runs and any exception continues to the caller:

~~~php
try {
    updateSeveralRecords();
} finally {
    releaseLock();
}
~~~

Nested finally blocks each run while an exception unwinds the stack. A finally block also runs when the try or catch returns. It is suitable for cleanup such as closing a file, releasing a lock, or restoring temporary state.

### Do not override the failure in finally

Avoid returning or throwing from finally. A return in finally can replace a value returned by try or catch. If finally throws while another exception is already travelling, the new exception becomes the propagated one and can obscure the original control flow.

Also remember the abrupt termination cases: if the process exits or a fatal condition stops execution, finally may not run. Do not call exit() in code that relies on finally for cleanup.

## Throw as an expression

Since PHP 8.0, throw is an expression, so it can appear in expression contexts:

~~~php
<?php

$name = $inputName !== ''
    ? $inputName
    : throw new InvalidArgumentException('Name is required.');
~~~

This can be concise, but use a regular if statement when it makes validation and recovery easier to follow. On PHP versions before 8.0, throw was a statement and had to be used as a standalone statement.

## An end-to-end example

This example validates a request, handles expected failures distinctly, translates a storage failure while preserving its cause, and always releases a simulated lock:

~~~php
<?php

declare(strict_types=1);

final class InsufficientStockException extends RuntimeException
{
}

final class OrderServiceException extends RuntimeException
{
}

function reserveStock(int $available, int $requested): string
{
    if ($requested <= 0) {
        throw new InvalidArgumentException(
            'Requested quantity must be positive.'
        );
    }

    if ($requested > $available) {
        throw new InsufficientStockException(
            'There is not enough stock.'
        );
    }

    return "Reserved $requested item(s).";
}

function placeOrder(int $available, int $requested): string
{
    echo "Lock acquired.\n";

    try {
        try {
            return reserveStock($available, $requested);
        } catch (InsufficientStockException $exception) {
            throw new OrderServiceException(
                'The order could not be completed.',
                0,
                $exception
            );
        }
    } finally {
        echo "Lock released.\n";
    }
}

try {
    echo placeOrder(5, 2) . "\n";
    echo placeOrder(5, 8) . "\n";
} catch (InvalidArgumentException $exception) {
    echo 'Invalid request: ' . $exception->getMessage() . "\n";
} catch (OrderServiceException $exception) {
    echo 'Order failure: ' . $exception->getMessage() . "\n";
    echo 'Cause: ' . $exception->getPrevious()->getMessage() . "\n";
}
~~~

The first call returns normally; finally releases the lock before the return reaches the caller. The second call wraps the stock exception. Finally still releases the lock, then the wrapped exception bubbles to the outer catch.

## Common mistakes

- Writing code after throw and expecting it to run.
- Catching a broad parent type before a specific child type.
- Assuming sibling catches handle exceptions thrown by another catch block.
- Catching and swallowing an exception without a valid recovery action.
- Replacing a lower-level exception without preserving it as the previous cause.
- Using exceptions for routine checks that are clearer as conditionals.
- Returning or throwing from finally and replacing an in-flight result or exception.
- Opening a resource inside a try and then closing it unconditionally without checking that opening succeeded.
- Calling exit() while depending on finally for cleanup.
- Assuming finally handles every fatal or abrupt process termination.

## Try it yourself

1. Add a statement after throw and confirm it is skipped.
2. Throw LengthException and arrange the catches from most specific to most general.
3. Throw an exception in a helper and catch it several calls higher.
4. Wrap a RuntimeException in a domain exception and inspect getPrevious().
5. Rethrow an exception from a catch and handle it in an outer try.
6. Use finally with a file handle on both success and failure paths.
7. Add a return to finally in a scratch example and observe how it changes the return value.
8. On PHP 8.0 or later, use throw as an expression, then rewrite it with an if statement.

## Key takeaways

- Throw a Throwable object to signal a failure that should interrupt normal control flow.
- Catch the most specific type that the current layer can handle.
- Uncaught exceptions bubble through callers until a matching catch or the global boundary.
- Wrap exceptions only when adding useful context, and preserve the lower-level cause.
- Use finally for cleanup, not for returning alternate results or hiding failures.
- Sibling catch blocks do not catch exceptions thrown by another catch block.
- finally cannot guarantee cleanup after abrupt process termination.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [057 - Errors, exceptions, and Throwable](057-errors-exceptions-and-throwable.md) | [Open Q&A](../qnas/058-try-catch-finally-and-throwing-exceptions.md) | [Open example](../examples/058-try-catch-finally-and-throwing-exceptions.md) | [059 - PHP errors, exceptions, and error categories](059-php-errors-exceptions-and-error-categories.md) |
