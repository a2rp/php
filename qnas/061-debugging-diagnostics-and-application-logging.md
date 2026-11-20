# 061 - Debugging, Diagnostics, and Application Logging: Q&A

Review the debugging workflow, PHP inspection tools, environment checks, and safe application logging. Try each question before reading its answer. See the [chapter lesson](../lessons/061-debugging-diagnostics-and-application-logging.md).

## Multiple-choice questions

### 1. What is a useful first step when investigating a bug?

A. Suppress all PHP diagnostics
B. Reproduce the issue and record the input and expected behavior
C. Rewrite the whole application
D. Add `var_dump()` to every file

**Answer:** B. Reproduce the issue and record the input and expected behavior.

**Explanation:** A repeatable case makes it easier to isolate the cause and verify the fix.

### 2. Which command checks a PHP file for syntax errors without executing it?

A. `php -l path/to/file.php`
B. `php --run path/to/file.php`
C. `php --ini path/to/file.php`
D. `php -m path/to/file.php`

**Answer:** A. `php -l path/to/file.php`.

**Explanation:** The lint option checks syntax only.

### 3. What does `php -v` display?

A. The CLI PHP version
B. The active request body
C. The application's log history
D. All installed database tables

**Answer:** A. The CLI PHP version.

**Explanation:** Compare the CLI version with the version used by the web server when behavior differs.

### 4. What does `php --ini` show?

A. The CLI configuration files used by PHP
B. Every error that has happened on the website
C. The current Git branch
D. The contents of the web server's access log

**Answer:** A. The CLI configuration files used by PHP.

**Explanation:** A web SAPI may load a different configuration.

### 5. Which tool is intended to inspect a variable's type and value during local debugging?

A. `var_dump()`
B. `error_reporting()`
C. `restore_error_handler()`
D. `random_bytes()`

**Answer:** A. `var_dump()`.

**Explanation:** It displays type and value, including nested array and object information.

### 6. Which statement about `var_dump()` in a public web response is safest?

A. It is always harmless
B. It may expose sensitive data or corrupt the response
C. It automatically redacts passwords
D. It writes only to a protected server log

**Answer:** B. It may expose sensitive data or corrupt the response.

**Explanation:** Use temporary inspection only in controlled development and remove it afterward.

### 7. What is `debug_backtrace()` used for?

A. Generating the call stack
B. Checking PHP syntax
C. Rotating log files
D. Creating a request ID

**Answer:** A. Generating the call stack.

**Explanation:** A backtrace helps identify the sequence of calls that led to a point in execution.

### 8. Which option omits function arguments from a `debug_backtrace()` result?

A. `DEBUG_BACKTRACE_IGNORE_ARGS`
B. `E_ALL`
C. `JSON_THROW_ON_ERROR`
D. `ENT_QUOTES`

**Answer:** A. `DEBUG_BACKTRACE_IGNORE_ARGS`.

**Explanation:** Arguments may contain secrets; omitting them can reduce exposure and memory use.

### 9. Why should a public `phpinfo()` page be avoided?

A. It changes every PHP source file
B. It can expose configuration, paths, extensions, and environment details
C. It disables error logging
D. It only works from the command line

**Answer:** B. It can expose configuration, paths, extensions, and environment details.

**Explanation:** If used for local investigation, remove or protect it afterward.

### 10. What does `error_get_last()` return?

A. A full history of every PHP diagnostic
B. Information about the most recent PHP diagnostic, or null
C. The currently registered exception handler
D. A list of all application log files

**Answer:** B. Information about the most recent PHP diagnostic, or null.

**Explanation:** It is a limited lookup, not a diagnostic history.

### 11. What does `error_log($message)` do by default?

A. Sends the message to PHP's configured logging destination
B. Displays the message in bold HTML
C. Sends the message to the current visitor by email
D. Saves the message in a PHP variable

**Answer:** A. Sends the message to PHP's configured logging destination.

**Explanation:** The destination depends on PHP and hosting configuration.

### 12. Why can CLI and web PHP behave differently on the same computer?

A. They may use different versions, extensions, or configuration files
B. CLI cannot run PHP functions
C. Web PHP ignores all configuration
D. The source code is automatically rewritten

**Answer:** A. They may use different versions, extensions, or configuration files.

**Explanation:** Inspect the runtime that actually exhibits the problem.

### 13. Which item is generally inappropriate to put in an application log?

A. A stable event name
B. A request identifier
C. A password or access token
D. A safe internal record ID

**Answer:** C. A password or access token.

**Explanation:** Keep credentials and other sensitive values out of logs.

### 14. Why might an application attach a request ID to log records?

A. To correlate events from one request across components
B. To replace authentication checks
C. To encode the user's password
D. To prevent PHP from reporting errors

**Answer:** A. To correlate events from one request across components.

**Explanation:** A reference ID can also help support teams find internal details without exposing them to the user.

### 15. Where should an exception generally be logged?

A. At every function that catches and rethrows it
B. At a boundary that has enough context and makes the final handling decision
C. In every template
D. Only in the browser console

**Answer:** B. At a boundary that has enough context and makes the final handling decision.

**Explanation:** Repeated logging at each layer creates duplicate records for one failure.

## Short-answer questions

### 16. Why is a reproducible case useful?

**Answer:** It provides a consistent way to observe the failure and compare behavior before and after a change.

**Explanation:** It narrows the investigation and helps confirm that the actual issue was fixed.

### 17. Why should you read the first meaningful diagnostic in a sequence?

**Answer:** Later errors may be consequences of the original failure.

**Explanation:** Fixing the earliest cause may resolve several follow-on symptoms.

### 18. What should you compare if a script works in CLI but fails on the server?

**Answer:** Compare PHP versions, loaded extensions, configuration files, environment variables, permissions, and SAPI-specific behavior.

**Explanation:** The CLI and server process may not share the same runtime setup.

### 19. What is the difference between `print_r()` and `var_dump()`?

**Answer:** `print_r()` provides a readable representation; `var_dump()` includes types and values.

**Explanation:** Both are temporary inspection tools and may expose sensitive data.

### 20. When is `var_export()` useful?

**Answer:** It can produce a parsable representation of many PHP values for inspection.

**Explanation:** Do not confuse its output with a safe logging format or assume every object can be reconstructed from it.

### 21. What is one risk of logging raw request bodies?

**Answer:** They may contain passwords, tokens, payment data, or personal information.

**Explanation:** Log only the fields needed to diagnose the event, and redact sensitive values.

### 22. Why sanitize untrusted text before including it in a log message?

**Answer:** Newlines and control characters can forge or distort log entries.

**Explanation:** Treat request data as untrusted even when it is sent to an internal log.

### 23. What makes a log record structured?

**Answer:** It contains named fields, such as time, level, event, request ID, and safe context.

**Explanation:** Structured fields are easier for log tools to search, filter, and aggregate than free-form sentences.

### 24. Why use a consistent timestamp convention such as UTC?

**Answer:** It makes events from different servers and services easier to compare.

**Explanation:** Local timezone differences can make incident timelines confusing.

### 25. Why should log access be restricted?

**Answer:** Logs can contain internal implementation details and sometimes sensitive operational context.

**Explanation:** Apply access controls, retention, and deletion policies appropriate to the data.

### 26. How should a web application respond when it logs an internal exception?

**Answer:** It should return a safe, generic message rather than the exception's internal details.

**Explanation:** A request ID can help the user report the issue without revealing a stack trace.

### 27. Why should an exception usually be logged once?

**Answer:** Repeated logs at each propagation layer create noise and make a single failure look like many failures.

**Explanation:** Log at the boundary that can add useful context and decide the final response.

### 28. What are the limits of `error_get_last()`?

**Answer:** It reports only the most recent diagnostic, not a complete history, and it is not a recovery mechanism.

**Explanation:** Use normal logging and exception handling for application observability.

### 29. What should you do with temporary debugging output after resolving a bug?

**Answer:** Remove it or replace it with intentional, safe logging.

**Explanation:** Debug output can leak data or break response formats if left in production.

### 30. What is the purpose of a breakpoint?

**Answer:** It pauses execution so a developer can inspect state and step through code.

**Explanation:** An IDE debugger can reduce the need for scattered temporary output, but it should be restricted to controlled environments.

## Scenarios and code review

### 31. A user reports an error but cannot provide the internal trace. What can the application return?

**Answer:** Return a generic message with a request or incident ID that support can use to find the protected log record.

**Explanation:** This supports troubleshooting without exposing internal details.

### 32. A developer adds `var_dump($_SERVER)` to investigate a production bug. What is the risk?

**Answer:** It may expose headers, paths, environment information, cookies, or other request data in the response.

**Explanation:** Use controlled logs or a secure debugging environment and remove the dump.

### 33. A log entry includes the raw value of a user-controlled name with newline characters. What could happen?

**Answer:** The value could create forged extra log lines or make the record misleading.

**Explanation:** Validate or sanitize untrusted values and prefer structured fields with a logger that safely encodes them.

### 34. A service logs an exception and rethrows it; three callers each log and rethrow the same exception. What is the likely result?

**Answer:** One failure may generate four duplicate log entries.

**Explanation:** Log once at a boundary with enough context to decide the final handling.

### 35. An error appears only on a web server, not when running `php file.php`. What should be checked first?

**Answer:** Compare the web SAPI's PHP version, configuration, extensions, environment, and file permissions with the CLI process.

**Explanation:** The local CLI is not necessarily the same runtime as the deployed application.

### 36. A log includes an exception message that contains a raw database query and customer email. Is that safe?

**Answer:** Not automatically. Redact sensitive information or log a safe event code and exception class instead.

**Explanation:** Exception messages are internal data and may include user input or implementation details.

### 37. A team leaves a `phpinfo()` page on a public server because it helped diagnose a configuration mismatch. What should they do?

**Answer:** Remove it or restrict it to an authenticated, controlled environment after the investigation.

**Explanation:** The page can reveal a broad view of the runtime and environment.

### 38. An operations log shows two identical exception entries for one request. What design issue might cause this?

**Answer:** Multiple layers may log the same exception while rethrowing it.

**Explanation:** Choose one boundary for recording the failure and pass context upward without duplicate logging.

### 39. A developer catches every `Throwable` inside a low-level helper and returns null. Why might this make debugging harder?

**Answer:** It hides the original failure and removes information needed by the layer that can make a recovery decision.

**Explanation:** Catch only when the current layer can handle the failure meaningfully; otherwise let it propagate.

### 40. A shutdown function uses `error_get_last()` and assumes every value is a fatal error. What is wrong?

**Answer:** The returned value can describe a non-fatal most-recent diagnostic as well.

**Explanation:** Inspect its type and treat the result as limited evidence, not a complete incident record.

## Coding questions

### 41. Generate a random request ID suitable for correlating logs in one request.

**Answer:**

~~~php
$requestId = bin2hex(random_bytes(16));
~~~

**Explanation:** Generate it once at the request boundary and pass it to the operations that need it.

### 42. Encode a structured log record as JSON and send it to PHP's configured error log.

**Answer:**

~~~php
$record = [
    'time' => gmdate(DATE_ATOM),
    'level' => 'info',
    'event' => 'order.created',
    'request_id' => $requestId,
];

error_log(json_encode($record, JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR));
~~~

**Explanation:** The configured destination depends on the PHP environment. Add only safe context and protect the destination.

### 43. Use `debug_backtrace()` without including function arguments.

**Answer:**

~~~php
$trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 8);
~~~

**Explanation:** Omitting arguments can reduce memory use and avoid capturing secrets passed to functions.

### 44. Catch and record an exception internally while returning a generic public response.

**Answer:**

~~~php
try {
    processOrder($order);
} catch (Throwable $exception) {
    error_log(sprintf(
        'order.process.failed request_id=%s exception=%s',
        $requestId,
        $exception::class
    ));

    http_response_code(500);
    echo 'The order could not be processed. Reference: '
        . htmlspecialchars($requestId, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
}
~~~

**Explanation:** The log records a safe identifier and exception class; the visitor receives no trace or exception message.

### 45. Inspect the most recent diagnostic without assuming one exists.

**Answer:**

~~~php
$lastError = error_get_last();

if ($lastError !== null) {
    error_log(sprintf(
        'Last diagnostic type=%d file=%s line=%d',
        $lastError['type'],
        $lastError['file'],
        $lastError['line']
    ));
}
~~~

**Explanation:** This examines only the most recent diagnostic. It does not provide a full history or make a failed script recover.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [060 - Error reporting, handlers, and assertions](../lessons/060-error-reporting-handlers-and-assertions.md) | [Read lesson](../lessons/061-debugging-diagnostics-and-application-logging.md) | [Open example](../examples/061-debugging-diagnostics-and-application-logging.md) | [062 - Classes, objects, and object-oriented PHP](../lessons/062-classes-objects-and-object-oriented-php.md) |
