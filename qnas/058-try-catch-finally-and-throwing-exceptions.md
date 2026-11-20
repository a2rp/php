# 058 - Try, Catch, Finally, and Throwing Exceptions: Q&A

Review exception flow, catch matching, bubbling, wrapping, and cleanup with finally. Try each question before reading its answer. See the [chapter lesson](../lessons/058-try-catch-finally-and-throwing-exceptions.md).

## Multiple-choice questions

### 1. What value can PHP throw with the throw keyword?

A. Any string
B. Any integer
C. An object implementing Throwable
D. Only an instance of Exception

**Answer:** C. An object implementing Throwable.

**Explanation:** Both Exception and Error objects implement Throwable. A string or arbitrary object cannot be thrown.

### 2. What happens to ordinary statements after a throw in the same path?

A. They run before PHP looks for a catch
B. They are skipped while PHP searches for a matching catch
C. They run after finally but before catch
D. They are automatically retried

**Answer:** B. They are skipped while PHP searches for a matching catch.

**Explanation:** Throw interrupts the current path and starts exception handling.

### 3. Which catch block handles a thrown object?

A. Every catch block in the same try
B. The first catch block whose type matches
C. The last catch block only
D. Any catch block with the same variable name

**Answer:** B. The first catch block whose type matches.

**Explanation:** PHP checks catches in order and runs only the first compatible one.

### 4. If a function does not catch a thrown exception, what normally happens?

A. PHP discards it
B. It bubbles to the caller in search of a matching catch
C. PHP converts it into null
D. The function silently returns false

**Answer:** B. It bubbles to the caller in search of a matching catch.

**Explanation:** The search continues up the call stack. If it reaches the global boundary without a handler, the program terminates.

### 5. Which catch order is correct when PaymentTimeoutException extends RuntimeException?

A. RuntimeException, then PaymentTimeoutException
B. PaymentTimeoutException, then RuntimeException
C. Alphabetical order
D. Either order; both blocks always run

**Answer:** B. PaymentTimeoutException, then RuntimeException.

**Explanation:** The parent catch also matches its child. Put the specific child first so it can be handled separately.

### 6. How are InvalidArgumentException and RuntimeException related?

A. InvalidArgumentException extends RuntimeException
B. RuntimeException extends InvalidArgumentException
C. They are separate sibling branches under Exception
D. They are aliases for the same class

**Answer:** C. They are separate sibling branches under Exception.

**Explanation:** A catch for either of those classes does not catch the other class.

### 7. What must follow a try block?

A. At least one catch or finally block
B. Exactly two catch blocks
C. A return statement
D. A global exception handler

**Answer:** A. At least one catch or finally block.

**Explanation:** A try can have catches, finally, or both, but it cannot stand alone.

### 8. What is finally mainly used for?

A. Retrying failed operations
B. Cleanup that should happen after the try/catch path
C. Hiding exceptions
D. Selecting which catch runs

**Answer:** B. Cleanup that should happen after the try/catch path.

**Explanation:** Closing a resource or releasing a lock are common uses.

### 9. Which PHP version first allows a catch variable to be omitted?

A. PHP 5.0
B. PHP 7.1
C. PHP 8.0
D. PHP 8.5

**Answer:** C. PHP 8.0.

**Explanation:** Earlier versions require the caught exception variable even if the code does not use it.

### 10. Since which PHP version can a catch clause list multiple exception types with a pipe?

A. PHP 5.0
B. PHP 7.1
C. PHP 8.0
D. PHP 8.3

**Answer:** B. PHP 7.1.

**Explanation:** Use a union catch when unrelated exception types share the same recovery action.

### 11. What is the third argument to the standard Exception constructor used for?

A. The source line
B. The previous Throwable in the causal chain
C. The HTTP status code
D. The severity level

**Answer:** B. The previous Throwable in the causal chain.

**Explanation:** Passing the original exception preserves the low-level cause when a higher layer wraps it.

### 12. Since which PHP version is throw allowed as an expression?

A. PHP 5.5
B. PHP 7.1
C. PHP 8.0
D. PHP 8.4

**Answer:** C. PHP 8.0.

**Explanation:** Before PHP 8.0, throw was a statement and had to stand alone.

### 13. Does a finally block normally run when a try or catch returns?

A. No, return skips finally
B. Yes, finally runs before the return completes
C. Only if the return value is null
D. Only if an exception was thrown

**Answer:** B. Yes, finally runs before the return completes.

**Explanation:** The return value is evaluated, then finally executes before control returns to the caller.

### 14. What can happen if finally itself returns a value?

A. It can override a value returned from try or catch
B. It is ignored in every PHP version
C. It converts the return into an exception
D. It runs only after process shutdown

**Answer:** A. It can override a value returned from try or catch.

**Explanation:** Avoid returning from finally because it can replace the intended result.

### 15. What is a safer action when a lower layer fails but the current layer can add context?

A. Discard the original exception
B. Wrap it in a higher-level exception and preserve it as previous
C. Return an empty string
D. Throw a plain string

**Answer:** B. Wrap it in a higher-level exception and preserve it as previous.

**Explanation:** The caller receives a useful domain-level type while maintainers can inspect the original cause.

## Short-answer questions

### 16. When should code throw an exception?

**Answer:** When the current operation cannot complete normally and a caller should decide how to recover or report the failure.

**Explanation:** Use a conditional or result value for ordinary expected branching when it is clearer.

### 17. What is the minimum type required for a thrown value?

**Answer:** The value must be an object implementing Throwable.

**Explanation:** Standard Exception and Error objects meet this requirement; arbitrary values do not.

### 18. What code runs after a matching catch block finishes?

**Answer:** Execution continues after the try/catch structure.

**Explanation:** PHP does not execute later sibling catch blocks for the same thrown object.

### 19. Where should an exception be caught?

**Answer:** At the layer that can meaningfully recover, translate it, or provide a safe response.

**Explanation:** Catching too early without recovery can hide a failure from the layer that knows what to do.

### 20. Why should a specific child exception be caught before its parent?

**Answer:** A parent catch matches instances of its child classes, so it can intercept them first.

**Explanation:** Put the narrowest relevant cases before broader fallback cases.

### 21. What happens if no catch in the current function matches?

**Answer:** The exception bubbles up to its caller.

**Explanation:** It continues up the call stack until a catch handles it or it reaches the global boundary.

### 22. Can a try block use finally without a catch block?

**Answer:** Yes.

**Explanation:** Finally can perform cleanup, after which an uncaught exception continues to the caller.

### 23. Name two appropriate tasks for finally.

**Answer:** Closing a file handle and releasing a lock.

**Explanation:** Finally is meant for cleanup that should happen after normal completion and exception unwinding.

### 24. Why should code avoid returning from finally?

**Answer:** A return from finally can replace a value returned from the try or catch block.

**Explanation:** It can make the function report success or a different result than the main path intended.

### 25. What happens if a finally block throws while another exception is already propagating?

**Answer:** The exception from finally becomes the propagated exception; the original may be attached as its previous exception.

**Explanation:** This can obscure the original failure and is why finally should focus on cleanup.

### 26. What is exception chaining?

**Answer:** It is preserving a lower-level Throwable as the previous cause of a higher-level exception.

**Explanation:** Pass it as the third argument to the standard Exception constructor and inspect it with getPrevious().

### 27. How should a layer rethrow the same exception when it adds no useful context?

**Answer:** Use throw with the caught exception object.

**Explanation:** Do not replace the exception with a generic one and lose its original details.

### 28. Does a sibling catch block handle an exception thrown inside another catch block?

**Answer:** No.

**Explanation:** An exception thrown by a catch must be handled by an enclosing try or a caller.

## True or false

### 29. A thrown exception skips later statements in the same execution path.

**Answer:** True.

**Explanation:** PHP unwinds the current path and searches for a matching catch.

### 30. Every catch block that matches a thrown object runs in sequence.

**Answer:** False.

**Explanation:** Only the first matching catch runs.

### 31. A catch for RuntimeException also catches InvalidArgumentException.

**Answer:** False.

**Explanation:** They are sibling classes under Exception, not parent and child.

### 32. A catch for RuntimeException can match an application-defined class that extends RuntimeException.

**Answer:** True.

**Explanation:** Catching a parent type also matches instances of its child classes.

### 33. A finally block can run when an exception is not caught in the current function.

**Answer:** True.

**Explanation:** Finally runs during stack unwinding before the exception continues to a caller.

### 34. Calling exit() in a try block still guarantees that finally runs.

**Answer:** False.

**Explanation:** Abrupt termination with exit() skips finally blocks.

### 35. Throwing an exception inside a catch block is handled by a later sibling catch from the same try.

**Answer:** False.

**Explanation:** Sibling catches handle the original try body. A newly thrown object must reach an enclosing handler or caller.

### 36. A custom exception can preserve another exception as its previous cause.

**Answer:** True.

**Explanation:** The previous Throwable is passed to the exception constructor and remains available through getPrevious().

## Predict the result or behavior

### 37. What does this print?

~~~php
<?php

try {
    echo "A";
    throw new RuntimeException('Stop');
    echo "B";
} catch (RuntimeException $exception) {
    echo "C";
} finally {
    echo "D";
}
~~~

**Answer:**

~~~text
ACD
~~~

**Explanation:** Throw skips B, the matching catch prints C, and finally prints D.

### 38. Which catch runs?

~~~php
<?php

final class PaymentTimeoutException extends RuntimeException
{
}

try {
    throw new PaymentTimeoutException('Timed out');
} catch (PaymentTimeoutException $exception) {
    echo 'specific';
} catch (RuntimeException $exception) {
    echo 'general';
}
~~~

**Answer:** The first catch runs and prints specific.

**Explanation:** The child exception is matched before its RuntimeException parent.

### 39. What is printed when the try block returns?

~~~php
<?php

function result(): string
{
    try {
        return 'try';
    } finally {
        echo "cleanup ";
    }
}

echo result();
~~~

**Answer:**

~~~text
cleanup try
~~~

**Explanation:** Finally executes before the pending return value is delivered to the caller.

### 40. What does this function return?

~~~php
<?php

function result(): string
{
    try {
        return 'try';
    } finally {
        return 'finally';
    }
}

echo result();
~~~

**Answer:**

~~~text
finally
~~~

**Explanation:** A return from finally overrides the return from try, which is why this pattern should be avoided.

### 41. Does the later catch print handled?

~~~php
<?php

try {
    throw new RuntimeException('first');
} catch (RuntimeException $exception) {
    throw new LogicException('second');
} catch (LogicException $exception) {
    echo 'handled';
}
~~~

**Answer:** No. The new LogicException is not caught by the sibling catch and becomes uncaught unless an outer handler exists.

**Explanation:** Sibling catches do not handle exceptions thrown by another catch block.

### 42. On PHP 8.0 or later, what value is assigned?

~~~php
<?php

$input = '';
$name = $input !== ''
    ? $input
    : throw new InvalidArgumentException('Name is required.');
~~~

**Answer:** No value is assigned; InvalidArgumentException is thrown.

**Explanation:** In PHP 8.0 and later, throw is an expression that can be used in this conditional expression.

## Coding questions

### 43. Write a function that throws InvalidArgumentException if its integer quantity is not positive.

**Answer:**

~~~php
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

**Explanation:** The function rejects invalid input with a specific exception and returns the valid quantity normally.

### 44. Wrap a RuntimeException as OrderProcessingException while preserving the original cause.

**Answer:**

~~~php
try {
    loadOrderRecord();
} catch (RuntimeException $exception) {
    throw new OrderProcessingException(
        'The order could not be loaded.',
        0,
        $exception
    );
}
~~~

**Explanation:** Passing the caught exception as the third argument preserves it for getPrevious() and protected diagnostics.

### 45. Show a try/finally pattern that always closes an opened file handle.

**Answer:**

~~~php
$handle = fopen('records.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the file.');
}

try {
    processRecords($handle);
} finally {
    fclose($handle);
}
~~~

**Explanation:** The open result is checked before entering the cleanup block. Once the resource exists, finally closes it on success or when an exception unwinds the stack.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [057 - Errors, exceptions, and Throwable](../lessons/057-errors-exceptions-and-throwable.md) | [Read lesson](../lessons/058-try-catch-finally-and-throwing-exceptions.md) | [Open example](../examples/058-try-catch-finally-and-throwing-exceptions.md) | [059 - PHP errors, exceptions, and error categories](../lessons/059-php-errors-exceptions-and-error-categories.md) |
