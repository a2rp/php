# 004 - Install PHP and Inspect the Runtime: Complete Example

This example creates a small command-line report about the PHP runtime currently executing a script. It helps you confirm which PHP installation, configuration, and extensions are active in that terminal.

## Question

How can you inspect the PHP runtime and configuration used by the current command-line session?

## Goal

Create a PHP script that reports the runtime version, version ID, SAPI, loaded configuration file, and whether the optional mbstring extension is loaded.

## Complete code

Save this as **runtime-report.php**:

~~~php
<?php

$configurationFile = php_ini_loaded_file();
$mbstringLoaded = extension_loaded('mbstring');

echo 'PHP version: ', PHP_VERSION, PHP_EOL;
echo 'PHP version ID: ', PHP_VERSION_ID, PHP_EOL;
echo 'Runtime interface: ', PHP_SAPI, PHP_EOL;
echo 'Loaded php.ini: ', ($configurationFile !== false ? $configurationFile : '(none)'), PHP_EOL;
echo 'mbstring extension: ', ($mbstringLoaded ? 'loaded' : 'not loaded'), PHP_EOL;
~~~

GitHub displays this code as text. It does not run the PHP program inside the Markdown page.

## Run it locally

1. Save the file as **runtime-report.php**.
2. Open a terminal in the folder containing the file.
3. Run the script with the PHP command:

~~~sh
php runtime-report.php
~~~

To inspect the same CLI installation with built-in commands, also run:

~~~sh
php -v
php --ini
php -m
~~~

On PowerShell, locate the executable with:

~~~powershell
Get-Command -All php
~~~

On macOS or Linux, use:

~~~sh
command -v php
~~~

## Illustrative output

A particular PHP 8.6.0 CLI installation might report:

~~~text
PHP version: 8.6.0
PHP version ID: 80600
Runtime interface: cli
Loaded php.ini: C:\tools\php\php.ini
mbstring extension: loaded
~~~

This is sample output, not a promise about your computer. Your version, configuration path, and mbstring status can differ. If no php.ini is loaded, the script prints **(none)**. If mbstring is not enabled for this runtime, it prints **not loaded**.

## How the script works

### Read the version and runtime interface

**PHP_VERSION** returns the runtime version as text. **PHP_VERSION_ID** returns a numeric version value. **PHP_SAPI** identifies the Server API used by the running PHP process. When this script is run from a terminal, the value is usually **cli**.

### Find the loaded configuration file

**php_ini_loaded_file()** returns the path of the configuration file used by this runtime, or false when there is no loaded php.ini. The conditional expression turns that false value into the readable text **(none)**.

### Check an optional extension

**extension_loaded('mbstring')** returns true when that extension is loaded in this runtime. The example reports the result without assuming that every PHP installation has mbstring enabled.

The built-in commands provide related information: **php -v** reports the CLI version, **php --ini** lists configuration locations, and **php -m** lists loaded modules.

## Why your output can differ

- PHP may be installed in a different folder.
- More than one PHP executable may be on the computer, and PATH determines which one the shell finds first.
- A PHP installation may have no loaded php.ini or may load configuration from another directory.
- Extensions can be built in, enabled, disabled, or unavailable in a particular installation.
- A web server can use a different PHP runtime from the command line.

This report describes only the runtime that executed this script. It does not verify the PHP version, configuration, or extensions used by a web server.

## Try changing it

1. Compare the script's version with the output from **php -v**.
2. Compare its configuration path with **php --ini**.
3. Check whether **mbstring** appears in **php -m**.
4. If more than one PHP executable is installed, compare **Get-Command -All php** or **command -v php** with the report.
5. Explain why the report cannot prove which PHP runtime a website uses.

## Key takeaways

- A PHP script can inspect its own runtime with built-in constants and functions.
- The output describes the PHP executable that ran the script.
- **php --ini** and **php -m** help inspect configuration and loaded extensions.
- Optional extensions may differ between PHP installations.
- CLI information does not automatically describe a web server's PHP runtime.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [003 - PHP releases, versions, and compatibility](../lessons/003-php-releases-versions-and-compatibility.md) | [Read this lesson](../lessons/004-install-php-and-inspect-the-runtime.md) | [Open Q&A](../qnas/004-install-php-and-inspect-the-runtime.md) | [005 - Run PHP from the command line and built-in server](../lessons/005-run-php-from-the-command-line-and-built-in-server.md) |
