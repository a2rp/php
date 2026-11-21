# PHP version compatibility and migration guides — Example

## Scenario

A legacy PHP application currently runs on PHP 7.4 and must move to a supported PHP 8 release. The example targets PHP 8.4 to demonstrate a multi-version upgrade. Before using any target in a real project, confirm the current PHP support schedule and that the host provides the target version and required extensions.

The goal is to identify incompatibilities before production, update the code and dependency set in a reviewed branch, verify the actual runtime, and deploy the PHP runtime and application as one tested release.

## Step 1: Record the current platform

Collect the version and extension details from each execution environment, not only a developer terminal.

~~~text
php -v
php -m
composer show --platform
~~~

Also record the PHP-FPM runtime, queue workers, scheduled commands, build image, and any CLI job containers. Their versions or extensions may differ from the interactive CLI.

Treat full environment diagnostics as operational information. Do not publish them on a public page.

## Step 2: Read every migration guide in order

For an upgrade from PHP 7.4 to PHP 8.4, review the migration guides for PHP 8.0, 8.1, 8.2, 8.3, and 8.4. Search the application and dependencies for each relevant deprecation, removal, changed default, type change, and behavior change.

The official guides are a checklist of runtime changes, not a guarantee that your application's code is compatible. Test the code paths that the application actually uses.

## Step 3: Replace an API removed in PHP 8

For example, the legacy function each() was removed in PHP 8. A loop that used it:

~~~php
<?php
$items = [
    'starter' => 'Beginner',
    'advanced' => 'Advanced',
];

while (($entry = each($items)) !== false) {
    echo $entry['key'] . ': ' . $entry['value'] . PHP_EOL;
}
~~~

can use foreach:

~~~php
<?php
$items = [
    'starter' => 'Beginner',
    'advanced' => 'Advanced',
];

foreach ($items as $key => $value) {
    echo $key . ': ' . $value . PHP_EOL;
}
~~~

Expected output from either version is:

~~~text
starter: Beginner
advanced: Advanced
~~~

Search for other removed or deprecated APIs in the official migration guides and in the dependencies. Do not assume this single code change is sufficient for a whole application upgrade.

## Step 4: Update the declared platform and dependencies

After checking the application and package compatibility, update the Composer requirements to the chosen minimum runtime and required extensions:

~~~json
{
  "require": {
    "php": "^8.4",
    "ext-json": "*",
    "ext-pdo": "*"
  },
  "config": {
    "platform": {
      "php": "8.4.0"
    }
  }
}
~~~

This example is for an application whose minimum supported runtime is PHP 8.4. The Composer platform value helps dependency resolution target that runtime. It does not install PHP or prove the production environment is configured correctly.

Review dependencies that do not support the target. Upgrade them in a branch, review their release notes, and commit the resulting lock-file changes. Avoid resolving new packages directly on a production server.

## Step 5: Validate the real target runtime

Run these checks in the same build image or environment intended for deployment:

~~~text
composer validate --strict
composer install --no-interaction --prefer-dist
composer check-platform-reqs
~~~

Then run the project's actual checks on the target PHP interpreter. For example:

~~~text
php -l src/CustomerService.php
vendor/bin/phpunit
~~~

Use the project's real test command and cover important web requests, CLI commands, database work, and background jobs. If the project claims to support more than one PHP version, run the test suite on the lowest supported version and the newest target as separate CI jobs.

The platform requirements check verifies the actual PHP and extension packages available to the process. The compatibility test suite provides evidence that application behavior works on that runtime. Both are needed.

## Step 6: Deploy the runtime and application together

Use a staged release:

1. Build the application and dependencies with the target PHP runtime.
2. Run the compatibility and test checks before traffic is changed.
3. Deploy the same reviewed artifact and lock file to a small canary or staging environment.
4. Confirm PHP-FPM, CLI commands, queue workers, and scheduled jobs use the intended runtime and extensions.
5. Check logs for errors and deprecations; verify important user flows.
6. Expand traffic after the release is healthy.
7. Restart or replace long-running workers so they no longer run the previous runtime.
8. Keep the last known-good code, dependency lock file, and runtime image available for rollback.

If the release fails, roll back the tested artifact and runtime together. Do not run an earlier application with a dependency set that requires the new PHP version.

## Example result checklist

| Check | Expected result |
| --- | --- |
| PHP target | The target release is currently supported and available in the deployment environment |
| Composer PHP constraint | Matches the project's tested minimum |
| Required extensions | Present in CLI, FPM, workers, and build environments |
| Dependency lock file | Reviewed and installed from the tested build |
| Migration-guide review | Every intervening major version has been reviewed |
| Tests | Pass on the lowest supported and target runtimes |
| Web and background processes | Use the same intended runtime and compatible extensions |
| Rollback | Restores a compatible combination of code, dependencies, and runtime |

## Common mistakes in this upgrade

- Testing only on a newer local CLI while PHP-FPM still uses the old runtime.
- Updating the Composer constraint without checking whether the application uses removed APIs.
- Assuming the platform configuration changes the PHP interpreter.
- Forgetting PHP extensions in the web or job runtime.
- Skipping migration guides for intermediate versions.
- Updating every dependency on production while deploying.
- Rolling back only the code while leaving incompatible dependencies or runtime settings.

## Expected result

The project declares the runtime it actually supports, dependencies are resolved for the chosen target, required extensions are present, and compatibility has been checked on the actual interpreter. The deployed artifact, dependency lock file, PHP runtime, and background processes move together with a known-good rollback option.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [144 - Database migration and deployment strategies](../lessons/144-database-migration-and-deployment-strategies.md) | [Lesson](../lessons/145-php-version-compatibility-and-migration-guides.md) | [Q&A](../qnas/145-php-version-compatibility-and-migration-guides.md) | [146 - PHPDoc, API documentation, and type annotations](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) |