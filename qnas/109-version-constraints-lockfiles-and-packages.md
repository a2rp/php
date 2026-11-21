# 109 - Version constraints, lock files, and packages: Q&A

## Multiple-choice questions

### 1. What does a Composer version constraint describe?

- **A.** Which package versions Composer may consider for resolution
- **B.** The only file Composer can install
- **C.** The version already installed on every computer
- **D.** The PHP version currently running in production

**Answer: A.** A constraint defines an allowed range or set of versions. **Explanation:** For an application, the lock file records the exact versions selected from those constraints.

### 2. Where does an application record the exact resolved package versions for repeatable installs?

- **A.** `composer.lock`
- **B.** `.gitignore`
- **C.** `autoload-dev`
- **D.** The repository's README only

**Answer: A.** The lock file records the resolved package versions and references. **Explanation:** `composer install` uses it when it is present and consistent with the manifest.

### 3. For a package following semantic versioning, what range does `^1.4.2` normally allow?

- **A.** `>=1.4.2` and `<2.0.0`
- **B.** Exactly `1.4.2`
- **C.** `>=1.4.2` and `<1.5.0`
- **D.** Every version from `0.0.0` onward

**Answer: A.** The caret constraint allows compatible releases below the next major boundary for a nonzero major version. **Explanation:** The actual installed version is recorded in the lock file.

### 4. What range does `^0.4` normally represent?

- **A.** `>=0.4.0` and `<0.5.0`
- **B.** `>=0.4.0` and `<1.0.0`
- **C.** Exactly `0.4.0`
- **D.** `>=0.0.0` and `<0.4.0`

**Answer: A.** Caret constraints use a narrower boundary for pre-1.0 releases. **Explanation:** Early versions do not carry the same compatibility expectations as a stable nonzero major version.

### 5. What range does `~1.4.2` normally allow?

- **A.** `>=1.4.2` and `<1.5.0`
- **B.** `>=1.4.2` and `<2.0.0`
- **C.** Exactly `1.4.2`
- **D.** `>=1.4.2` without an upper bound

**Answer: A.** With three version components, the tilde keeps the major and minor components fixed while allowing the final component to move. **Explanation:** The number of specified components affects the upper boundary.

### 6. What does `composer install` do when a valid lock file is present?

- **A.** Installs the versions recorded in the lock file
- **B.** Selects the newest versions allowed by every constraint
- **C.** Deletes the lock file after each package install
- **D.** Installs only development branches

**Answer: A.** Install synchronizes the installed packages to the lock file's resolved versions. **Explanation:** Use `composer update` when the project intentionally wants to resolve newer versions.

### 7. What does `composer update` normally do?

- **A.** Resolves package versions again within the manifest constraints and updates the lock file
- **B.** Only checks PHP syntax without changing files
- **C.** Installs the exact existing lock versions and never changes them
- **D.** Publishes a package release

**Answer: A.** Update selects versions that satisfy the current constraints and writes the resulting resolution to the lock file. **Explanation:** It may change multiple direct and transitive packages, so review the full diff.

### 8. Why should an application normally commit `composer.lock`?

- **A.** So team members, CI, and deployments can install the same resolved dependency set
- **B.** So Composer can ignore `composer.json`
- **C.** So package constraints can never be updated
- **D.** So PHP can run without a vendor directory

**Answer: A.** Committing the application lock file makes installs reproducible. **Explanation:** Planned updates change the lock file and should be reviewed and committed.

### 9. Does a library's `composer.lock` force its consumers to install the same exact dependency versions?

- **A.** No; consumers resolve the library's constraints with their own project and lock file
- **B.** Yes; all consumers copy the library lock file
- **C.** Yes, but only when the package name starts with `vendor/`
- **D.** Only if consumers use `composer install --no-dev`

**Answer: A.** A library publishes its constraints as package metadata; consuming applications create their own resolution. **Explanation:** A library may keep a lock file for its own development and tests, but it does not pin consumer installs.

### 10. What does a constraint such as `dev-main` request?

- **A.** Development code from the `main` branch rather than a stable tagged release
- **B.** The latest stable patch release only
- **C.** A package with no source code
- **D.** A package that Composer has already audited

**Answer: A.** The constraint refers to a moving development branch. **Explanation:** Use it intentionally; an application lock file records the selected reference for a resolved install.

### 11. What does `composer why-not vendor/library 3.0.0` help explain?

- **A.** Which dependency constraints prevent version `3.0.0` from being selected
- **B.** Which PHP file contains a syntax error
- **C.** Why a package was published
- **D.** Which commit created the lock file

**Answer: A.** `why-not` helps identify constraints blocking a proposed package version. **Explanation:** It is useful when the solver cannot select the desired version.

### 12. What does a PHP or extension requirement in `composer.json` do?

- **A.** Declares a platform constraint that Composer can consider during resolution
- **B.** Installs PHP or the extension automatically
- **C.** Converts an extension into a Composer plugin
- **D.** Makes the same runtime available on every server

**Answer: A.** Platform constraints help Composer choose compatible package versions. **Explanation:** They do not install the runtime; check the actual deployment environment separately.

## Short-answer questions

### 13. How do `composer.json` and `composer.lock` differ?

**Answer:** `composer.json` declares direct requirements and allowed version constraints; `composer.lock` records the exact resolved versions for an application. **Explanation:** One states what can be selected, and the other records what was selected.

### 14. What is the difference between `composer install` and `composer update` in a project with a lock file?

**Answer:** Install uses the versions in the lock file; update resolves versions again within the current constraints and changes the lock file. **Explanation:** Use install for repeatable setup and update for planned dependency changes.

### 15. Why might `composer update vendor/library` change other packages too?

**Answer:** Composer may need to change related dependencies to satisfy the target package's requirements. **Explanation:** Inspect every changed lock entry and test the resulting package graph.

### 16. What does a wildcard constraint such as `1.4.*` permit?

**Answer:** Versions matching the `1.4` series. **Explanation:** It does not allow a different minor series such as `1.5`; the lock file records the exact selected patch version.

### 17. What does a constraint such as `>=1.4 <2.0` mean?

**Answer:** The version must be at least `1.4` and lower than `2.0`. **Explanation:** Space-separated comparisons combine as AND, so both boundaries must be satisfied.

### 18. What is the purpose of `minimum-stability` and per-package stability flags?

**Answer:** They control whether Composer may select prerelease or development versions. **Explanation:** Keep the project default conservative and opt into lower stability only when a specific dependency requires it.

### 19. Why does a reusable library define dependency constraints even if it has its own lock file?

**Answer:** Consumers use the library's manifest constraints when resolving the library with their application dependencies. **Explanation:** The library's lock file does not choose exact versions for consumer projects.

### 20. What is a Composer package repository?

**Answer:** A source of package metadata and package versions from which Composer can resolve dependencies. **Explanation:** Composer uses its default repository and can also be configured with other repository types.

### 21. What is the difference between a stable release tag and a development branch constraint?

**Answer:** A stable tag names a release; a branch constraint such as `dev-main` follows development code that can move. **Explanation:** A branch may change before a stable release is published.

### 22. Why should an application not run `composer update` automatically during every deployment?

**Answer:** It may select newer allowed versions at deployment time and change the dependency set unexpectedly. **Explanation:** Resolve and review updates earlier, commit the lock file, and deploy with `composer install`.

### 23. What can a package's `conflict` metadata express?

**Answer:** It can declare that the package cannot be installed with a specified package or version. **Explanation:** Composer considers conflicts alongside requirements when solving the dependency graph.

### 24. What should you inspect when a dependency cannot be resolved?

**Answer:** Check available package versions, conflicting requirements, PHP and extension requirements, stability settings, repository configuration, credentials, and manifest-lock consistency. **Explanation:** Solver failures usually identify one or more of these constraints.

## Practical and scenario questions

### 25. Your manifest allows `^2.3`, the lock file contains `2.3.7`, and `2.4.0` is released. What will `composer install` normally install?

**Answer:** It will install the locked `2.3.7` version. **Explanation:** The manifest allows the newer version, but install reproduces the lock file; run an intentional update to select a newer one.

### 26. A team member deletes `composer.lock` before deploying so Composer will choose the latest releases. What should the team do instead?

**Answer:** Restore and commit the intended lock file, resolve any desired updates in a planned change, review the full diff, and deploy with `composer install`. **Explanation:** Deleting the lock file makes the deployment resolve a different package graph at that moment.

### 27. A library maintainer expects users to install only the versions in the library's lock file. What is the correction?

**Answer:** Consumers resolve the library's declared constraints with their own dependencies and lock file. **Explanation:** The library should keep its constraints accurate and test them; its lock file is for its own development workflow.

### 28. A package's stable release is incompatible with a needed unreleased fix. The team considers requiring `dev-main`. What should it consider first?

**Answer:** Confirm the package identity and branch, understand that it is moving development code, assess the risk, and use it only as a deliberate temporary dependency. **Explanation:** Prefer a stable tagged release once the fix is published and preserve the chosen reference through the application lock file.

### 29. Composer says no version satisfies the project's requirements. Which command can help show why a proposed version is blocked?

**Answer:** Run `composer why-not vendor/library 3.0.0` with the package and version being investigated. **Explanation:** It reports constraints from the dependency graph that prevent the selection.

### 30. The project allows `^2.0`, but a dependency requires `^3.0` of the same package. Why can Composer fail to resolve?

**Answer:** The root and dependency constraints have no overlapping version range. **Explanation:** The project must choose compatible package versions or revise the root requirement if the application supports the newer major.

### 31. A new release is compatible with the package but requires PHP 8.3, while the production server runs PHP 8.2. What should happen?

**Answer:** Keep or select a package version compatible with PHP 8.2, or upgrade the actual runtime deliberately. **Explanation:** A manifest platform constraint helps prevent selecting incompatible versions, but it does not install PHP.

### 32. A developer added `^3.0` to `composer.json` by hand, but `composer.lock` still records a 2.x package. What should they do?

**Answer:** Run an intentional Composer update that resolves the new constraint, inspect the complete manifest and lock-file changes, then commit them together. **Explanation:** Editing the manifest alone does not update the recorded dependency resolution.

### 33. A package exists in a private VCS repository, but Composer cannot find it. What should the team inspect?

**Answer:** Confirm the root project's repository configuration, package metadata and branch or tag names, access permissions, and credentials. **Explanation:** Repository declarations in a dependency are not automatically inherited by the root application.

### 34. An application uses `>=1.0` for a package that may release breaking major versions. What is the risk?

**Answer:** Composer may be allowed to select a future incompatible major release. **Explanation:** Add an intentional upper compatibility boundary and update deliberately.

### 35. In Windows PowerShell, a caret constraint passed to `composer require` behaves unexpectedly. What should the developer do?

**Answer:** Follow the shell's caret escaping rules or edit the manifest constraint, then verify what Composer recorded. **Explanation:** PowerShell command-line parsing can transform carets; Composer's version-constraint documentation describes the required escaping for that shell.

### 36. A library claims support for `php >=8.0`, but its source uses a feature introduced later. What should the maintainer correct?

**Answer:** Fix the code or raise the declared PHP minimum to the actual supported version, then test the declared range. **Explanation:** Incorrect platform metadata can let Composer install a package that fails at runtime.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [108 - Composer autoloading and PSR-4](../lessons/108-composer-autoloading-and-psr-4.md) | [Lesson](../lessons/109-version-constraints-lockfiles-and-packages.md) | [Example](../examples/109-version-constraints-lockfiles-and-packages.md) | [110 - PHP-FIG standards and PSR recommendations](../lessons/110-php-fig-standards-and-psr-recommendations.md) |
