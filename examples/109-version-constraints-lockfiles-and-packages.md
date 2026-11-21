# 109 - Version constraints, lock files, and packages: Example

## Goal

Build a small application that depends on a local Composer package. Use a version constraint to select a compatible package release, inspect the generated lock file, and update the package deliberately.

## Question

How can an application depend on a versioned local package, record the exact selected version for repeatable installs, and handle a package release that no longer fits the current constraint?

## Assumptions

- PHP 8.1 or newer and Composer are available.
- Commands are run from the application root.
- The local package and application live in the same repository for this exercise.
- No third-party package download is required.

## 1. Create the application manifest

Make a directory named `versioned-package-demo` and create `composer.json`:

~~~json
{
    "name": "example/versioned-package-demo",
    "type": "project",
    "description": "A PHP application demonstrating package versions and lock files",
    "repositories": [
        {
            "type": "path",
            "url": "packages/greeting-library",
            "options": {
                "versions": {
                    "demo/greeting-library": "1.2.0"
                },
                "symlink": true
            }
        }
    ],
    "require": {
        "php": "^8.1",
        "demo/greeting-library": "^1.0"
    }
}
~~~

The `path` repository tells this root project where to find the local package. The repository metadata says the local package is version `1.2.0`, which satisfies the application's `^1.0` constraint. The symlink option is useful during local development when the operating system supports it; Composer may use a platform-appropriate link.

Repository configuration belongs to the root project. A dependency's own repository declarations are not automatically inherited by an application that installs it.

## 2. Create the local package

Create `packages/greeting-library/composer.json`:

~~~json
{
    "name": "demo/greeting-library",
    "type": "library",
    "description": "A local greeting library for the version constraint example",
    "require": {
        "php": "^8.1"
    },
    "autoload": {
        "psr-4": {
            "Demo\\Greeting\\": "src/"
        }
    }
}
~~~

The package name must match the name under the path repository's `versions` option and the application's `require` section. Its PSR-4 rule lets Composer load its class.

Create `packages/greeting-library/src/Greeting.php`:

~~~php
<?php

declare(strict_types=1);

namespace Demo\Greeting;

final class Greeting
{
    public function message(string $name): string
    {
        return 'Hello from the versioned package, ' . $name . '!';
    }
}
~~~

## 3. Add the application entry point

Create `app.php` in the project root:

~~~php
<?php

declare(strict_types=1);

use Demo\Greeting\Greeting;

require __DIR__ . '/vendor/autoload.php';

$greeting = new Greeting();
echo $greeting->message('Mira') . PHP_EOL;
~~~

Create `.gitignore`:

~~~gitignore
/vendor/
~~~

## 4. Resolve the package and create the lock file

From the project root, run:

~~~sh
composer update
composer validate
composer show --direct
composer show demo/greeting-library
php app.php
~~~

The initial `composer update` resolves the local package within the declared constraints, writes `composer.lock`, installs dependencies into `vendor/`, and generates the autoloader. Because the application allows `^1.0` and the path repository exposes `1.2.0`, Composer can select that release.

Expected output from the application:

~~~text
Hello from the versioned package, Mira!
~~~

Inspect `composer.lock` to see the exact package version Composer selected. The file also records package reference information as available for the repository. Its precise generated contents depend on the Composer version and local repository state; do not write or edit this file manually.

## 5. Reproduce the selected resolution

Commit the application manifest, lock file, package source, and ignore rule. Then a fresh checkout can run:

~~~sh
composer install
~~~

Composer uses `composer.lock` to install the resolved versions instead of selecting a newer version just because it is available. Keep `vendor/` ignored; it can be recreated from the committed files.

## 6. Publish a compatible local package version

Suppose the local package has a compatible release `1.3.0`. Update the path repository's version metadata in the root `composer.json`:

~~~json
"versions": {
    "demo/greeting-library": "1.3.0"
}
~~~

The application's `^1.0` constraint still permits that version. Resolve the targeted package update:

~~~sh
composer update demo/greeting-library
~~~

Composer updates the selected version in the lock file. Inspect the complete diff because related packages can also change during a targeted update.

## 7. Handle a new major version

Suppose the package later releases `2.0.0`. The application's `^1.0` constraint excludes that major. If the project has reviewed and accepted the new major, update both the path repository metadata and the application's requirement:

~~~json
{
    "repositories": [
        {
            "type": "path",
            "url": "packages/greeting-library",
            "options": {
                "versions": {
                    "demo/greeting-library": "2.0.0"
                },
                "symlink": true
            }
        }
    ],
    "require": {
        "php": "^8.1",
        "demo/greeting-library": "^2.0"
    }
}
~~~

Then resolve and review the change:

~~~sh
composer update demo/greeting-library
composer validate
~~~

If the project kept `^1.0`, Composer should not select `2.0.0`. A major version change can include breaking changes, so update application code and run its full checks before committing the new lock file.

## 8. Diagnose a blocked version

With the package at `2.0.0` but the application still requiring `^1.0`, inspect the solver explanation:

~~~sh
composer why-not demo/greeting-library 2.0.0
~~~

The constraint boundary explains why the solver cannot select the package version. If the project does not intend to upgrade, keep the `^1.0` requirement and use a compatible `1.x` release instead.

## Example source-control state

The application repository should contain:

~~~text
versioned-package-demo/
├── .gitignore
├── app.php
├── composer.json
├── composer.lock
└── packages/
    └── greeting-library/
        ├── composer.json
        └── src/
            └── Greeting.php
~~~

Track both Composer manifests and the application lock file. Do not track generated `vendor/` in this example.

## What this example demonstrates

1. A root project can declare a local path repository for development.
2. Package metadata supplies a version that the root constraint can match.
3. `^1.0` allows compatible `1.x` versions but excludes `2.0.0`.
4. `composer update` resolves versions and writes the application's lock file.
5. `composer install` reproduces the locked package set in a fresh checkout.
6. A targeted update can change the selected package and related lock entries.
7. A new major release requires a deliberate constraint and compatibility decision.
8. `composer why-not` helps explain why a version cannot be selected.
9. The generated lock file should be reviewed and never hand-edited.

## Important notes

- The path repository and its version override are for this local demonstration; a published package is normally resolved through a package repository and release metadata.
- Keep the local package's version metadata accurate and consistent with the release being demonstrated.
- The Composer manifest, not the lock file, declares the allowed compatibility range.
- Commit the application lock file after a reviewed update.
- Repository credentials for private packages must remain outside source control.
- The PHP source in this example is provided for study and was not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [108 - Composer autoloading and PSR-4](../lessons/108-composer-autoloading-and-psr-4.md) | [Q&A](../qnas/109-version-constraints-lockfiles-and-packages.md) | [Lesson](../lessons/109-version-constraints-lockfiles-and-packages.md) | [110 - PHP-FIG standards and PSR recommendations](../lessons/110-php-fig-standards-and-psr-recommendations.md) |
