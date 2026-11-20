# 003 - PHP Releases, Versions, and Compatibility: Complete Example

This example checks whether the PHP runtime running a command-line script meets a minimum version. It demonstrates one small compatibility check; it does not prove that a complete application or its dependencies are compatible.

## Question

How can a PHP script compare the installed runtime with a project's minimum PHP version?

## Goal

Print the current PHP version, compare it with a minimum version, and stop with a clear message when the runtime is too old.

## Complete code

Save this as **check-runtime.php**:

~~~php
<?php

$minimumVersion = '8.4.0';
$currentVersion = PHP_VERSION;

echo 'Current PHP version: ', $currentVersion, PHP_EOL;

if (version_compare($currentVersion, $minimumVersion, '<')) {
    echo 'Required: PHP ', $minimumVersion, ' or newer.', PHP_EOL;
    exit(1);
}

echo 'This runtime meets the minimum version.', PHP_EOL;
~~~

GitHub displays the code as text. It does not execute PHP in this Markdown page.

## Run it locally

1. Save the file as **check-runtime.php**.
2. Open a terminal in the folder that contains it.
3. Run:

~~~sh
php check-runtime.php
~~~

This is a command-line example. It checks the PHP runtime used by that terminal. A web server may use a different PHP runtime.

## Expected output

On PHP 8.6.0, the script prints:

~~~text
Current PHP version: 8.6.0
This runtime meets the minimum version.
~~~

On PHP 8.3.0, it prints:

~~~text
Current PHP version: 8.3.0
Required: PHP 8.4.0 or newer.
~~~

The exact current version line depends on the PHP installation used to run the script.

## How it works

### Store the minimum and read the current runtime

~~~php
$minimumVersion = '8.4.0';
$currentVersion = PHP_VERSION;
~~~

The minimum is a string that represents the example project's requirement. **PHP_VERSION** provides the version of the runtime executing this script.

### Compare version strings

~~~php
version_compare($currentVersion, $minimumVersion, '<')
~~~

The **version_compare** function understands version strings and the comparison operator. Here, the condition is true when the current runtime is older than 8.4.0.

If the condition is true, the script prints the requirement and exits with a nonzero status. Otherwise, it prints that the minimum version check passed.

### Why the check is limited

This check only compares the runtime with one minimum version. It does not check whether:

- Required PHP extensions are installed
- The application's libraries support this runtime
- Configuration matches the application's needs
- The code's behavior is correct on this runtime
- The web server uses the same runtime as the terminal

A project should also declare its version requirements in its dependency or deployment configuration and run its checks in the environments it supports.

## Try changing it

1. Change **8.4.0** to another minimum version and run the script again.
2. Compare results from two PHP installations if you have more than one.
3. Explain why a passing version check does not prove that every dependency works.
4. Explain why a terminal version check may not describe the PHP runtime serving a website.
5. Look at the support snapshot in the lesson and explain why support status is different from a numeric minimum check.

## Key takeaways

- **PHP_VERSION** reports the runtime version as text.
- **version_compare** can compare version strings.
- A minimum version check can reject a runtime that is too old.
- A passing check only verifies that threshold, not the whole application's compatibility.
- CLI and web-server PHP runtimes may differ.
- Version requirements should be checked alongside dependencies, extensions, configuration, and application behavior.

## Chapter navigation

| Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- |
| [Read this lesson](../lessons/003-php-releases-versions-and-compatibility.md) | [Open Q&A](../qnas/003-php-releases-versions-and-compatibility.md) | [004 - Install PHP and inspect the runtime](../lessons/004-install-php-and-inspect-the-runtime.md) |
