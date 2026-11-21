# 105 - Command-line applications and arguments

## Goals

After this lesson, you should be able to:

- Explain how PHP's CLI SAPI differs from PHP running behind a web server.
- Run a PHP script from a terminal and inspect the PHP executable and configuration.
- Read positional arguments and options safely.
- Choose between direct argument parsing and getopt().
- Read from standard input and write to standard output or standard error.
- Use clear help text, validation, and process exit codes.
- Build command-line tools that are predictable in scripts and pipelines.

## What is the PHP CLI?

The command-line interface (CLI) is a PHP SAPI designed to run PHP programs from a shell, terminal, task scheduler, or another process.

A CLI process does not have the same request and response lifecycle as a web application. It can receive command-line arguments and standard input, write to standard output and standard error, and return an exit status to the process that started it.

Common uses include:

- One-time maintenance scripts.
- Importing or exporting data.
- Generating reports.
- Scheduled jobs and queue workers.
- Local development tools.
- Automation in deployment pipelines.

CLI PHP and web-server PHP may use different PHP versions, configuration files, loaded extensions, environment variables, file permissions, and working directories. Check the runtime where the script will actually run.

## Check the CLI runtime

Run these commands in a terminal:

~~~sh
php -v
php --ini
php -m
~~~

They display the CLI version, loaded configuration files, and available extensions.

To verify the SAPI from PHP:

~~~php
<?php

echo PHP_SAPI, PHP_EOL;
~~~

When run with the CLI SAPI, this prints:

~~~text
cli
~~~

PHP provides other SAPIs, such as those used by a web server. Do not decide that a script is running from the command line just because argv exists; check PHP_SAPI when the distinction matters.

On Windows, the CLI executable is commonly php.exe. A system can have multiple PHP installations, so inspect which executable the shell resolves:

~~~sh
where.exe php
~~~

On Unix-like systems, use:

~~~sh
command -v php
~~~

If PHP is not found, install or configure the CLI executable and add its directory to PATH. The command php -v should show the expected runtime before using a script.

## Run a PHP file

Given this file:

~~~text
project/
└── hello.php
~~~

Run it from the project directory:

~~~sh
php hello.php
~~~

You can also pass an explicit path:

~~~sh
php /path/to/project/hello.php
~~~

On Windows, quote paths that contain spaces:

~~~sh
php "C:/Users/Example User/project/hello.php"
~~~

The shell starts the PHP process and passes each argument as a separate string. The shell's quoting rules decide how text becomes arguments, so the exact command syntax differs between PowerShell, Command Prompt, Bash, and other shells.

## Current working directory and script directory

A relative file path is usually resolved from the process's current working directory, not automatically from the directory containing the PHP script.

These are different values:

- getcwd() reports the current working directory.
- __DIR__ identifies the directory containing the current PHP file.

Use __DIR__ when a program needs a file packaged beside its source:

~~~php
$configPath = __DIR__ . '/config/defaults.json';
~~~

Use a user-provided path relative to the current working directory only when that is the intended command behavior. Document the rule so callers know where to run the command from.

Do not use chdir() to hide an unclear path policy. Resolve paths deliberately and report a useful error when a file is missing.

## The argument vector

In CLI execution, PHP exposes the command-line arguments through $argv when available. $argv is an array of strings. The first item, index 0, is normally the script name. User-supplied arguments begin at index 1.

$argc counts every item in $argv, including the script name. It is not the count of user-supplied arguments alone.

For example, with:

~~~sh
php greet.php Mina
~~~

the conceptual argument vector is:

~~~text
$argv[0] = greet.php
$argv[1] = Mina
$argc = 2
~~~

Read arguments by index only after checking that they exist. All command-line input is untrusted input: validate its count, format, range, and meaning before using it.

A minimal positional-argument script:

~~~php
<?php

declare(strict_types=1);

if ($argc !== 2) {
    fwrite(STDERR, "Usage: php greet.php <name>\n");
    exit(2);
}

$name = trim($argv[1]);

if ($name === '') {
    fwrite(STDERR, "Name must not be empty.\n");
    exit(2);
}

echo 'Hello, ', $name, '!', PHP_EOL;
~~~

The string still needs context-appropriate handling if it is later used in HTML, SQL, a shell command, a file path, or another format.

## Positional arguments and options

Positional arguments are identified by their position:

~~~sh
php import.php customers.csv
~~~

Options are named and describe a behavior or setting:

~~~sh
php report.php --format=json --limit=20
~~~

A command should document:

- Its command name and purpose.
- Required and optional arguments.
- Supported options and accepted values.
- Defaults.
- Whether option values may use a space or equals sign.
- How to request help.
- Exit statuses and important side effects.

Choose one clear option style. Long options such as --format=json are self-describing and work well in user-facing commands.

## Parse simple commands explicitly

For a small fixed command, a short manual parser can make accepted syntax and validation visible. Reject unknown options, duplicate options, missing values, and unexpected arguments instead of silently ignoring mistakes.

This pattern supports a required option value in either --name Mina or --name=Mina form:

~~~php
$arguments = $argv;
array_shift($arguments); // Remove the script name.

$options = [];
$allowedValueOptions = ['name'];
$allowedFlags = ['help'];

for ($index = 0; $index < count($arguments); $index++) {
    $token = $arguments[$index];

    if (!str_starts_with($token, '--')) {
        throw new InvalidArgumentException(
            'Unexpected positional argument.'
        );
    }

    $optionText = substr($token, 2);
    $equalsPosition = strpos($optionText, '=');

    if ($equalsPosition === false) {
        $optionName = $optionText;
        $value = null;
    } else {
        $optionName = substr($optionText, 0, $equalsPosition);
        $value = substr($optionText, $equalsPosition + 1);
    }

    if (in_array($optionName, $allowedFlags, true)) {
        if ($value !== null || array_key_exists($optionName, $options)) {
            throw new InvalidArgumentException(
                'Flag was given an invalid value or repeated.'
            );
        }

        $options[$optionName] = true;
        continue;
    }

    if (!in_array($optionName, $allowedValueOptions, true)) {
        throw new InvalidArgumentException('Unknown option.');
    }

    if (array_key_exists($optionName, $options)) {
        throw new InvalidArgumentException('Option was repeated.');
    }

    if ($value === null) {
        $index++;

        if (!isset($arguments[$index])) {
            throw new InvalidArgumentException(
                'Option is missing its value.'
            );
        }

        $value = $arguments[$index];
    }

    if ($value === '') {
        throw new InvalidArgumentException('Option value is empty.');
    }

    $options[$optionName] = $value;
}
~~~

This is a teaching pattern, not a universal command parser. Add the syntax the command needs, such as positional values after --, and test edge cases. For a larger CLI application, a maintained command-line library can provide subcommands, consistent help, and richer validation.

Do not use eval() to interpret an argument. Do not concatenate untrusted arguments into a shell command. Prefer a process API that accepts an argument array when a child process must be started, and validate each argument for its intended purpose.

## Parsing options with getopt()

getopt() parses short and long options declared in a specification.

- A short option character without a colon is a flag.
- One colon means the option requires a value.
- Two colons mean the option's value is optional.
- Long options use an array of names, with the same colon convention.

Example:

~~~php
$restIndex = null;

$options = getopt(
    'f:',
    ['format:', 'help'],
    $restIndex
);
~~~

This declares -f VALUE, --format VALUE, and --help. A flag is returned with the value false, so check whether it exists with array_key_exists() rather than treating its value as a Boolean true.

getopt() has important behavior to understand:

- Option parsing stops at the first non-option; later options are not parsed.
- Put options before positional arguments or use -- as an explicit separator.
- The third rest-index parameter is available from PHP 7.1 and identifies where unparsed arguments begin.
- Optional values have distinct rules; for long options, prefer an equals sign such as --color=always when supplying an optional value.
- getopt() does not replace validation of returned values or a clear usage message.

When the command needs strict control over syntax, unknown options, duplicates, and positional values, use a manual parser or a dedicated CLI library rather than assuming getopt() enforces every policy.

## Standard input, output, and error

A CLI process has three standard streams:

- STDIN receives input.
- STDOUT carries normal results.
- STDERR carries diagnostics, warnings, and usage errors.

Keep data output on STDOUT so it can be redirected or piped. Write errors to STDERR so they do not corrupt generated data.

~~~php
fwrite(STDOUT, "Report complete.\n");
fwrite(STDERR, "Warning: one row was skipped.\n");
~~~

Read a line from standard input:

~~~php
$line = fgets(STDIN);

if ($line === false) {
    fwrite(STDERR, "No input was received.\n");
    exit(1);
}

$value = trim($line);
~~~

A prompt is useful for a human but can block an automated job. For commands used in automation, accept required data through explicit arguments, options, or standard input. Do not prompt for a secret in a way that echoes it to the terminal; prefer a protected environment or secret-management mechanism where appropriate.

## Help and usage messages

A help message should explain how to invoke the command without requiring the user to read its source.

Example:

~~~text
Usage: php task-report.php [options] [--] [file]

Options:
  --format=table|json   Select the output format. Default: table.
  --limit=N             Show at most N records. Default: 100.
  --help                Display this help message.

Exit status:
  0  Command completed.
  1  Input or runtime failure.
  2  Invalid command usage.
~~~

Support --help before opening files or making changes. Keep help output stable and concise because scripts and users may rely on it.

## Exit statuses

The exit status is an integer reported to the operating system when the program ends.

- Use 0 for success.
- Use a nonzero status for failure.
- Use consistent documented values for different failure categories where useful.

In PHP, exit(0) or exit(1) ends a program with that integer status. A top-level return from a CLI script also ends execution, but a function return only returns from that function.

Example:

~~~php
if ($inputIsInvalid) {
    fwrite(STDERR, "Invalid input.\n");
    exit(2);
}

echo "Success.\n";
exit(0);
~~~

Check the status in the calling shell or automation. A message that looks successful is not enough if the process returned failure, and a warning printed to STDERR does not by itself determine the exit status.

## Shell quoting and safe input

The shell parses quoting, variables, redirects, and special characters before PHP receives arguments. The argv array contains the resulting values; it does not contain the original shell syntax.

Quote paths with spaces according to the shell in use. When writing instructions, provide commands for the intended shell or keep commands portable.

Treat all arguments and standard input as untrusted:

- Validate numbers as numbers and enforce a sensible range.
- Allowlist accepted formats, modes, and action names.
- Avoid dynamic SQL; use prepared statements for values.
- Do not pass raw input to eval().
- Avoid shell command construction by concatenating strings.
- Restrict file operations to intended locations and check failures.
- Keep secrets out of source code, command histories, and diagnostic output.

Escaping is context-specific. Shell escaping does not make a value safe for SQL, HTML, a path, or a log message.

## File output and destructive commands

A CLI tool can delete or overwrite data quickly and without a browser confirmation. Before changing files or records:

- Make the target path explicit.
- Reject an empty path or a path outside the allowed directory when appropriate.
- Check whether the target exists and whether overwriting is allowed.
- Prefer a dry-run option for bulk or destructive work.
- Report how many records or files changed.
- Use a backup or transaction strategy where possible.
- Return a failure status when the operation is incomplete.

Do not treat a CLI tool as trusted merely because it runs locally. It may be launched by a scheduler, a deployment system, or another process with different permissions.

## Environment and execution context

A scheduled command may not inherit the same working directory, PATH, environment variables, or user permissions as an interactive terminal. Make important dependencies explicit.

Useful practices:

- Use an absolute path to the expected PHP executable in automation when PATH may differ.
- Use __DIR__ for files shipped alongside the script.
- Check required extensions and environment values at startup.
- Set time zone and locale deliberately when output depends on them.
- Avoid relying on a user's interactive shell profile.
- Send logs to a controlled destination and rotate them.
- Make jobs safe to retry when they may be interrupted.

Environment configuration and secret handling are covered in the next chapter.

## A Unix shebang

On Unix-like systems, a script may start with a shebang:

~~~php
#!/usr/bin/env php
<?php

declare(strict_types=1);

echo "Hello from PHP CLI.\n";
~~~

After making the file executable, it can be invoked directly:

~~~sh
chmod +x hello
./hello
~~~

The shebang must be the first line. The /usr/bin/env form finds php through PATH. This convention is not a replacement for the PHP command and is not interpreted the same way by every platform. On Windows, use php.exe script.php unless a compatible shell and file association are configured.

## Common mistakes

### Counting argv entries as user arguments

$argc includes the script name at index 0. Compare it with the expected total count, or calculate the user argument count as $argc - 1.

### Assuming the web and CLI runtimes match

They can load different PHP versions, extensions, and configuration. Inspect each runtime separately.

### Resolving files from the script directory by accident

Relative paths normally depend on the current working directory. Use __DIR__ for files packaged with the script.

### Printing errors to standard output

Diagnostics mixed into a CSV or JSON result can corrupt a pipeline. Write errors to STDERR.

### Ignoring a nonzero exit status

Automation should inspect the process status and stop or alert when a command fails.

### Assuming getopt() parses options anywhere

It stops at the first non-option. Put options first or delimit positional arguments with --, and validate the returned values.

### Trusting a command-line value

Arguments are user-controlled data. Validate ranges, allowed names, file paths, and any SQL values before using them.

### Prompting during a scheduled job

An unattended task may wait forever for input. Provide an explicit non-interactive mode or required input channel.

## Key takeaways

- CLI is a PHP SAPI for scripts and shell applications, with a process-oriented input/output model.
- Verify the CLI PHP executable, version, configuration, and extensions.
- $argv includes the script name at index 0; $argc counts that item too.
- Relative file paths use the process working directory; use __DIR__ for files beside the script.
- Use STDOUT for results, STDERR for diagnostics, and documented nonzero exit statuses for failures.
- Validate every option and argument, including ranges and file paths.
- getopt() is useful for declared options but stops at the first non-option.
- Keep commands predictable for automation: document help, defaults, exit codes, and side effects.
- Do not concatenate untrusted input into shell commands or SQL.
- Make scheduled jobs explicit about paths, environment, permissions, and retry behavior.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [104 - Schema design and database migrations](104-schema-design-and-database-migrations.md) | [Q&A](../qnas/105-command-line-applications-and-arguments.md) | [Example](../examples/105-command-line-applications-and-arguments.md) | [106 - Environment configuration and secrets](106-environment-configuration-and-secrets.md) |
