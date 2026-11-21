# Production configuration, permissions, and logging — Q&A

Review production PHP settings, filesystem access, secrets, and logging with these questions. Each answer follows its question so you can check your reasoning immediately.

## Multiple-choice questions

### 1. Which production error configuration is the safest default?

- A. Display every PHP error and disable logging.
- B. Disable both display and logging.
- C. Log reported errors and do not display detailed PHP errors to visitors.
- D. Suppress every warning with the error-control operator.
**Answer: C.** A production application should retain diagnostic information internally while keeping stack traces and server details out of public responses. A common starting point is **error_reporting = E_ALL**, **display_errors = Off**, and **log_errors = On**.

### 2. What does **display_errors = Off** do?

- A. It prevents PHP from detecting errors.
- B. It stops PHP from sending reported errors in the response.
- C. It guarantees that every error is written to an application log.
- D. It disables PHP-FPM.
**Answer: B.** Display and logging are separate. Configure logging deliberately; turning display off by itself does not guarantee that an error is recorded.

### 3. Why is **php --ini** not enough to verify a website's production settings?

- A. It only works for JavaScript applications.
- B. It shows the settings for the CLI PHP runtime, which may differ from PHP-FPM.
- C. It changes the FPM pool automatically.
- D. It only checks file permissions.
**Answer: B.** Different PHP installations or SAPIs can load different configuration files. Verify the PHP-FPM service and pool that handle the web requests.

### 4. Which files should normally be writable by PHP-FPM?

- A. Every application file and directory.
- B. Only explicitly designated runtime locations, such as needed cache or upload storage.
- C. The deployment scripts and server configuration.
- D. Every file under the web root.
**Answer: B.** Keep code and deployment configuration read-only to the request-handling process. Grant write access only where the application actually needs it.

### 5. What is a safe way to provide a production API key?

- A. Commit it in a PHP source file.
- B. Put it in a public configuration page.
- C. Supply it through a protected deployment or secret-management mechanism.
- D. Add it to every log entry for easier debugging.
**Answer: C.** Secrets should not be committed, served publicly, or logged. Give access only to the identities that need the secret.

## Short-answer questions

### 6. Explain the difference between **error_reporting**, **display_errors**, and **log_errors**.

**Answer:** **error_reporting** selects which PHP errors are reported. **display_errors** controls whether those errors are included in the response. **log_errors** enables sending them to the configured logging destination. Production normally reports broadly, logs internally, and does not display detailed errors to visitors.

### 7. Why should a public **phpinfo()** page be removed after diagnostics?

**Answer:** It can reveal runtime versions, configuration paths, extensions, and environment details. Use a restricted diagnostic method and remove temporary files before public traffic reaches the deployment.

### 8. What should a safe error response contain?

**Answer:** An appropriate status code, a brief non-sensitive message, and optionally an opaque request ID for support. It should not include a stack trace, SQL, credentials, filesystem paths, or raw exception details.

### 9. Name four data types that should not appear in ordinary application logs.

**Answer:** Examples include passwords, API tokens, cookies or session IDs, authorization headers, payment-card data, and unnecessary sensitive personal data. Redact sensitive values before writing events.

### 10. What is a request ID used for?

**Answer:** It correlates a visitor's safe error response with a protected internal log event. It should be opaque and must not encode a password, session secret, or other private value.

### 11. Why must logs be rotated and monitored?

**Answer:** Unbounded logs can fill disk space and disrupt the application. Monitoring also helps detect logging failures, repeated errors, and storage problems before they cause an outage.

### 12. What is log injection?

**Answer:** It occurs when untrusted input, such as a newline or control character, is written without validation and makes forged lines appear to be real events. Validate, normalize, or encode user-controlled values before logging them.

## Configuration and permissions scenarios

### 13. A developer disables displayed errors and sees no errors in the logs either. What should they check?

**Answer:** Confirm that **log_errors** is enabled, the effective PHP-FPM configuration was loaded, and the configured destination is correct and writable or collected by the platform. Also confirm that the correct FPM service was reloaded. Do not turn public error display back on as the production fix.

### 14. A cache write fails, so someone plans to make the whole project world-writable. Is that appropriate? What should they do?

**Answer:** No. A world-writable mode such as **777** lets every local account write and may allow an attacker to change application code. Identify the exact cache directory, assign it to the appropriate runtime identity or group, grant only the required access there, and keep the source tree read-only to PHP-FPM.

### 15. Are example modes such as **0750** for directories and **0640** for files universal requirements?

**Answer:** No. They are illustrative modes, not universal values. The correct owner, group, and modes depend on the operating system, deployment user, FPM user, and hosting model. The important rule is least privilege: grant only the required read, write, and traversal access.

### 16. What must be true when user-uploaded files are served by a website?

**Answer:** Store them in a controlled upload location and configure the web server so uploaded files cannot execute as PHP or other server-side code. Do not trust a filename extension as the only safeguard.

### 17. PHP-FPM uses environment clearing. How should the application receive a required environment label?

**Answer:** Keep environment clearing enabled when appropriate and explicitly pass the required non-secret variable to the pool or platform. Provide credentials through protected secret configuration rather than hard-coding them into a committed pool file.

### 18. Why should an FPM slow log be protected?

**Answer:** It may include script paths and stack information that reveal internal application details. Restrict access, choose a protected destination, and set retention deliberately.

### 19. Do **open_basedir** and disabled functions replace filesystem permissions or process isolation?

**Answer:** No. They may provide additional defense in depth, but they do not replace operating-system permissions, isolation, a correctly scoped document root, or application authorization.

## Practical review

### 20. A production request returns a generic 500 page. What should an operator inspect first?

**Answer:** Use the request ID to locate the protected application event. Check the PHP-FPM and web-server logs, confirm the relevant logging destinations are collecting output, and investigate the failure without exposing detailed diagnostics to the visitor. Avoid logging credentials or dumping full request data while debugging.

### 21. Put these deployment checks in a sensible order: verify runtime behavior, set protected configuration, check permissions, validate configuration, reload the service.

**Answer:**

1. Set protected production configuration and secrets.
2. Check ownership and permissions for source, secrets, logs, uploads, and runtime directories.
3. Validate the PHP and PHP-FPM configuration.
4. Reload the correct service.
5. Verify behavior with a successful request and a controlled failure.

**Explanation:** Configuration and permissions must be in place before the service loads them. The final check confirms that the running service behaves as intended.

### 22. What should a safe internal error event record when an unexpected exception occurs?

**Answer:** Record enough non-sensitive context to find and classify the failure, such as a timestamp, severity, opaque request ID, stable event name, and safe exception type. Do not automatically include the exception message if it might contain a secret, query value, path, or user input.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [142 - Apache, Nginx, and PHP-FPM integration](../lessons/142-apache-nginx-and-php-fpm-integration.md) | [Lesson](../lessons/143-production-configuration-permissions-and-logging.md) | [Example](../examples/143-production-configuration-permissions-and-logging.md) | [144 - Database migration and deployment strategies](../lessons/144-database-migration-and-deployment-strategies.md) |