# Production configuration, permissions, and logging — Example

## Goal

Configure a small PHP application so errors are logged privately, visitors receive safe responses, application code is read-only to PHP-FPM, and only required runtime paths are writable.

The paths, users, groups, and service settings below are examples. Adapt them to your host; do not copy ownership changes blindly into a live server.

## Example application layout

~~~text
/srv/shop/
├── current/
│   ├── public/              # Only this directory is the web document root
│   │   └── index.php
│   ├── src/                 # Application source
│   └── config/              # Non-secret configuration
└── shared/
    ├── cache/                # Writable only when the app needs it
    └── uploads/              # Writable only when uploads are used

/etc/shop/
└── runtime.env               # Protected configuration, outside the web root
~~~

The web server serves only /srv/shop/current/public. It must not serve the repository root, /etc/shop, logs, backups, or runtime storage.

| Location | Deployment access | PHP-FPM access |
| --- | --- | --- |
| Application source and public assets | Update during a controlled release | Read only |
| Cache | Usually no request-time access | Write only if required |
| Upload storage | Only if deployment needs it | Narrow write access if uploads are handled |
| Runtime secrets | Provision through a protected process | Read only for the pool that needs them |
| Logs | Configure and rotate through operations | Write through the selected logging setup |

Use the narrowest permissions that allow the application to run. Do not make the whole project writable to fix one cache or upload error.

## PHP production error settings

A production PHP configuration can use:

~~~ini
error_reporting = E_ALL
display_errors = Off
display_startup_errors = Off
log_errors = On
; Configure a file destination only if it is private and writable by FPM:
; error_log = /var/log/shop/php-error.log
~~~

The PHP-FPM worker must be able to write to a configured file destination. Keep that file outside the public document root, restrict access, and arrange rotation and monitoring. If the platform collects process error output, configure and verify that collection instead of inventing a writable path.

## PHP-FPM pool example

Pass only required, non-secret environment values to the pool:

~~~ini
[shop]
clear_env = yes
env[APP_ENV] = production
catch_workers_output = yes
~~~

Provide credentials through the host's protected secret mechanism. Do not put real passwords or API keys in a pool file committed to source control. Validate the active pool configuration and reload the correct FPM service. A CLI inspection does not prove that the FPM worker loaded the same settings.

## Safe failure handling in PHP

This illustrative endpoint handles an unexpected failure at one application boundary. Replace the simulated exception with the real operation and use the framework logger if the application has one.

~~~php
<?php
declare(strict_types=1);

header('Content-Type: application/json; charset=utf-8');

$requestId = bin2hex(random_bytes(8));

try {
    // Replace with the application operation.
    throw new RuntimeException('Simulated internal failure');
} catch (Throwable $exception) {
    // Do not automatically log the exception message. It may contain
    // paths, query values, or sensitive input.
    error_log(sprintf(
        'level=error event=request_failed request_id=%s exception_type=%s',
        $requestId,
        get_class($exception)
    ));

    http_response_code(500);

    echo json_encode(
        [
            'error' => 'An unexpected error occurred.',
            'requestId' => $requestId,
        ],
        JSON_THROW_ON_ERROR
    );
}
~~~

This example uses PHP 7.3 or later for JSON_THROW_ON_ERROR. The request ID is randomly generated for correlation and does not contain user data or a secret. In a framework application, use its established exception boundary instead of adding a catch block to every controller.

### Example visitor response

The request ID changes on each request:

~~~json
{
  "error": "An unexpected error occurred.",
  "requestId": "7c91a2d4e55f8901"
}
~~~

The visitor sees a short message and an identifier for support. The response does not reveal the exception message, stack trace, file path, or configuration.

### Example application log event

The event content written by the PHP example is:

~~~text
level=error event=request_failed request_id=7c91a2d4e55f8901 exception_type=RuntimeException
~~~

PHP or the platform may add a timestamp or prefix. Protect the destination, collect it reliably, rotate it, and monitor for repeated errors or storage problems. This event records the exception type but not its message.

## Deployment checks

1. Supply production secrets outside source control and outside the public document root.
2. Confirm the web root points only to the public directory.
3. Confirm PHP-FPM can read the release but cannot rewrite application source.
4. Confirm only intended cache or upload directories are writable.
5. Verify uploaded files cannot execute as server-side code.
6. Validate PHP and FPM configuration before reloading.
7. Reload the service that handles web requests and verify its effective behavior.
8. Make one successful request and one controlled failing request.
9. Confirm the visitor sees the generic response and the protected log contains the matching request ID.
10. Remove temporary diagnostic files before public traffic.

Do not leave a public phpinfo() page available during verification. Diagnostic output and logs can reveal internal implementation details.

## Expected result

- Visitor responses do not expose internal PHP details.
- A protected log event provides a request ID and safe error classification.
- PHP-FPM cannot modify application source.
- Only required runtime directories are writable.
- Settings are verified in the FPM process that actually handles requests.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [142 - Apache, Nginx, and PHP-FPM integration](../lessons/142-apache-nginx-and-php-fpm-integration.md) | [Lesson](../lessons/143-production-configuration-permissions-and-logging.md) | [Q&A](../qnas/143-production-configuration-permissions-and-logging.md) | [144 - Database migration and deployment strategies](../lessons/144-database-migration-and-deployment-strategies.md) |