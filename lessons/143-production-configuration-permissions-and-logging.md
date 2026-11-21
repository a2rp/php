# Production configuration, permissions, and logging

A PHP application needs different settings and safeguards when it serves real users. Development should help you find mistakes quickly. Production should protect secrets, avoid exposing internal details, allow only the required processes to write files, and preserve enough diagnostic information for the team to investigate failures.

This chapter explains how to configure PHP and PHP-FPM for production, control filesystem access, and collect useful logs without placing private data in them.

## Learning goals

After this chapter, you should be able to:

- Separate application code from environment-specific configuration and secrets.
- Configure PHP errors so users see safe responses while maintainers can investigate failures.
- Explain why CLI PHP settings may differ from PHP-FPM settings.
- Give the web and PHP processes only the filesystem access they need.
- Write useful, privacy-conscious logs and protect them from tampering.
- Validate configuration and permissions as part of deployment.

## Development and production have different needs

A local development environment should make problems visible to the developer. A production environment serves users and must not disclose stack traces, filesystem paths, SQL details, credentials, or configuration values in HTTP responses.

| Concern | Development | Production |
| --- | --- | --- |
| PHP errors | Visible locally when useful | Logged internally, not displayed to visitors |
| Configuration | Convenient local settings | Supplied by the deployment environment |
| Secrets | Local-only test values | Protected platform or secret-store values |
| Code files | Developer may edit them | Runtime process should normally read, not modify, them |
| Writable data | Local development storage | Only explicitly designated runtime directories |
| Logs | Convenient local output | Restricted, rotated, monitored, and retained deliberately |

Do not copy a development configuration into production without reviewing each setting.

## Keep code, configuration, and secrets separate

Application code should not contain environment-specific values such as database passwords, API tokens, signing keys, or production hostnames. Supply these values through the hosting platform, a secret manager, or a protected configuration file that is outside the public document root and excluded from source control.

Good practices include:

- Commit a configuration template containing names and safe example values, never real credentials.
- Store production secrets in the deployment platform or an approved secret store.
- Grant access to secrets only to the deployment and runtime identities that need them.
- Keep secret files outside the public document root and use restrictive ownership and permissions.
- Redact secret values from deployment output, diagnostics, and logs.
- Rotate any credential that was accidentally committed, logged, or exposed. Removing it from the latest commit does not make the old value safe again.
- Keep production, staging, and local values separate.

A .env file is not automatically safe. If a project uses one, ensure the real file is not committed, cannot be fetched over HTTP, and is readable only by the required processes. A stronger deployment arrangement keeps it outside the web root entirely.

The web document root should contain only files intended to be public, often a dedicated public directory. Source files, deployment configuration, logs, backups, and secrets should live elsewhere.

## Configure PHP errors safely

For production, record useful PHP errors internally and keep them out of visitor responses. A production PHP configuration commonly includes:

~~~ini
error_reporting = E_ALL
display_errors = Off
display_startup_errors = Off
log_errors = On
~~~

error_reporting controls which errors PHP reports. display_errors controls whether reported errors are sent in the response. log_errors enables sending them to the configured logging destination. These are separate controls: turning off display does not mean errors should be ignored.

Choose the error destination for the deployment:

- On a traditional server, configure a protected error-log destination that the PHP-FPM worker can write to.
- In a container or managed platform, sending errors to the process error stream may be preferable when the platform collects and protects that stream.
- Do not point logs to a path that is inside the public document root.
- Confirm log ownership, available disk space, retention, and rotation.

Do not suppress errors with @ or empty exception handlers to make production appear quiet. Handle errors deliberately, return a safe response, and retain enough internal context to investigate.

## Verify the settings for the actual PHP runtime

A machine may have multiple PHP installations and configuration files. The command-line SAPI used by php can load different configuration from the PHP-FPM service that handles web requests. A CLI check alone does not prove that a browser request uses the same settings.

During deployment:

1. Identify which PHP binary and configuration the command line uses.
2. Identify the configuration and pool loaded by the running PHP-FPM service.
3. Validate the edited configuration with the appropriate PHP or FPM configuration check.
4. Reload or restart the correct service using the platform's deployment procedure.
5. Confirm behavior through a restricted health or diagnostic check.
6. Remove temporary diagnostic files before making the deployment public.

Commands such as php --ini and php -i can help inspect the CLI runtime. Treat their output as sensitive because it may include paths and configuration details. Never leave a public phpinfo() page on a production site: it can expose runtime versions, paths, extensions, and environment details.

A setting change has no effect until the relevant process loads it. Check the running service, not only the file you edited.

## Pass only intended environment variables to PHP-FPM

PHP-FPM pools can control which environment variables workers receive. With environment clearing enabled, a worker does not inherit every variable from the service manager by default. Explicitly pass only the variables the application needs.

~~~ini
[shop]
clear_env = yes
env[APP_ENV] = production
catch_workers_output = yes
~~~

This example passes a non-secret environment label. Configure secrets through the deployment platform's protected mechanism; do not put credentials directly into a pool configuration committed to the repository.

catch_workers_output can route worker output to the FPM logging destination. Choose and verify the destination deliberately. Do not assume that output from PHP, the web server, FPM, and the application all goes to one place.

## Apply least privilege to files and directories

A web request should not give PHP permission to rewrite application source code. Separate identities and writable locations so that a compromised request cannot freely alter the entire application or deployment.

A typical access model is:

| Path or resource | Deployment identity | PHP-FPM identity |
| --- | --- | --- |
| Application source and configuration templates | Can update during a controlled release | Can read what it needs; normally cannot write |
| Public assets | Can publish during a release | Can read and serve |
| Upload storage | Usually not needed | Can write only if uploads are part of the application |
| Cache or generated runtime data | Usually not needed during requests | Can write only to the designated directories |
| Secrets | Can provision through a protected deployment process | Can read only the secrets required at runtime |
| Logs | Can configure or rotate | Can write through the chosen logging destination |

The exact user, group, and mode depend on the operating system and hosting platform. Decide them explicitly instead of applying broad permissions such as world-writable access. A mode such as 777 lets every local account write and is almost never an appropriate application fix.

Also:

- Keep upload and runtime-data directories outside the executable source tree when practical.
- If uploaded files must be served, configure the web server so they cannot be executed as PHP or another server-side program.
- Do not let request-handling code change its own source, deployment scripts, or server configuration.
- Make log and secret paths writable only by their intended owners.
- Check parent-directory permissions too; a secure-looking file can still be replaced if an untrusted user can write to its containing directory.
- Recheck permissions after deployments, archive extraction, container builds, and ownership changes.

PHP restrictions such as open_basedir or disabled functions may add defense in depth, but they do not replace operating-system permissions, process isolation, a correct document root, or application authorization.

## Design useful application logs

A useful operational log helps answer what happened, when, and which request was affected. It should not reproduce the entire request or expose user secrets.

A structured event might contain:

~~~text
timestamp=2026-11-21T12:30:00Z level=error request_id=req-7c91 event=payment_service_unavailable
~~~

Useful fields can include:

- A timestamp with a consistent timezone, preferably UTC.
- A severity level such as debug, info, warning, or error.
- A generated request or correlation ID.
- A stable event name and safe exception type.
- A service or environment name, when helpful.
- A status code or other non-sensitive outcome.

Avoid logging:

- Passwords, access tokens, API keys, signing secrets, or database credentials.
- Cookies, session IDs, authorization headers, or reset links.
- Full payment-card data or sensitive personal data.
- Full request bodies by default.
- Unfiltered exception messages when they may contain query values, paths, or user-supplied data.

Use an opaque request ID to connect a user's safe error response to a protected internal event. Validate or encode user-controlled values before putting them into log records. Newlines and control characters in untrusted input can make forged log entries appear to be genuine events.

## Protect the logging pipeline

A log is useful only if it is available, trustworthy, and handled within the application's privacy requirements.

- Restrict who can read, change, and delete logs.
- Keep logs outside the public document root.
- Use rotation or a managed logging service so files do not fill the disk.
- Set retention and deletion rules that match operational and privacy needs.
- Monitor collection failures, storage capacity, and repeated application errors.
- Protect centralized log access with appropriate authentication and audit controls.
- Avoid logging the same exception at every layer; record it once at the boundary that can add useful request context.

PHP-FPM can also produce access, worker-output, or slow-request logs. Enable only the logs that help operate the service, protect their destinations, and review whether their format includes request details you do not want to retain. A slow log can include script paths and stack information, so access to it should be limited.

## Return safe errors to visitors

Visitors need a clear outcome and a way to report the problem. They do not need a stack trace or the server's directory layout.

A safe error response should:

- Use an appropriate HTTP status code.
- Give a short, non-sensitive explanation.
- Include an opaque request ID when support needs to locate the internal event.
- Avoid exception messages, SQL statements, filesystem paths, and configuration values.

Catch unexpected failures at a deliberate application boundary. Log a safe event with the request ID, return a generic response, and do not continue as if an important operation succeeded. Authentication, authorization, and validation errors should be handled according to their intended behavior rather than all being converted into server errors.

## A deployment checklist

Before routing production traffic to a release:

1. Supply production configuration and secrets from protected deployment settings.
2. Confirm the web document root contains only public files.
3. Check PHP and PHP-FPM production error settings.
4. Confirm the intended FPM pool receives only required environment variables.
5. Confirm source files are not writable by the request-handling process.
6. Confirm only designated storage and cache paths are writable.
7. Verify uploaded files cannot execute as server-side code.
8. Confirm logs are private, collected, rotated, and monitored.
9. Validate configuration and reload the correct service.
10. Test one successful request and one controlled failure; confirm the visitor sees a safe response and the internal log contains a useful, redacted event.
11. Remove temporary diagnostic endpoints and files.

## Common mistakes

- Turning off both error display and error logging, leaving failures invisible.
- Assuming a CLI configuration check proves the FPM configuration is correct.
- Publishing phpinfo() or a diagnostic file and forgetting to remove it.
- Storing production credentials in source control or inside the public directory.
- Making the entire application writable to fix one cache or upload failure.
- Using 777 as a quick permissions workaround.
- Logging complete requests, cookies, tokens, passwords, or raw exception details.
- Enabling detailed slow or access logs without protecting and rotating them.
- Changing an INI file but not reloading the process that uses it.

## Key takeaways

- Production should log PHP errors internally while keeping detailed errors out of visitor responses.
- CLI and PHP-FPM may load different configuration; verify the runtime that serves requests.
- Keep secrets outside source control and the public document root.
- Make application code read-only to the request-handling identity and grant write access only to specific runtime directories.
- Logs should provide safe, useful context without credentials or unnecessary personal data.
- Protect, rotate, monitor, and retain logs deliberately.
- Validate the complete deployment after configuration or permission changes.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [142 - Apache, Nginx, and PHP-FPM integration](142-apache-nginx-and-php-fpm-integration.md) | [Q&A](../qnas/143-production-configuration-permissions-and-logging.md) | [Example](../examples/143-production-configuration-permissions-and-logging.md) | [144 - Database migration and deployment strategies](144-database-migration-and-deployment-strategies.md) |