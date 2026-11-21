# Safe error handling and private information

Errors are part of normal software operation: a request can contain invalid input, a database can become unavailable, or an unexpected bug can interrupt processing. A safe application gives users a useful, non-sensitive response while preserving enough protected diagnostic information for maintainers to investigate.

The central rule is simple: **show a clear outcome to the user; keep implementation details in access-controlled logs.**

## Learning objectives

After this lesson, you should be able to:

- distinguish PHP errors, exceptions, and other throwables;
- configure PHP to record diagnostics without displaying them to visitors;
- return appropriate HTTP status codes and generic error messages;
- log useful, redacted diagnostic context with a request identifier;
- protect logs from secrets, personal data, injection, unauthorized access, and unbounded growth;
- place exception handling at a request or job boundary without hiding failures.

## Errors, exceptions, and throwables

PHP has several kinds of failures:

- **Warnings and notices** are diagnostics emitted while a program continues or partially continues. They can indicate incorrect assumptions or misuse of an API.
- **Exceptions** represent conditions that code can throw and, where appropriate, catch with try and catch.
- **Errors** represent engine or programming failures, such as a type error. Both Error and Exception implement Throwable, so a boundary can catch either when it must convert an unexpected failure into a safe response.
- **Parse and startup failures** may happen before application code executes. A handler installed inside a PHP script cannot reliably handle failures that prevent the script from starting. Configure PHP itself for production behavior.

Do not catch every failure just to continue. Catch a known exception when the application can recover or map it to a meaningful result. At the outer request or job boundary, catch unexpected Throwable only to record a safe diagnostic and finish the operation with a generic failure response. Do not silently discard it.

## Separate the user response from diagnostics

An error response is part of the application interface. It should tell the user what happened at a useful level and what they can do next, without revealing how the application is built.

A public response should not expose:

- stack traces, source code, filesystem paths, or environment variables;
- SQL statements, database hostnames, credentials, or internal service names;
- access tokens, session identifiers, passwords, or encryption keys;
- raw exception messages that might contain submitted values or infrastructure details;
- request bodies, cookies, or other private user data.

For an unexpected server failure, return an appropriate status such as HTTP 500 and a stable, general error code. A support or request identifier can help correlate the response with a protected log entry. Generate that identifier on the server; do not blindly reflect an arbitrary identifier supplied by a visitor.

For a known client error, return an accurate 4xx status and a safe explanation. For example, invalid input can be reported as a validation error. Do not turn every failure into HTTP 500, and do not return HTTP 200 merely because the error was handled.

### HTML and API responses

An HTML page can display a short message and a support ID. A JSON API should return a consistent JSON shape and the correct content type. Neither format should include an exception trace.

~~~json
{
  "error": "internal_error",
  "message": "The request could not be completed.",
  "requestId": "server-generated-id"
}
~~~

Only return fields that are part of the documented public API. Avoid including a filesystem path or raw upstream response in an error payload.

## Configure PHP for production

Production configuration should keep diagnostics out of the response and send them to a protected logging destination. Configure this in the PHP runtime or deployment configuration, not only with application code that might run after a startup or parse failure.

~~~ini
error_reporting = E_ALL
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/php/application-error.log
~~~

The path above is an example. Choose a destination outside the public document root that the PHP process can write to and that only authorized operators can read. Configure log rotation, retention, monitoring, and disk limits for the actual deployment.

Keep useful reporting enabled. Turning off display does not mean turning off detection: E_ALL allows diagnostics to be recorded for investigation. In local development, visible diagnostics can help, but development settings must not be deployed to production.

The PHP distribution's development and production configuration templates have different defaults. Review the active runtime configuration for the PHP SAPI serving the application; CLI and web-server PHP can load different configuration files.

## Handle unexpected failures at a boundary

Framework exception middleware is usually the best place to handle request failures because it can set response formats, status codes, and logging consistently. For a small application, a top-level try/catch or an uncaught-exception handler can provide a fallback.

A safe boundary should:

1. create or obtain a server-side request ID;
2. record a minimal, redacted diagnostic in a protected log;
3. set the correct HTTP status and content type;
4. send a generic HTML or JSON response;
5. avoid rendering the exception message or trace.

For an API, set headers before writing the body. If output has already begun, PHP may be unable to change headers or status reliably; keep output out of bootstrap and error-handling code.

A global exception handler is a last-resort boundary for uncaught exceptions. It is not a substitute for handling expected conditions near the code that understands them. It also does not replace PHP runtime configuration for failures that occur before the handler can be registered.

## Log only what is needed

A useful event log commonly includes:

- a timestamp added by the logger;
- a stable event name and severity;
- a server-generated request or job ID;
- a safe route or operation name;
- an exception category or internal error code where useful;
- the outcome and status code.

Do not log an entire request or an exception blindly. Exception messages and stack traces can contain submitted values, file paths, SQL, tokens, or other confidential details. If detailed diagnostics are needed, use a protected logging system with access controls and a deliberate redaction policy. Make sure sensitive values are removed before they reach that system.

Avoid logging:

- passwords, authentication secrets, session IDs, cookies, access tokens, and API keys;
- database connection strings or encryption keys;
- full payment-card data or other highly sensitive personal information;
- complete request bodies, uploaded file contents, or private headers without a specific, reviewed need.

Use data minimization: collect only fields needed to diagnose or audit the event, restrict who can access them, and delete them according to a retention policy. Masking is not sufficient if the remaining value still identifies a person or allows a secret to be used.

## Protect log integrity and availability

Logs are a security-sensitive data store. Keep them outside the public document root, restrict read and write access, and protect them in transit when they are sent to a central logging service. Monitor important events and failures, and retain logs only as long as needed.

Treat visitor-controlled text as untrusted. A newline or control character in a value copied into a plain-text log can forge additional-looking records. Prefer structured logging with a trusted logger that encodes fields; otherwise validate, bound, and sanitize untrusted values before writing them. Do not concatenate raw request data into log lines.

Protect against log flooding and disk exhaustion through size limits, rotation, retention, rate-aware alerting, and operational monitoring. Do not suppress every repeated failure: aggregate or rate-limit noisy events while retaining enough evidence to detect an attack or outage.

## Handle known failures close to their source

A known failure should be mapped where the application has enough context to choose a safe result:

- A missing record can produce a 404 response.
- Invalid input can produce a 400 or 422 response, based on the API contract.
- A failed permission check can produce 403 without explaining private authorization rules.
- A temporary dependency outage can produce a generic 503 response if retry may help.
- An unexpected bug should produce a generic 500 response and a protected diagnostic.

Avoid using PHP's error-control operator as a general error-handling strategy. It can hide diagnostics that would otherwise help identify a defect. Check return values and handle expected failures explicitly; configure reporting and logging for unexpected failures.

## Safe error-handling checklist

Before deploying an application, confirm that:

- production uses display_errors = Off and display_startup_errors = Off;
- error reporting remains enabled and log_errors = On;
- the active PHP configuration and logging destination are verified for the web SAPI;
- logs are outside the public document root and have restrictive permissions;
- unexpected failures return the correct status and a generic, documented response;
- request IDs are generated or validated safely and never contain arbitrary visitor text;
- exceptions, SQL, traces, secrets, cookies, and full request bodies are not exposed or logged indiscriminately;
- structured logging safely encodes untrusted fields;
- log access, retention, rotation, capacity, monitoring, and redaction are configured;
- a controlled failure has been checked in both the browser/API response and the protected log.

## Key takeaways

- Keep public messages useful and generic; keep diagnostics private and access-controlled.
- In production, report errors and log them, but do not display them to visitors.
- Catch expected failures where they can be handled; use a top-level boundary for unexpected failures.
- A request ID can connect a generic response to a protected log entry.
- Never assume exception messages or request data are safe to show or log.
- Protect logs against disclosure, injection, tampering, excessive retention, and disk exhaustion.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [134 - HTTPS, security headers, and secure transport](134-https-security-headers-and-secure-transport.md) | [Q&A](../qnas/135-safe-error-handling-and-private-information.md) | [Example](../examples/135-safe-error-handling-and-private-information.md) | [136 - Profiling PHP applications](136-profiling-php-applications.md) |