# 081 - Autoloading classes and files: Q&A

Each answer appears directly after its question, with an explanation to make the reasoning clear.

## Multiple-choice questions

### 1. What does `require` do if its target file cannot be loaded?

- A. It silently skips the file.
- B. It emits only a warning and always continues.
- C. It raises an error and normal execution cannot continue.
- D. It automatically searches every directory.

**Answer:** C. It raises an error and normal execution cannot continue.

**Explanation:** Use `require` for a file the program needs. `include` reports a warning when a file is missing and allows execution to continue.

### 2. What is the purpose of `require_once`?

- A. It loads a file only when PHP is running from the command line.
- B. It requires the file if it has not already been included during the request.
- C. It loads every class in the same directory.
- D. It registers an autoloader.

**Answer:** B. It requires the file if it has not already been included during the request.

**Explanation:** The `_once` variants prevent repeated evaluation of the same file. They do not discover or load related files automatically.

### 3. Which value does `__DIR__` provide?

- A. The current working directory of the process.
- B. The directory containing the current PHP file.
- C. The path to the PHP executable.
- D. The directory of the requested URL.

**Answer:** B. The directory containing the current PHP file.

**Explanation:** It is useful for building stable paths that do not depend on where the process was started.

### 4. Which function registers an autoload callback?

- A. `register_class_loader()`
- B. `include_class()`
- C. `spl_autoload_register()`
- D. `require_namespace()`

**Answer:** C. `spl_autoload_register()`.

**Explanation:** PHP calls registered callbacks when it needs to resolve an undefined class-like symbol.

### 5. Which statement about `__autoload()` is correct for PHP 8 and newer?

- A. It is required for every project.
- B. It was removed; use `spl_autoload_register()` instead.
- C. It is the Composer autoloader.
- D. It loads functions and constants automatically.

**Answer:** B. It was removed; use `spl_autoload_register()` instead.

**Explanation:** `__autoload()` was deprecated in PHP 7.2 and removed in PHP 8.0.

### 6. Which declarations can PHP autoload when they are referenced?

- A. Classes, interfaces, traits, and enums.
- B. Only global functions.
- C. Every PHP file in the project.
- D. Only constants.

**Answer:** A. Classes, interfaces, traits, and enums.

**Explanation:** An autoloader is triggered by unresolved class-like names; it is not a general file-discovery mechanism.

### 7. Does `use App\Billing\Invoice;` load the file that declares `Invoice`?

- A. Yes, every `use` statement includes its target.
- B. No, it creates an alias; a declaration must already be loaded or autoloaded.
- C. Yes, but only when the class is instantiated.
- D. Only if the path matches a directory.

**Answer:** B. It creates an alias; it does not load the file.

**Explanation:** Namespaces, imports, and autoloading have separate jobs. `use` changes how a name can be written in a file.

### 8. What does an autoload callback receive for a fully qualified class name?

- A. The class name with a leading namespace separator in every case.
- B. The class name without a leading namespace separator.
- C. The class's source file contents.
- D. The class's directory path.

**Answer:** B. The class name without a leading namespace separator.

**Explanation:** A class such as `App\Billing\Invoice` is passed as that name, without an initial `\`.

### 9. What should a namespace-to-path loader do when the requested class is outside its namespace prefix?

- A. Search the entire disk.
- B. Return without loading a file.
- C. Throw an exception immediately.
- D. Load a similarly named class.

**Answer:** B. Return without loading a file.

**Explanation:** Other registered autoloaders may handle that class. A loader should only handle the prefixes it owns.

### 10. After adding a PSR-4 mapping to `composer.json`, what command regenerates Composer's autoloader?

- A. `composer dump-autoload`
- B. `composer reload-php`
- C. `php namespace --build`
- D. `composer require-autoload`

**Answer:** A. `composer dump-autoload`.

**Explanation:** Composer regenerates the generated autoload files from the project's configuration.

### 11. Which file is normally included to activate Composer's generated autoloader?

- A. `composer.json`
- B. `vendor/autoload.php`
- C. `src/autoload.php`
- D. `composer.lock.php`

**Answer:** B. `vendor/autoload.php`.

**Explanation:** Include this generated file from the application bootstrap or entry point so Composer can load configured classes and dependencies.

### 12. With the PSR-4 mapping `"App\\": "src/"`, where does `App\\Billing\\Invoice` map?

- A. `src/App/Billing/Invoice.php`
- B. `src/Billing/Invoice.php`
- C. `App/Billing/Invoice.php`
- D. `vendor/App/Billing/Invoice.php`

**Answer:** B. `src/Billing/Invoice.php`.

**Explanation:** The base prefix `App\\` maps to `src/`. The remaining namespace segments map to subdirectories and the class name maps to a PHP file.

## Short-answer questions

### 13. What is file inclusion in PHP?

**Answer:** It is executing another PHP file from the current file with constructs such as `include` or `require`.

**Explanation:** An included file can declare symbols needed by the rest of the program and may return a value to the including code.

### 14. When should `require` be preferred over `include`?

**Answer:** When the program cannot operate correctly without the file.

**Explanation:** A missing required file stops normal execution with an error; a missing included file produces a warning and execution can continue.

### 15. What problem do class autoloaders solve?

**Answer:** They remove the need to manually list every class file before using those classes.

**Explanation:** PHP invokes an autoload callback when code first refers to an unresolved class-like name, giving the callback a chance to load the declaration.

### 16. Why should a custom autoloader use `__DIR__` to build its base path?

**Answer:** It anchors the path to the directory of the loader file.

**Explanation:** A path built from `__DIR__` does not change just because the process is launched with a different current working directory.

### 17. What is PSR-4 in the context of autoloading?

**Answer:** A convention for mapping namespace prefixes to base directories and mapping the remaining namespace and class name to a file path.

**Explanation:** It lets projects follow a predictable namespace-to-file structure. Composer can generate an autoloader from a PSR-4 mapping.

### 18. Does registering an autoloader execute every PHP file in the mapped directory?

**Answer:** No.

**Explanation:** The callback runs in response to an unresolved class-like name and should load only the file that corresponds to that name.

### 19. Can PHP register more than one autoloader?

**Answer:** Yes. `spl_autoload_register()` can register multiple callbacks.

**Explanation:** PHP tries the registered callbacks in order. A callback that does not own a name should return without preventing later callbacks from trying.

### 20. What happens if no autoloader defines a requested class-like symbol?

**Answer:** PHP reports that the requested class, interface, trait, or enum could not be found.

**Explanation:** Autoloading gives callbacks a chance to make the declaration available; it does not guarantee that a matching file exists.

### 21. Does class autoloading automatically load ordinary functions and constants?

**Answer:** No.

**Explanation:** Autoloading is triggered by class-like symbols. Explicitly include the file containing functions or constants, or use an appropriate Composer file-autoload configuration.

### 22. Why should a loader check `is_file()` before requiring its candidate path?

**Answer:** It avoids requiring a missing file and lets other registered autoloaders try.

**Explanation:** Each loader should handle only its own mapping. An absent candidate is not necessarily an error if another loader can resolve the class.

## Scenario questions

### 23. A file in `src/Services/InvoiceService.php` declares `namespace App\Services; class InvoiceService`. What PSR-4 mapping could load it?

**Answer:** Map `App\\` to `src/`.

**Explanation:** The remaining namespace segment `Services` becomes a directory, and `InvoiceService` becomes the filename `InvoiceService.php`.

### 24. A script works from the project root but fails when launched by a scheduled job. It uses `require 'config.php';`. What is a reliable fix?

**Answer:** Build the path from a known file location, for example `require __DIR__ . '/config.php';`.

**Explanation:** A bare relative path can depend on the process's current working directory, which may differ for a scheduled job.

### 25. A developer adds a Composer PSR-4 mapping but the new class still cannot be found. What should be checked first?

**Answer:** Confirm the mapping, namespace declaration, class name, directory, filename casing, and that `composer dump-autoload` was run.

**Explanation:** All parts of the path must agree. Composer's generated autoloader also needs to be refreshed after configuration changes.

### 26. A project has a custom loader for `App\\` and Composer's loader for vendor classes. The custom loader receives `Vendor\\Tool\\Client`. What should it do?

**Answer:** Return without loading a file.

**Explanation:** The custom loader owns only the `App\\` prefix; Composer or another callback may handle the vendor class.

### 27. A class is in the correct file, but its namespace says `App\Billing` while the autoload mapping expects `App\Finance`. What is wrong?

**Answer:** The namespace does not match the configured namespace-to-directory mapping.

**Explanation:** Autoloaders resolve a class name to a path. The namespace declaration inside the file must still declare the requested fully qualified name.

### 28. The application needs a helper function from `src/helpers.php`. Should it expect a class autoloader to load that file?

**Answer:** No. Include the file explicitly or configure Composer's `autoload.files` feature.

**Explanation:** A function call does not trigger class autoloading.

### 29. A custom loader receives `App\Models\User` and uses `App\\` mapped to `src/`. What relative path should it build?

**Answer:** `src/Models/User.php`.

**Explanation:** Remove the matched prefix, replace namespace separators in the remaining name with directory separators, and append `.php`.

### 30. A file path is built directly from a request parameter such as `$_GET['class']`. Why is this unsafe?

**Answer:** Untrusted input could influence which file the application tries to load.

**Explanation:** Map known namespace prefixes and class names through controlled paths. Do not turn arbitrary user input into an include or require path.

## Code-reading questions

For questions 31–35, assume the project has a loader registered with the following mapping: `App\\` maps to `src/`. The loader strips that prefix, converts namespace separators to directory separators, appends `.php`, and requires the file only when `is_file()` returns true.

### 31. Which file is requested for `new App\Billing\Invoice()`?

**Answer:** `src/Billing/Invoice.php`.

**Explanation:** After stripping `App\\`, the relative class name is `Billing\\Invoice`, which maps to the nested path `Billing/Invoice.php` under `src/`.

### 32. If `App\Billing\Invoice` is already declared before it is used, does PHP need to call the autoloader for that class?

**Answer:** No.

**Explanation:** Autoloading is a fallback for an unresolved class-like name. PHP does not need to load a class that is already declared.

### 33. What should the loader do when asked for `Other\Logger`?

**Answer:** It should return without trying to load it.

**Explanation:** `Other\\Logger` does not begin with the loader's `App\\` prefix, so that loader does not own it.

### 34. If the requested path does not exist, what should this loader do?

**Answer:** Return without requiring the missing file.

**Explanation:** This allows later registered autoloaders to try. If no loader eventually defines the requested symbol, PHP reports it as missing.

### 35. Does a `use App\Billing\Invoice;` statement make question 31's file load automatically without an autoloader?

**Answer:** No.

**Explanation:** The import provides a short alias only. A matching class declaration must be included explicitly or found by an autoloader.

## Key takeaways

- Answers follow each question so the material can be reviewed immediately.
- Use `include` for optional files and `require` for files the program depends on.
- `spl_autoload_register()` is the modern way to register class autoload callbacks.
- An autoloader should map only known namespaces, use stable paths, and leave unrelated names to other loaders.
- Composer can generate PSR-4 autoloading from a namespace-prefix mapping.
- Class autoloading does not replace loading files that contain only functions or constants.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [080 - Namespaces, imports, and name resolution](../lessons/080-namespaces-imports-and-name-resolution.md) | [Lesson](../lessons/081-autoloading-classes-and-files.md) | [Example](../examples/081-autoloading-classes-and-files.md) | [082 - Attributes and metadata](../lessons/082-attributes-and-metadata.md) |
