# 106 - Environment configuration and secrets: Q&A

## Multiple-choice questions

### 1. Why should database credentials be kept outside PHP source code?

- **A.** The same code can run in different environments without embedding credentials
- **B.** PHP cannot read strings from source files
- **C.** Environment variables automatically encrypt every secret
- **D.** It makes database queries run faster

**Answer: A.** External configuration lets development, testing, and production use different values with the same application code. **Explanation:** Environment variables are only a delivery mechanism; they do not automatically encrypt or securely store a secret.

### 2. What does getenv('DB_HOST') return when that variable is not available?

- **A.** false
- **B.** An empty PHP array in every version
- **C.** The local machine name
- **D.** The value of DB_NAME

**Answer: A.** getenv() returns false when the named environment variable does not exist. **Explanation:** Check for false explicitly, and decide whether an empty string is also invalid for that setting.

### 3. Why can $_ENV be empty even when an environment value is accessible through getenv()?

- **A.** PHP's variables_order configuration may not populate $_ENV
- **B.** getenv() only works in JavaScript
- **C.** $_ENV contains only command-line options
- **D.** Environment values are always deleted by PDO

**Answer: A.** Population of $_ENV depends on PHP configuration, including variables_order. **Explanation:** Use and document a consistent lookup method, and verify it in the actual SAPI.

### 4. What should a configuration example file such as .env.example contain?

- **A.** Variable names and safe placeholders, not real credentials
- **B.** A production database password
- **C.** A developer's personal access token
- **D.** A dump of every variable on the server

**Answer: A.** An example file should document the required names and safe defaults or blank placeholders. **Explanation:** It is normally committed, so real secrets must not be placed in it.

### 5. Does PHP automatically parse a file named .env?

- **A.** No; a library, framework, or application code must load it
- **B.** Yes, whenever the file is in the project root
- **C.** Only when a database extension is enabled
- **D.** Only on Windows

**Answer: A.** PHP does not load .env files by itself. **Explanation:** Understand and configure the dependency responsible for loading a local environment file.

### 6. What is the default value of PHP-FPM's clear_env pool setting?

- **A.** yes
- **B.** no
- **C.** It is always inherited from $_ENV
- **D.** It cannot be configured

**Answer: A.** PHP-FPM defaults clear_env to yes. **Explanation:** FPM clears the worker environment before adding values explicitly configured for the pool, so a shell variable may not reach a web request.

### 7. Which is a safe diagnostic for a missing password setting?

- **A.** “DB_PASSWORD is not configured”
- **B.** Print the full password to STDERR
- **C.** var_dump the complete configuration array
- **D.** Add the password to a public URL

**Answer: A.** Report the setting name or its presence without printing the value. **Explanation:** STDERR and server logs can still be read by others or copied, so they must not contain secrets.

### 8. What does filter_var('false', FILTER_VALIDATE_BOOLEAN, FILTER_NULL_ON_FAILURE) return?

- **A.** Boolean false
- **B.** Boolean true
- **C.** The string 'false'
- **D.** null

**Answer: A.** The recognized string 'false' is parsed as Boolean false. **Explanation:** Do not use a plain truthiness check for configuration strings; nonempty strings such as 'false' are truthy in PHP.

### 9. Which practice best limits the damage from a leaked database credential?

- **A.** Give each application a separate account with only necessary permissions
- **B.** Share an administrator account with every application
- **C.** Store a copy of the secret in every source file
- **D.** Use the same password indefinitely

**Answer: A.** Separate least-privilege credentials limit access and make rotation more manageable. **Explanation:** A credential should grant only the actions and resources that the application needs.

### 10. What should happen after a real credential is committed to Git?

- **A.** Revoke or rotate it; deleting the visible file alone is not enough
- **B.** Add the file to .gitignore and take no other action
- **C.** Rename the variable in PHP
- **D.** Wait until the next release

**Answer: A.** Treat the credential as potentially copied and revoke or rotate it. **Explanation:** The value may remain in Git history, clones, caches, logs, or other copies.

### 11. Why should an application fail fast when a required secret is missing?

- **A.** It avoids accepting work in an incomplete or unsafe configuration
- **B.** It automatically creates a secure secret
- **C.** It ensures $_ENV is always populated
- **D.** It makes secret rotation unnecessary

**Answer: A.** Startup validation makes the configuration problem visible before a partial request or job fails later. **Explanation:** Do not silently use a production credential or endpoint fallback.

### 12. Which statement best describes environment variables as a secret mechanism?

- **A.** They can deliver values to a process but are not a vault by themselves
- **B.** They are encrypted at rest by PHP automatically
- **C.** They cannot be inherited by another process
- **D.** They are safe to print because they are temporary

**Answer: A.** Environment variables are a common configuration delivery method, but they need runtime access controls. **Explanation:** Secrets may be exposed through process inspection, debugging, child processes, dumps, or logs.

## Short-answer questions

### 13. What is the difference between configuration and a secret?

**Answer:** Configuration describes how the application should run; a secret is configuration that grants access or protects data. **Explanation:** A host or feature setting may be non-sensitive, while a password, token, signing key, or encryption key needs stronger protection.

### 14. How should a PHP application read a required environment value?

**Answer:** Call getenv() and explicitly reject false or an empty value when the setting is required. **Explanation:** The caller should not assume a missing setting has a safe default, and the error must not disclose the secret itself.

### 15. Why should a password helper avoid trimming the value automatically?

**Answer:** Whitespace may be part of the secret. **Explanation:** Validate a value according to its documented format; normalize ordinary identifiers only when that is the intended policy.

### 16. Why should the application validate environment values before using them?

**Answer:** Environment values are strings and may be missing or malformed. **Explanation:** Convert and check booleans, integers, ports, URLs, names, and ranges before passing them to database or service APIs.

### 17. What is a safe use for .env.example?

**Answer:** It documents required environment variable names using blank values or clearly fake placeholders. **Explanation:** It helps developers configure their local environment without placing a real credential in the repository.

### 18. Why does adding .env to .gitignore not fix a secret that was already committed?

**Answer:** Ignore rules affect future untracked files; they do not erase existing commits or copies. **Explanation:** Revoke or rotate the exposed credential, then assess repository history and distribution.

### 19. Why may a variable set in an interactive shell be missing in a PHP-FPM request?

**Answer:** The web service may have started with a different environment, and FPM clears worker environments by default unless values are explicitly added. **Explanation:** Configure required values through the service's supported deployment or pool mechanism and verify the web SAPI.

### 20. Why should an application avoid a silent default for a missing production signing key?

**Answer:** A shared or fake fallback key can make signatures predictable or invalidate security assumptions. **Explanation:** Fail startup and require the deployment to provide a proper key.

### 21. How should an application report whether a secret is configured?

**Answer:** It may report presence or absence, but should never print the secret value. **Explanation:** A boolean status helps diagnose configuration while reducing accidental disclosure in logs and output.

### 22. Why should local, test, staging, and production credentials be separate?

**Answer:** Separate credentials limit cross-environment access and reduce damage from accidental exposure. **Explanation:** Tests should not be able to access production services, and local debugging should not reveal production data.

### 23. What is a basic secret rotation sequence?

**Answer:** Issue a replacement, configure the application, verify it works, and revoke the old credential. **Explanation:** A planned overlap can support zero-downtime rotation, but the old value should be removed promptly.

### 24. Why should the entire environment not be printed for debugging?

**Answer:** The process environment may contain unrelated passwords, tokens, and service credentials. **Explanation:** Report only the specific setting's presence and safe metadata needed to diagnose the issue.

## Practical scenarios

### 25. getenv('APP_DEBUG') returns the string 'false', but the application enters its debug branch using if ($debugValue). Why?

**Answer:** The string is nonempty and therefore truthy in PHP. **Explanation:** Parse it explicitly with a Boolean validator and reject values outside the supported spellings.

### 26. $_ENV['DB_HOST'] is missing, but a CLI test shows getenv('DB_HOST') works. What should be checked?

**Answer:** Check variables_order and the actual SAPI configuration, then use the application's documented environment lookup method consistently. **Explanation:** PHP can expose a process value through getenv() without populating $_ENV.

### 27. A database variable is present in the terminal but missing in the web app running under PHP-FPM. What should the operator check?

**Answer:** Check the FPM pool configuration and how the service receives environment values, including clear_env and explicit env entries. **Explanation:** FPM clears worker environments by default; do not assume an interactive shell's variables are inherited.

### 28. A developer commits .env with a real API token and then deletes it in the next commit. What should happen next?

**Answer:** Revoke or rotate the token and investigate where it may have been copied. **Explanation:** The previous commit can remain in history and existing clones, so deleting the working-tree file does not restore secrecy.

### 29. The application logs “Invalid password: [value]” while diagnosing a connection failure. What is the correction?

**Answer:** Remove the password from the log and report a sanitized error such as a failed database connection plus a safe code or setting name. **Explanation:** Logs are another output channel and may be accessible beyond the application process.

### 30. APP_DEBUG is missing in production, and code silently defaults it to true. Why is this dangerous, and what should happen instead?

**Answer:** Debug output may reveal stack traces, filesystem paths, or configuration details. **Explanation:** Validate the required setting and fail startup or use an explicitly safe production default of false.

### 31. A local .env file is listed in .gitignore, but git status still shows it as tracked. Why?

**Answer:** The file was already added to Git before the ignore rule was applied. **Explanation:** Remove it from tracking, rotate any real credentials, and inspect the history; do not assume the ignore rule is retroactive.

### 32. A developer uses getenv('DB_PORT') directly as a port without conversion. What should the code do?

**Answer:** Validate the string as an integer and enforce the allowed port range before connecting. **Explanation:** Environment values arrive as strings and malformed input should fail during configuration validation.

### 33. A service has a .env.example with DB_PASSWORD=production-secret so that deployments “work out of the box.” What should change?

**Answer:** Replace it with a blank value or obviously fake placeholder and provision the real secret outside the repository. **Explanation:** Example files are commonly committed and copied; they must never contain active credentials.

### 34. An operator rotates a database password in the secret manager but the running application still uses the old password. What should be checked?

**Answer:** Check whether the PHP process has received the new setting and whether the service must be reloaded or restarted. **Explanation:** Existing workers may retain their start-time environment until recycled; verify the new credential before revoking the old one.

### 35. A developer calls phpinfo() on a public production URL to check environment configuration. What is safer?

**Answer:** Remove public phpinfo() and use an access-controlled diagnostic that reports only whether specific settings are present. **Explanation:** phpinfo() can expose extensive runtime and request details, including potentially sensitive configuration.

### 36. A production database connection fails, and the team wants to dump the complete configuration array to logs. What is a safer diagnostic plan?

**Answer:** Log a sanitized failure code and non-secret context, such as the environment name and whether required settings are present. **Explanation:** Never serialize passwords, tokens, or the whole environment; detailed logs must also be access-controlled.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [105 - Command-line applications and arguments](../lessons/105-command-line-applications-and-arguments.md) | [Lesson](../lessons/106-environment-configuration-and-secrets.md) | [Example](../examples/106-environment-configuration-and-secrets.md) | [107 - Composer and dependency management](../lessons/107-composer-and-dependency-management.md) |
