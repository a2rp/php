# 108 - Composer autoloading and PSR-4: Q&A

## Multiple-choice questions

### 1. What does an autoloader do in PHP?

- **A.** Loads a class file when code first requests a class that is not yet defined
- **B.** Installs PHP on the computer
- **C.** Rewrites every class into one file
- **D.** Starts the web server

**Answer: A.** An autoloader can resolve and include a class file when the class is first needed. **Explanation:** Composer generates and registers an autoloader from the project's configuration.

### 2. Which file should an application normally include to activate Composer's generated autoloading?

- **A.** `vendor/autoload.php`
- **B.** `composer.lock`
- **C.** `composer.json`
- **D.** `vendor/composer/autoload_psr4.php` directly

**Answer: A.** Include `vendor/autoload.php` once from the application entry point. **Explanation:** It loads Composer's generated autoloading setup; do not include generated internals individually.

### 3. Given the mapping `App\` to `src/`, where does `App\Service\Greeting` map under PSR-4?

- **A.** `src/Service/Greeting.php`
- **B.** `src/App/Service/Greeting.php`
- **C.** `vendor/App/Service/Greeting.php`
- **D.** `src/Greeting/Service.php`

**Answer: A.** The namespace segments after the mapped prefix become directories, and the class name becomes the `.php` file. **Explanation:** The `App\` prefix itself is not repeated below `src/`.

### 4. Why should the PSR-4 namespace prefix include its trailing separator?

- **A.** It prevents a prefix such as `App` from also matching a different prefix such as `Application`
- **B.** PHP requires every directory name to end in a slash
- **C.** Composer uses it as a package version
- **D.** It enables `composer.lock`

**Answer: A.** A separator makes the namespace boundary explicit. **Explanation:** A mapping for `App\` is distinct from one for `Application\`.

### 5. In `composer.json`, what does the JSON key `"App\\"` represent?

- **A.** The PHP namespace prefix `App\`
- **B.** The class name `App`
- **C.** A directory named `App\\`
- **D.** Two separate namespace prefixes

**Answer: A.** JSON escapes each backslash, so two backslash characters in the JSON text represent one backslash in the namespace prefix. **Explanation:** Keep the final namespace separator in the configured prefix.

### 6. After changing the PSR-4 mapping in `composer.json`, what command rebuilds Composer's generated autoloader?

- **A.** `composer dump-autoload`
- **B.** `php --clean`
- **C.** `git reset --hard`
- **D.** `composer remove`

**Answer: A.** `composer dump-autoload` regenerates the autoload files from current Composer configuration. **Explanation:** Do not edit generated files under `vendor/` by hand.

### 7. In a typical development setup with an existing PSR-4 mapping, what normally happens after adding a new class under its mapped directory?

- **A.** The PSR-4 loader can find it when requested without rebuilding the mapping
- **B.** The class must be added to `composer.lock` by hand
- **C.** The entire application must be rewritten
- **D.** The class is loaded only after a server restart in every case

**Answer: A.** A normal PSR-4 lookup derives the file path from the class name and mapping. **Explanation:** Regenerate the autoloader when configuration changes; optimized or authoritative production classmaps may require rebuilding after source changes.

### 8. What is `autoload-dev` intended for in a root project?

- **A.** Classes used only for that project's development, such as tests
- **B.** Runtime classes that every consuming application needs
- **C.** PHP extensions installed on the server
- **D.** Package download credentials

**Answer: A.** `autoload-dev` keeps test or development class mappings separate from normal package runtime autoload rules. **Explanation:** A library's consumers should not receive its test classes as runtime autoload mappings.

### 9. When is Composer's `files` autoload mechanism useful?

- **A.** For functions or compatibility files that cannot be loaded by requesting a class name
- **B.** For every class in a PSR-4 application
- **C.** For choosing the PHP executable
- **D.** For storing dependency versions

**Answer: A.** Composer includes configured files when its autoloader is loaded. **Explanation:** Such files can execute code immediately, so use the mechanism sparingly and avoid unnecessary side effects.

### 10. When can a classmap be useful?

- **A.** When integrating legacy files or directories that do not follow a predictable PSR-4 layout
- **B.** When PHP source files must be encrypted
- **C.** When the application has no classes
- **D.** When ignoring all dependency files

**Answer: A.** A classmap indexes class-like declarations found in configured files or directories. **Explanation:** After adding classes to a classmapped directory, regenerate the map so it includes them.

### 11. Why can a class load on one developer's machine but fail on a case-sensitive server?

- **A.** The namespace, directory, or file capitalization does not exactly match
- **B.** Composer always disables namespaces on servers
- **C.** The PHP server changes every class name to lowercase
- **D.** The server has no filesystem

**Answer: A.** Some local filesystems tolerate case mismatches that a case-sensitive filesystem will not. **Explanation:** Match namespace segments, directory names, and file names exactly.

### 12. What risk comes with Composer's authoritative classmap option?

- **A.** Classes absent from the generated classmap may not be found through normal PSR-4 fallback
- **B.** It deletes `composer.json`
- **C.** It installs a second PHP runtime
- **D.** It makes every class globally accessible

**Answer: A.** Authoritative mode limits lookups to the generated classmap. **Explanation:** Generate it from the complete production source tree, and rebuild it when classes change.

## Short-answer questions

### 13. What is the difference between a namespace and a PSR-4 mapping?

**Answer:** A namespace is part of a PHP declaration's name; a PSR-4 mapping connects a namespace prefix to a directory. **Explanation:** Composer uses the remaining namespace and class name to derive a file path.

### 14. What is the fully qualified name of a class declared as `namespace App\Service; class Greeting {}`?

**Answer:** `App\Service\Greeting`. **Explanation:** The namespace and class name combine to form the fully qualified class name.

### 15. Does a PHP `use` statement load a class file?

**Answer:** No. It creates an alias for the name used in that file. **Explanation:** The autoloader resolves the class when code actually requests it.

### 16. If `App\` maps to `src/`, what path corresponds to `App\Service\Greeting`?

**Answer:** `src/Service/Greeting.php`. **Explanation:** Remove the mapped prefix, convert the remaining namespace separator to a directory separator, and append the class name plus `.php`.

### 17. Why should application code include `vendor/autoload.php` before requesting Composer-managed classes?

**Answer:** It registers Composer's generated class loader so those classes can be resolved. **Explanation:** Without it, PHP may report that the class is not found unless another loader already knows the class.

### 18. What does `composer dump-autoload` do?

**Answer:** It regenerates Composer's autoload files from the current Composer configuration. **Explanation:** Run it after changing mappings or other autoload rules; never modify generated internals manually.

### 19. What is one reason to map a PSR-4 prefix to multiple directories?

**Answer:** A project may need to keep one namespace across two source locations, such as during a deliberate legacy migration. **Explanation:** Composer can check the configured locations; a single clear source directory is simpler for new code.

### 20. Why should tests normally use `autoload-dev` rather than the main runtime `autoload` mapping?

**Answer:** Test classes are development-only and should not become part of a package's runtime autoload surface. **Explanation:** This keeps the package's production autoload rules focused on classes consumers need.

### 21. What does Composer's classmap mechanism record?

**Answer:** It records class-like declarations discovered in configured files or directories so they can be looked up by name. **Explanation:** Regenerate the autoloader after adding classes in a classmapped source location.

### 22. Why should `files` autoload entries be limited?

**Answer:** Composer includes those files when the generated autoloader is loaded, so their top-level code can run immediately. **Explanation:** Prefer class autoloading for classes and use file inclusion only for functions or compatibility code that needs it.

### 23. What is an optimized autoloader intended to improve?

**Answer:** It can reduce class lookup work, especially in a production build. **Explanation:** Optimization should be generated from the source tree that will actually be deployed.

### 24. Why should generated files under `vendor/composer/` not be edited directly?

**Answer:** Composer owns and regenerates them, so manual changes can be overwritten. **Explanation:** Change `composer.json` or source files, then rebuild the autoloader.

## Practical and scenario questions

### 25. Your application reports `Class "App\Service\Greeting" not found`. What are the first three things to inspect?

**Answer:** Confirm the entry point includes the correct `vendor/autoload.php`, verify the `App\` to `src/` mapping, and check that the class declaration and file path match the name and capitalization. **Explanation:** Most PSR-4 failures are a missing loader, incorrect mapping, or namespace/path mismatch.

### 26. You add this mapping: `"App": "src/"`. Why should it be changed to include a namespace separator?

**Answer:** Use the prefix `App\` so it matches the namespace boundary explicitly. **Explanation:** Without the separator, the shorter prefix can overlap with names such as `Application\`.

### 27. The class is declared as `App\Service\Greeting`, but the file is saved as `src/service/greeting.php`. Why may deployment fail?

**Answer:** The directory and file capitalization do not match the PSR-4-derived path `src/Service/Greeting.php`. **Explanation:** A case-sensitive filesystem treats those paths as different.

### 28. You changed `composer.json` from mapping `App\` to `src/` to mapping it to `app/`, but the old path still seems active. What should you run and inspect?

**Answer:** Run `composer dump-autoload`, then confirm the mapping, files, and generated autoloader path. **Explanation:** The generated autoloader must reflect the current configuration.

### 29. A class under `tests/` loads locally but is unavailable to consumers of your library. What is a likely explanation?

**Answer:** It is mapped with `autoload-dev`, which is for the root project's development classes rather than consumer runtime classes. **Explanation:** Put only classes consumers need in the package's regular `autoload` rules.

### 30. A new class works in development but is missing from a production deployment using an authoritative classmap. What should the build do?

**Answer:** Regenerate the optimized authoritative autoloader after all production source files have been added. **Explanation:** Authoritative mode only knows classes present in its generated map.

### 31. A legacy file declares several global functions and has no class names for Composer to request. Which autoload mechanism could include it?

**Answer:** A carefully chosen Composer `files` entry. **Explanation:** Composer includes it when the autoloader is loaded, so keep its top-level behavior limited and predictable.

### 32. A project has classes under `legacy/` that do not follow a stable namespace-to-directory rule. Which Composer approach may fit?

**Answer:** A classmap for the legacy directory, while using PSR-4 for new namespaced code. **Explanation:** This avoids forcing an inaccurate directory convention onto existing files.

### 33. The `use App\Service\Greeting;` line is present, but PHP still says the class is missing. Why is that not enough?

**Answer:** `use` only aliases the class name; it does not include the class or register Composer's loader. **Explanation:** Include the correct `vendor/autoload.php` and verify the PSR-4 mapping and file path.

### 34. A project works on Windows but fails on Linux with a class-not-found error. What should be checked first?

**Answer:** Compare exact capitalization in the namespace declaration, directory names, and `.php` filename. **Explanation:** The local filesystem may have hidden a case mismatch that Linux exposes.

### 35. Your PSR-4 map uses `App\` to `src/`, but the file is at `src/App/Service/Greeting.php`. What is the path mistake?

**Answer:** The mapped namespace prefix has been repeated as a directory. The expected path is `src/Service/Greeting.php`. **Explanation:** Only namespace segments after the prefix are appended beneath the mapped directory.

### 36. You add a new class inside an already mapped PSR-4 directory, and the ordinary development autoloader finds it. Must you run `composer dump-autoload` after every new class?

**Answer:** Not in a standard PSR-4 development setup. **Explanation:** The loader derives the path when the class is requested. Regenerate after configuration changes, and rebuild optimized or authoritative maps when the deployment process requires it.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [107 - Composer and dependency management](../lessons/107-composer-and-dependency-management.md) | [Lesson](../lessons/108-composer-autoloading-and-psr-4.md) | [Example](../examples/108-composer-autoloading-and-psr-4.md) | [109 - Version constraints, lockfiles, and packages](../lessons/109-version-constraints-lockfiles-and-packages.md) |
