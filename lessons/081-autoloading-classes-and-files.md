# 081 - Autoloading classes and files

## Learning goals

By the end of this lesson, you will be able to:

- Load a PHP file explicitly with `include` or `require`.
- Explain the difference between `include`, `require`, and their `_once` forms.
- Register a class autoloader with `spl_autoload_register()`.
- Map a namespace and class name to a predictable PHP file path.
- Describe how Composer generates and loads a PSR-4 autoloader.
- Recognize what autoloading does not load and how to avoid common mistakes.

## 1. Why loading files matters

PHP executes a file when the program includes or requires it. A file can declare classes, interfaces, traits, enums, functions, and constants. Before code can use a declaration, PHP must have loaded the file that contains it.

A small script can load each dependency directly:

```php
<?php

require __DIR__ . '/src/Invoice.php';
require __DIR__ . '/src/InvoiceRepository.php';
require __DIR__ . '/src/InvoiceController.php';
```

This is clear for a tiny program. As a project grows, repeating a long list of class files in every entry point becomes tedious. Class autoloading lets PHP ask registered callbacks to load a class-like symbol when the code first needs it.

## 2. `include` and `require`

Both constructs execute the selected file at the point where they appear. They can also return a value if the included file explicitly returns one.

| Construct | If the file cannot be loaded | Common use |
| --- | --- | --- |
| `include` | Emits a warning and the script can continue | Optional file whose absence is handled |
| `require` | Produces an error and stops normal execution | Essential file the program cannot continue without |
| `include_once` | Like `include`, but avoids including the same file more than once | Optional file that must not be evaluated twice |
| `require_once` | Like `require`, but avoids requiring the same file more than once | Essential bootstrap or dependency file that must not be evaluated twice |

Use `__DIR__` to build a path from the directory containing the current file:

```php
<?php

require __DIR__ . '/config/database.php';
```

This path does not depend on the process's current working directory. Avoid relying on a bare path such as `require 'config/database.php';` when the script can be started from different directories.

### When to use `_once`

`include_once` and `require_once` keep PHP from evaluating a given file again if it has already been included during the request. They can prevent duplicate function or class declarations in manual-loading code.

With a well-designed autoloader, use `require` inside the loader after checking that the target file exists. The loader is called only when PHP needs an unresolved class-like name; `_once` is usually unnecessary there.

## 3. Class autoloading

Register a callback with `spl_autoload_register()`. When PHP encounters an unresolved class-like name, it passes the name to the registered callback. The callback can locate and load the matching file. PHP then checks whether the requested declaration is available.

```php
<?php

spl_autoload_register(static function (string $class): void {
    $file = __DIR__ . '/src/' . $class . '.php';

    if (is_file($file)) {
        require $file;
    }
});

$invoice = new Invoice();
```

In this simple convention, `Invoice` maps to `src/Invoice.php`. This works for a small project with simple class names. A namespaced project needs a mapping from namespace segments to directories.

Autoloading can load classes, interfaces, traits, and enums. It does not automatically load a file just because the file exists, and it is not a general loader for ordinary functions or constants. Include a file explicitly when code needs declarations that are not triggered by a class-like name.

## 4. A namespace-to-file mapping

A namespace and a directory are separate concepts. An autoloader defines the relationship between them. A simple PSR-4-style loader can map the `App\\` namespace prefix to the `src/` directory:

```php
<?php

spl_autoload_register(static function (string $class): void {
    $prefix = 'App\\';
    $baseDirectory = __DIR__ . '/src/';

    // This loader is responsible only for the App namespace.
    if (!str_starts_with($class, $prefix)) {
        return;
    }

    // Remove the namespace prefix, then turn namespace separators into
    // directory separators.
    $relativeClass = substr($class, strlen($prefix));
    $relativePath = str_replace('\\', DIRECTORY_SEPARATOR, $relativeClass);
    $file = $baseDirectory . $relativePath . '.php';

    // Leave unresolved names for any later registered autoloaders.
    if (is_file($file)) {
        require $file;
    }
});
```

Suppose the project contains this file:

```text
project/
├── bootstrap.php
└── src/
    └── Billing/
        └── Invoice.php
```

And `src/Billing/Invoice.php` declares:

```php
<?php

namespace App\Billing;

class Invoice
{
    public function number(): string
    {
        return 'INV-100';
    }
}
```

When code requests `App\\Billing\\Invoice`, the loader removes `App\\`, converts the remaining namespace separator into a directory separator, and builds `src/Billing/Invoice.php`.

This example uses `str_starts_with()`, which is available in PHP 8.0 and newer. File and namespace casing should be consistent, especially on case-sensitive filesystems.

## 5. Multiple autoloaders and unresolved names

PHP can have multiple registered autoloaders. They are called in registration order until the requested declaration is found or the queue is exhausted. A callback should quietly return when it does not own a namespace or the expected file is absent. That gives other loaders a chance to handle the name.

If none of the registered loaders defines the requested class-like symbol, PHP reports that the class, interface, trait, or enum could not be found. An autoloader should not guess at arbitrary paths, emit unrelated output, or throw an exception simply because a class is outside its mapping.

`__autoload()` is obsolete: it was deprecated in PHP 7.2 and removed in PHP 8.0. Use `spl_autoload_register()` for custom autoloading.

## 6. Composer autoloading and PSR-4

Most modern PHP projects use Composer to manage dependencies and generate the autoloader. A project can map its own namespace in `composer.json`:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  }
}
```

With this mapping, the class `App\\Billing\\Invoice` belongs in `src/Billing/Invoice.php`. The namespace declaration inside that file must match the class name:

```php
<?php

namespace App\Billing;

class Invoice
{
}
```

After adding or changing the autoload mapping, regenerate Composer's autoloader from the project root:

```sh
composer dump-autoload
```

Then load the generated file once from the application's entry point or bootstrap:

```php
<?php

require __DIR__ . '/../vendor/autoload.php';

$invoice = new App\Billing\Invoice();
```

Composer creates `vendor/autoload.php` and registers the configured autoloaders. The application does not need a hand-written `require` for every class. Keep Composer's generated `vendor/` directory out of source control when the project's setup expects contributors or deployment to install dependencies; commit `composer.json` and usually `composer.lock` for an application.

Composer also supports other autoload strategies, including class maps and explicitly included files. PSR-4 is a common choice for project classes because it gives namespaces a predictable directory layout.

## 7. Autoloading and namespaces solve different problems

A namespace gives a symbol its full name, such as `App\\Billing\\Invoice`. Autoloading decides how PHP can find the file that declares that symbol. Declaring a namespace does not create a directory, include a file, or register an autoloader.

Likewise, `use App\\Billing\\Invoice;` creates a short alias in the current file. It does not load the class. The class must already be declared or an autoloader must be registered to locate it.

## 8. Common mistakes

- **Expecting a namespace to load its file:** namespaces name symbols; explicit includes or autoloaders load files.
- **Expecting `use` to load a class:** imports create aliases only.
- **Building paths from the working directory:** use a stable base such as `__DIR__`.
- **Mapping the wrong namespace prefix:** ensure the prefix, directory, declaration, and filename agree.
- **Ignoring letter case:** keep directory, filename, namespace, and class casing consistent.
- **Making a loader search everywhere:** map only the namespace prefixes the loader owns.
- **Requiring an absent candidate file:** check `is_file()` before `require` so later autoloaders can try.
- **Trying to use `__autoload()`:** it was removed in PHP 8.0; register callbacks with `spl_autoload_register()`.
- **Assuming class autoloading loads functions:** include the relevant file or use a Composer file-autoload configuration when appropriate.

## Key takeaways

- `include` and `require` execute files; `require` treats a missing essential file as a stopping error.
- `_once` variants prevent the same file from being included repeatedly.
- `spl_autoload_register()` registers callbacks that PHP can ask to load unresolved class-like symbols.
- A custom loader should map known namespace prefixes to stable paths and quietly ignore names it does not own.
- Composer can generate a PSR-4 autoloader from a namespace-to-directory mapping.
- Namespaces, imports, and autoloading have related but distinct jobs.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [080 - Namespaces, imports, and name resolution](080-namespaces-imports-and-name-resolution.md) | [Q&A](../qnas/081-autoloading-classes-and-files.md) | [Example](../examples/081-autoloading-classes-and-files.md) | [082 - Attributes and metadata](082-attributes-and-metadata.md) |
