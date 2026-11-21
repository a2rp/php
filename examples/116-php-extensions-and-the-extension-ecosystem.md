# 116 - PHP extensions and the extension ecosystem: Example

## Goal

Create a small PHP project that declares a required native extension, inspects the active CLI runtime, and checks a required capability before using it.

## Question

How can a project document an extension requirement and diagnose the case where the PHP runtime used by the application has not loaded that extension?

## Assumptions

- PHP 8.1 or newer and Composer are installed.
- Commands are run from the project directory.
- `mbstring` is required by this example; `curl` is optional.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
extension-check/
├── composer.json
└── check-runtime.php
~~~

## 2. Declare the extension requirement

Create `composer.json`:

~~~json
{
  "name": "learning/php-extension-check",
  "type": "project",
  "require": {
    "php": "^8.1",
    "ext-mbstring": "*"
  }
}
~~~

The `ext-mbstring` platform requirement tells Composer that the project needs the extension in the PHP environment used during dependency installation. It does not download a Composer library or enable the extension in every PHP SAPI. `curl` is optional here, so it is deliberately not declared as required.

## 3. Inspect and check the active PHP runtime

Create `check-runtime.php`:

~~~php
<?php

declare(strict_types=1);

$requiredExtensions = ['mbstring'];
$optionalExtensions = ['curl'];
$missingRequired = [];

$loadedIni = php_ini_loaded_file();
$scannedIni = php_ini_scanned_files();

printf("PHP version: %s\n", PHP_VERSION);
printf("SAPI: %s\n", PHP_SAPI);
printf("Loaded php.ini: %s\n", $loadedIni === false ? '(none)' : $loadedIni);
printf(
    "Scanned INI files: %s\n",
    $scannedIni === false ? '(none)' : trim(str_replace(PHP_EOL, ', ', $scannedIni))
);

foreach ($requiredExtensions as $extension) {
    $loaded = extension_loaded($extension);
    printf(
        "%s: %s (required)\n",
        $extension,
        $loaded ? 'loaded' : 'missing'
    );

    if (!$loaded) {
        $missingRequired[] = $extension;
    }
}

foreach ($optionalExtensions as $extension) {
    $loaded = extension_loaded($extension);
    printf(
        "%s: %s (optional)\n",
        $extension,
        $loaded ? 'loaded' : 'missing'
    );
}

if ($missingRequired !== []) {
    fwrite(
        STDERR,
        'Missing required extensions: ' . implode(', ', $missingRequired) . PHP_EOL
    );
    exit(1);
}

$name = 'CAFÉ';
printf("Lowercase check: %s\n", mb_strtolower($name, 'UTF-8'));
~~~

The script reports the PHP version, SAPI, and INI configuration for the process that runs it. It checks `mbstring` before calling `mb_strtolower()`, so a missing required module produces a clear startup failure instead of an undefined-function error. A missing optional `curl` module is reported but does not stop the script.

## 4. Install dependencies and run the preflight

From `extension-check/`, run:

~~~sh
composer install
composer check-platform-reqs
php check-runtime.php
~~~

Composer checks the declared `ext-mbstring` requirement while resolving or installing the project. `check-platform-reqs` checks the actual platform requirements of installed packages against the PHP environment running Composer. Then the PHP script reports what the CLI process loaded and exercises one `mbstring` function.

Example output when the CLI runtime has `mbstring` loaded and `curl` is absent:

~~~text
PHP version: 8.3.x
SAPI: cli
Loaded php.ini: <path to the CLI php.ini, or (none)>
Scanned INI files: <additional INI files, or (none)>
mbstring: loaded (required)
curl: missing (optional)
Lowercase check: café
~~~

The version and configuration paths depend on the PHP installation. The optional extension line will say `loaded` if `curl` is available. If `mbstring` is missing, the script writes the missing requirement to standard error and exits with a nonzero status before calling `mb_strtolower()`.

## 5. Resolve a missing module for the correct runtime

If `mbstring` is missing:

1. Run `php -v`, `php --ini`, and `php -m` to identify the CLI PHP binary and its configuration.
2. Install the compatible `mbstring` package using the platform's PHP package manager or enable the matching module according to that distribution's instructions.
3. If changing startup configuration, reload the relevant PHP service.
4. Run the inspection commands again against the intended runtime.
5. Also verify the PHP SAPI that serves the application; a CLI check does not verify PHP-FPM or a web-server process.

For a supported third-party development extension, PIE is the current installer. For example, a developer can install Xdebug into the PHP runtime targeted by PIE with:

~~~sh
pie install xdebug/xdebug
~~~

This installs a PHP extension, not a project library. Confirm the target PHP executable and keep development-only debugging tools out of production unless the deployment has a deliberate need for them. PECL is deprecated; use current PIE or platform package guidance instead.

## 6. Verify web PHP safely

If the CLI check passes but web requests still fail, confirm the web runtime's version, SAPI, loaded configuration file, and loaded modules. A local or access-restricted diagnostic route can temporarily run checks such as `extension_loaded('mbstring')` and `php_ini_loaded_file()`.

Do not leave a public `phpinfo()` page or diagnostic endpoint enabled. It can disclose detailed runtime and environment configuration. Remove the temporary diagnostic after confirming the web process, and restart or reload that service after startup configuration changes.

## Important notes

- Composer's `ext-*` requirement documents a runtime capability; it does not install or enable a native module.
- The script checks the CLI process. The production SAPI must be checked separately.
- `curl` is optional in this example and will not cause a failure when absent.
- The expected PHP version and INI paths are illustrative; they vary by machine.
- A module must match the PHP version/API, operating system, architecture, build settings, and required system libraries.
- PIE replaces deprecated PECL for supported third-party extension installation.
- Do not ignore platform requirements to make a dependency installation appear successful; resolve the actual environment mismatch.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [115 - Caching data and application output](../lessons/115-caching-data-and-application-output.md) | [Q&A](../qnas/116-php-extensions-and-the-extension-ecosystem.md) | [Lesson](../lessons/116-php-extensions-and-the-extension-ecosystem.md) | [117 - Testing PHP applications with PHPUnit](../lessons/117-testing-php-applications-with-phpunit.md) |
