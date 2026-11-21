# 108 - Composer autoloading and PSR-4: Example

## Goal

Create a small PHP project where Composer loads application classes from `src/` using PSR-4, while a development-only class lives under `tests/`.

## Question

How can a PHP project map a namespace to source folders, load a class without manually including its file, and keep test-only classes out of the runtime autoload rules?

## Assumptions

- PHP 8.1 or newer is available on the command line.
- Composer is installed and available as `composer`.
- The commands are run from the project root.
- This example shows autoloading and namespace mapping; it does not install a test framework.

## 1. Create the project structure

Create a directory called `psr4-demo` and add this `composer.json`:

~~~json
{
    "name": "example/psr4-demo",
    "type": "project",
    "description": "A small PHP project demonstrating Composer PSR-4 autoloading",
    "require": {
        "php": "^8.1"
    },
    "autoload": {
        "psr-4": {
            "Demo\\": "src/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Demo\\Tests\\": "tests/"
        }
    }
}
~~~

In JSON, each backslash is written twice. The `Demo\\` key represents the PHP namespace prefix `Demo\`; `Demo\\Tests\\` represents `Demo\Tests\`.

Create `.gitignore`:

~~~gitignore
/vendor/
~~~

Create the following directories and files:

~~~text
psr4-demo/
├── .gitignore
├── app.php
├── composer.json
├── src/
│   └── Service/
│       └── Greeting.php
└── tests/
    ├── check-dev-autoload.php
    └── Support/
        └── Label.php
~~~

Composer will also create `composer.lock` and `vendor/` when dependencies are installed. The lock file belongs in Git for an application; `vendor/` is generated and ignored here.

## 2. Add an application class

Create `src/Service/Greeting.php`:

~~~php
<?php

declare(strict_types=1);

namespace Demo\Service;

final class Greeting
{
    public function message(string $name): string
    {
        return 'Hello, ' . $name . '!';
    }
}
~~~

The class's fully qualified name is `Demo\Service\Greeting`. The `Demo\` prefix maps to `src/`, so the remaining `Service` namespace segment becomes a directory and `Greeting` becomes the PHP file name:

~~~text
Demo\Service\Greeting
          ↓
src/Service/Greeting.php
~~~

Directory and file capitalization must match the namespace and class name, especially when deploying to a case-sensitive filesystem.

## 3. Load the Composer autoloader and use the class

Create `app.php` in the project root:

~~~php
<?php

declare(strict_types=1);

use Demo\Service\Greeting;

require __DIR__ . '/vendor/autoload.php';

$greeting = new Greeting();
echo $greeting->message('Mira') . PHP_EOL;
~~~

The `use` statement is an alias for the class name. The `require` loads Composer's generated class loader, which finds the PHP file when `new Greeting()` requests that class.

## 4. Generate Composer files and run the application

From `psr4-demo/`, run:

~~~sh
composer install
composer validate
php app.php
~~~

Because this new project has no `composer.lock`, `composer install` resolves the platform requirement, writes a lock file, and generates the autoloader. There are no third-party packages in this demonstration, but Composer still generates `vendor/autoload.php` for the project's own namespace mapping.

Expected output:

~~~text
Hello, Mira!
~~~

## 5. Add a development-only class

Create `tests/Support/Label.php`:

~~~php
<?php

declare(strict_types=1);

namespace Demo\Tests\Support;

final class Label
{
    public static function text(): string
    {
        return 'Development-only class loaded';
    }
}
~~~

The `Demo\Tests\` prefix maps to `tests/`, so `Demo\Tests\Support\Label` resolves to `tests/Support/Label.php`. This mapping is under `autoload-dev`, so it is for this project's development and test code rather than a library's consumer-facing runtime classes.

Create `tests/check-dev-autoload.php`:

~~~php
<?php

declare(strict_types=1);

use Demo\Tests\Support\Label;

require dirname(__DIR__) . '/vendor/autoload.php';

echo Label::text() . PHP_EOL;
~~~

With the development autoloader generated, run:

~~~sh
composer dump-autoload
php tests/check-dev-autoload.php
~~~

Expected output:

~~~text
Development-only class loaded
~~~

An ordinary `composer install` in development includes root `autoload-dev` mappings. A production installation with `--no-dev` omits development-only autoload configuration, so application runtime code must not depend on classes from `tests/`.

## 6. Add another class under the existing PSR-4 mapping

Create `src/Service/Status.php` with namespace `Demo\Service` and class `Status`:

~~~php
<?php

declare(strict_types=1);

namespace Demo\Service;

final class Status
{
    public function text(): string
    {
        return 'Application class loaded';
    }
}
~~~

A regular PSR-4 loader can derive this class's file path from the existing mapping, so adding a class under the already mapped `src/` directory normally does not require `composer dump-autoload` during development. If you change `composer.json` autoload configuration, regenerate the autoloader.

## 7. Understand the source-control workflow

For this application, commit:

~~~text
composer.json
composer.lock
.gitignore
app.php
src/
tests/
~~~

Do not commit the generated `vendor/` directory in this workflow. A teammate or build environment can recreate it with `composer install` using the committed Composer files.

When the project needs a production-optimized autoloader, build it after all application source files are present:

~~~sh
composer dump-autoload --optimize
~~~

An authoritative classmap is a stricter optional optimization. It only knows classes included in the generated map, so regenerate it after source changes and use it only when the release process builds from the complete production source tree:

~~~sh
composer dump-autoload --optimize --classmap-authoritative
~~~

## Troubleshooting

| Symptom | Likely cause | Check |
| --- | --- | --- |
| `Class not found` | Composer's autoloader was not included | Include the correct `vendor/autoload.php` before requesting the class |
| Class not found only on Linux | Case mismatch in a directory or file name | Match namespace, directory, and file capitalization exactly |
| Composer cannot map a class | Prefix or base directory is wrong | Confirm the namespace prefix ends in a separator and the path is relative to the project root |
| Mapping changes do not take effect | Generated autoloader is stale | Run `composer dump-autoload` and inspect Composer output |
| Test class is absent in production | It is configured under `autoload-dev` | Keep test classes out of runtime code and use the normal `autoload` section for application classes |
| A recently added class is absent in authoritative mode | The generated classmap predates that class | Rebuild the optimized autoloader after adding the source file |

## What this example demonstrates

1. A Composer PSR-4 mapping connects `Demo\` to `src/`.
2. Namespace suffixes map to directories and class names map to `.php` files.
3. Including `vendor/autoload.php` activates Composer's generated loader.
4. A `use` statement aliases a class name; it does not load the class file itself.
5. `autoload-dev` can map project test classes separately from runtime classes.
6. A standard PSR-4 mapping can find new classes beneath an already mapped directory.
7. Generated autoload files and `vendor/` are recreated from project files.
8. Production autoloader optimization should be built from the complete release source.

## Important notes

- Keep namespace, directory, and file capitalization consistent.
- Do not repeat the mapped namespace prefix as a directory beneath the mapping root.
- Run `composer dump-autoload` after changing autoload configuration.
- Do not edit generated files under `vendor/composer/` by hand.
- Avoid letting runtime code depend on test-only `autoload-dev` classes.
- The PHP source in this example is provided for study and was not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [107 - Composer and dependency management](../lessons/107-composer-and-dependency-management.md) | [Q&A](../qnas/108-composer-autoloading-and-psr-4.md) | [Lesson](../lessons/108-composer-autoloading-and-psr-4.md) | [109 - Version constraints, lockfiles, and packages](../lessons/109-version-constraints-lockfiles-and-packages.md) |
