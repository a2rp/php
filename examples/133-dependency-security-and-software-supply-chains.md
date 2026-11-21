# 133 - Dependency security and software supply chains: Example

## Question

Create a small PHP application that uses a Composer package, records its resolved dependency tree, checks the dependency metadata and available security advisories, and installs the reviewed versions for production.

The example should show which files to commit, which generated directory should not be committed, and why an audit result is not a complete safety guarantee.

## Project files

Create this structure:

~~~text
dependency-review-demo/
├── composer.json
└── src/
    └── run.php
~~~

### `composer.json`

~~~json
{
  "name": "example/dependency-review-demo",
  "type": "project",
  "description": "A small Composer dependency review example",
  "license": "MIT",
  "require": {
    "php": "^8.1",
    "monolog/monolog": "^3.0"
  },
  "autoload": {
    "psr-4": {
      "DependencyReviewDemo\\": "src/"
    }
  },
  "config": {
    "allow-plugins": {},
    "sort-packages": true
  }
}
~~~

The empty `allow-plugins` object means this project has not approved any Composer plugin. If the project later needs a plugin, review it and add only its exact package name.

### `src/run.php`

~~~php
<?php

declare(strict_types=1);

require dirname(__DIR__) . '/vendor/autoload.php';

if (!class_exists(Monolog\Logger::class)) {
    throw new RuntimeException('The expected Composer package is not available.');
}

echo 'Application started; Monolog is available.', PHP_EOL;
~~~

## Resolve and review the dependency tree

Use Composer 2.2 or newer for the explicit plugin allowlist in this manifest. From the project directory, resolve the dependency set during development:

~~~sh
composer update --no-interaction
~~~

This creates `composer.lock` with the selected versions and package metadata. Review and validate the files before committing:

~~~sh
composer validate --strict
composer show --direct
composer why psr/log
composer audit --locked
git diff -- composer.json composer.lock
~~~

The first command checks the manifest and lock-file consistency. The package listing shows direct requirements; the dependency explanation command shows why `psr/log` is present in this example's tree. The audit checks the locked packages against the currently available advisories and configured policies. Review the complete lock-file diff, including transitive package changes.

Commit both `composer.json` and `composer.lock` for this application. Do not commit the generated `vendor/` directory:

~~~gitignore
/vendor/
~~~

## Install from the reviewed lock file

A developer or CI job can install the exact resolved versions from the committed lock file:

~~~sh
composer install --no-interaction
php src/run.php
~~~

Expected application output:

~~~text
Application started; Monolog is available.
~~~

A production build can omit development-only packages and optimize the autoloader:

~~~sh
composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader
composer audit --locked --no-dev
php src/run.php
~~~

The example has no development dependencies, but real applications should still audit and review their development tools because they execute in local and CI environments.

## Update a dependency deliberately

When a security fix or planned update is needed, make the change in a development branch. For example:

~~~sh
composer update monolog/monolog --with-dependencies
composer validate --strict
composer audit --locked
git diff -- composer.json composer.lock
~~~

Read the package release notes and inspect every changed direct and transitive version before merging. Run the application's tests and relevant integration checks in its normal CI workflow. After review, commit the updated manifest and lock file together. Deploy with `composer install` from that reviewed lock file rather than resolving a new dependency tree during deployment.

## Expected review points

- The project declares its PHP and direct package requirements.
- The lock file records the resolved packages and must be committed for this application.
- The generated vendor directory is installed from the lock file and is not committed in this example.
- No Composer plugin is permitted unless it is deliberately allowlisted.
- The project has no root Composer scripts. If scripts are added, review what code they execute.
- The `composer audit` report reflects available findings at the time of the check. It cannot prove that every package is safe or detect every unknown compromise.
- The package name and project setup are for demonstration. Review the current package source, maintenance, license, advisories, and transitive dependencies before adopting any dependency in a real project.
- Build Composer dependencies with an unprivileged identity and without production secrets or unnecessary repository write access.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [132 - Managing secrets and production configuration](../lessons/132-managing-secrets-and-production-configuration.md) | [Lesson](../lessons/133-dependency-security-and-software-supply-chains.md) | [Q&A](../qnas/133-dependency-security-and-software-supply-chains.md) | [134 - HTTPS, security headers, and secure transport](../lessons/134-https-security-headers-and-secure-transport.md) |