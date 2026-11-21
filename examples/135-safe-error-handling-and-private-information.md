# Safe error handling and private information — example

This example demonstrates a small JSON endpoint that keeps the public response generic and writes a minimal diagnostic record to PHP's configured error log. The request ID connects the response to the log without exposing exception details.

## Goal

When an unexpected failure occurs, the endpoint should:

- return HTTP 500 with a stable JSON error code and a generic message;
- generate a server-side request ID and return it in the response header and body;
- write a structured, minimal record to the protected PHP error log;
- avoid sending or logging the raw exception message and stack trace.

## Production PHP configuration

Configure the PHP runtime for the web server or PHP-FPM process. The example path must be replaced with a destination outside the public document root that the PHP process can write to.

~~~ini
error_reporting = E_ALL
display_errors = Off
display_startup_errors = Off
log_errors = On
error_log = /var/log/php/application-error.log
~~~

Protect the log file and its directory with restrictive permissions. Configure rotation, retention, capacity monitoring, and authorized access for the target environment. Verify the active configuration for the web SAPI; it can differ from the CLI configuration.

## Complete endpoint

Save this as a PHP endpoint such as public/api.php. Replace the deliberate exception with the real application operation. This example deliberately fails so the safe error path is visible.

~~~php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=utf-8');

$requestId = bin2hex(random_bytes(16));
header('X-Request-ID: ' . $requestId);

try {
    // Replace this deliberate failure with the real request operation.
    // The message represents internal detail and must not reach the visitor.
    throw new RuntimeException('Database operation failed for an internal host');
} catch (Throwable $exception) {
    $logRecord = [
        'timestamp' => gmdate('c'),
        'event' => 'request_failed',
        'request_id' => $requestId,
        'exception_type' => get_class($exception),
        'http_status' => 500,
    ];

    $encodedLogRecord = json_encode(
        $logRecord,
        JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
    );

    // PHP sends this to the protected error_log destination configured above.
    // Do not append the raw exception message or trace to this record.
    error_log($encodedLogRecord);

    http_response_code(500);

    $response = [
        'error' => 'internal_error',
        'message' => 'The request could not be completed.',
        'requestId' => $requestId,
    ];

    echo json_encode(
        $response,
        JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
    );
}
~~~

The code catches Throwable at the outer request boundary so an unexpected Exception or Error cannot become a detailed public response. In a framework application, use its exception middleware to apply the same policy consistently.

The log record intentionally includes only stable operational metadata. A production team that needs deeper diagnostics should send appropriately redacted information to a protected logging system with strict access and retention controls. Do not add the exception message, request body, cookies, authorization header, SQL, or trace without a reviewed need and a reliable redaction policy.

## Expected HTTP response

The request returns HTTP 500 with a JSON body similar to this:

~~~http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json; charset=utf-8
X-Request-ID: 9c53a20b0e914ce4a6d26

{
  "error": "internal_error",
  "message": "The request could not be completed.",
  "requestId": "9c53a20b0e914ce4a6d26"
}
~~~

The ID shown is illustrative; the endpoint generates a new random value per request. The response does not contain the exception message, host, filesystem path, or stack trace.

## Expected protected log entry

The PHP error log receives a JSON record similar to this:

~~~json
{
  "timestamp": "2026-01-15T12:34:56+00:00",
  "event": "request_failed",
  "request_id": "9c53a20b0e914ce4a6d26",
  "exception_type": "RuntimeException",
  "http_status": 500
}
~~~

The timestamp and ID are illustrative. The real log is written to the configured server-side destination, not returned to the caller.

## Adapt the example safely

- Catch and map expected failures near the code that understands them. Use the outer boundary only as a fallback for unexpected failures.
- Return a documented 4xx status for invalid client input; do not report it as an unexpected 500.
- If the endpoint serves HTML, return a generic page instead of this JSON response while preserving the status and support ID.
- If a trusted proxy supplies a request ID, validate its length and character set before accepting it. Otherwise generate one in the application.
- Keep request headers and body fields out of logs unless a specific operational requirement has been reviewed and sensitive fields are redacted.
- Configure the logger to safely encode any untrusted values. Never concatenate raw user-controlled text into a plain-text log line.
- Keep production display disabled in PHP configuration. A script-level handler cannot cover every parse or startup failure.
- If logging fails, keep the visitor response generic and alert through deployment monitoring; never fall back to printing the diagnostic into the response.

## Review checklist

- [ ] The web SAPI has error reporting enabled, display disabled, and logging enabled.
- [ ] The configured log destination is outside the public document root and access-controlled.
- [ ] A controlled exception returns HTTP 500 and the generic JSON shape.
- [ ] The response includes a server-generated request ID but no exception details.
- [ ] The log entry includes that same ID and excludes the exception message and trace.
- [ ] Logs have rotation, retention, access controls, and capacity monitoring.
- [ ] No password, token, cookie, SQL, or full request body is written by this handler.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [134 - HTTPS, security headers, and secure transport](../lessons/134-https-security-headers-and-secure-transport.md) | [Lesson](../lessons/135-safe-error-handling-and-private-information.md) | [Q&A](../qnas/135-safe-error-handling-and-private-information.md) | [136 - Profiling PHP applications](../lessons/136-profiling-php-applications.md) |