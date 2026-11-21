# 105 - Command-line applications and arguments: Q&A

## Multiple-choice questions

### 1. What is PHP CLI?

- **A.** A SAPI for running PHP programs from a shell or terminal
- **B.** A database extension
- **C.** A web browser
- **D.** A syntax used only inside HTML

**Answer: A.** CLI is PHP's command-line SAPI. **Explanation:** It runs a process that can receive arguments and standard input, write output streams, and return an exit status.

### 2. What is normally stored at $argv[0] in a CLI script?

- **A.** The script name or path
- **B.** The first user-supplied argument
- **C.** The current PHP version
- **D.** The process exit status

**Answer: A.** The first argv entry is normally the script name or path. **Explanation:** User-supplied arguments usually begin at index 1, so check indexes before reading them.

### 3. What does $argc count?

- **A.** Every entry in $argv, including the script name
- **B.** Only the options that begin with two hyphens
- **C.** The number of PHP files in a directory
- **D.** The number of lines written to STDOUT

**Answer: A.** $argc counts the full argument vector, including its script-name entry. **Explanation:** A command expecting exactly one user argument usually expects $argc to equal 2.

### 4. Which constant represents the command's normal output stream?

- **A.** STDOUT
- **B.** STDERR
- **C.** STDIN
- **D.** PHP_SAPI

**Answer: A.** STDOUT is the normal output stream. **Explanation:** Keep result data on STDOUT so it can be redirected or piped.

### 5. Where should a command normally write diagnostic errors?

- **A.** STDERR
- **B.** STDOUT only
- **C.** The user's SQL database
- **D.** The $argv array

**Answer: A.** Diagnostics should normally go to STDERR. **Explanation:** This keeps errors from corrupting structured data such as JSON or CSV written to STDOUT.

### 6. What does a zero exit status conventionally mean?

- **A.** The command completed successfully
- **B.** The command received no arguments
- **C.** PHP is using the web SAPI
- **D.** The command printed an error

**Answer: A.** Zero conventionally indicates success; nonzero indicates failure. **Explanation:** Shell scripts and schedulers can use the status to decide whether to continue, retry, or alert.

### 7. What does a relative path such as data.json usually depend on when a CLI script opens it?

- **A.** The process's current working directory
- **B.** The PHP manual's location
- **C.** The web server's document root in every case
- **D.** The value of $argc

**Answer: A.** Relative paths are normally resolved from the process's current working directory. **Explanation:** Use __DIR__ for files packaged beside a PHP script.

### 8. What happens when getopt() encounters a non-option argument?

- **A.** Option parsing stops at the first non-option
- **B.** It converts the argument into a PHP variable
- **C.** It always moves the argument to the end of $argv
- **D.** It runs the argument as a shell command

**Answer: A.** getopt() stops parsing options at the first non-option. **Explanation:** Put options before positional arguments or use -- as a separator, then inspect the remaining arguments.

### 9. In a getopt() option specification, what does a single colon after an option mean?

- **A.** The option requires a value
- **B.** The option is a Boolean flag
- **C.** The option is unknown
- **D.** The option must appear twice

**Answer: A.** One colon means that the option requires a value. **Explanation:** Two colons indicate an optional value; no colon means the option is a flag.

### 10. Why should a command support a help option before opening files or changing data?

- **A.** Users can learn the syntax without causing work or side effects
- **B.** It makes invalid data safe automatically
- **C.** It changes the PHP SAPI to CLI
- **D.** It guarantees the script has database access

**Answer: A.** Help should explain usage without requiring the command's main work. **Explanation:** A help request should not unexpectedly read a file, prompt, or modify data.

### 11. Which is a safe way to pass a value to a SQL query in a PHP CLI application?

- **A.** Bind it through a prepared statement
- **B.** Concatenate it directly into SQL
- **C.** Call eval() on it
- **D.** Write it to STDERR

**Answer: A.** Prepared statements keep values separate from SQL syntax. **Explanation:** CLI arguments are untrusted input too; running locally does not make them safe.

### 12. What is the main purpose of a Unix shebang such as #!/usr/bin/env php?

- **A.** It helps the operating system find the interpreter when the file is run directly
- **B.** It configures a database table
- **C.** It sets the PHP namespace
- **D.** It makes a script executable on every operating system automatically

**Answer: A.** A shebang identifies the interpreter for direct execution in compatible Unix-like environments. **Explanation:** The file may also need executable permission, and Windows commonly runs scripts through php.exe instead.

## Short-answer questions

### 13. How can a script check which SAPI is executing it?

**Answer:** Compare PHP_SAPI with the expected SAPI name, such as cli. **Explanation:** The presence of argv alone is not a reliable SAPI check because similar values may exist in other contexts.

### 14. What is the difference between getcwd() and __DIR__?

**Answer:** getcwd() reports the process working directory; __DIR__ identifies the directory containing the current PHP file. **Explanation:** Use the working directory for paths the caller supplies relative to where the command is launched, and __DIR__ for files packaged beside the script.

### 15. Why must command-line arguments be validated?

**Answer:** They are external input and may be missing, malformed, out of range, or intentionally hostile. **Explanation:** Validate type, format, allowed values, path, and business meaning before using an argument.

### 16. How should a command handle a missing required argument?

**Answer:** Print concise usage or an error to STDERR and return a documented nonzero exit status. **Explanation:** Automation can detect the failure, and the user learns how to invoke the command correctly.

### 17. What do the three standard streams represent?

**Answer:** STDIN carries input, STDOUT carries normal results, and STDERR carries diagnostics. **Explanation:** Separating them allows callers to pipe or redirect result data without mixing it with errors.

### 18. What does the -- token commonly signal in a command's argument syntax?

**Answer:** It marks the end of options, so later tokens are treated as positional arguments. **Explanation:** The exact parser must support this behavior; do not assume every parser handles options after positional arguments the same way.

### 19. What is one limitation of getopt() that matters when a command has positional arguments?

**Answer:** It stops parsing options at the first non-option argument. **Explanation:** Options placed after that argument are not parsed. Put options first or use -- and inspect the unparsed portion.

### 20. Why should a CLI command document its exit statuses?

**Answer:** Callers need to distinguish success from invalid usage and runtime failure. **Explanation:** Shell scripts, schedulers, and deployment systems commonly make decisions based on process status rather than message text.

### 21. Why can a command that works interactively fail in a scheduler?

**Answer:** The scheduled process may have a different working directory, PATH, environment, user permissions, PHP configuration, or available input. **Explanation:** Make executable paths, files, required settings, and noninteractive behavior explicit.

### 22. Why should data output and error output not be mixed?

**Answer:** An error message on STDOUT can corrupt a structured result or pipeline. **Explanation:** Send normal result data to STDOUT and diagnostics to STDERR.

### 23. Why is building a shell command by concatenating raw arguments dangerous?

**Answer:** Shell metacharacters in an argument can change the command being executed. **Explanation:** Avoid a shell when possible; if starting a child process, use an API that accepts an argument array and validate each value for its purpose.

### 24. What should a destructive command consider before deleting or overwriting files?

**Answer:** It should make the target explicit, validate the path, check overwrite rules, offer a dry run for bulk actions, and report the result. **Explanation:** CLI operations can change many files quickly and may run with broad permissions.

## Practical scenarios

### 25. A developer checks if $argc equals 1 before reading $argv[1] as the first user argument. What is the bug?

**Answer:** With no user arguments, $argc is 1; the script must reject or handle that case before accessing index 1. **Explanation:** $argv[0] is the script name, so one user argument makes $argc equal 2.

### 26. A script emits JSON on STDOUT, but also prints a warning with echo. A downstream JSON parser fails. How should the script change?

**Answer:** Keep JSON data on STDOUT and write the warning to STDERR. **Explanation:** Callers can redirect or pipe STDOUT as machine-readable output while still seeing diagnostics separately.

### 27. A command accepts --limit=999999999 with no validation. What should be added?

**Answer:** Parse the value as an integer and enforce a documented sensible range before using it. **Explanation:** Syntax parsing alone does not establish that a value is safe or useful for the operation.

### 28. A PHP script opens config/settings.json, but fails when launched by a scheduler from another directory. How should the path be resolved if the file ships beside the script?

**Answer:** Build the path from __DIR__. **Explanation:** A relative path uses the process's current working directory, which can differ between an interactive terminal and a scheduler.

### 29. A command uses getopt() for --format and --limit, followed by a filename. A caller places --limit after the filename, and the limit is ignored. Why?

**Answer:** getopt() stops parsing at the first non-option. **Explanation:** Put options before the filename or use -- to mark the positional-argument section and handle the remaining arguments deliberately.

### 30. A scheduled task waits forever because the script calls fgets(STDIN) for confirmation. What is a better design?

**Answer:** Provide a noninteractive mode with explicit required arguments or a documented input file, and avoid prompting in unattended execution. **Explanation:** A scheduler may have no human to answer the prompt.

### 31. A web page can display a useful command's error message, but the command returns exit status 0 on failure. Why is this a problem?

**Answer:** Automation may treat the failed run as success because it relies on the process status. **Explanation:** Return a nonzero status when required work fails and document the statuses.

### 32. A command works with php in a terminal but fails under cron because the executable is not found. What should be checked?

**Answer:** Check the scheduler's PATH and use the expected PHP executable path explicitly if necessary. **Explanation:** Scheduled jobs may not load the user's interactive shell profile or environment.

### 33. A command writes both a report and a diagnostic to STDOUT, and a caller redirects output to report.txt. Where will the diagnostic go?

**Answer:** It will be mixed into report.txt and may corrupt it. **Explanation:** Write diagnostics to STDERR so the caller can redirect result data independently.

### 34. A CLI script accepts a filename argument and passes it directly into a shell command string. What is the risk and a safer direction?

**Answer:** Special shell characters can change the executed command. **Explanation:** Avoid constructing shell strings; use a process API that accepts an argument array, and validate the file path and permitted location.

### 35. A user invokes a tool with --help, but it first connects to the production database and then displays help. What should change?

**Answer:** Parse and handle --help before opening connections or performing other work. **Explanation:** Help should be a side-effect-free way to learn the command syntax.

### 36. A command runs a bulk delete and provides no dry-run or count of affected records. What improvements would make it safer to operate?

**Answer:** Validate the target, add a dry-run mode, preview or confirm the selection where appropriate, report how many records will change, and return a failure status if the operation is incomplete. **Explanation:** A CLI can execute destructive operations quickly and may be run unattended, so its impact and result must be clear.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [104 - Schema design and database migrations](../lessons/104-schema-design-and-database-migrations.md) | [Lesson](../lessons/105-command-line-applications-and-arguments.md) | [Example](../examples/105-command-line-applications-and-arguments.md) | [106 - Environment configuration and secrets](../lessons/106-environment-configuration-and-secrets.md) |
