# PHP version compatibility and migration guides

PHP applications run in different environments: a developer's command line, a web server's PHP-FPM workers, queue workers, scheduled commands, and production hosts. They may not all use the same PHP version or extensions. Version compatibility means knowing which runtimes the project supports, declaring that range accurately, and testing upgrades before deploying them.

A version upgrade can change syntax availability, function behavior, warnings, removed APIs, extension behavior, and dependency requirements. Plan the upgrade from the project's actual current version to a supported target, and review every intervening migration guide.

## Learning goals

After this chapter, you should be able to:

- Choose and document a supported PHP version range.
- Declare PHP and extension requirements in Composer.
- Understand what Composer's platform setting does and does not prove.
- Test the lowest and highest supported runtimes.
- Read migration guides and resolve deprecations before a feature is removed.
- Upgrade dependencies and PHP in controlled, diagnosable steps.
- Verify the version used by CLI, PHP-FPM, and long-running workers.

## Track support, not just version numbers

PHP release branches move through support stages:

| Stage | What it means for a project |
| --- | --- |
| Active support | Bug and security fixes are provided under the branch's support policy. |
| Security-only support | Only qualifying security fixes are provided; plan a move to a newer supported branch. |
| End of life | The branch no longer receives official fixes; schedule an upgrade promptly. |

The exact branches and dates change. Check PHP's official supported-versions schedule when choosing a target and again when planning a later release. Do not copy a version list into a long-lived tutorial or project policy and assume it stays current.

Choose a target that is supported for the expected lifetime of the release and is available in production. Confirm hosting, operating-system packages, container images, required extensions, and deployment tools can provide it.

## Declare the supported range

Composer's PHP requirement describes the minimum and allowed version range for dependency resolution. Include required PHP extensions as platform dependencies too.

~~~json
{
  "require": {
    "php": "^8.3",
    "ext-pdo": "*",
    "ext-json": "*"
  }
}
~~~

This is an example constraint, not a permanent recommendation. Select a minimum version that the project actually tests and still fits its support policy. A constraint that is too broad can claim compatibility the code does not have; one that is too narrow can unnecessarily prevent valid installations.

A library may support a range of PHP versions because its users deploy different environments. An application usually has more control and can choose a narrower production target. In either case, every version included by the constraint should be part of the compatibility plan.

Keep the dependency lock file committed for applications. It records the resolved dependency versions so deployments can install the reviewed set rather than resolving a different set at release time.

## Composer's platform setting is a solver target

Composer can be configured to resolve dependencies as if a particular PHP version were installed. This is useful when development uses a newer runtime than production:

~~~json
{
  "config": {
    "platform": {
      "php": "8.3.0"
    }
  }
}
~~~

This tells dependency resolution to respect the configured platform version. It does not change the PHP interpreter, extensions, or production server, and it does not prove that code was tested on that runtime.

After installing dependencies on the actual deployment environment, run:

~~~text
composer check-platform-reqs
~~~

This checks the real PHP runtime and installed extensions against the package requirements. Include it in a build or deployment check and stop the release when required platform packages are missing.

Do not use Composer's platform setting to hide an outdated production runtime. Set it to the lowest runtime the project intends to support, then verify the actual environment separately.

## Test the full runtime matrix

Test each runtime the project claims to support. At minimum, exercise the lowest supported version and the newest target version. If the project spans multiple supported branches, include those branches in continuous integration when practical.

The matrix should check:

- PHP syntax and type declarations.
- Unit and integration behavior.
- Deprecation and warning output.
- Required PHP extensions and extension versions.
- Composer dependency installation from the lock file.
- CLI commands, web requests, and background jobs.
- Database drivers, serializers, and other runtime integrations the application uses.

A lint command only parses code with the PHP interpreter that ran it. Run it with each target interpreter if compatibility across versions matters. Likewise, a successful local test does not prove the PHP-FPM or queue-worker runtime matches the developer's CLI.

## Audit the change between versions

Before moving from a current version to a target:

1. Record the PHP version and extensions used by production CLI, PHP-FPM, queue workers, schedulers, and build jobs.
2. Choose a supported target available in the deployment environment.
3. Read the migration guide for every major version crossed, in order.
4. Review backward-incompatible changes, deprecations, changed defaults, removed functions, and extension updates.
5. Search the application and dependencies for affected APIs and behavior.
6. Upgrade dependencies in a branch and resolve their new PHP requirements.
7. Run the test matrix on the target version and inspect warnings.
8. Deploy the runtime and code through a staged release with a rollback plan.

For example, a project moving across several PHP major versions should read each intervening guide, not just the guide for the final version. A change can be introduced as a deprecation in one release and removed in a later release.

Migration guides list changes, but they cannot know which code paths your project uses. Search the codebase, inspect runtime warnings, and test behavior that matters to the application.

## Fix deprecations before they become removals

A deprecation notice signals that a behavior or API is discouraged and may change or be removed later. Treat deprecations as migration work while the old runtime still runs:

- Enable suitable development or CI reporting so notices are visible to maintainers.
- Find the call site and the owning dependency.
- Replace obsolete APIs with supported alternatives.
- Update dependencies that still rely on removed behavior.
- Add a focused regression check for the behavior being changed.
- Keep production responses free of diagnostic output; do not hide the problem by suppressing all notices.

Do not wait until an upgrade turns a deprecation into a fatal error. Also review behavior changes that do not produce a deprecation, such as comparison, type-conversion, default, or error-handling changes.

## Upgrade dependencies deliberately

A PHP upgrade and a dependency upgrade are related but separate changes. A safe sequence is:

1. Preserve the current lock file and record the known-good release.
2. Update the PHP runtime in an isolated development or CI environment.
3. Install the locked dependencies and check platform requirements.
4. Upgrade packages that do not support the target runtime.
5. Review package changelogs and breaking changes.
6. Run tests and static checks.
7. Update the lock file in a reviewed change.
8. Deploy the tested lock file and target PHP runtime together.

Avoid changing the PHP major version, every dependency, and large parts of application behavior in one unreviewable step. Smaller changes make failures easier to locate and rollback easier to reason about.

For production deployment, prefer the project's reviewed lock file and the dependency installation mode intended for production. Do not run an unrestricted dependency update on a production server.

## Verify every PHP execution environment

An application can appear healthy on the command line while web requests fail, or web traffic can use a new version while old queue workers remain active.

Check separately:

- The PHP executable used by development and build commands.
- The PHP-FPM version and loaded configuration serving web requests.
- Queue consumers and scheduled jobs.
- Container or host images used by deployment.
- The extension set available in each runtime.

After changing PHP, restart or replace long-running processes so they load the intended runtime. Verify a real request and a representative background job. Do not leave a public diagnostic page that reveals runtime details.

## Runtime and library compatibility

For an application, a declared PHP constraint should match the deployment runtime and the code that CI tests.

For a reusable library:

- Declare the minimum PHP version and extension requirements honestly.
- Avoid using syntax or APIs newer than the declared minimum.
- Test the oldest supported runtime and a current supported runtime.
- Clearly communicate breaking changes in a release.
- Remove a runtime from the support range deliberately, with a new release and migration notes.

Feature checks can help when an optional capability varies by environment, but they are not a substitute for declaring required extensions and dependencies. Prefer stable abstractions and dependency APIs over scattering version-number checks through application code.

## Common mistakes

- Choosing an end-of-life runtime because it still works on one machine.
- Claiming a PHP range that CI never tests.
- Testing only the newest runtime and missing incompatibility with the declared minimum.
- Assuming Composer's platform setting installs or validates the configured runtime.
- Checking CLI PHP but not PHP-FPM, queue workers, or schedulers.
- Reading only the final migration guide while skipping changes from versions in between.
- Ignoring deprecations until an API is removed.
- Updating PHP and all dependencies directly on production.
- Removing the lock file from an application and resolving different dependencies on each deployment.
- Suppressing warnings instead of fixing their cause.

## Upgrade checklist

- [ ] Check the official support schedule and select a supported target.
- [ ] Record actual PHP and extension versions in every execution environment.
- [ ] Confirm the Composer PHP and extension constraints match the tested support range.
- [ ] Read each intervening migration guide.
- [ ] Resolve deprecations and known incompatibilities.
- [ ] Test the lowest supported and target PHP versions.
- [ ] Run Composer's real-platform requirements check in the deployment environment.
- [ ] Upgrade dependencies in a branch and review the lock-file change.
- [ ] Deploy web and background runtimes consistently.
- [ ] Verify requests, jobs, logs, and rollback readiness after release.

## Key takeaways

- PHP support schedules change; choose targets from the current official schedule.
- Composer constraints describe compatibility, while CI and real-platform checks provide evidence.
- Test every version the project claims to support.
- Review all migration guides between the current and target versions.
- Fix deprecations before APIs are removed.
- Upgrade dependencies in a controlled branch and deploy the reviewed lock file.
- Verify CLI, PHP-FPM, queue workers, and scheduled jobs independently.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [144 - Database migration and deployment strategies](144-database-migration-and-deployment-strategies.md) | [Q&A](../qnas/145-php-version-compatibility-and-migration-guides.md) | [Example](../examples/145-php-version-compatibility-and-migration-guides.md) | [146 - PHPDoc, API documentation, and type annotations](146-phpdoc-api-documentation-and-type-annotations.md) |