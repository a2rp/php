# 081 - Autoloading classes and files: Example

## Question

How can a small PHP application map the `App\\` namespace to its `src/` directory, automatically load an interface and classes when they are first used, and then use Composer to provide the same mapping?

## Project structure

Create these files:

```text
autoload-demo/
├── autoload.php
├── demo.php
└── src/
    ├── Contracts/
    │   └── Greeter.php
    ├── Models/
    │   └── Visitor.php
    └── Services/
        └── ConsoleGreeter.php
```

## Complete PHP example

### `autoload.php`

```php
<?php

declare(strict_types=1);

spl_autoload_register(static function (string $class): void {
    $prefix = 'App\\';
    $baseDirectory = __DIR__ . '/src/';

    // This callback handles only classes in the App namespace.
    if (!str_starts_with($class, $prefix)) {
        return;
    }

    // Convert the class name after App\\ into a path under src/.
    $relativeClass = substr($class, strlen($prefix));
    $relativePath = str_replace('\\', DIRECTORY_SEPARATOR, $relativeClass);
    $file = $baseDirectory . $relativePath . '.php';

    // Leave missing files to any other registered autoloaders.
    if (is_file($file)) {
        require $file;
    }
});
```

### `src/Contracts/Greeter.php`

```php
<?php

declare(strict_types=1);

namespace App\Contracts;

interface Greeter
{
    public function greet(string $name): string;
}
```

### `src/Models/Visitor.php`

```php
<?php

declare(strict_types=1);

namespace App\Models;

class Visitor
{
    public function __construct(private string $name)
    {
    }

    public function name(): string
    {
        return $this->name;
    }
}
```

### `src/Services/ConsoleGreeter.php`

```php
<?php

declare(strict_types=1);

namespace App\Services;

use App\Contracts\Greeter;

class ConsoleGreeter implements Greeter
{
    public function greet(string $name): string
    {
        return "Hello, {$name}! Welcome to PHP autoloading.";
    }
}
```

### `demo.php`

```php
<?php

declare(strict_types=1);

require __DIR__ . '/autoload.php';

use App\Contracts\Greeter;
use App\Models\Visitor;
use App\Services\ConsoleGreeter;

$visitor = new Visitor('Maya');
$greeter = new ConsoleGreeter();

echo $greeter->greet($visitor->name()) . "\n";
echo 'Greeter interface loaded: ';
echo interface_exists(Greeter::class, false) ? "yes\n" : "no\n";
echo 'Visitor class loaded: ';
echo class_exists(Visitor::class, false) ? "yes\n" : "no\n";
```

Run the entry point from the project directory:

```sh
php demo.php
```

This example requires PHP 8.0 or newer because the loader uses `str_starts_with()`.

## Expected output

```text
Hello, Maya! Welcome to PHP autoloading.
Greeter interface loaded: yes
Visitor class loaded: yes
```

## How it works

1. `demo.php` loads `autoload.php` once with a path based on `__DIR__`.
2. Creating `Visitor` requests `App\\Models\\Visitor`. The callback removes the `App\\` prefix and loads `src/Models/Visitor.php`.
3. Creating `ConsoleGreeter` requests `App\\Services\\ConsoleGreeter`. The same mapping loads `src/Services/ConsoleGreeter.php`.
4. While PHP declares `ConsoleGreeter`, it needs the `App\\Contracts\\Greeter` interface. PHP invokes the autoloader again, which loads `src/Contracts/Greeter.php`.
5. The two `use` statements make the names shorter in `demo.php`; the autoloader, not the imports, is what loads the declarations.
6. The second argument `false` in `interface_exists()` and `class_exists()` prevents those checks from triggering another autoload attempt. The output therefore confirms that the interface and class have already been loaded.

The mapping follows this rule:

```text
App\\Models\\Visitor
  App\\ prefix -> src/
  remaining name -> Models\\Visitor
  namespace separators -> directory separators
  resulting file -> src/Models/Visitor.php
```

The namespace, directory, filename, and declared class or interface must agree. Keep their letter casing consistent, particularly when code may run on a case-sensitive filesystem.

## Composer PSR-4 alternative

For a real project, Composer can generate this autoloader. Add this mapping to `composer.json` in the project root:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "src/"
    }
  }
}
```

Regenerate Composer's autoload files:

```sh
composer dump-autoload
```

Then replace this line in `demo.php`:

```php
require __DIR__ . '/autoload.php';
```

with:

```php
require __DIR__ . '/vendor/autoload.php';
```

The PHP class and interface files stay in the same locations. Composer now performs the namespace-to-directory mapping, so the hand-written `autoload.php` is no longer needed.

## Try these changes

- Request a class outside the `App\\` prefix and observe that this loader returns without trying to load it.
- Rename `src/Models/Visitor.php` without updating the class's expected path, then restore it and note why the path and name must match.
- Remove the `Greeter` interface file and observe that PHP cannot finish declaring `ConsoleGreeter`.
- Change the greeting text and run `php demo.php` again.
- Add a second namespace prefix to `composer.json`, regenerate the autoloader, and place a class in its mapped directory.

## Key takeaways

- `spl_autoload_register()` lets PHP request a file when a class-like name is first needed.
- The callback should map only known prefixes and should not require a missing candidate file.
- `use` shortens names; it does not load files.
- PSR-4 maps namespace prefixes to directories and remaining name segments to paths.
- Composer can generate and register the same kind of mapping through `vendor/autoload.php`.
- Functions and constants still need an explicit file-loading strategy.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [080 - Namespaces, imports, and name resolution](../lessons/080-namespaces-imports-and-name-resolution.md) | [Q&A](../qnas/081-autoloading-classes-and-files.md) | Example | [082 - Attributes and metadata](../lessons/082-attributes-and-metadata.md) |
