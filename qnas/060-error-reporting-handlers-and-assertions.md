# 060 - Error Reporting, Handlers, and Assertions: Q&A

Use these questions to review diagnostic configuration, custom handlers, uncaught exceptions, and assertions. Try each one before reading its answer. See the [chapter lesson](../lessons/060-error-reporting-handlers-and-assertions.md).

## Multiple-choice questions

### 1. What does `error_reporting(E_ALL)` do?

A. Displays every diagnostic in a browser
B. Selects all currently defined diagnostic levels for reporting
C. Turns every diagnostic into a Throwable
D. Writes all messages to a log file

**Answer:** B. It selects all currently defined diagnostic levels for reporting.

**Explanation:** Reporting level, display, and logging are separate controls.

### 2. What does `display_errors` control?

A. Whether PHP sends diagnostic details in its response
B. Which error levels a custom handler receives
C. Whether exceptions can be thrown
D. Whether PHP parses source files

**Answer:** A. It controls whether PHP displays diagnostic details in the response.

**Explanation:** Production responses should not expose paths, stack traces, or sensitive internals.

### 3. Which setting enables PHP's error-log output?

A. `display_errors`
B. `error_reporting`
C. `log_errors`
D. `zend.assertions`

**Answer:** C. `log_errors`.

**Explanation:** The destination is configured separately, commonly through `error_log` or the hosting environment.

### 4. Which is a suitable production web configuration pattern?

A. Display full errors and disable logging
B. Report useful levels, keep public display off, and log privately
C. Set reporting to zero and show visitors a stack trace
D. Convert every diagnostic to an HTML response

**Answer:** B. Report useful levels, keep public display off, and log privately.

**Explanation:** Operators need details in protected logs; visitors should receive safe, generic responses.

### 5. How should multiple diagnostic levels be combined?

A. With bitwise OR (`|`)
B. With string concatenation
C. With the keyword `and`
D. By adding the constant names

**Answer:** A. With bitwise OR (`|`).

**Explanation:** The constants are bit flags; for example, `E_WARNING | E_USER_WARNING`.

### 6. What does `error_reporting()` return when called without an argument?

A. The last exception
B. The currently active diagnostic mask
C. The configured log file
D. The current error handler

**Answer:** B. The currently active diagnostic mask.

**Explanation:** When passed a new mask, it changes the runtime setting and returns the previous mask.

### 7. Which expression excludes deprecations from an `E_ALL` mask?

A. `E_ALL | E_DEPRECATED`
B. `E_ALL & ~E_DEPRECATED`
C. `E_ALL && E_DEPRECATED`
D. `E_DEPRECATED - E_ALL`

**Answer:** B. `E_ALL & ~E_DEPRECATED`.

**Explanation:** This is a bitmask operation. Suppressing deprecations should not replace updating code that needs migration.

### 8. What is the purpose of the second argument to `set_error_handler()`?

A. It selects which diagnostic levels invoke the callback
B. It sets the exception class to throw
C. It configures the log destination
D. It changes `display_errors`

**Answer:** A. It selects which diagnostic levels invoke the callback.

**Explanation:** This callback mask is distinct from the global `error_reporting()` mask.

### 9. Which callback parameters are used by a PHP diagnostic error handler?

A. Severity, message, file, and line
B. Request, response, route, and status
C. Exception, previous exception, and stack
D. File handle and byte count only

**Answer:** A. Severity, message, file, and line.

**Explanation:** PHP passes diagnostic context to the callback. The old error context argument is no longer passed in PHP 8.

### 10. What does a custom error handler returning `false` mean?

A. PHP's standard error handler should continue
B. PHP should silently ignore every error
C. The handler has thrown an exception
D. The current script should restart

**Answer:** A. PHP's standard error handler should continue.

**Explanation:** Return `false` when the custom handler has not fully handled that diagnostic.

### 11. What does a custom error handler returning `true` generally mean?

A. The custom handler handled the diagnostic
B. PHP will throw a `TypeError`
C. The error was added to `E_ALL`
D. The handler was removed

**Answer:** A. The custom handler handled the diagnostic.

**Explanation:** Returning a non-false result prevents PHP's standard handler from continuing for that diagnostic.

### 12. Does the `error_reporting()` mask alone prevent a registered custom handler from being called?

A. Yes, always
B. No; a custom handler can still be called and should inspect the active mask if needed
C. Yes, but only for warnings
D. It depends only on `display_errors`

**Answer:** B. No; a custom handler can still be called and should inspect the active mask if needed.

**Explanation:** The handler's level mask and the global reporting mask serve different purposes.

### 13. Which call restores the previous diagnostic handler?

A. `restore_error_handler()`
B. `restore_exception_handler()`
C. `error_reporting()`
D. `error_get_last()`

**Answer:** A. `restore_error_handler()`.

**Explanation:** PHP stores custom error handlers on a stack; restoring returns to the prior handler.

### 14. Which PHP problems cannot generally be handled by a user-defined diagnostic handler?

A. Every `E_USER_WARNING`
B. Some engine-level failures such as parse and compile errors
C. All exceptions
D. Every warning from an included file

**Answer:** B. Some engine-level failures such as parse and compile errors.

**Explanation:** Several fatal and compile-time categories are outside the user-defined error handler's scope.

### 15. What does `set_exception_handler()` handle?

A. Diagnostics that are included in `E_ALL`
B. A `Throwable` that was not caught by normal `try/catch`
C. Only `E_WARNING`
D. Only `AssertionError`

**Answer:** B. A `Throwable` that was not caught by normal `try/catch`.

**Explanation:** It is a final boundary for uncaught exceptions and errors, not a diagnostic error handler.

## Short-answer questions

### 16. Explain the difference between reporting, displaying, and logging.

**Answer:** Reporting selects which diagnostic levels PHP processes. Displaying controls whether details appear in the response. Logging controls whether PHP writes diagnostics to a log destination.

**Explanation:** A production application can report and log diagnostic details while hiding them from visitors.

### 17. Why is `E_ALL` useful during development?

**Answer:** It surfaces a broad set of diagnostics, including notices, warnings, and deprecations that may reveal bugs or outdated code.

**Explanation:** Catching issues during development makes them easier to investigate before deployment.

### 18. Why should production not show detailed errors to visitors?

**Answer:** Details can reveal source paths, database information, configuration, request data, or stack traces.

**Explanation:** Keep detailed records in protected logs and return a generic, safe response.

### 19. Why can `ini_set('display_errors', '1')` fail to help with a parse error in the same file?

**Answer:** PHP must parse the file before executing its statements, so the call may never run.

**Explanation:** Startup and parse-time behavior must be configured outside the code that failed to execute.

### 20. How is a custom diagnostic handler different from an exception handler?

**Answer:** A diagnostic handler receives selected PHP error levels. An exception handler receives a `Throwable` that escaped all matching `try/catch` blocks.

**Explanation:** A `TypeError` is not an `E_WARNING`, and configuring one handler does not configure the other.

### 21. Why should a temporary diagnostic handler be restored in a `finally` block?

**Answer:** The previous handler is restored whether the protected code returns normally or throws.

**Explanation:** This prevents a local handler from unintentionally changing how later code handles diagnostics.

### 22. What should a handler do when its callback mask includes a diagnostic but the active reporting mask excludes it?

**Answer:** If suppressed diagnostics should remain suppressed, inspect `error_reporting()` and return `false` when the severity bit is not active.

**Explanation:** The callback can still be invoked regardless of the reporting mask, so the handler must enforce this policy itself.

### 23. What is the purpose of an uncaught-exception handler?

**Answer:** It provides a last-resort place to record an uncaught `Throwable` and coordinate a safe final response.

**Explanation:** It does not recover execution or resume the code after the uncaught failure.

### 24. What can `error_get_last()` tell a shutdown callback, and what is one limitation?

**Answer:** It can return information about the most recent PHP diagnostic. It does not guarantee that every failure can be recovered or that the last diagnostic is the one the application expected.

**Explanation:** A shutdown callback is a limited reporting mechanism, not a replacement for normal exception handling.

### 25. Why can the CLI and web server use different PHP configurations?

**Answer:** The command-line SAPI and a web-server SAPI may load different configuration files or overrides.

**Explanation:** Inspect the runtime that actually serves the application; `php --ini` only reports the CLI configuration.

### 26. What are the three documented `zend.assertions` modes?

**Answer:** `1` generates and executes assertion code, `0` generates but skips it at runtime, and `-1` does not generate it.

**Explanation:** When a process starts with `-1`, assertion code cannot be enabled later for that process.

### 27. Why should an assertion not be used to validate a visitor's input?

**Answer:** Assertions can be disabled or omitted, so input validation must use normal conditions and explicit error handling.

**Explanation:** Security and correctness must not depend on an assertion being active.

### 28. Why should an assertion expression not perform a required side effect?

**Answer:** The expression may not run when assertion code is disabled.

**Explanation:** Perform the operation explicitly, handle its result, and use an assertion only for an optional internal invariant.

### 29. What changed about string expressions passed to `assert()` in PHP 8?

**Answer:** A string argument is treated like an ordinary string value; it is no longer evaluated as PHP source code.

**Explanation:** Pass a boolean expression such as `assert($count >= 0)`, not a source-code string.

### 30. Which assertion-related INI settings were deprecated in PHP 8.3?

**Answer:** The `assert.*` settings were deprecated, including `assert.active`, `assert.exception`, `assert.warning`, `assert.bail`, and `assert.callback`.

**Explanation:** Do not build new code around these deprecated settings; understand `zend.assertions` for the target runtime.

## Scenarios and code review

### 31. A production page shows a database password in a warning. Which configuration and response changes are needed?

**Answer:** Disable public error display, enable logging to a protected destination, and return a generic response to the visitor.

**Explanation:** Hiding the response does not mean disabling reporting. Operators still need the diagnostic details in secured logs.

### 32. A handler logs a diagnostic and then returns `false`. What might happen?

**Answer:** PHP's standard handler may also process the diagnostic, potentially causing duplicate display or reporting.

**Explanation:** Return `false` only when the standard handler should continue. Return `true` after the custom handler fully handles it.

### 33. An application uses `assert($user->isAdmin())` before showing an admin page. Is this a safe authorization check?

**Answer:** No. Authorization must be enforced with an ordinary conditional that runs regardless of assertion configuration.

**Explanation:** An assertion is not a security boundary and can be disabled.

### 34. A custom handler is expected to catch an uncaught `InvalidArgumentException`. Why does it not?

**Answer:** `InvalidArgumentException` is a thrown `Throwable`, not a diagnostic level delivered to `set_error_handler()`.

**Explanation:** Catch it with `try/catch` or record it with a suitable uncaught-exception handler if it escapes.

### 35. The current handler is installed inside a library function and never restored. What is the risk?

**Answer:** Later code may continue using the library's handler, changing application-wide diagnostic behavior unexpectedly.

**Explanation:** Scope temporary handler changes and restore the previous handler in a `finally` block.

### 36. A handler should only process warnings. How can its callback mask be configured?

**Answer:** Pass `E_WARNING` as the second argument to `set_error_handler()`.

**Explanation:** The callback mask limits which diagnostic severities invoke that handler; it does not set the global reporting or display policy.

### 37. Why is `assert($saved = saveRecord($record))` a bug-prone design?

**Answer:** The save operation and assignment may be skipped when assertions are disabled.

**Explanation:** Call `saveRecord()` outside the assertion, handle failure explicitly, and assert only a resulting invariant if useful.

### 38. A team wants to turn every warning into an exception globally. What should they consider first?

**Answer:** They should identify which warnings are truly exceptional, preserve reporting-mask behavior, and test the effect at a narrow application boundary.

**Explanation:** A blanket conversion can turn recoverable or third-party diagnostics into unexpected control flow. A selected handler is easier to reason about.

### 39. Why is disabling deprecation reporting not a complete upgrade plan?

**Answer:** It hides information about code that may need changes for a later PHP version.

**Explanation:** Use the diagnostics to find and fix compatibility issues; suppress them only as a deliberate, temporary policy.

### 40. A `register_shutdown_function()` callback is being used to recover from every fatal or parse error. What is wrong with this expectation?

**Answer:** Shutdown callbacks cannot make failed code resume, and some parse/startup failures happen before the callback can be registered.

**Explanation:** Treat shutdown inspection as limited last-resort logging for failures reached after registration.

## Coding questions

### 41. Write code that reports all diagnostic levels during a development run.

**Answer:**

~~~php
error_reporting(E_ALL);
~~~

**Explanation:** This selects all current PHP diagnostic levels. Whether details are displayed or logged is controlled separately.

### 42. Combine `E_WARNING` and `E_USER_WARNING`, then check whether `E_WARNING` is included.

**Answer:**

~~~php
$levels = E_WARNING | E_USER_WARNING;
$warningSelected = ($levels & E_WARNING) !== 0;
~~~

**Explanation:** Bitwise OR combines flags; bitwise AND tests one flag.

### 43. Write a custom handler that falls through when the severity is not in the active reporting mask.

**Answer:**

~~~php
set_error_handler(
    static function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        if ((error_reporting() & $severity) === 0) {
            return false;
        }

        error_log(sprintf('%s in %s:%d', $message, $file, $line));
        return true;
    },
    E_WARNING | E_USER_WARNING
);
~~~

**Explanation:** The callback checks the global reporting mask, logs selected warnings, and returns `false` for PHP's standard handler to continue when the severity is masked.

### 44. Show how to restore a temporary error handler even if protected code throws.

**Answer:**

~~~php
set_error_handler($handler, E_WARNING);

try {
    performOperation();
} finally {
    restore_error_handler();
}
~~~

**Explanation:** `finally` runs on normal completion and while an exception unwinds through the block.

### 45. Replace an assertion-based check of a required request value with runtime validation.

**Answer:**

~~~php
$name = $_POST['name'] ?? '';

if (!is_string($name) || trim($name) === '') {
    throw new InvalidArgumentException('A non-empty name is required.');
}
~~~

**Explanation:** This validation runs whether assertions are enabled or disabled. An assertion may document an internal invariant after validation, but it must not replace validation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [059 - PHP errors, exceptions, and error categories](../lessons/059-php-errors-exceptions-and-error-categories.md) | [Read lesson](../lessons/060-error-reporting-handlers-and-assertions.md) | [Open example](../examples/060-error-reporting-handlers-and-assertions.md) | [061 - Debugging, diagnostics, and application logging](../lessons/061-debugging-diagnostics-and-application-logging.md) |
