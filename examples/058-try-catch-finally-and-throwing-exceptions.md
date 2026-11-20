# 058 - Try, Catch, Finally, and Throwing Exceptions: Example

## Question

How can an order operation reject invalid input, handle an expected stock failure, preserve the original cause, and always release cleanup state?

## Complete example

Save this as a PHP file and run it with PHP 8.0 or later. The lock messages simulate acquiring and releasing a resource so the exception paths are easy to see.

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
            // Add service-level context and preserve the lower-level cause.
            throw new OrderServiceException(
                'The order could not be completed.',
                0,
                $exception
            );
        }
    } finally {
        // Runs before a return completes and while an exception bubbles out.
        echo "Lock released.\n";
    }
}

function attemptOrder(
    string $label,
    int $available,
    int $requested
): void {
    echo $label . "\n";

    try {
        $result = placeOrder($available, $requested);
        echo 'Result: ' . $result . "\n";
    } catch (InvalidArgumentException) {
        // PHP 8 permits omitting the variable when it is not needed.
        echo "Request error: Requested quantity must be positive.\n";
    } catch (OrderServiceException $exception) {
        echo 'Order failure: ' . $exception->getMessage() . "\n";
        echo 'Cause: ' . $exception->getPrevious()->getMessage() . "\n";
    }

    echo "\n";
}

attemptOrder('Attempt 1: valid quantity', 5, 2);
attemptOrder('Attempt 2: invalid quantity', 5, 0);
attemptOrder('Attempt 3: unavailable stock', 5, 8);

// Since PHP 8.0, throw can be used as an expression.
function requireLabel(string $label): string
{
    return $label !== ''
        ? $label
        : throw new InvalidArgumentException('A label is required.');
}

try {
    requireLabel('');
} catch (InvalidArgumentException) {
    echo "Empty label rejected.\n";
}

// A union catch can handle unrelated types with the same recovery action.
try {
    throw new JsonException('The JSON input is malformed.');
} catch (JsonException | InvalidArgumentException $exception) {
    echo 'Shared input handler: ' . get_class($exception) . "\n";
}
~~~

## Expected output

~~~text
Attempt 1: valid quantity
Lock acquired.
Lock released.
Result: Reserved 2 item(s).

Attempt 2: invalid quantity
Lock acquired.
Lock released.
Request error: Requested quantity must be positive.

Attempt 3: unavailable stock
Lock acquired.
Lock released.
Order failure: The order could not be completed.
Cause: There is not enough stock.

Empty label rejected.
Shared input handler: JsonException
~~~

## How it works

- reserveStock() throws InvalidArgumentException for an invalid quantity and InsufficientStockException when the request exceeds available stock.
- placeOrder() catches only the stock-specific exception. It wraps that failure as OrderServiceException and passes the original as the previous exception.
- InvalidArgumentException and OrderServiceException are separate branches under Exception, so the outer catches handle different cases.
- finally prints the release message on a successful return and on both exception paths. It runs before the function returns or the exception continues to the caller.
- attemptOrder() catches the expected input failure separately from a service failure. The service handler can read the original stock error through getPrevious().
- requireLabel() uses the PHP 8.0 throw-expression syntax. Its catch omits the variable because only the type matters.
- The union catch handles JsonException and InvalidArgumentException with the same response. Neither is a subclass of the other.
- The simulated lock is only output for demonstration. Real resource acquisition should be checked before entering a cleanup path, and abrupt process termination can prevent finally from running.

## Try it yourself

1. Change Attempt 1 to request more items than are available and follow the exception to the outer handler.
2. Change Attempt 2 to request a negative quantity and confirm the same input exception is selected.
3. Add a custom exception extending InsufficientStockException and put its catch before the parent catch.
4. Remove the third argument when creating OrderServiceException and observe that getPrevious() no longer exposes the stock failure.
5. Add a return statement inside finally in a scratch copy and see how it overrides the intended result.
6. Replace the throw expression in requireLabel() with a regular if statement for PHP versions before 8.0.
7. Add a catch block that throws a new exception, then handle it in an enclosing try rather than a sibling catch.

## Common mistakes

- Expecting code after throw to run.
- Catching a parent before its specific child.
- Assuming a sibling catch handles an exception thrown by another catch.
- Wrapping an exception without preserving it as the previous cause.
- Putting a return or a new exception in finally and replacing the intended flow.
- Assuming finally runs after exit() or every fatal termination.
- Using exceptions for ordinary checks that are clearer as conditionals.
- Omitting the check that a resource was successfully opened before relying on finally to close it.

## Key takeaways

- Use exceptions to signal failures that need a caller's decision.
- Catch specific failure types and let unrelated failures continue upward.
- Preserve the original cause when adding higher-level context.
- Use finally for cleanup on success and exception paths.
- Avoid overriding pending results or exceptions in finally.
- PHP 7.1 added union catches; PHP 8.0 added optional catch variables and throw expressions.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [057 - Errors, exceptions, and Throwable](057-errors-exceptions-and-throwable.md) | [Open Q&A](../qnas/058-try-catch-finally-and-throwing-exceptions.md) | [Read lesson](../lessons/058-try-catch-finally-and-throwing-exceptions.md) | [059 - PHP errors, exceptions, and error categories](059-php-errors-exceptions-and-error-categories.md) |
