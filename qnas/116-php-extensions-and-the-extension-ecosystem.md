# 116 - PHP extensions and the extension ecosystem: Q&A

## Multiple-choice questions

### 1. What does a PHP extension generally provide?

- **A.** Capabilities exposed to PHP by the runtime, such as functions, classes, or drivers
- **B.** A Composer autoloader for every project
- **C.** A replacement for the PHP language
- **D.** A guarantee that every server has identical configuration

**Answer: A.** Extensions add capabilities to the PHP runtime. **Explanation:** They may be built into PHP or loaded as modules when PHP starts.

### 2. Where is a typical Composer library installed?

- **A.** Into a project dependency directory such as `vendor/`
- **B.** Into the PHP binary itself
- **C.** Into the operating system kernel
- **D.** Into the PHP extension directory in every case

**Answer: A.** Composer normally installs PHP packages for a project and provides their autoloading. **Explanation:** A native extension is installed into a PHP runtime and must be enabled there.

### 3. What does `ext-mbstring` in `composer.json` declare?

- **A.** A PHP runtime requirement for the `mbstring` extension
- **B.** A Composer library named `mbstring`
- **C.** A request to compile PHP from source
- **D.** A runtime setting that enables every extension

**Answer: A.** `ext-*` entries describe extension platform requirements. **Explanation:** The requirement helps dependency checks report a missing capability; it does not install the extension.

### 4. Which command lists modules loaded by the CLI PHP executable?

- **A.** `php -m`
- **B.** `php --packages`
- **C.** `composer modules`
- **D.** `php --server-extensions`

**Answer: A.** `php -m` lists loaded modules for that CLI runtime. **Explanation:** It does not prove that PHP-FPM, a web server, or another PHP binary has the same modules.

### 5. What does `php --ini` help identify?

- **A.** The configuration file and additional INI directories used by that PHP process
- **B.** Every extension source file in the project
- **C.** The active database schema
- **D.** Composer's autoload map only

**Answer: A.** The command reports INI configuration information for the selected PHP executable. **Explanation:** A different SAPI or binary may read a different configuration.

### 6. Which tool is currently recommended for installing supported third-party PHP extensions?

- **A.** PIE, the PHP Installer for Extensions
- **B.** PECL as the recommended new installer
- **C.** Composer's normal `require` command for every native module
- **D.** `dl()` inside a web request

**Answer: A.** PIE replaces PECL as the recommended extension installer; PECL is deprecated. **Explanation:** Composer installs project packages, while PIE installs PHP extensions into a runtime.

### 7. What does `extension_loaded('curl')` check?

- **A.** Whether the `curl` extension is loaded in the current PHP process
- **B.** Whether a remote website responds
- **C.** Whether a Composer package is installed
- **D.** Whether a cURL command-line binary exists

**Answer: A.** `extension_loaded()` reports whether a named PHP extension is loaded. **Explanation:** The result applies to the runtime executing that code.

### 8. What is a typical startup directive for a regular PHP extension?

- **A.** `extension=mbstring`
- **B.** `require=mbstring`
- **C.** `composer=mbstring`
- **D.** `include_path=mbstring`

**Answer: A.** Regular extensions are commonly enabled with an `extension` directive. **Explanation:** Follow the specific extension's installation instructions; Zend extensions commonly use `zend_extension`.

### 9. Why might `php -m` show an extension that a web request cannot use?

- **A.** CLI and web processes may use different PHP binaries or configuration files
- **B.** PHP modules only work in Composer scripts
- **C.** The extension is automatically removed on HTTP requests
- **D.** A loaded CLI extension proves the web server is configured correctly

**Answer: A.** Different SAPIs or installations can load different modules and INI files. **Explanation:** Inspect the actual runtime that handles the web request.

### 10. What can make a downloaded native extension binary fail to load?

- **A.** A mismatch in PHP version/API, architecture, build settings, or system libraries
- **B.** A comment in `composer.json`
- **C.** A PHP variable name that is too short
- **D.** Using a namespaced class in the application

**Answer: A.** Native modules must match the runtime and platform that load them. **Explanation:** A file existing in the extension directory is not proof of binary compatibility.

### 11. Which directive is commonly used to load a Zend extension such as a profiler or opcode tool?

- **A.** `zend_extension`
- **B.** `php_package`
- **C.** `composer_extension`
- **D.** `zend_include`

**Answer: A.** Zend extensions use the `zend_extension` startup directive. **Explanation:** Do not guess the directive; use the extension's own installation instructions.

### 12. What is a sound reason to use FFI?

- **A.** A carefully assessed need to call a native library when the developer understands its API and risks
- **B.** A general way to speed up all PHP arrays
- **C.** A replacement for every maintained extension
- **D.** A tool for installing Composer packages

**Answer: A.** FFI provides low-level native interfacing and requires care. **Explanation:** It is not a general speed optimization or a casual replacement for a supported extension.

## Short-answer questions

### 13. Explain the difference between a Composer package and a PHP extension.

**Answer:** A Composer package is usually project-level PHP code loaded through Composer. A PHP extension is a capability installed and enabled in the PHP runtime. **Explanation:** Installing one does not automatically install the other.

### 14. Why should a project declare required extensions in `composer.json`?

**Answer:** It communicates runtime capabilities the project needs and allows platform checks to report missing extensions. **Explanation:** It helps fail early, but it does not install the module or prove every SAPI is configured identically.

### 15. What information does `php -v` provide when diagnosing a module problem?

**Answer:** It identifies the version and build information for the PHP executable invoked by the command. **Explanation:** This helps distinguish multiple local PHP installations and check a module's compatibility target.

### 16. What does `php --ri mbstring` help inspect?

**Answer:** It displays information about the loaded `mbstring` extension in that CLI runtime. **Explanation:** If it reports the extension is absent, check installation and the configuration used by that binary.

### 17. What is a SAPI, and why does it matter for extension checks?

**Answer:** A SAPI is the interface through which PHP runs, such as CLI or a web-server/FPM process. **Explanation:** Different SAPIs can load different binaries, INI files, extensions, and settings.

### 18. Why may PHP need a service restart after changing extension configuration?

**Answer:** PHP reads startup configuration when its process starts, so existing workers may still have the old configuration. **Explanation:** Reload the relevant service and verify the new state in the process that serves the application.

### 19. What does `extension_loaded('curl')` tell you, and what does it not tell you?

**Answer:** It tells whether the current process has loaded the named extension. It does not prove that all network requests will succeed or that another SAPI has it loaded. **Explanation:** Runtime capability checks are specific to the executing process and do not replace application-level error handling.

### 20. Why is `function_exists()` not a full extension-health check?

**Answer:** It checks only whether one function is defined. **Explanation:** It does not confirm the whole module is loaded, every optional feature is configured, or the function can successfully reach an external service.

### 21. What is PIE used for?

**Answer:** PIE installs supported third-party PHP extensions into a PHP runtime. **Explanation:** It is separate from Composer's normal installation of PHP libraries into a project.

### 22. Why is PECL no longer the recommended extension installation method?

**Answer:** PECL has been deprecated and PIE has replaced it as the recommended installer for third-party extensions. **Explanation:** Follow current PHP extension installation guidance and prefer PIE or compatible platform packages.

### 23. Name four compatibility properties to check before loading a prebuilt native extension.

**Answer:** PHP/module API version, operating system, CPU architecture, and build settings such as thread safety; also check required system libraries. **Explanation:** The exact binary must match the runtime that loads it.

### 24. Why should a temporary `phpinfo()` page not remain publicly accessible?

**Answer:** It can reveal detailed PHP and environment configuration useful for diagnosing but unnecessary for public visitors. **Explanation:** Restrict it during diagnosis and remove it afterward.

## Scenario questions

### 25. Composer reports that `ext-mbstring` is missing, but the `mbstring` Composer package is already in `vendor/`. What should you do?

**Answer:** Install or enable the native `mbstring` extension in the PHP runtime used by the application, then check the actual runtime. **Explanation:** A Composer library with a similar name is not the extension platform requirement.

### 26. `php -m` lists `curl`, but the web application reports an undefined cURL function. How should you investigate?

**Answer:** Inspect the web SAPI's PHP version, loaded INI file, scanned INI directory, and extension list; install or enable the module for that runtime and reload the service. **Explanation:** The CLI and web process may not use the same PHP installation or configuration.

### 27. An extension DLL is present in the Windows `ext` folder but PHP warns that it cannot load it. What should you check?

**Answer:** Check the configured extension directory, the PHP version and architecture, thread-safety/build compatibility, required dependent DLLs, and startup logs. **Explanation:** Presence in the directory does not guarantee the binary matches the PHP process.

### 28. A developer adds `extension=opcache` and PHP reports that the module is not a valid extension. What should they verify?

**Answer:** Check the extension's instructions and use the appropriate Zend extension directive, commonly `zend_extension=opcache`, with a compatible module. **Explanation:** Zend extensions are loaded through a different startup mechanism from regular modules.

### 29. A package uses `mbstring` only to improve a nonessential search feature. How should it handle environments without the extension?

**Answer:** Mark the capability as optional if the package can function without it, isolate the feature, and provide a clear fallback or disabled state. **Explanation:** Do not declare an extension as mandatory if the application genuinely works without it.

### 30. The team adds `ext-pdo_mysql` to `composer.json`. Does Composer now install the MySQL driver?

**Answer:** No. The declaration lets Composer check for the platform capability; the driver still must be installed and enabled for the PHP runtime. **Explanation:** A platform requirement describes an environment dependency rather than a project package installation.

### 31. A server update enables an extension for CLI PHP, but existing FPM workers do not show it. What is the likely next step?

**Answer:** Confirm the FPM runtime's configuration and reload or restart its service, then verify from a request handled by FPM. **Explanation:** Existing workers may retain startup configuration and a web SAPI may use a different INI path.

### 32. `function_exists('imagecreatefrompng')` returns false, and the application needs image resizing. What should be checked?

**Answer:** Check whether the appropriate image extension is installed and loaded in the active runtime, along with its relevant configuration and platform support. **Explanation:** A function check reveals the missing function but not why the module is unavailable.

### 33. A deployment uses a very old native module build with a new PHP release. The file exists but PHP refuses to load it. Why?

**Answer:** The module may target a different PHP module API, platform, architecture, build mode, or dependency set. **Explanation:** Install or build a compatible version rather than copying an arbitrary binary into the extension directory.

### 34. A project team wants to install Xdebug in development but not production. What is a sensible setup?

**Answer:** Install and enable Xdebug only in the development or debugging runtime, and keep production runtime requirements and configuration separate. **Explanation:** Debugging extensions add capabilities that may be unnecessary in production and affect its footprint or behavior.

### 35. A developer considers using FFI to avoid installing a supported, maintained extension. What should they evaluate?

**Answer:** Whether native interfacing is truly required, the safety and maintenance of the native API, process privileges, and whether FFI can be restricted where unused. **Explanation:** FFI is a powerful low-level boundary, not a general replacement or speed shortcut.

### 36. `composer check-platform-reqs` succeeds in CI, but production still fails with a missing extension. What could explain the difference?

**Answer:** CI and production may use different PHP executables, SAPIs, containers, INI files, deployment images, or extension configurations. **Explanation:** Platform checks must run against an environment that represents the actual runtime, and the web process should be verified after deployment.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [115 - Caching data and application output](../lessons/115-caching-data-and-application-output.md) | [Lesson](../lessons/116-php-extensions-and-the-extension-ecosystem.md) | [Example](../examples/116-php-extensions-and-the-extension-ecosystem.md) | [117 - Testing PHP applications with PHPUnit](../lessons/117-testing-php-applications-with-phpunit.md) |
