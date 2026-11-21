# 109 - Version constraints, lock files, and packages

## Goals

After this lesson, you should be able to:

- Read common Composer version constraints and translate them into allowed ranges.
- Explain how version constraints differ from the exact versions in a lock file.
- Use stable releases and development branches deliberately.
- Explain the different lock-file roles in applications and libraries.
- Update one dependency without casually changing the entire dependency graph.
- Understand how package versions and repositories help Composer find code.
- Diagnose why a requested version cannot be installed.
- Review package metadata and platform requirements before adopting a dependency.

## Package, version, and repository

Composer identifies a package by its package name and version. A package commonly contains PHP source, a `composer.json` manifest, and metadata describing its dependencies and where its files can be obtained.

A repository is a source of package metadata. Composer uses Packagist by default and can also use configured repositories. A repository may offer release archives (dist), source checkouts, or both. Composer selects a package version that satisfies the project's constraints and the rest of the dependency graph.

For a versioned library, maintainers commonly create version-control tags for releases. Those versions are visible to Composer through package metadata. A branch such as `main` is development code and is not automatically the same as a stable release such as `1.2.0`.

A project's own `composer.json` normally declares a package version range, not the exact commit or files that will be installed on every machine.

## Version constraints

A constraint describes which versions Composer may choose. It does not say which version was chosen in the last successful resolution; that is recorded in `composer.lock` for an application.

Composer combines constraints from the root project and every package in the dependency graph. The solver must find a set of versions where all those constraints can be satisfied together.

### Exact versions

An exact constraint permits one version:

~~~json
{
    "require": {
        "vendor/library": "1.4.2"
    }
}
~~~

This can be useful when a project has a specific reason to hold one release, but it greatly restricts updates. It is usually not the right default for a maintained application or reusable library.

### Comparison ranges

Comparison operators express explicit boundaries:

~~~json
{
    "require": {
        "vendor/library": ">=1.4 <2.0"
    }
}
~~~

A space or comma combines comparisons as AND. The package version must satisfy both the lower and upper conditions. The `||` operator expresses alternatives, such as `^1.0 || ^2.0`.

Avoid an unbounded lower-only constraint such as `>=1.4` unless the project has intentionally reviewed the compatibility consequences. It can permit future major releases with breaking changes.

### Wildcard constraints

A wildcard allows a matching section of the version series:

~~~json
{
    "require": {
        "vendor/library": "1.4.*"
    }
}
~~~

This accepts matching 1.4 releases while excluding a different minor series. Wildcards are concise, but caret constraints are commonly preferred when the project's compatibility policy follows semantic versioning.

### Tilde constraints

The tilde operator allows the last specified part to move while keeping the earlier part fixed:

~~~text
~1.4       allows >=1.4.0 and <2.0.0
~1.4.2     allows >=1.4.2 and <1.5.0
~~~

The number of specified components matters. Read the constraint carefully instead of assuming that every tilde range ends at the same boundary.

### Caret constraints

The caret operator allows compatible releases without crossing the next significant version boundary:

~~~text
^1.4.2     allows >=1.4.2 and <2.0.0
^0.4       allows >=0.4.0 and <0.5.0
^0.0.4     allows >=0.0.4 and <0.0.5
~~~

For a nonzero major version, a caret range normally permits later minor and patch releases below the next major version. Before `1.0.0`, Composer uses a narrower boundary because early releases do not promise the same compatibility guarantees.

Caret is a common choice when a package follows semantic versioning, but it is not proof that every permitted release is compatible with your application. Review changes and run the project's checks when updating.

### Stability and development versions

Composer prefers stable releases by default. Development branches and prereleases have lower stability. Use them only when the project has a clear reason, such as testing an unreleased fix or working on a package in development.

A branch constraint can look like:

~~~json
{
    "require": {
        "vendor/library": "dev-main"
    }
}
~~~

That selects development code from the branch, rather than a stable tagged release. It may change as the branch moves, so a committed application lock file is especially important for recording the exact selected reference.

Minimum-stability settings and per-package stability flags can allow prereleases or development versions. Keep the project-wide default conservative and permit lower stability only for the package that needs it.

## Constraint versus lock file

The constraint and lock file answer different questions:

| File or value | What it means |
| --- | --- |
| Constraint in `composer.json` | Which package versions the project allows Composer to consider |
| Entry in `composer.lock` | Which exact package version and reference Composer resolved for this application |

For example, a manifest may allow `^2.3`. After resolving dependencies, the lock file could select `2.3.7`. A later `composer install` uses the locked version; it does not automatically move to a newer `2.x` release just because one is available.

This separation lets an application express a compatibility range while keeping deployed and team installs repeatable.

## Application lock files

An application should normally commit both `composer.json` and `composer.lock`.

- `composer.json` records the direct requirements and allowed ranges.
- `composer.lock` records the selected versions and references for direct and transitive dependencies.
- `composer install` installs the versions in the lock file when it exists.
- `composer update` resolves versions again within the constraints and updates the lock file.

A fresh application with no lock file can run `composer install` to resolve the current manifest and create one. The resulting lock file should be reviewed and committed. After that, new checkouts and deployments can install the same resolved set with `composer install`.

Do not delete the lock file just to force newer packages during deployment. Plan an update, review the changes, and deploy from the new committed lock file.

## Library lock files

A reusable library publishes its constraints in its `composer.json`. When an application installs that library, Composer resolves the library's constraints together with the application's other requirements and records the result in the application's lock file.

A library's own lock file does not force consuming applications to install those exact versions. The library can still use a lock file for its own local development or a reproducible test setup; consumers resolve from its package metadata and their own lock file.

Library maintainers should test against a useful range of dependency versions. For example, CI can test the newest allowed versions and, when appropriate, the lowest supported versions. This helps reveal constraints that are too narrow or minimum versions that are inaccurate.

## Updating dependencies safely

### Update the whole dependency graph

~~~sh
composer update
~~~

This resolves newer versions allowed by the manifest and updates the lock file. It can change many direct and transitive packages. Use it intentionally, inspect the full diff, and run the project's checks.

### Update one package

~~~sh
composer update vendor/library
~~~

This targets the named package, but Composer may also change related dependencies needed to satisfy constraints. Inspect every changed lock-file entry rather than assuming only one package moved.

### Change a constraint

If the project needs a newer major series, change the manifest deliberately, then resolve and review it:

~~~sh
composer require vendor/library:^3.0
~~~

On Windows PowerShell, the caret is a special command-line character for Composer invocations. Follow the shell's escaping rules or edit the constraint in `composer.json`, then use Composer to resolve it and verify the final manifest.

### Reproduce an existing resolution

~~~sh
composer install
~~~

Use this for a fresh checkout, CI build, or deployment when the intended lock file is present. It synchronizes installed packages to the recorded versions rather than selecting the newest allowed releases.

## Why a constraint may not resolve

Composer may be unable to find a package version because:

- No configured repository provides a package release matching the constraint.
- The package's available tags or branches do not match the requested version.
- Another dependency requires an incompatible version of the same package.
- The selected package requires a newer PHP version or a missing extension.
- The requested prerelease is not allowed by the project's stability settings.
- A private repository is missing or its credentials are unavailable.
- The manifest was changed but the lock file was not resolved again.

Useful inspection commands include:

~~~sh
composer validate
composer show vendor/library --all
composer why vendor/library
composer why-not vendor/library 3.0.0
composer show --platform
~~~

The `why` command explains which dependency requires a package. `why-not` helps identify constraints blocking a selected version. The available command options and output depend on the Composer release; use command help when needed.

## Platforms and package compatibility

PHP versions and extensions are platform packages. A library can declare requirements such as:

~~~json
{
    "require": {
        "php": "^8.1",
        "ext-json": "*"
    }
}
~~~

These constraints help Composer avoid versions that cannot run on the declared platform. They do not install PHP or an extension. Verify the real deployment environment with `composer check-platform-reqs`.

A package can also declare a `conflict` rule when it cannot work with a particular package or version. Composer's solver considers these rules alongside `require` constraints.

Do not bypass a genuine compatibility issue with `--ignore-platform-reqs`. Resolve the environment or package selection instead.

## Package metadata and package sources

A Composer package's manifest can contain metadata such as:

- Its package name, such as `vendor/library`.
- Its package type, such as `library` or `project`.
- Its description and license declaration.
- Its PHP and extension requirements.
- Its autoload rules.
- Its dependencies, conflicts, suggestions, and replacement declarations.

Review this metadata before depending on a package. Confirm the package identity, maintenance state, supported PHP versions, required extensions, dependencies, license, and release history.

Repositories tell Composer where package metadata can be found. The default public package repository is enabled in a typical Composer setup. A project can configure additional Composer, VCS, or path repositories when needed. Repository configuration belongs to the root project; repository declarations in a dependency are not automatically inherited by the consuming application.

Do not put private repository credentials in a committed manifest. Store credentials in protected Composer configuration or a secured environment.

## Release tags and package versions

A release tag gives Composer a stable version to resolve. A package maintainer should:

1. Keep the package's `composer.json` requirements accurate.
2. Run tests against the supported PHP and dependency range.
3. Update release notes and version-control history.
4. Create a release tag that follows the project's versioning policy.
5. Ensure the package repository metadata can expose the tagged release.

A tag name alone does not guarantee compatibility. Consumers still need to review package maintenance and update safely.

For a temporary development branch or fork, use an explicit development constraint and a deliberate repository configuration. Replace it with a stable release when the fix or feature is published.

## Common mistakes

### Treating `^2.0` as an exact pin

It allows a range of compatible releases below the next major boundary. The exact selected version is in the application lock file.

### Assuming a new release is installed by `composer install`

With a lock file, install uses the locked versions. Run an intentional update to resolve newer allowed versions.

### Committing an application without its lock file

Every environment can resolve the same constraints at a different time and select a different dependency set. Commit the application lock file.

### Expecting a library lock file to control consumers

Consumers resolve the library's constraints with their whole project and write their own lock file. A library's lock file is for the library's own development workflow.

### Using an unbounded version range casually

A range can allow breaking releases that the application has never reviewed. Prefer explicit compatibility boundaries and update deliberately.

### Allowing development stability project-wide

A permissive project-wide minimum stability can admit unstable releases across many dependencies. Keep the default conservative and opt in narrowly when needed.

### Updating on every deployment

`composer update` can choose new versions at deployment time. Resolve and review updates earlier, commit the lock file, and deploy with `composer install`.

### Ignoring transitive lock-file changes

A targeted update may affect related packages. Read the complete diff and verify the resulting dependency graph.

### Treating platform configuration as an installed runtime

A platform setting influences resolution but does not install PHP or extensions. Check the actual runtime separately.

## Key takeaways

- A version constraint describes the versions Composer may select; a lock file records the versions selected for an application.
- Caret, tilde, exact, wildcard, and comparison constraints have different boundaries.
- Pre-1.0 caret ranges are narrower than nonzero-major ranges.
- Commit `composer.lock` for an application and use `composer install` for repeatable builds.
- A library's constraints are resolved by each consuming application; its lock file does not pin consumer installs.
- Review the entire lock-file diff after an update, including transitive dependencies.
- Package metadata, repository source, platform requirements, and stability affect resolution.
- Treat development branches as moving code and use them intentionally.
- Check the target PHP environment and private repository configuration when resolution fails.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [108 - Composer autoloading and PSR-4](108-composer-autoloading-and-psr-4.md) | [Q&A](../qnas/109-version-constraints-lockfiles-and-packages.md) | [Example](../examples/109-version-constraints-lockfiles-and-packages.md) | [110 - PHP-FIG standards and PSR recommendations](110-php-fig-standards-and-psr-recommendations.md) |
