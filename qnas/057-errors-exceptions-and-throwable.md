# 057 - Errors, Exceptions, and Throwable: Q&A

Review PHP diagnostic errors, the Throwable hierarchy, exception flow, handlers, cleanup, and production reporting. Try each question before reading its answer. See the [chapter lesson](../lessons/057-errors-exceptions-and-throwable.md).

## Multiple-choice questions

### 1. Which statement best describes an E_WARNING diagnostic?

A. It is always an Exception object
B. It is a severity-based diagnostic and may also accompany a false return value
C. It is the same as a TypeError
D. It automatically stops every script

**Answer:** B. It is a severity-based diagnostic and may also accompany a false return value.

**Explanation:** A diagnostic warning is not automatically a Throwable. Check documented return values or deliberately convert selected diagnostics.

### 2. Which interface is implemented by both Exception and Error?

A. Iterator
B. Throwable
C. Serializable
D. Stringable

**Answer:** B. Throwable.

**Explanation:** Both branches can be thrown and caught with try/catch.

### 3. Which hierarchy is correct?

A. Error extends Exception
B. Exception extends Error
C. Exception and Error are separate branches implementing Throwable
D. Throwable extends Exception

**Answer:** C. Exception and Error are separate branches implementing Throwable.

**Explanation:** TypeError is under Error, while RuntimeException is under Exception.

### 4. What is a sensible diagnostic reporting level during development?

A. E_ERROR only
B. E_ALL
C. 0
D. E_DEPRECATED only

**Answer:** B. E_ALL.

**Explanation:** Reporting all diagnostic levels in development helps reveal warnings, notices, deprecations, and other issues early.

### 5. What is a safer production web configuration for diagnostics?

A. Display every error to visitors
B. Hide diagnostics from the response and log them privately
C. Turn off reporting and logging
D. Put stack traces into every HTML page

**Answer:** B. Hide diagnostics from the response and log them privately.

**Explanation:** Public error output can reveal paths, queries, configuration, or secrets. Protected logs help maintainers investigate failures.

### 6. What does error_reporting(E_ALL) do?

A. Converts all warnings into exceptions
B. Selects diagnostic levels to report
C. Catches every fatal error
D. Sends errors to a database

**Answer:** B. It selects diagnostic levels to report.

**Explanation:** It does not turn diagnostics into Throwable objects; display and logging are configured separately.

### 7. What happens to statements after a throw in the same try path?

A. They always execute before the catch
B. They are skipped while PHP searches for a matching catch
C. They run only if the exception is an Error
D. They are converted into finally statements

**Answer:** B. They are skipped while PHP searches for a matching catch.

**Explanation:** The thrown object travels up the call stack until PHP finds a matching catch or reaches the uncaught boundary.

### 8. In what order should related catch types normally be listed?

A. Parent class before child class
B. Child class before parent class
C. Alphabetical order only
D. Order does not matter

**Answer:** B. Child class before parent class.

**Explanation:** A catch for the parent also matches child instances, so an earlier parent catch could make the child catch unreachable.

### 9. What is finally primarily useful for?

A. Hiding every exception
B. Cleanup that should happen after the try/catch path
C. Replacing all catch blocks
D. Showing a stack trace to visitors

**Answer:** B. Cleanup that should happen after the try/catch path.

**Explanation:** Use it to close resources or release locks on success and failure. Abrupt termination can prevent it from running.

### 10. Which class is a typical parent for an application-specific operational exception?

A. RuntimeException
B. stdClass
C. TypeError
D. Throwable

**Answer:** A. RuntimeException.

**Explanation:** Custom exceptions normally extend Exception or one of its subclasses. User classes cannot implement Throwable directly.

### 11. What is the purpose of passing a previous exception to a new exception?

A. To suppress all logging
B. To preserve the lower-level cause for diagnosis
C. To turn an exception into a warning
D. To continue the failed operation

**Answer:** B. To preserve the lower-level cause for diagnosis.

**Explanation:** The previous exception can be retrieved with getPrevious() and logged privately.

### 12. Which built-in type represents a type mismatch detected by PHP?

A. TypeError
B. RuntimeWarning
C. ParseNotice
D. InvalidArgumentDiagnostic

**Answer:** A. TypeError.

**Explanation:** TypeError is an Error branch Throwable and commonly indicates a programming defect or invalid call.

### 13. Which diagnostic types cannot be handled by a user-defined set_error_handler() callback?

A. E_WARNING and E_NOTICE
B. E_ERROR, E_PARSE, E_CORE_ERROR, E_CORE_WARNING, E_COMPILE_ERROR, and E_COMPILE_WARNING
C. E_USER_WARNING only
D. All diagnostics can be handled

**Answer:** B. Those listed fatal, parse, core, and compile diagnostic types.

**Explanation:** A custom handler has limits and cannot replace proper error configuration or fixes.

### 14. What is set_exception_handler() for?

A. Resuming a failed operation from the beginning
B. A final callback for uncaught Throwable objects
C. Handling every diagnostic warning
D. Changing the exception hierarchy

**Answer:** B. A final callback for uncaught Throwable objects.

**Explanation:** It can log and produce a safe final response, but it does not resume the interrupted code.

### 15. What can a shutdown callback use to inspect the last PHP diagnostic?

A. error_get_last()
B. getPrevious()
C. error_reporting()
D. restore_error_handler()

**Answer:** A. error_get_last().

**Explanation:** Together with register_shutdown_function(), it can provide supplementary logging for some fatal diagnostics.

## Short-answer questions

### 16. Name PHP's two broad failure-reporting mechanisms covered in this chapter.

**Answer:** Severity-based diagnostic errors and Throwable objects.

**Explanation:** Diagnostics may be displayed or logged; Throwable objects are thrown and handled with try/catch.

### 17. Is an ordinary PHP warning automatically an Exception?

**Answer:** No.

**Explanation:** A warning is a diagnostic. A function may also return false, so check its documented result or convert selected warnings deliberately.

### 18. What is the relationship between Exception, Error, and Throwable?

**Answer:** Exception and Error are separate class branches that both implement the Throwable interface.

**Explanation:** Code can catch Throwable to handle both branches, but broad catches need a deliberate recovery policy.

### 19. Why use E_ALL in development?

**Answer:** It makes the development environment report all diagnostic levels so issues can be fixed before release.

**Explanation:** It does not mean every diagnostic should be shown to production users.

### 20. Why should production web responses not display raw diagnostics?

**Answer:** They can expose sensitive implementation details such as paths, SQL, configuration, and stack traces.

**Explanation:** Log details to a protected destination and show users a generic message.

### 21. Does error_reporting() configure whether errors appear in the browser?

**Answer:** No. It selects which diagnostic levels are reported; display_errors and log_errors control display and logging behavior.

**Explanation:** Reporting, display, and logging are related but distinct configuration concerns.

### 22. Where should an exception normally be caught?

**Answer:** At the layer that can meaningfully recover, translate the failure, or add useful context.

**Explanation:** Catching and swallowing an exception without recovery can make failed work appear successful.

### 23. Why should a catch for InvalidArgumentException come before a catch for RuntimeException when both are relevant?

**Answer:** A broader parent catch can match a child before the more specific catch gets a chance.

**Explanation:** Put the most specific matching type first.

### 24. What should a finally block contain?

**Answer:** Cleanup that should occur after the try/catch path, such as closing a handle or releasing a lock.

**Explanation:** Avoid returning or throwing from finally because that can replace an in-flight result or exception.

### 25. How should an application-specific exception usually be defined?

**Answer:** Extend Exception or an appropriate subclass such as RuntimeException.

**Explanation:** Throwable itself is not intended to be directly implemented by user-defined classes.

### 26. What does an exception's previous-exception field preserve?

**Answer:** The lower-level exception that caused or informed a higher-level exception.

**Explanation:** It provides a causal chain for private logs and debugging while callers receive a useful higher-level type.

### 27. Why might a custom error handler check error_reporting() before converting a diagnostic?

**Answer:** A custom handler may still be called when the @ operator suppresses a diagnostic, so the handler can honor the current reporting mask.

**Explanation:** In modern PHP, suppression does not guarantee that a user-defined handler will not run.

### 28. What is the role and limitation of a shutdown callback checking error_get_last()?

**Answer:** It can log information about some fatal diagnostics at shutdown, but it cannot recover from every fatal, startup, or compile failure.

**Explanation:** Treat it as supplementary diagnostics, not ordinary exception handling.

## True or false

### 29. error_reporting(E_ALL) automatically turns every PHP diagnostic into an exception.

**Answer:** False.

**Explanation:** It selects diagnostic levels. A custom handler is needed to convert selected diagnostics into ErrorException objects.

### 30. A TypeError is in the Error branch of the Throwable hierarchy.

**Answer:** True.

**Explanation:** TypeError implements Throwable through the Error class hierarchy.

### 31. Exception extends Error.

**Answer:** False.

**Explanation:** Exception and Error are separate branches; both implement Throwable.

### 32. A finally block normally runs while PHP unwinds a thrown Throwable.

**Answer:** True.

**Explanation:** It is intended for cleanup during normal flow and exception unwinding, although abrupt process termination can prevent it.

### 33. Catching Throwable and continuing is always a safe way to recover.

**Answer:** False.

**Explanation:** Throwable includes programming errors. Continue only when the application has a sound recovery policy.

### 34. A custom error handler can catch every parse and fatal error.

**Answer:** False.

**Explanation:** Several error types, including E_PARSE and E_ERROR, cannot be handled by a user-defined error handler.

### 35. set_exception_handler() resumes the statement that failed.

**Answer:** False.

**Explanation:** It handles an uncaught Throwable at the final boundary; it does not resume the interrupted path.

### 36. A production application should generally log diagnostics privately rather than display implementation details to visitors.

**Answer:** True.

**Explanation:** Private logs support investigation while generic public messages avoid disclosing internals.

## Predict the result or behavior

### 37. What is printed?

~~~php
<?php

try {
    echo "A";
    throw new RuntimeException('Failed');
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

**Explanation:** The throw skips B, the matching catch prints C, and finally prints D.

### 38. Which catch runs first?

~~~php
try {
    throw new InvalidArgumentException('Bad value');
} catch (InvalidArgumentException $exception) {
    echo 'specific';
} catch (RuntimeException $exception) {
    echo 'general';
}
~~~

**Answer:** The InvalidArgumentException catch runs and prints specific.

**Explanation:** The specific child class is checked before its RuntimeException parent.

### 39. What is the output if no exception is thrown?

~~~php
try {
    echo "work";
} finally {
    echo "cleanup";
}
~~~

**Answer:**

~~~text
workcleanup
~~~

**Explanation:** finally runs after the try block even when the operation completes normally.

### 40. What is the result?

~~~php
<?php

function expectInteger(int $value): int
{
    return $value;
}

try {
    expectInteger('five');
} catch (TypeError $error) {
    echo 'type';
}
~~~

**Answer:**

~~~text
type
~~~

**Explanation:** Passing a non-numeric string to a parameter declared int in strict typing context raises TypeError, which can be caught.

### 41. Does the following handler guarantee that every PHP error will become an ErrorException?

~~~php
set_error_handler(function () {
    throw new ErrorException();
});
~~~

**Answer:** No.

**Explanation:** Some fatal, parse, core, and compile error types cannot be handled by set_error_handler(). A handler also should use the actual severity and diagnostic details.

### 42. If an uncaught exception reaches set_exception_handler(), does code after the failing statement continue?

**Answer:** No. The exception handler is a final boundary; it does not resume the interrupted operation.

**Explanation:** It can log the failure and send a safe final response, after which request execution ends.

## Coding questions

### 43. Write a function that rejects a non-positive quantity with InvalidArgumentException and otherwise returns it.

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

**Explanation:** The function documents its valid input contract and signals invalid input with a specific exception.

### 44. Convert only unsuppressed E_WARNING diagnostics into ErrorException and restore the previous handler afterward.

**Answer:**

~~~php
set_error_handler(
    function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
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
    E_WARNING
);

try {
    performOperationThatMayWarn();
} finally {
    restore_error_handler();
}
~~~

**Explanation:** The mask limits conversion to E_WARNING. The reporting check respects suppression state, and finally restores the prior handler.

### 45. Show a safe uncaught-exception handler that logs details but sends a generic response.

**Answer:**

~~~php
set_exception_handler(
    function (Throwable $throwable): void {
        error_log((string) $throwable);

        http_response_code(500);
        echo 'An unexpected error occurred.';
    }
);
~~~

**Explanation:** The protected log contains diagnostic detail for maintainers. The public response does not disclose paths, stack traces, SQL, or secrets.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [056 - Serialization and safe data exchange](../lessons/056-serialization-and-safe-data-exchange.md) | [Read lesson](../lessons/057-errors-exceptions-and-throwable.md) | [Open example](../examples/057-errors-exceptions-and-throwable.md) | [058 - try, catch, finally, and throwing exceptions](../lessons/058-try-catch-finally-and-throwing-exceptions.md) |
