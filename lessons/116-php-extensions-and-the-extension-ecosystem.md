# 116 - PHP extensions and the extension ecosystem

## Goals

After this lesson, you should be able to:

- Explain what a PHP extension adds and how it differs from a Composer library.
- Distinguish core functionality, extensions bundled with PHP, third-party extensions, and Zend extensions.
- Inspect the extensions and configuration loaded by a specific PHP runtime.
- Install and enable an extension using the appropriate package manager or PIE.
- Diagnose common PHP version, architecture, configuration-file, and SAPI mismatches.
- Declare required extensions so a project can report missing runtime capabilities.
- Treat native extensions and FFI as code that needs compatibility and security review.

## What is a PHP extension?

A PHP extension adds functionality to the PHP runtime. It can expose functions, classes, constants, stream wrappers, or other capabilities to PHP code. Examples include `mbstring` for multibyte string operations, `curl` for network requests, `gd` for image operations, `pdo_mysql` for a MySQL PDO driver, and OPcache for compiled bytecode caching.

Extensions are often implemented in native code and can interact closely with PHP and the operating system. Some are built into a PHP binary; others are loaded as shared modules when PHP starts. A missing extension is therefore usually an environment or deployment issue, not a missing PHP source file.

## Extensions and Composer packages are different

A Composer library is usually PHP code installed into a project, often under `vendor/`. A PHP extension is installed into a PHP runtime and must be available to the process running the application.

| Concern | Composer package | PHP extension |
| --- | --- | --- |
| Installed into | A project or shared PHP library directory | A PHP runtime's extension location and configuration |
| Typical declaration | A package name in `composer.json` | An `ext-*` platform requirement in `composer.json` |
| Loaded by | Composer autoloading | PHP startup configuration or a built-in module |
| Example | A date or HTTP client library | `ext-curl`, `ext-mbstring`, or `ext-pdo_mysql` |

A PHP library may require an extension to exist. Installing the library does not install that extension for the runtime. Composer can resolve PHP packages while the application still fails at runtime because a required extension is absent.

Declare required runtime capabilities in the project so the dependency manager can check them:

~~~json
{
  "require": {
    "php": "^8.2",
    "ext-mbstring": "*",
    "ext-pdo_mysql": "*"
  }
}
~~~

Use the PHP version range and extension requirements that the application actually supports. Do not add requirements for optional capabilities unless the project truly cannot work without them. On a deployment or CI machine, `composer check-platform-reqs` checks requirements against the real PHP runtime and extensions available there.

## Extension categories

### Core functionality

Some facilities are part of PHP's core and cannot be removed by choosing a different extension build. The core language, arrays, classes, error handling, and other built-in facilities belong here.

### Extensions bundled with PHP

Many extensions are distributed with PHP but may still be optional in a particular build or package. PDO drivers, `mbstring`, `gd`, and `intl` are examples of extensions that may need explicit installation or enabling depending on the platform and PHP distribution.

### Third-party extensions

Extensions maintained outside the PHP source distribution add capabilities such as debugging, profiling, or specialized integrations. Check whether an extension supports the PHP version and platform you deploy before installing it.

### Zend extensions

Zend extensions integrate at a lower runtime level and may provide facilities such as debugging, profiling, or opcode caching. Their startup directive is commonly `zend_extension`, while regular PHP modules use `extension`. Follow the instructions for the specific extension; do not change one directive into the other by guesswork.

## Inspect the PHP runtime you are using

A computer can have multiple PHP installations. The command-line PHP binary, a web server module, PHP-FPM, a container, and a hosting service may each use a different version, extension directory, or configuration file.

Start by inspecting the CLI runtime:

~~~sh
php -v
php --ini
php -m
php --ri mbstring
~~~

- `php -v` shows the version and build information for that executable.
- `php --ini` reports the configuration file and scanned configuration directories used by the CLI process.
- `php -m` lists loaded modules and Zend modules.
- `php --ri name` displays information about a loaded extension when it provides such information.

From PHP code, `extension_loaded('mbstring')` checks whether an extension is loaded. `get_loaded_extensions()` returns names of loaded extensions. `function_exists('mb_strtolower')` checks for one function, but a function check is narrower than confirming the module and does not establish that every feature is configured and usable.

If the CLI shows an extension but a web request does not, inspect the actual web runtime. It may load a different `php.ini`, use a separate scanned INI directory, or run a different PHP version. A temporarily protected diagnostic page can report the web SAPI and loaded configuration, but `phpinfo()` exposes detailed environment information. Do not leave it publicly accessible; remove it after diagnosis. Restart or reload the relevant web service after startup configuration changes.

## Install and enable an extension

First identify the exact PHP executable and version that needs the capability. Then choose an installation path supported by that environment:

1. Prefer the operating system or platform package manager when it provides a compatible extension package for the deployed PHP version.
2. Use PIE, the PHP Installer for Extensions, for supported third-party extensions. PECL is deprecated; PIE replaces it as the recommended installer for extensions outside the PHP distribution.
3. Follow the platform's documented build process only when a compatible package is unavailable and you can maintain the native build and its system dependencies.

For a third-party extension published for PIE, installation uses the extension package name. For example, the Xdebug extension package can be installed with:

~~~sh
pie install xdebug/xdebug
~~~

PIE installs an extension for a PHP runtime. It does not add a normal application library under `vendor/`. Confirm which PHP installation PIE targets, especially on a development machine with multiple binaries. In automated builds, make the chosen extension package and target runtime explicit rather than relying on an interactive prompt.

Depending on the operating system, a package manager or extension installer may also enable the module. Otherwise, PHP's startup configuration may contain a directive such as:

~~~ini
extension=mbstring
zend_extension=opcache
~~~

Use the exact module name and directive specified by the extension's installation instructions. PHP loads its main `php.ini` when the process starts and may scan additional INI files. For web-server processes, a restart or reload may be needed before the change takes effect. Do not edit a production runtime casually: test the change in a matching environment and keep a rollback path.

## Compatibility matters

A native extension must be compatible with the PHP runtime and operating system that loads it. A prebuilt binary may need to match:

- The PHP version and module API it was built for.
- The operating system and processor architecture.
- Whether the PHP build is thread-safe or non-thread-safe, where the platform distinguishes them.
- Debug or release build settings and other build options.
- Required system libraries and their compatible versions.

A DLL or shared object that exists on disk can still fail to load if one of these does not match. On Windows, check the extension directory, DLL architecture, PHP build type, and dependent DLLs. On Unix-like systems, check the module path, build tools, PHP development tools, and required system libraries. Use package-manager builds where possible because they can coordinate compatible components.

After installing, verify the module in the exact runtime that will run the application. A successful `php -m` in a terminal does not prove that PHP-FPM or the web server has loaded it.

## Handle missing optional capabilities deliberately

If an extension is optional, isolate code that depends on it and give a clear fallback or a helpful error. Check the capability at the application boundary or deployment health check rather than scattering silent checks throughout the code.

~~~php
if (!extension_loaded('curl')) {
    throw new RuntimeException('The cURL extension is required for outbound HTTP requests.');
}
~~~

For a required extension, a deployment check should fail early with a useful message rather than letting a production request fail later with an undefined function error. A Composer platform requirement helps communicate the dependency, but it cannot install every native system library or guarantee that a web SAPI is configured exactly like the CLI.

## Native code and security

An extension runs with the privileges of the PHP process and can affect process stability. Install only extensions the application needs, keep them maintained, and obtain builds from trusted sources. Review compatibility and security advisories when upgrading PHP or an extension.

FFI is a separate facility for calling native libraries from PHP. It is powerful and can expose low-level memory and operating-system interfaces. It is not a general replacement for a maintained PHP extension and should only be used when the developer understands the native API and its safety requirements. Restrict its availability through runtime configuration where it is not needed.

Do not use `dl()` as an application-level installation strategy. Extension availability should be managed as part of the PHP runtime or deployment image, so startup failures are visible and all workers use a consistent configuration.

## A practical extension checklist

Before adding an extension, ask:

1. Does the application require a native capability, or would a maintained Composer library be enough?
2. Is the extension already included and loaded in every target runtime?
3. Is it actively maintained and compatible with the PHP version, operating system, architecture, and SAPI?
4. Is the extension requirement declared and checked in CI or deployment?
5. Does the installation method fit the environment: system package, PIE, or a maintained build?
6. Have the web process and CLI process both been checked after configuration changes?
7. Is the extension needed in production, or only in development and debugging?
8. Is there a safe failure or fallback if an optional capability is missing?

## Key takeaways

- Extensions add capabilities to the PHP runtime; Composer packages are usually installed into a project.
- Declare required extensions as `ext-*` platform requirements and check the actual deployment runtime.
- `php --ini`, `php -m`, `php --ri`, and `extension_loaded()` help diagnose what a particular process loaded.
- CLI, web server, PHP-FPM, and containers can use different PHP versions and INI files.
- Prefer compatible system packages or PIE for third-party extension installation. PECL is deprecated.
- Native extension builds must match the PHP runtime, platform, and required libraries.
- Install only needed extensions, keep them maintained, and treat FFI as a low-level capability.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [115 - Caching data and application output](115-caching-data-and-application-output.md) | [Q&A](../qnas/116-php-extensions-and-the-extension-ecosystem.md) | [Example](../examples/116-php-extensions-and-the-extension-ecosystem.md) | [117 - Testing PHP applications with PHPUnit](117-testing-php-applications-with-phpunit.md) |
