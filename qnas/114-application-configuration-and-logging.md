# 114 - Application configuration and logging: Q&A

## Multiple-choice questions

### 1. What is application configuration for?

- **A.** Values that can change how an application runs without changing its business rules
- **B.** Replacing every application service with a global variable
- **C.** Storing user passwords in source code
- **D.** Logging every request body

**Answer: A.** Configuration describes settings such as environment, endpoints, timeouts, and log levels. **Explanation:** Keep those settings separate from the business behavior that uses them.

### 2. What does `getenv('MISSING_VALUE')` return when the variable is not available?

- **A.** `false`
- **B.** An empty array
- **C.** The integer `0`
- **D.** A PHP exception automatically

**Answer: A.** `getenv()` returns `false` when a named environment variable does not exist. **Explanation:** Check for absence before parsing or using the value.

### 3. Why should environment values be parsed and validated?

- **A.** Environment variables are strings and may be missing or malformed
- **B.** PHP automatically converts every setting to the right type
- **C.** Environment variables cannot hold text
- **D.** Parsing prevents all deployment errors

**Answer: A.** A string such as `"truue"` should not silently become an unintended boolean. **Explanation:** Convert values deliberately and fail early when a required setting is invalid.

### 4. Does a plain PHP process automatically parse a `.env` file?

- **A.** No; a framework or configured dotenv loader must do that
- **B.** Yes, every PHP request loads one from the repository root
- **C.** Yes, but only when Composer is installed
- **D.** No PHP application can use environment variables

**Answer: A.** `.env` parsing is provided by a framework or an explicitly configured package, not by PHP automatically. **Explanation:** Without a loader, the runtime must supply the process variables.

### 5. Which file should contain real production credentials?

- **A.** An approved deployment environment or secret store, not a committed example file
- **B.** A public `.env.example`
- **C.** A PHP class committed to source control
- **D.** A log message for easier troubleshooting

**Answer: A.** Credentials should be supplied through a protected deployment mechanism. **Explanation:** Example files should contain only harmless placeholders.

### 6. What is the highest-severity PSR-3 log level?

- **A.** `emergency`
- **B.** `debug`
- **C.** `info`
- **D.** `notice`

**Answer: A.** PSR-3 places `emergency` above `alert`, `critical`, `error`, `warning`, `notice`, `info`, and `debug`. **Explanation:** Use consistent levels so operators can filter and alert meaningfully.

### 7. Where should variable values generally go in a structured log call?

- **A.** The context array
- **B.** A concatenated secret-bearing message string
- **C.** The PHP source filename
- **D.** The HTTP status code only

**Answer: A.** A stable message with named context values is easier to search and process. **Explanation:** Context can carry safe identifiers and diagnostic values without rebuilding the message text for each event.

### 8. What context key is commonly used to pass an exception to a PSR-3 logger?

- **A.** `exception`
- **B.** `stack`
- **C.** `fatal`
- **D.** `throwable_text`

**Answer: A.** Use the `exception` context key when passing an exception object to a PSR-3 logger. **Explanation:** Logger implementations can recognize and format it appropriately.

### 9. Which value should generally not be written to application logs?

- **A.** An authentication token or password
- **B.** A safe order identifier
- **C.** A request correlation identifier
- **D.** A failure category

**Answer: A.** Secrets such as tokens and passwords should not be logged. **Explanation:** Logs may be retained, exported, and accessed by more people or systems than source code.

### 10. What should a web client receive when an unexpected server exception occurs in production?

- **A.** A safe generic error response
- **B.** The complete stack trace and environment variables
- **C.** The database connection string
- **D.** A success response with the exception text

**Answer: A.** Return a safe error response and preserve appropriate diagnostic detail in protected logs. **Explanation:** Public error output can expose implementation details and secrets.

### 11. What is a formal audit trail different from?

- **A.** Ordinary diagnostic logs
- **B.** A PHP class
- **C.** An environment variable
- **D.** A Composer autoloader

**Answer: A.** Audit records may need guarantees for completeness, integrity, access, and retention beyond ordinary operational logs. **Explanation:** Do not assume an application log automatically satisfies audit requirements.

### 12. Why configure log rotation or retention?

- **A.** To control storage growth and how long records containing operational data are kept
- **B.** To make every log entry an audit record
- **C.** To encrypt all application secrets automatically
- **D.** To replace log access permissions

**Answer: A.** Unbounded logs can consume storage and retain information longer than necessary. **Explanation:** Rotation, retention, and permissions are part of operating a logging destination.

## Short-answer questions

### 13. Name three values that commonly belong in configuration.

**Answer:** The active environment, an API endpoint, and the minimum log level are examples. **Explanation:** Configuration values may differ between local, test, and production environments.

### 14. Why build a typed settings object during application startup?

**Answer:** It centralizes parsing and validation and gives the application predictable types. **Explanation:** Invalid required configuration can fail before the affected request or job begins.

### 15. What is a reasonable purpose for a checked-in `.env.example` file?

**Answer:** Document required environment variable names with harmless sample values. **Explanation:** It should not contain real credentials or production secrets.

### 16. What does the log level control?

**Answer:** Which severities a logger or its handlers accept for output. **Explanation:** It is separate from debug mode, which can change the application's diagnostic behavior.

### 17. Why use stable log messages with structured context?

**Answer:** It keeps event text consistent while making varying values easier to search, filter, and process. **Explanation:** Context fields can be indexed independently by log tools.

### 18. What is a safe reason to include a request ID in logs?

**Answer:** It can connect related log records for one request without copying the entire request into each record. **Explanation:** The identifier should not itself expose sensitive data.

### 19. Why should an exception usually be logged at one useful boundary instead of every layer?

**Answer:** Repeated logging creates duplicate entries for one failure. **Explanation:** A boundary can add relevant operation context, while centralized error handling creates a safe response.

### 20. How should a service get the configuration value it needs?

**Answer:** Receive that setting or a cohesive settings object from the application boundary. **Explanation:** The service should not repeatedly read all environment variables or parse files itself.

### 21. Why are file permissions important for a log destination?

**Answer:** Log records can contain private operational or personal data. **Explanation:** Restrict access to the processes and operators who need it.

### 22. What is the risk of using `phpinfo()` or dumping all environment variables on a public page?

**Answer:** It can expose credentials, endpoints, and other deployment details. **Explanation:** Diagnostic tools that reveal environment state must not be publicly available in production.

### 23. Is a debug log level the same as enabling debug mode?

**Answer:** No. **Explanation:** Debug mode changes application diagnostics; log level controls which severities are written by configured handlers.

### 24. Why might an audit trail require a separate design?

**Answer:** It may need stronger completeness, integrity, access, and retention guarantees than ordinary diagnostic logging. **Explanation:** An operational log is not automatically a compliant or tamper-resistant audit record.

## Practical and scenario questions

### 25. A required database setting is missing, but the application silently uses an empty string and fails on the first request. What should change?

**Answer:** Validate the required value during startup and fail with a clear configuration error. **Explanation:** Early failure is easier to diagnose than a delayed connection error in an unrelated code path.

### 26. `APP_DEBUG=truue` is interpreted as truthy because the code checks whether the string is non-empty. What is a safer approach?

**Answer:** Parse the value using a strict boolean parser and reject an unrecognized value. **Explanation:** A typo should not silently enable or disable debug behavior.

### 27. A repository includes `.env.example` with a working database password for a shared test server. Is that safe?

**Answer:** No. Replace it with a harmless placeholder and supply the real credential through a protected runtime or secret store. **Explanation:** A committed example file is visible to repository readers and can be copied elsewhere.

### 28. A service calls `getenv('PAYMENT_TOKEN')` on every payment operation. How can configuration handling improve?

**Answer:** Load and validate the token at startup, then inject it only into the payment adapter that needs it. **Explanation:** This centralizes configuration and makes the service dependency visible.

### 29. A developer logs the full `Authorization` header to troubleshoot login failures. What should they log instead?

**Answer:** A safe request identifier, failure category, and minimal non-sensitive context. **Explanation:** Authentication headers contain credentials that must not be retained in ordinary logs.

### 30. One exception is logged in a repository, service, controller, and global handler. What should the team do?

**Answer:** Choose one useful logging boundary, include the necessary safe context there, and avoid duplicate entries as the exception propagates. **Explanation:** Repeated traces make one incident appear to be several failures.

### 31. A production page displays a stack trace only when a request fails. What should be changed?

**Answer:** Disable public debug output and return a generic response; keep appropriate diagnostics in protected logs. **Explanation:** Stack traces can reveal paths, queries, and implementation details.

### 32. A team labels every routine successful event as `error`. What is the likely operational effect?

**Answer:** Error alerts become noisy and real failures become harder to notice. **Explanation:** Select levels based on event severity and use `info` or `notice` for ordinary significant events.

### 33. A log file is writable by every user on the host and grows without limit. What controls are missing?

**Answer:** Restrictive permissions, rotation or centralized collection, retention, and monitoring. **Explanation:** Logs consume storage and can contain sensitive data.

### 34. A framework automatically loads `.env`, but a plain PHP script in the same repository does not. Is the script broken?

**Answer:** Not necessarily; plain PHP does not automatically parse `.env` files. **Explanation:** The script needs the framework or a configured dotenv loader, or the runtime must provide environment variables directly.

### 35. A log context includes a full user object with password hash, email, and reset token. What is a safer replacement?

**Answer:** Include only the specific safe identifier and diagnostic fields needed for the event, and exclude secrets and unnecessary personal data. **Explanation:** Minimize what is written to retained operational records.

### 36. The product needs an immutable record of who changed a financial approval and when. Can a regular debug log alone be assumed to meet this need?

**Answer:** No. Design a specific audit record with the required integrity, access, and retention guarantees. **Explanation:** Diagnostic logs and formal audit trails serve different purposes.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [113 - Dependency injection and service design](../lessons/113-dependency-injection-and-service-design.md) | [Lesson](../lessons/114-application-configuration-and-logging.md) | [Example](../examples/114-application-configuration-and-logging.md) | [115 - Caching data and application output](../lessons/115-caching-data-and-application-output.md) |
