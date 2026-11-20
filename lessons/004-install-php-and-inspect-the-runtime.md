# 004 - Install PHP and Inspect the Runtime

To write PHP, you need a PHP runtime installed on your computer. An editor can help you write source code, but it does not execute PHP. This chapter installs the command-line runtime and shows how to inspect its version, configuration, and extensions.

The steps depend on your operating system. Use a PHP version that is still supported and meets the requirements of the project you plan to run. The supported versions change over time; the previous chapter explains how to check them.

## What you will learn

- What you need to install to begin working with PHP
- How to make the PHP command available in a terminal
- What PATH does
- How to check the installed version
- How to find the loaded configuration file
- How to inspect installed extensions
- Why a terminal PHP runtime may differ from a website's runtime

## What is installed?

For this stage, the important part is the **PHP runtime**, including the command-line interface (CLI). The runtime parses and executes PHP programs. You do not need to install a particular editor, framework, or database to inspect the runtime.

A PHP installation can also include extensions, configuration files, and ways to connect PHP to a web server. Those components depend on the operating system and how PHP was installed. This chapter focuses on the CLI; running a PHP script and starting the local development server are covered next.

## Before installing

1. Check the operating system and processor architecture of your computer.
2. Check the current PHP support schedule and the version required by your project.
3. Choose an installation source that provides a maintained PHP build for your operating system.
4. Follow that source's instructions for the PHP CLI package or binary.
5. Keep track of where the PHP executable and configuration files are installed.

A system package manager may offer a version that is older than the newest PHP branch. Verify the version it installs instead of assuming the package name means it is current.

## Windows setup

One common Windows setup uses a prebuilt PHP distribution:

1. Choose a maintained PHP build from the official PHP for Windows distribution or a trusted package manager. Match it to your Windows architecture and intended use.
2. Extract it into a stable folder, for example **C:\tools\php**.
3. If the distribution provides **php.ini-development** and **php.ini-production**, copy **php.ini-development** to **php.ini** for local development. Do not use development error-display settings on a public production server.
4. Add the folder that contains **php.exe** to your Windows **Path** environment variable. Add the folder, not the path to the executable itself.
5. Close and reopen the terminal so it reads the updated environment.
6. Check that PowerShell can find PHP with **Get-Command -All php**.

If the terminal cannot find PHP, try the executable by its full path, such as **C:\tools\php\php.exe -v**. If that works, the installation is present and the remaining issue is usually the Path setting or an old terminal session.

A package manager can also install PHP on Windows. In either case, check which executable PowerShell resolves when more than one PHP installation exists.

## macOS and Linux setup

On macOS and Linux, install the PHP CLI package using a maintained package manager or installation method for that operating system. Package names and available versions differ between distributions. Check that the selected package meets the project's PHP version requirement, then open a new terminal and inspect the installation using the commands below.

Avoid compiling PHP from source for a first installation unless you specifically need to build or customize the runtime. Source builds require additional development tools and configuration.

## Confirm the terminal can find PHP

The PHP executable is commonly named **php**. Use the command for your terminal to find it:

~~~powershell
Get-Command -All php
~~~

On macOS or Linux, use:

~~~sh
command -v php
~~~

These commands show the executable location resolved by the current shell. If multiple paths are listed, the first resolved executable is the one the shell will usually run when you type **php**.

## Check the version

Run:

~~~sh
php -v
~~~

The output shows the PHP version and identifies the CLI runtime. Compare the installed version with the version your project requires and with PHP's current support schedule.

If the shell reports that PHP is not found, the executable may not be installed, or its containing folder may not be on PATH. On Windows, check the Path environment variable and open a fresh terminal. On macOS or Linux, check the package installation and the command path.

## Find the configuration file

PHP reads settings from a configuration file named **php.ini** when one is available. To see where the CLI searches for and loads configuration, run:

~~~sh
php --ini
~~~

The output reports the configuration search path, the loaded **php.ini** file, and any additional configuration files that were scanned. If it reports that no configuration file is loaded, PHP may still run with built-in defaults. Whether a configuration file is required depends on the installation and the application.

Do not guess which **php.ini** is active. Use **php --ini** to locate the file for the runtime you are inspecting before changing settings.

## Inspect extensions

Extensions add capabilities to PHP. A project may need extensions for tasks such as image processing, database access, or international text handling. The PHP installation may include some extensions already, while others may need to be installed or enabled separately.

List the extensions loaded by the CLI runtime:

~~~sh
php -m
~~~

To inspect one extension's settings, use:

~~~sh
php --ri json
~~~

The JSON extension is used here only as an example. A project's actual extension requirements depend on its code and dependencies. Having PHP installed does not mean every possible extension is enabled.

## Check the CLI and web runtimes separately

The commands in this chapter inspect the **CLI runtime**. A web server can use a different PHP installation, configuration file, or set of extensions. This is common when a computer has multiple PHP versions or when a hosting provider manages the web runtime separately.

If a PHP command works in a terminal but a website behaves differently, inspect the PHP configuration used by that web server as well. Do not publish a full configuration report from a public website: it can reveal environment details that should remain private.

## Common setup problems

| Symptom | What to check |
| --- | --- |
| The terminal says php is not recognized or not found. | Confirm PHP is installed, add the folder containing the executable to PATH, and open a new terminal. |
| The wrong PHP version appears. | Find all resolved PHP executables and check which one appears first in PATH. |
| A setting change has no effect. | Run php --ini and edit the configuration file used by the runtime you are checking. |
| An application reports a missing extension. | Compare its requirements with php -m, then install or enable the required extension for the correct runtime. |
| CLI behavior differs from website behavior. | The CLI and web server may use different PHP versions, configuration, or extensions. |

## Try it yourself

1. Find the path of the PHP executable your terminal resolves.
2. Run **php -v** and record the version and runtime type shown.
3. Run **php --ini** and identify the loaded configuration file.
4. Run **php -m** and choose one listed extension to inspect with **php --ri**.
5. Explain why a project can work in the terminal but fail on a web server.
6. If you have multiple PHP versions installed, determine which one your terminal uses first.

## Key points

- PHP must be installed as a runtime; an editor alone cannot execute it.
- PATH helps the terminal find the PHP executable.
- **php -v** reports the CLI version, **php --ini** reports configuration locations, and **php -m** lists loaded extensions.
- Check the PHP version against current support information and project requirements.
- Do not assume the CLI and web server use the same PHP runtime.
- Install or enable only the extensions the application needs.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [003 - PHP releases, versions, and compatibility](003-php-releases-versions-and-compatibility.md) | [Open Q&A](../qnas/004-install-php-and-inspect-the-runtime.md) | [Open example](../examples/004-install-php-and-inspect-the-runtime.md) | [005 - Run PHP from the command line and built-in server](005-run-php-from-the-command-line-and-built-in-server.md) |
