# Safe error handling and private information — Q&A

These questions review the ideas in [Chapter 135](../lessons/135-safe-error-handling-and-private-information.md). Every answer appears immediately after its question, with an explanation where it helps.

## Multiple-choice questions

### 1. Which production setup best protects visitors while preserving diagnostics?

A. Display every PHP error and do not keep logs.
B. Disable error reporting and display every exception in the page.
C. Keep error reporting enabled, disable display, and log errors to a protected destination.
D. Suppress all errors with the error-control operator.

**Answer: C.**

**Explanation:** Production should detect errors and record them for authorized maintainers, while keeping implementation details out of public responses. Disabling display does not mean disabling error reporting.

### 2. What should an unexpected server failure normally return to a visitor?

A. The stack trace and source file path.
B. A generic message, an appropriate server-error status, and optionally a server-generated request ID.
C. The raw exception message with HTTP 200.
D. The database error text so the user can report it.

**Answer: B.**

**Explanation:** A generic response prevents internal details from leaking. A request ID lets support staff correlate the report with a protected log entry.

### 3. Which information should generally be excluded from application logs?

A. A stable event name.
B. A server-generated request ID.
C. A safe operation name and status.
D. Passwords, session IDs, access tokens, and full request bodies.

**Answer: D.**

**Explanation:** These values are secrets or private data. Logging them creates another place where they can be exposed or misused.

### 4. What does setting display_errors to Off accomplish?

A. It disables PHP diagnostics entirely.
B. It prevents PHP diagnostics from being displayed in the response.
C. It automatically encrypts the log file.
D. It catches parse errors in application code.

**Answer: B.**

**Explanation:** Keep error reporting enabled and configure logging separately. PHP startup configuration is also needed for failures that happen before application code runs.

### 5. Which HTTP status is generally appropriate for an unexpected server-side failure?

A. 200
B. 301
C. 404
D. 500

**Answer: D.**

**Explanation:** HTTP 500 communicates an unexpected server failure. Known client errors should use an appropriate 4xx status instead of being mislabeled as server errors.

### 6. Why use a request ID in an error response?

A. It contains the exception trace in encoded form.
B. It lets the user identify a password reset token.
C. It gives support a safe value to correlate with protected diagnostic logs.
D. It replaces authorization checks.

**Answer: C.**

**Explanation:** Generate the ID on the server and include it in both the generic response and the log. Do not copy an arbitrary visitor-provided value into a response or log.

## Short-answer questions

### 7. What is the difference between an exception and an unexpected failure handled at the application boundary?

**Answer:** An exception is a value thrown by code to represent an exceptional condition. Code close to the operation may catch a known exception and recover or map it to a safe result. A request-boundary handler is a final fallback for unexpected uncaught failures.

**Explanation:** Catching everything too early can hide defects or prevent the framework from handling the request correctly. Handle expected cases where the application has enough context, then use the outer boundary for safe reporting and a generic response.

### 8. Why should a production system use E_ALL while display_errors is Off?

**Answer:** E_ALL keeps diagnostics detectable and available for logs, while display_errors = Off keeps those details out of visitor responses.

**Explanation:** Detection and presentation are separate concerns. Turning off display is not a reason to hide diagnostics from maintainers.

### 9. Why is an exception message not automatically safe to show or log?

**Answer:** It may contain submitted data, SQL, internal paths, service details, credentials, or other sensitive values.

**Explanation:** Treat messages and traces as potentially sensitive. Redact them before using a protected diagnostic system, and never send them directly to a visitor.

### 10. Why does a script-level handler not replace PHP runtime configuration?

**Answer:** Some failures, such as parse or startup failures, can occur before application code executes and before a script-level handler is installed.

**Explanation:** Configure the PHP runtime for production so display and logging behavior is in place early. Verify the configuration used by the web SAPI, which may differ from the CLI configuration.

### 11. What is log injection, and how can it be reduced?

**Answer:** Log injection happens when untrusted input, such as a newline, makes a forged or misleading entry appear in a log.

**Explanation:** Prefer a structured logger that safely encodes fields. Do not concatenate raw visitor input into log lines; validate, bound, or sanitize any user-controlled values that truly need to be logged.

### 12. What is data minimization in logging?

**Answer:** It means collecting only the fields needed to diagnose or audit an event, limiting who can access them, and deleting them according to a retention policy.

**Explanation:** Fewer retained sensitive fields mean fewer opportunities for accidental disclosure or misuse. Masking alone is not enough if the remaining value still identifies someone or acts as a secret.

## True or false

### 13. In production, turning off display_errors means the application should also turn off all error reporting.

**Answer: False.**

**Explanation:** Keep error reporting enabled and route diagnostics to a protected log. The public response and the private operational record have different audiences.

### 14. A top-level catch can make every kind of PHP failure recoverable.

**Answer: False.**

**Explanation:** A request boundary can handle many uncaught Throwables, but it cannot reliably handle failures that stop the script before it starts, such as some parse or startup failures.

### 15. It is safe to log every request body because logs are for developers.

**Answer: False.**

**Explanation:** Request bodies can contain passwords, personal information, uploaded content, or tokens. Log only reviewed fields that are necessary, and apply access controls and retention limits.

### 16. A handled error can still require an HTTP error status.

**Answer: True.**

**Explanation:** Handling means the application produced a deliberate response; it does not mean the request succeeded. A validation failure might use 400 or 422, while an unexpected server failure might use 500.

## Applied scenarios

### 17. A database operation throws an exception containing its hostname and query. What should the public API return?

**Answer:** Return a generic error payload with an appropriate status, such as 500 for an unexpected internal failure, and optionally a server-generated request ID. Keep the raw exception details out of the response.

**Explanation:** The client needs to know that the operation failed, not the database topology or SQL. Record only safe diagnostic details in a protected system.

### 18. A user submits an invalid email address. Should this normally be reported as HTTP 500?

**Answer:** No. Return the API's documented client-error status, commonly 400 or 422, with a safe validation message.

**Explanation:** The request is invalid, but the server is functioning. Reserve 500 for unexpected server-side failures.

### 19. A production response contains a PHP path and stack trace. What should be changed?

**Answer:** Disable error display in the active production PHP configuration, keep error logging enabled, and make the request boundary return a generic response.

**Explanation:** Check the web SAPI configuration, not only CLI settings. Also verify that the protected log destination is writable by PHP and not publicly served.

### 20. A request ID comes from an HTTP header. Can the application write it to a log unchanged?

**Answer:** Not safely by default. Generate a new server-side ID, or validate a trusted upstream ID against a strict format and length before accepting it.

**Explanation:** An arbitrary header can contain control characters or attacker-chosen text. The ID should remain a safe correlation token, not a channel for log injection.

### 21. A warning repeats thousands of times and fills the disk. What should the team do?

**Answer:** Fix the underlying issue, rotate and bound logs, monitor capacity, and use alerting or carefully designed aggregation/rate controls for repeated events.

**Explanation:** Suppressing every event can hide an attack or ongoing outage. Availability controls should reduce flooding while retaining enough evidence to investigate.

## Code-reading questions

### 22. What is unsafe about this handler?

~~~php
set_exception_handler(function (Throwable $exception): void {
    echo $exception->getMessage();
    echo $exception->getTraceAsString();
});
~~~

**Answer:** It prints the raw exception message and full trace directly to the visitor.

**Explanation:** Either can expose submitted values, internal paths, SQL, secrets, or service details. A production handler should log a carefully redacted diagnostic to a protected destination and return a generic response with an appropriate status.

### 23. Does the following configuration disable error detection?

~~~ini
error_reporting = E_ALL
display_errors = Off
log_errors = On
~~~

**Answer: No.** Error reporting remains enabled, display is disabled, and PHP is configured to log errors.

**Explanation:** The logging destination still needs to be configured or supplied by the runtime, protected from public access, and managed with retention and rotation.

## Practice exercise

### 24. Write a short API failure response policy for an unexpected exception. Which details belong in the response, and which belong in protected diagnostics?

**Answer:** The response should use HTTP 500 and a stable public error code, a generic message, and optionally a server-generated request ID. Protected diagnostics can include the same ID, a safe event name, timestamp, severity, safe operation name, status, and a carefully redacted exception category.

**Explanation:** Keep passwords, tokens, cookies, raw request bodies, SQL, raw exception messages, and traces out of the public response. Do not add those fields to logs without a reviewed need and a safe redaction policy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [134 - HTTPS, security headers, and secure transport](../lessons/134-https-security-headers-and-secure-transport.md) | [Lesson](../lessons/135-safe-error-handling-and-private-information.md) | [Example](../examples/135-safe-error-handling-and-private-information.md) | [136 - Profiling PHP applications](../lessons/136-profiling-php-applications.md) |