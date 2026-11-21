# Deployment environments and release workflows — Q&A

Use these questions to review release planning, environment configuration, PHP-specific deployment details, database compatibility, and recovery. Answers and explanations appear immediately after each question.

## Multiple-choice questions

### 1. What is an environment?

- A. A Git commit
- B. A place where an application runs with its own configuration and services
- C. A PHP class
- D. A database migration

**Answer: B.**

**Explanation:** Development, CI, staging, and production are environments. They may use different credentials and services while running the same tested application artifact.

### 2. Which release approach best verifies that production receives the version tested in staging?

- A. Rebuild the application manually on the production server
- B. Promote the same identifiable artifact and inject production configuration at runtime
- C. Edit source files after copying them
- D. Run dependency updates after the release starts

**Answer: B.**

**Explanation:** Promoting the same artifact reduces differences between what passed checks and what is released. Runtime configuration supplies environment-specific values without changing the application code.

### 3. What does Composer's lockfile provide for a project deployment?

- A. The exact selected dependency versions for a repeatable installation
- B. Production database credentials
- C. Automatic rollback of database migrations
- D. A PHP-FPM configuration file

**Answer: A.**

**Explanation:** Composer uses the lockfile to install the resolved dependency versions. Committing it lets local, CI, staging, and production installs use the same dependency set.

### 4. Where should production secrets normally be stored?

- A. In committed PHP source files
- B. In a protected environment configuration or secret manager
- C. In a public example configuration file
- D. In the application artifact without access controls

**Answer: B.**

**Explanation:** Secrets should be provided securely at runtime and restricted to the systems and people that need them. A committed example may show variable names, but should contain placeholders rather than real credentials.

### 5. Which database migration strategy is safest when old and new application versions may run at the same time?

- A. Drop the old column before deploying new code
- B. Use an expand, migrate, and later contract sequence
- C. Rename every table during deployment
- D. Assume application rollback reverses database changes

**Answer: B.**

**Explanation:** Add compatible schema first, deploy code that can work during transition, migrate data, switch usage, and remove old structures only after no running code needs them.

### 6. What does a readiness check indicate?

- A. Whether an instance should currently receive traffic
- B. Whether a source file exists
- C. Whether a user has permission to deploy
- D. Whether every database row is valid

**Answer: A.**

**Explanation:** Readiness is about whether an instance can safely serve requests now. A liveness check instead asks whether the process is running and able to continue.

### 7. OPcache timestamp validation is disabled. What deployment action may be needed for new PHP code to take effect?

- A. Refresh the cache using the server's supported mechanism or restart the PHP service
- B. Clear the browser history
- C. Run composer update
- D. Rename the database

**Answer: A.**

**Explanation:** If OPcache does not check source timestamps, files changing on disk may not immediately replace the cached scripts. Deployment should deliberately refresh the runtime cache.

## Short-answer questions

### 8. What is the difference between an environment and a release?

**Answer:** An environment is a runtime context with configuration and services. A release is an identifiable application version deployed into an environment.

**Explanation:** The same release can be promoted from staging to production while each environment provides different database endpoints, credentials, and feature settings.

### 9. Why should production dependencies be installed from the lockfile rather than resolved anew during deployment?

**Answer:** Installing from the lockfile reproduces the dependency versions selected and reviewed for that application revision.

**Explanation:** Resolving new versions during production can change dependencies independently of the tested source revision. It makes failures harder to reproduce and releases less predictable.

### 10. Why is composer update usually not part of a normal production release?

**Answer:** It resolves dependency versions and may change the dependency set instead of installing the exact versions recorded for the release.

**Explanation:** Dependency updates should be reviewed and tested as a source change, with the resulting lockfile committed. A release should install that reviewed lockfile.

### 11. What is the purpose of development, CI, staging, and production environments?

**Answer:** Development supports local work, CI runs automated checks, staging provides a production-like release check, and production serves real users.

**Explanation:** Clear environment boundaries make it easier to find configuration mistakes and prevent non-production activity from using live credentials or data.

### 12. Why should the same artifact be promoted from staging to production?

**Answer:** It keeps the production code and dependencies identical to the version that passed staging checks.

**Explanation:** Rebuilding or manually editing after testing can introduce unverified differences. Configuration should vary by environment, while the artifact remains unchanged where practical.

### 13. Describe the expand, migrate, contract approach.

**Answer:** First add a backward-compatible schema change, then deploy code that can work during the transition, migrate data, switch the application to the new structure, and remove the old structure in a later release.

**Explanation:** This supports deployments where old and new application instances overlap. It also gives time to validate the new representation before destructive cleanup.

### 14. Why does a code rollback not necessarily restore service after a database migration?

**Answer:** The new schema or transformed data may no longer be compatible with the old code.

**Explanation:** Database changes and application releases are separate. A safe rollback plan checks schema compatibility and data effects; recovery may require a forward fix or a restore procedure.

### 15. Why should the CLI PHP runtime and the web PHP runtime be checked separately?

**Answer:** They may use different PHP versions, configuration files, extensions, or environment variables.

**Explanation:** A Composer command can pass under CLI PHP while PHP-FPM lacks a required extension or uses a different setting. Verify both execution paths against the application's requirements.

### 16. What is the difference between a liveness check and a readiness check?

**Answer:** Liveness asks whether the process is running and can continue. Readiness asks whether the instance should receive traffic now.

**Explanation:** A live process can still be unready because a required startup step has not finished or a necessary dependency is unavailable. Keep checks small and avoid exposing secrets.

### 17. Name useful information to record for each deployment.

**Answer:** Record the source revision or artifact identifier, environment, deployment time, migration state, checks performed, and outcome.

**Explanation:** This information connects observed errors to a specific release and helps operators make an informed rollback or recovery decision.

## Scenarios and code reasoning

### 18. A release installs a package version that differs from staging, even though the source commit is the same. What should you check?

**Answer:** Check whether the lockfile is present and committed, whether deployment used Composer install rather than update, and whether the build artifact was rebuilt with different dependency inputs.

**Explanation:** A source revision alone does not identify dependencies if the lockfile or build process is missing or inconsistent.

### 19. The new release adds a required database column and immediately removes the old one. During rollout, old application instances begin failing. What went wrong?

**Answer:** The schema change was not backward-compatible with the old instances that were still serving traffic.

**Explanation:** During a rolling deployment, code versions can overlap. Add the new structure first, deploy compatible code, migrate and verify usage, and remove the old structure later.

### 20. The application files on disk show the new version, but requests still behave like the old version. What PHP deployment issue should be investigated?

**Answer:** Check OPcache configuration and whether PHP workers or the relevant opcode cache were refreshed after deployment.

**Explanation:** When timestamp validation is disabled, OPcache may keep serving cached scripts until they are invalidated or the PHP service is restarted by the supported mechanism.

### 21. A health endpoint returns success, but users report that checkout is broken after a release. What should the release check include?

**Answer:** Verify critical application flows such as checkout, plus relevant database and payment integration behavior, logs, error rate, and latency.

**Explanation:** A shallow health endpoint does not prove that every user-facing path or dependency works. Release verification should focus on the behavior affected by the change.

### 22. A database migration fails halfway through a large backfill. What properties make recovery safer?

**Answer:** Make the backfill observable, bounded, resumable, and safe to rerun; record progress and avoid assuming the whole migration is atomic.

**Explanation:** Large data changes may exceed practical transaction limits or hold locks too long. A repeatable batch process can resume after a partial failure.

### 23. A production deployment fails after a destructive schema migration. Should the operator immediately switch the code pointer to the prior release?

**Answer:** Not without checking compatibility and data state. A code rollback may not work with the changed schema or transformed data.

**Explanation:** Stop or limit traffic to the unhealthy version, assess the failure, and choose a compatible code rollback, forward repair, feature disablement, or verified data recovery plan.

### 24. Staging tests pass, but someone edits a PHP file directly on the production server afterward. Why is this a release problem?

**Answer:** The production state no longer matches the tested and identifiable artifact.

**Explanation:** Manual edits cannot be reliably reproduced, reviewed, or rolled back. Make the correction in source control, test it, and release a new artifact.

## Practice exercise

### 25. Propose a release workflow for a PHP application with Composer dependencies and a database.

**Answer:**

1. Commit application code, Composer manifest, and lockfile.
2. In CI, install from the lockfile, verify the PHP version and required extensions, and run formatting, static analysis, tests, and dependency checks.
3. Build an identifiable artifact with production dependencies and record its source revision.
4. Deploy that same artifact to staging with staging-only credentials and services.
5. Verify migrations, key user flows, logs, and health checks in staging.
6. Promote the artifact to production with protected production configuration.
7. Run only backward-compatible migrations needed for the rollout, direct traffic to the new release, refresh PHP caches when required, and verify critical flows and operational metrics.
8. Keep the prior compatible release and follow the planned rollback or recovery procedure if checks fail.

**Explanation:** This workflow separates application code from environment configuration, promotes the version that was tested, and treats database and runtime changes as part of the release rather than as afterthoughts.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [140 - Background jobs, queues, and workers](../lessons/140-background-jobs-queues-and-workers.md) | [Lesson](../lessons/141-deployment-environments-and-release-workflows.md) | [Example](../examples/141-deployment-environments-and-release-workflows.md) | [142 - Apache, Nginx, and PHP-FPM integration](../lessons/142-apache-nginx-and-php-fpm-integration.md) |