# Deployment environments and release workflows

Deployment moves a tested application version from development into environments where people rely on it. A release is the controlled change from one running version to another, including its code, dependencies, configuration, database compatibility, and background processes.

A dependable workflow makes each release identifiable and repeatable. It verifies the exact version before release, supplies environment-specific settings safely, deploys without editing live files by hand, checks that the new version works, and provides a recovery plan.

## Learning objectives

After this lesson, you should be able to:

- distinguish an environment from a release;
- explain why development, testing, staging, and production need deliberate configuration;
- deploy the exact dependency versions recorded in the lockfile;
- describe a build, test, artifact, and promotion workflow;
- plan database changes that remain compatible during a release;
- perform health checks and choose a safe rollback or recovery action;
- identify PHP runtime, extension, and OPcache concerns during deployment.

## Environments and configuration

An **environment** is a place where an application runs with a particular configuration and set of services. Common environments include:

- **Development:** fast local feedback, test data, debugging tools, and detailed errors.
- **Testing or CI:** automated checks run in a controlled, repeatable environment.
- **Staging:** a production-like environment used for release checks, integration checks, and acceptance.
- **Production:** the live environment serving real users and real data.

The environment names are not important by themselves. What matters is knowing which services, credentials, PHP runtime, extensions, and configuration each environment uses.

Keep environment-specific values outside committed source code. These include database credentials, API tokens, encryption keys, mail credentials, host names, and feature switches. Supply them through the platform's protected environment configuration or a secret manager. A local development environment file may be useful, but it should be excluded from version control; provide a safe example file containing placeholder names, not real secrets.

Do not copy production secrets into development or staging. If production data is needed for a test, use approved, minimized, and sanitized data. Give each environment separate credentials with only the permissions it needs.

Use the same application artifact across staging and production when practical, while injecting different configuration at runtime. This helps ensure that the code validated in staging is the code that users receive. Avoid rebuilding or manually editing the application after it has passed release checks.

## Build the version that was tested

A release should be traceable to a source revision, such as a commit identifier or version tag. Automated checks commonly include:

1. install dependencies from the lockfile;
2. check formatting, static analysis, and automated tests;
3. verify the PHP version and required extensions;
4. inspect dependency security advisories according to project policy;
5. build a release artifact containing application code and production dependencies;
6. record the source revision and build result.

For a Composer application, commit both the dependency manifest and its lockfile. The manifest describes allowed dependency constraints; the lockfile records the selected versions. Use the lockfile to reproduce the dependency set instead of resolving new versions on the production server.

A typical production installation command is:

~~~sh
composer install --no-dev --optimize-autoloader --no-interaction --prefer-dist
~~~

The no-dev option omits development dependencies. Optimized autoloading is useful for production deployments. Run Composer in a controlled build or deployment step, and verify the target PHP version and extensions. Do not run composer update as part of an ordinary production release: it resolves dependency versions and can change the application beyond the source revision you reviewed.

The exact command and flags depend on the application. Review Composer plugins and scripts because they can execute code during installation. Do not ignore platform requirements just to make a deployment pass; verify that the actual production PHP runtime satisfies the package requirements.

A release artifact might be a package, container image, or versioned directory. Include only what the application needs at runtime. Keep generated caches, logs, uploads, user data, and secrets outside an artifact that is meant to be immutable.

## Promote releases deliberately

A small deployment may copy a prepared release to a server. A larger setup may use containers or a deployment platform. In either case, make the steps explicit and repeatable:

1. identify the exact revision and artifact;
2. verify that required checks passed;
3. provide environment configuration and secrets;
4. prepare files and dependencies without exposing a half-copied application;
5. run compatible database migrations and cache preparation;
6. direct traffic to the new release;
7. run health checks and observe the result;
8. retain the previous release and deployment metadata for recovery.

Where the platform supports it, keep each release in its own versioned directory and switch a current pointer or route traffic to the new version only after it is ready. This avoids changing files underneath active requests. Ensure that file permissions and ownership allow the runtime user to read code while limiting write access to only the required storage and cache directories.

For rolling or multi-server deployments, servers may temporarily run different versions at the same time. Code and database changes must tolerate that overlap. Blue-green deployment keeps old and new environments available during a switch, while canary deployment sends a limited amount of traffic to a new version first. These strategies reduce some risks but require appropriate routing, capacity, and data compatibility.

A deployment is more than a successful file copy. Track the revision, deployment time, environment, migration state, and outcome. Keep the process idempotent where possible so a retried deployment step does not create duplicate work or corrupt state.

## Database changes during a release

Database changes are often harder to roll back than code. A new application version may use a new column while old application instances are still serving requests. A safe migration plan accounts for that overlap.

A common approach is **expand, migrate, contract**:

1. **Expand:** add a new nullable column, table, or index without immediately removing the old structure.
2. **Deploy compatible code:** allow old and new application versions to operate while the schema is in transition.
3. **Migrate data:** backfill or transform existing rows in bounded batches when necessary.
4. **Switch usage:** have the application read and write the new representation, with checks that confirm the data is correct.
5. **Contract later:** remove the old column or behavior only after no running code needs it and recovery no longer depends on it.

Large migrations may need to run separately from the web request and be observable, resumable, and safe to repeat. Consider table locks, transaction size, query load, and the time a migration takes on production-sized data.

Do not assume that reverting application code also reverts its database changes. A destructive migration may make the prior code unusable. Prefer a forward repair or a compatible code rollback. Take and verify backups according to the recovery plan before high-risk changes; a backup that has never been restored is not a proven recovery method.

## Health checks and release verification

A **liveness check** asks whether a process is running and able to continue. A **readiness check** asks whether an instance should receive traffic. Keep checks small and safe; a readiness check may confirm that essential dependencies are reachable without exposing credentials or internal details.

After a release, verify more than the home page. Check critical user flows, application logs, error rate, latency, database connectivity, queue processing, and any key integration affected by the change. Watch the system long enough to catch delayed errors or background work failures.

If the new release is unhealthy, stop increasing traffic to it. Depending on the failure, route traffic back to the prior compatible release, disable a feature, or apply a forward fix. Preserve logs and deployment information so the cause can be investigated. Do not automatically roll back the database or overwrite user data without understanding the consequences.

## PHP runtime and cache considerations

The PHP version and extensions used in CI should match the production target closely. Check the command-line PHP runtime and the PHP-FPM or web-server runtime separately: they may load different configuration files or extensions. Composer's platform check can verify the actual PHP and extension requirements for an installed application.

Use a PHP branch that is still supported, and plan upgrades before the branch reaches end of security support. Test application code and dependencies against the intended target version before changing production.

If OPcache is configured not to check source timestamps, new files on disk may not be used until the relevant cache is invalidated or the PHP service is restarted. A deployment should deliberately refresh the runtime cache using the supported mechanism for the server. Do not assume that copying files alone makes every PHP worker execute the new code.

If the application uses generated configuration or route caches, rebuild them for the new release using the deployment process. Never include secret values in a shared or publicly downloadable build artifact.

## Rollback and recovery

A **rollback** returns application traffic to a previously deployed code version. A **recovery** restores correct service and data, which may require more than changing the code pointer.

Before release, answer these questions:

- Which known-good application version can receive traffic again?
- Will that version work with the current database schema?
- Can new writes made during the release still be read by the old version?
- What is the recovery process if a migration or backfill fails?
- Where are logs, backups, and deployment records stored?
- Who can stop or reverse a rollout?

Keep old release files until the new version is stable and the retention policy permits cleanup. For a multi-server or container deployment, know how to route traffic back and how to handle old and new instances together. Prefer reversible, backward-compatible schema changes; for irreversible changes, plan and verify recovery before running them.

## Common deployment mistakes

- Deploying uncommitted changes or a different artifact from the one tested.
- Running dependency updates on the production host without review.
- Committing real secrets or sharing production credentials with non-production systems.
- Assuming the CLI PHP configuration is the same as PHP-FPM.
- Copying code in place while live requests are using it.
- Running a long or destructive migration without a compatibility and recovery plan.
- Removing old schema or release files before the new release is verified.
- Treating a successful health endpoint as proof that all critical flows work.
- Forgetting to refresh OPcache or generated application caches.
- Rolling back code without checking whether the database still supports it.

## Key takeaways

- Keep environments deliberate and secrets outside the committed application.
- Promote the exact artifact that passed checks, using the lockfile for repeatable Composer installs.
- Make releases identifiable, repeatable, observable, and recoverable.
- Use backward-compatible database changes and delay destructive cleanup.
- Verify readiness and key application behavior after traffic switches.
- Match the production PHP runtime and extensions, and account for OPcache.
- Plan rollback around both code and data; a code rollback alone may not restore service.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [140 - Background jobs, queues, and workers](140-background-jobs-queues-and-workers.md) | [Q&A](../qnas/141-deployment-environments-and-release-workflows.md) | [Example](../examples/141-deployment-environments-and-release-workflows.md) | [142 - Apache, Nginx, and PHP-FPM integration](142-apache-nginx-and-php-fpm-integration.md) |