# 133 - Dependency security and software supply chains: Questions and answers

Use these questions to review Composer dependency management, package execution, audits, lock files, and supply-chain incident response.

## Multiple-choice questions

### 1. What is a transitive dependency?

- A. A package the application requires directly in `composer.json`.
- B. A package required by another dependency in the project.
- C. A PHP extension bundled with the language.
- D. A package that is never installed.

**Answer:** B. **Explanation:** A direct package can bring in additional packages. The complete dependency tree includes both direct and transitive dependencies.

### 2. What does `composer.lock` record?

- A. Only the application's direct package names.
- B. Resolved dependency versions and related package metadata for a reproducible install.
- C. Production credentials for each package.
- D. A guarantee that every dependency is safe.

**Answer:** B. **Explanation:** The lock file helps install the same resolved versions across environments. It does not certify that the selected code is harmless or vulnerability-free.

### 3. Which Composer command installs the locked versions when a lock file is present?

- A. `composer update`
- B. `composer install`
- C. `composer remove`
- D. `composer clear-cache`

**Answer:** B. **Explanation:** `composer install` uses the lock file when available. `composer update` resolves versions again and writes the updated results to the lock file.

### 4. What is `composer audit --locked` used for?

- A. It proves that every dependency has been manually reviewed.
- B. It checks the locked dependency set against available advisories and configured dependency policies.
- C. It updates every package to the latest version.
- D. It disables Composer plugins.

**Answer:** B. **Explanation:** It identifies known findings available to the configured audit sources. A clean result cannot prove that no unknown issue or malicious code exists.

### 5. Why should a project inspect a package before adding it?

- A. All packages with many downloads are guaranteed safe.
- B. Package name, ownership, maintenance, source, license, code, and transitive dependencies can affect risk.
- C. Composer does not resolve package versions.
- D. A package's README is the only input needed.

**Answer:** B. **Explanation:** A package can be abandoned, impersonated, compromised, incompatible, or introduce risky dependencies and behavior.

### 6. What does Composer's `config.allow-plugins` setting help control?

- A. Which package names can be used as PHP namespaces.
- B. Which Composer plugins are permitted to execute during Composer operations.
- C. Which PHP extensions are installed by the operating system.
- D. Which users can log in to the application.

**Answer:** B. **Explanation:** Composer 2.2 and newer allow an explicit plugin policy. Permit only plugins the project requires and trusts.

### 7. Which statement about Composer scripts and plugins is correct?

- A. They are always inert metadata.
- B. They can execute code during Composer operations and should be reviewed.
- C. They only run after deployment to production.
- D. They cannot access files available to the Composer process.

**Answer:** B. **Explanation:** Scripts and plugins run with the permissions of the user running Composer. Avoid running dependency-related code with unnecessary secrets or privileges.

### 8. A package is flagged by an advisory. What is the best next step?

- A. Hide the finding permanently.
- B. Check the affected range and dependency path, then update, replace, remove, or document a temporary mitigation.
- C. Delete `composer.lock` and deploy.
- D. Ignore it if it is transitive.

**Answer:** B. **Explanation:** Transitive packages can also execute in or affect the application. Determine exposure and choose a tracked remediation rather than suppressing the warning.

## Short-answer questions

### 9. What is the difference between `composer.json` and `composer.lock`?

**Answer:** `composer.json` declares requirements and version constraints. `composer.lock` records the resolved dependency versions for the project. **Explanation:** A range such as `^3.0` describes acceptable versions, while the lock file selects the actual resolved versions used by an install.

### 10. Why should an application commit its lock file?

**Answer:** It lets developers, CI, and deployments install the same resolved dependency versions rather than silently resolving different versions at different times. **Explanation:** This improves reproducibility, though package provenance and safety still require review.

### 11. Does a library's lock file force consumers to use those exact dependency versions?

**Answer:** No. Consumers resolve the library's declared constraints as part of their own dependency tree. **Explanation:** A library lock file can still help the library's own development and CI remain reproducible.

### 12. Why should `composer update` be a deliberate development operation?

**Answer:** It can change many resolved packages, including transitive dependencies, and updates the lock file. **Explanation:** Review the full change, validate compatibility, run checks, and commit the manifest and lock file together.

### 13. What does a successful Composer audit prove?

**Answer:** It means the audited packages did not match the findings returned by the configured sources and policies at that time. **Explanation:** The audit may not know about undisclosed or newly discovered problems and does not establish the intent of every package.

### 14. Why should Composer not run as root or an administrator?

**Answer:** Install, update, plugins, and scripts can execute package or project code with the permissions of the Composer process. **Explanation:** Run builds as an unprivileged identity and restrict its file, network, secret, and repository access.

### 15. Why is an SBOM useful, and what does it not do?

**Answer:** It lists components and versions so teams can identify affected packages when new findings appear. It does not itself prove the components are safe or guarantee the inventory is complete. **Explanation:** An SBOM supports inventory and response but does not replace audits, review, or build controls.

### 16. What should happen to a temporary advisory exception?

**Answer:** It should have a recorded justification, narrow scope, an owner, and an expiry or review date. **Explanation:** An exception is a tracked risk decision, not a permanent way to make the pipeline appear clean.

## Code review questions

### 17. What is risky about running this during every production deployment?

~~~sh
composer update
~~~

**Answer:** It resolves package versions again and can change the dependency tree instead of installing the exact versions already reviewed. **Explanation:** Resolve and review updates during development; deploy from a commit that includes its lock file with `composer install`.

### 18. Is this configuration a good default?

~~~json
{
  "config": {
    "allow-plugins": true
  }
}
~~~

**Answer:** No. It permits all Composer plugins to execute. **Explanation:** Use an explicit allowlist for only the required and reviewed plugin package names, or disallow plugins if the project needs none.

### 19. What is wrong with this CI setup?

~~~text
A pull request from any fork runs composer install with the production deployment token available.
~~~

**Answer:** Dependency scripts or plugins in untrusted changes could access or misuse the production token. **Explanation:** Do not expose production credentials to untrusted pull requests. Use isolated jobs, minimal permissions, and protected deployment environments.

### 20. A lock-file update changes 40 packages after adding one direct package. What should the reviewer do?

**Answer:** Inspect the entire lock-file diff, identify which packages changed and why, check constraints and sources, run audits and relevant project checks, then accept only an understood and necessary change. **Explanation:** One requested package can alter a broad transitive tree; the direct package change does not explain every changed version.

## Applied scenarios

### 21. A direct package is safe, but its transitive dependency has a security advisory. Is the application affected?

**Answer:** It may be. Identify why the transitive package is installed, determine the affected version and reachable behavior, and update or mitigate through the parent dependency if needed. **Explanation:** Indirect dependencies are still installed code and can be part of the application's attack surface.

### 22. The team sees a new plugin in `composer.lock` but nobody remembers approving it. What should happen?

**Answer:** Pause the change, identify which requirement introduced the plugin, inspect its source and purpose, and review the manifest, lock-file diff, and plugin policy before allowing it to run. **Explanation:** A lock-file entry is not approval for executable plugin code.

### 23. Composer audit returns no advisories, but a maintainer account for a dependency may have been compromised. Is the package cleared?

**Answer:** No. Check incident details and package provenance, suspend or pin the affected release as appropriate, determine whether the version was downloaded or deployed, and rebuild from reviewed inputs after selecting a trusted version. **Explanation:** Advisory scans may not yet contain information about a newly discovered compromise.

### 24. A critical fix is available but requires a major package version. What should the team do?

**Answer:** Assess the upgrade path and breaking changes, update the package and required dependents in a focused change, run compatibility and security checks, then release promptly. If immediate upgrade is impossible, document a narrow temporary mitigation and its owner. **Explanation:** Avoiding a breaking upgrade does not remove the vulnerability; the risk and remediation need active management.

## Coding exercise

### 25. Show a Composer configuration that explicitly blocks all plugins unless a required plugin is deliberately allowlisted. What commands should CI run on a reviewed project?

**Answer:**

~~~json
{
  "name": "example/reviewed-php-app",
  "type": "project",
  "require": {
    "php": "^8.2"
  },
  "config": {
    "allow-plugins": {
      "trusted-vendor/required-plugin": true
    },
    "sort-packages": true
  }
}
~~~

~~~sh
composer validate --strict
composer audit --locked
composer install --no-dev --prefer-dist --no-interaction
~~~

**Explanation:** The plugin list names one approved plugin; if the project needs none, keep the list empty. CI should install from the reviewed lock file, audit the locked set, and run as an unprivileged identity without unnecessary production credentials. Review required root scripts and decide whether they should run in that job.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [132 - Managing secrets and production configuration](../lessons/132-managing-secrets-and-production-configuration.md) | [Lesson](../lessons/133-dependency-security-and-software-supply-chains.md) | [Example](../examples/133-dependency-security-and-software-supply-chains.md) | [134 - HTTPS, security headers, and secure transport](../lessons/134-https-security-headers-and-secure-transport.md) |