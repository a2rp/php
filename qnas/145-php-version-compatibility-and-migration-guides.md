# PHP version compatibility and migration guides — Q&A

Review PHP support ranges, Composer platform requirements, deprecations, and upgrade planning. Each answer follows its question.

## Multiple-choice questions

### 1. What should determine the PHP versions a project claims to support?

- A. The versions installed on one developer's computer.
- B. The versions the project declares and actually tests, while considering official support and production availability.
- C. Every PHP version ever released.
- D. Only the latest version, regardless of production.
**Answer: B.** A compatibility range is a promise. Choose a supported target available in production and test every version the project claims to support.

### 2. What does Composer's PHP requirement describe?

- A. It installs or upgrades the PHP interpreter.
- B. It tells Composer which PHP versions are allowed for dependency resolution.
- C. It configures PHP-FPM.
- D. It proves the application has passed tests on those versions.
**Answer: B.** Composer uses the requirement to resolve packages. The requirement does not install the runtime or provide test evidence.

### 3. What does Composer's platform configuration do?

- A. It changes the PHP version running on the server.
- B. It tells Composer to resolve dependencies as if the configured PHP platform were present.
- C. It restarts queue workers.
- D. It validates every application code path.
**Answer: B.** It can make dependency resolution reflect a target platform. Check the real environment separately and test the code on the target interpreter.

### 4. Which Composer command checks the actual PHP and extension requirements of the installed packages?

- A. composer check-platform-reqs
- B. composer clear-cache
- C. composer dump-autoload
- D. composer diagnose --no-plugins
**Answer: A.** This check evaluates the real runtime and installed extensions, rather than treating Composer's simulated platform setting as proof of the server environment.

### 5. What is a safe way to upgrade across several PHP major versions?

- A. Read only the migration guide for the final version.
- B. Skip migration guides and suppress all warnings.
- C. Read each intervening guide in order, resolve compatibility issues, and test the target.
- D. Upgrade directly on the production server.
**Answer: C.** A behavior may be deprecated in one release and removed later. The project needs to account for every version crossed.

## Short-answer questions

### 6. Explain the difference between active support, security-only support, and end of life.

**Answer:** Active support includes fixes under the branch's active-support policy. Security-only support is narrower and limited to qualifying security fixes. End of life means official fixes have ended. Check the current PHP support schedule because branch dates change.

### 7. Why should a PHP version support list not be copied into a project and left there indefinitely?

**Answer:** Release branches move through support stages and reach end of life. A stale list can recommend a version that no longer receives fixes. Make checking the official current schedule part of version planning rather than relying on an old snapshot.

### 8. What is the purpose of the PHP requirement in Composer's dependency manifest?

**Answer:** It expresses the PHP range that the project or package expects. It should match the code, dependencies, deployment environments, and versions the project actually tests.

### 9. Why should an application commit its Composer lock file?

**Answer:** It records the resolved dependency versions. Deploying from the reviewed lock file avoids resolving a different dependency set at release time.

### 10. Does a Composer platform setting prove production is running that PHP version?

**Answer:** No. It affects dependency resolution only. It does not install PHP, change PHP-FPM, install extensions, or prove the code runs. Check the real production runtime and run the platform requirement check there.

### 11. What is the difference between a PHP syntax check and a compatibility test?

**Answer:** A syntax check parses files with one particular PHP interpreter. It does not prove that behavior, dependencies, extensions, web requests, and background jobs work on another version. Use the target interpreter and run the relevant test suite.

### 12. What should a PHP compatibility test matrix include?

**Answer:** Include the lowest supported runtime and the intended current target, plus other claimed versions where practical. Check syntax, tests, warnings and deprecations, required extensions, dependency installation, web requests, CLI commands, and long-running jobs.

### 13. What is a deprecation notice?

**Answer:** It warns that an API or behavior is discouraged and may change or be removed in a later version. Find and replace deprecated use, update affected dependencies, and add a focused regression check before upgrading.

### 14. Why can an upgrade break code even when the code still parses?

**Answer:** A version can change behavior, defaults, type handling, warnings, or extension behavior without introducing a syntax error. Review backward-incompatible changes and test important application behavior.

### 15. Why must CLI PHP and PHP-FPM be checked separately?

**Answer:** They may use different binaries, configuration, extensions, or versions. A CLI command can pass while web requests or queue workers still use another runtime.

### 16. Why should dependencies be upgraded in a branch instead of directly on a production server?

**Answer:** A branch makes the package and lock-file changes reviewable and testable. Production should install the already-reviewed dependency set rather than resolve new versions during deployment.

### 17. Why can changing PHP and every dependency at once make an upgrade harder?

**Answer:** If a regression occurs, it is difficult to identify which change caused it. Smaller, controlled steps make diagnosis, review, and rollback clearer.

## Upgrade scenarios

### 18. A project declares support for PHP 8.1, but CI only tests PHP 8.4. What is missing?

**Answer:** The project has not shown that its declared minimum works. Add the lowest supported version to the test matrix or raise the minimum requirement and document that support change.

### 19. Composer installation succeeds on a developer's computer, but production reports a missing PHP extension. Why?

**Answer:** The developer and production environments may have different extensions. Composer's platform simulation can also differ from the actual server. Check the real production platform with the Composer platform requirements command and install required extensions through the environment's managed setup.

### 20. A team is upgrading from an older PHP release to a much newer one. What should it inspect before changing production?

**Answer:** Record each runtime and extension, read every intervening migration guide, search for affected APIs and dependencies, resolve deprecations, test on the target runtime, verify Composer requirements, and stage the rollout with a recovery plan.

### 21. A deprecated API still works today. Is it safe to ignore?

**Answer:** Not as an upgrade plan. The notice is an early warning. Resolve it while the existing runtime still runs, then test the replacement so a later upgrade does not turn it into a fatal error.

### 22. Production web requests use a new PHP version, but scheduled jobs fail after the release. What should be checked?

**Answer:** Check the PHP binary, extensions, and configuration used by the scheduler and queue workers. Restart long-running processes where needed and verify each execution environment independently.

### 23. A library supports multiple PHP versions. How should it use a feature available only in the newest version?

**Answer:** Either raise and document the minimum supported version or provide an implementation compatible with the declared range. Test the lowest and newest supported runtimes. A runtime feature check may help for optional capabilities, but it does not replace correct package requirements.

### 24. What does a support schedule tell you that a version number alone does not?

**Answer:** It indicates whether a release branch is receiving active fixes, security-only fixes, or no official fixes, and when those stages end. That information helps plan a supported target and upgrade timeline.

## Practical exercise

### 25. Put these actions in a safer order: change production PHP, read migration guides, test on the target runtime, check dependencies and extensions, deploy the tested release.

**Answer:**

1. Record current runtime versions and extensions.
2. Choose a supported target and read every intervening migration guide.
3. Check application and dependency requirements, then resolve incompatibilities.
4. Test on the target PHP runtime and verify the real platform requirements.
5. Deploy the tested code, lock file, runtime, and extensions through a staged release.
6. Verify CLI, web, and background execution after deployment.

**Explanation:** Testing and compatibility review should happen before production changes. The final verification checks that every process is using the intended runtime.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [144 - Database migration and deployment strategies](../lessons/144-database-migration-and-deployment-strategies.md) | [Lesson](../lessons/145-php-version-compatibility-and-migration-guides.md) | [Example](../examples/145-php-version-compatibility-and-migration-guides.md) | [146 - PHPDoc, API documentation, and type annotations](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) |