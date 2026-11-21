# 107 - Composer and dependency management

## Goals

After this lesson, you should be able to:

- Explain what Composer manages in a PHP project.
- Read the role of composer.json, composer.lock, and vendor/.
- Add runtime and development packages with the right commands.
- Understand version constraints and why lock files matter.
- Install the locked dependencies in a fresh checkout.
- Update dependencies deliberately and review the result.
- Check platform requirements and security advisories.
- Keep package credentials and untrusted scripts under control.

## What is Composer?

Composer is PHP's dependency manager. It resolves packages, records project requirements, downloads package code, and generates files that connect packages to the application.

A package may be:

- A library used by the application.
- A development tool such as a test runner or static analyzer.
- A package required by another package.
- A plugin that extends Composer itself.

Composer packages are commonly identified with names such as vendor/package. Packagist is the default package repository for packages published there, and Composer can also use configured repositories.

Composer does not replace PHP, the database, or the application framework. It manages package requirements and their installation for a project.

## Important project files

### composer.json

composer.json describes the project's direct requirements and other Composer metadata. It is normally placed at the project root.

A small project might contain:

~~~json
{
    "name": "example/php-notes-demo",
    "type": "project",
    "require": {
        "php": "^8.1",
        "monolog/monolog": "^3.0"
    }
}
~~~

The require section lists packages needed at runtime. The PHP entry is a platform requirement: it declares which PHP versions the project supports.

composer.json usually contains version constraints rather than the exact versions installed on one machine.

### composer.lock

composer.lock records the exact package versions Composer resolved, including transitive dependencies. For an application, commit this file so developers, CI, and deployment install the same resolved dependency set.

The lock file is generated and updated by Composer. Avoid editing it by hand.

For a reusable library, Composer consumers resolve dependencies according to that library's composer.json; they do not use the library's lock file to choose versions in the consuming application. A library may still keep its lock file for consistent local development and tests.

### vendor/

Composer installs third-party package code and generated files in vendor/ by default. The vendor directory is generated from the project's dependency metadata and is normally ignored by Git.

A .gitignore entry commonly includes:

~~~gitignore
/vendor/
~~~

Commit the manifests and lock file for an application, then let each environment install vendor/ from them. Do not manually edit package code inside vendor/.

### The generated autoloader

Composer generates vendor/autoload.php. An application includes this file so Composer can load package classes and configured application classes.

~~~php
require __DIR__ . '/vendor/autoload.php';
~~~

Composer autoloading and PSR-4 are covered in the next chapter. In this lesson, focus on how the dependency files determine what is installed.

## Creating a Composer project

Check that Composer is available:

~~~sh
composer --version
~~~

If the command is not found, install Composer using its documented installation procedure and verify that the executable runs with the PHP version intended for the project.

Create composer.json in the project directory with composer init, or create a valid JSON file and validate it. For example:

~~~sh
composer init
~~~

Composer asks for package metadata and requirements. Review the generated file; do not blindly accept unsuitable values.

To add a runtime dependency:

~~~sh
composer require monolog/monolog
~~~

Composer updates composer.json, resolves compatible versions, writes composer.lock, and installs the packages into vendor/. The command chooses a compatible version according to the constraint it records.

To add a development-only package:

~~~sh
composer require --dev phpunit/phpunit
~~~

Development-only dependencies belong in require-dev. They are useful for local development, tests, and code quality tools, but are normally omitted from a production install made with --no-dev.

A package required by production application code belongs in require, even if the code path is rarely used. Marking a runtime dependency as development-only can make production fail when vendor/ is installed without dev packages.

## Install, update, require, and remove

### composer install

Use install to create or synchronize vendor/ from the project's dependency files.

- If composer.lock exists, Composer installs the exact locked versions, subject to platform compatibility.
- If there is no lock file, Composer resolves dependencies and creates one.

Typical fresh-checkout and deployment command:

~~~sh
composer install
~~~

Production install often excludes development dependencies:

~~~sh
composer install --no-dev --prefer-dist --no-interaction
~~~

The exact deployment command depends on the build process and Composer version. Run it in a controlled build or release step, not in a normal web request.

### composer update

Use update when intentionally resolving newer versions allowed by the current constraints:

~~~sh
composer update
~~~

It changes composer.lock and installs the newly selected versions. Review and commit the resulting lock-file changes.

For a targeted update:

~~~sh
composer update monolog/monolog
~~~

Composer may also update dependencies needed to satisfy that package. Inspect all lock-file changes; the named package may not be the only affected line.

Do not use composer update as the normal deployment command. It can select newer allowed releases and change the resolved dependency set.

### composer require

Use require to add a direct dependency:

~~~sh
composer require vendor/package
~~~

It edits composer.json and resolves packages. For a development-only dependency:

~~~sh
composer require --dev vendor/test-tool
~~~

Specifying a constraint is possible when the project needs a deliberate range:

~~~sh
composer require monolog/monolog:^3.0
~~~

Review the composer.json and composer.lock diff after the command.

### composer remove

Remove an unused direct dependency with:

~~~sh
composer remove vendor/package
~~~

Composer updates the manifest, lock file, and installed dependency set. Search the application for uses of the package before removing it.

## Version constraints

A version constraint describes which package versions Composer may select. It is a compatibility range, not necessarily the exact installed version.

Common forms include:

- An exact version such as 2.4.1: tightly pins the package version.
- A caret constraint such as ^3.0: allows compatible releases below the next breaking major boundary for nonzero major versions.
- A tilde constraint such as ~3.2: limits updates according to the specified precision.
- A wildcard such as 3.2.*: allows releases matching that component pattern.
- A range such as >=2.0 <3.0: states explicit lower and upper boundaries.

The caret operator has special behavior for 0.x versions because semantic versioning treats early releases differently. Check the actual constraint range instead of assuming every caret range means the same major rule.

A constraint controls what may be selected. The lock file records what was selected. Together they support controlled updates and repeatable installs.

Avoid unstable development versions in ordinary application requirements unless the project deliberately accepts them. Composer's default minimum stability is stable; stability flags can opt into less stable releases, but should be used intentionally.

## Direct and transitive dependencies

A direct dependency is one your project declares. A transitive dependency is required by another package.

For example:

~~~text
your application
└── logging package
    └── logging interface package
~~~

Declare packages your own code uses directly, even if they are currently installed transitively through another package. Otherwise, removing or changing the other package can unexpectedly remove a dependency your code relied on.

Do not add every transitive package as a direct requirement. Declare what the project itself uses.

## Platform requirements

Composer checks platform packages such as:

- php: the PHP version running Composer.
- ext-*: PHP extensions.
- lib-*: certain system libraries.

A package may require an extension that is installed in one environment but missing in another. Composer reports incompatible requirements during dependency resolution or installation.

Useful commands:

~~~sh
composer check-platform-reqs
composer show --platform
~~~

Run platform checks in the environment that will execute the application, especially in the production image or release build.

The config.platform setting can make Composer resolve as if a different PHP version or extension set were present. It is useful for a declared deployment target, but it does not install that PHP version or extension. Verify the actual runtime separately.

Avoid --ignore-platform-reqs as a routine fix. It suppresses compatibility checks; it does not make an incompatible package work.

## Lock files and team workflow

For an application, commit:

- composer.json
- composer.lock
- .gitignore rules for vendor/ and private local files

A teammate or deployment can then run composer install and receive the locked versions.

A normal dependency update workflow is:

1. Review the package's purpose and support status.
2. Update a package intentionally.
3. Inspect composer.json and composer.lock changes.
4. Run the project's tests and quality checks.
5. Review security advisories and license information.
6. Commit the manifest and lock changes together.
7. Deploy with composer install from the committed lock file.

Do not regenerate the lock file independently on every production host. That makes deployments resolve at different times and can produce different dependency sets.

## Security and trust

Dependencies are executable code. Installing a package or Composer plugin can run code as the user performing the install, and scripts declared by a project can run during Composer events.

Before adding a dependency:

- Confirm the package identity and source.
- Review whether it is maintained and appropriate for the project.
- Check its required PHP version, extensions, and dependency graph.
- Inspect its license and security history.
- Understand whether the package provides plugins or scripts.
- Avoid packages with unnecessary permissions or unclear provenance.

Composer's allow-plugins setting controls which Composer plugins may run. Grant permission only to plugins the project intentionally uses.

composer audit checks installed or locked packages against available security advisories and dependency policies. Run it regularly and review findings:

~~~sh
composer audit
~~~

An audit result is a useful signal, not proof that every dependency is safe. Keep the dependency graph current, review advisories, and plan upgrades.

Do not commit credentials used to access private repositories. Composer authentication may be stored in a user-level configuration file or supplied through a protected environment. Check the path and permissions before sharing a build machine.

## Scripts and plugins

composer.json may define scripts for tasks such as tests or formatting, and packages may provide Composer plugins.

These commands can run project or package code:

- composer install
- composer update
- composer run-script
- Composer plugin events

Review scripts and plugin permissions, especially before running Composer in an unfamiliar repository or with elevated privileges. Do not assume a dependency operation is a passive file download.

## Useful inspection commands

~~~sh
composer validate
composer show
composer show --direct
composer outdated
composer licenses
composer audit
~~~

- validate checks composer.json and lock-file consistency.
- show lists installed package information.
- show --direct helps inspect direct dependencies.
- outdated reports packages with newer versions available under their constraints.
- licenses lists package license information.
- audit checks for known advisories and configured dependency policies.

Command options and output can vary between Composer releases. Check the installed Composer command help when needed.

## Common mistakes

### Committing vendor/ but forgetting composer.lock

The dependency files should describe the project, and the lock file should preserve the resolved application versions. Commit the lock file and normally ignore vendor/.

### Running composer update during every deployment

That resolves new versions during deployment. Use composer install from the committed lock file for a repeatable deployment.

### Updating composer.json by hand and not updating the lock

The manifest and lock file can become inconsistent. Use Composer commands, then validate and review both files.

### Putting a runtime package in require-dev

A production installation with --no-dev will omit it. Put packages used by production code in require.

### Ignoring PHP or extension requirements

A dependency can install on one machine and fail in another. Check the platform in the actual runtime image.

### Treating --ignore-platform-reqs as a fix

It only suppresses checks. Resolve the version or extension mismatch instead of assuming incompatible code will run.

### Adding an unnecessary package

Every dependency adds maintenance and security surface. Prefer the smallest set that solves the actual requirement.

### Installing an unknown Composer plugin

Plugins and scripts can execute code. Review the package and allow only the plugins the project needs.

### Assuming composer audit installs updates

Audit reports policies and advisories; it does not update dependencies. Plan a compatible update, review it, and test the project.

## Key takeaways

- Composer manages PHP package requirements, resolution, installation, and generated dependency files.
- composer.json describes direct requirements and constraints; composer.lock records exact resolved versions.
- Applications should commit composer.lock and normally ignore vendor/.
- Use composer install for fresh setups and deployments; use composer update for intentional version changes.
- Put production packages in require and development tools in require-dev.
- Validate constraints and actual PHP or extension requirements in the target runtime.
- Dependencies, plugins, and scripts are code; review their source and permissions.
- Run composer audit and act on findings; an audit is one part of dependency maintenance.
- Keep private repository credentials out of source control.
- Composer autoload configuration and PSR-4 are covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [106 - Environment configuration and secrets](106-environment-configuration-and-secrets.md) | [Q&A](../qnas/107-composer-and-dependency-management.md) | [Example](../examples/107-composer-and-dependency-management.md) | [108 - Composer autoloading and PSR-4](108-composer-autoloading-and-psr-4.md) |
