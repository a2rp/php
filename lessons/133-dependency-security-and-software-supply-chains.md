# 133 - Dependency security and software supply chains

A PHP application includes more than its own source code. Composer packages, transitive dependencies, plugins, scripts, package registries, build tools, CI workflows, and deployment artifacts all contribute code or influence what reaches production. This connected path is the software supply chain.

A vulnerability or compromise anywhere in that path can affect the application. Reduce risk by choosing dependencies deliberately, keeping a committed lock file, reviewing changes, auditing known advisories, and building releases in a controlled environment.

## Learning goals

By the end of this chapter, you should be able to:

- Explain direct and transitive Composer dependencies.
- Describe the different roles of `composer.json` and `composer.lock`.
- Install the versions recorded in a lock file and update them deliberately.
- Audit dependencies for known vulnerabilities and other flagged packages.
- Review Composer plugins, scripts, package sources, and CI permissions.
- Respond to a vulnerable or compromised dependency.
- Explain why a lock file and an audit report cannot prove that every package is safe.

## Understand the dependency tree

A direct dependency is a package your project requires explicitly. A transitive dependency is installed because one of your direct dependencies requires it. Your application executes or loads code from both.

A project may also depend on code outside Composer: PHP extensions, the PHP runtime, operating-system packages, build tools, CI actions, deployment images, and private package registries. A complete inventory should include the components that actually reach production and the tools that build them.

Keep the dependency set as small as the application reasonably needs. Before adding a package, check its purpose, maintenance, release history, ownership, documentation, license, vulnerability history, and the packages it will bring in. Verify the exact package name and source; lookalike names and abandoned repositories can create risk.

## Understand Composer manifests and lock files

`composer.json` declares package requirements and constraints. A constraint such as `^3.0` allows a compatible range; it does not identify one exact version.

`composer.lock` records the resolved versions for the dependency tree. For an application, commit both files. A lock file helps developers, CI, and production install the same selected versions. It improves reproducibility but does not prove that the selected code is benign or free of vulnerabilities.

Use the commands for their intended roles:

~~~sh
composer validate --strict
composer audit --locked
composer install
~~~

- `composer validate --strict` checks manifest and lock-file consistency and reports validation issues.
- `composer audit --locked` checks the locked packages against Composer's configured dependency policies, including available security advisories.
- `composer install` uses the lock file when one is present.

Use `composer update` deliberately when changing the resolved dependency versions. Review its lock-file diff, test the application, and commit the resulting manifest and lock file together. Avoid running an unrestricted update as an ordinary production deployment step.

Libraries have a special detail: their lock file can support reproducible development and CI, but downstream consumers resolve versions according to the library's constraints. The library's lock file does not force each consumer to install those exact versions.

## Audit known issues

Run Composer's audit command regularly and in CI:

~~~sh
composer audit --locked
~~~

Composer can report known security advisories and packages flagged by configured dependency policies. An audit checks the data available to its configured sources at that time. A clean report is not proof that code is safe: a new vulnerability may not have an advisory yet, and an audit cannot establish the intent or integrity of every package.

When an advisory appears:

1. Identify the affected direct or transitive package and the locked version.
2. Check the advisory's affected and fixed version ranges.
3. Determine why the package is present and whether the vulnerable code path is reachable.
4. Update to a fixed compatible version, or replace or remove the package.
5. Review the manifest and lock-file diff, run relevant tests and checks, then deploy through the normal release process.
6. If no fix is available, reduce exposure with a documented temporary mitigation and owner; keep tracking the upstream fix.

Do not permanently suppress an advisory simply to make CI green. If a finding must be ignored temporarily, document its justification, scope, owner, and expiry, and review it until removed.

## Review update changes

A dependency update can change more than the direct package. It can replace transitive packages, introduce new plugins or scripts, alter autoloaded code, or change supported PHP and extension requirements.

Before accepting an update:

- Review every added, removed, or changed package in the lock file.
- Check whether the package is direct or transitive and why it changed.
- Inspect the exact name, version, source reference, and available integrity metadata.
- Check release notes and compatibility requirements.
- Review new Composer plugins and root scripts.
- Run project validation, security audits, tests, and the application checks relevant to the change.
- Make sure the change is expected and can be reverted or rolled forward.

Avoid arbitrary version constraints such as a moving development branch for production dependencies unless a reviewed workflow explicitly requires one. A stable version is not automatically trustworthy; provenance and review still matter.

## Composer plugins and scripts can execute code

Composer plugins and project-defined scripts can run code during Composer commands. Do not treat package installation as a passive download.

Composer 2.2 and newer provide `config.allow-plugins` to restrict which plugins may execute. Allow only named plugins that the project needs and trusts. Do not set the value to `true` for every plugin by default.

Review the root package's `scripts` entries and changes to them. Run Composer as a normal, unprivileged build user rather than as root or an administrator. Do not provide untrusted dependency code with production credentials, broad repository write access, deployment tokens, or access to unrelated files.

Composer supports `--no-plugins` and `--no-scripts` for workflows that intentionally disable those execution paths. Use them only when the project can install correctly without required plugin or script behavior; do not treat a blanket flag as a substitute for understanding the build.

## Protect package sources and the build pipeline

- Use trusted package repositories and secure transport.
- Inspect custom `repositories` configuration and private registry credentials.
- Restrict registry tokens to the required packages and read operations when possible.
- Avoid silently mixing trusted internal packages with untrusted public names; define and review source policies to reduce dependency-confusion risk.
- Protect CI workflows and deployment definitions because they can fetch packages, run code, and access credentials.
- Give pull-request jobs the minimum permissions required; do not expose secrets to untrusted contributions.
- Separate dependency resolution from production deployment. Build from reviewed inputs and promote the same artifact between environments where practical.
- Record what was built, from which commit, lock file, toolchain, and workflow.

A software bill of materials (SBOM) can help inventory included components and versions. It is useful for searching when an advisory appears, but it does not itself verify that a package is safe or that the inventory is complete.

## Production installation

Deploy from a reviewed commit with its lock file present. A typical production install is:

~~~sh
composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader
composer audit --locked --no-dev
~~~

The first command installs locked dependencies and omits packages declared only for development. The second audits the production dependency set. Use the PHP and extension versions required by the lock file; do not routinely bypass platform requirements to make a build succeed.

The root package's scripts and any explicitly permitted plugins may still run as part of Composer operations. Review them, restrict the build identity, and make a deliberate decision about whether a deployment should run them. Keep production credentials out of dependency-resolution and pull-request jobs whenever possible.

## Avoid common mistakes

- Running `composer update` on every deployment instead of installing the reviewed lock file.
- Assuming direct dependencies are the entire dependency tree.
- Accepting a package based on its name or download count alone.
- Treating a lock file, version constraint, audit command, or SBOM as a complete security guarantee.
- Allowing every Composer plugin or ignoring unfamiliar scripts.
- Running Composer as root or with production secrets available to package code.
- Updating many unrelated packages without reviewing the lock-file diff.
- Leaving vulnerable packages or temporary advisory exceptions without an owner and a removal plan.

## Dependency security checklist

- Keep `composer.json` and the application's `composer.lock` under version control.
- Use `composer install` for reproducible builds; update versions deliberately and review the lock diff.
- Run `composer validate --strict` and `composer audit --locked` in the development and release workflow.
- Inventory direct and transitive dependencies, including production and build-time components.
- Add only needed packages from verified sources and review their maintenance and provenance.
- Restrict Composer plugins with an explicit `allow-plugins` policy and inspect root scripts.
- Run Composer as an unprivileged user without unnecessary secrets or write permissions.
- Keep package registry and CI credentials scoped, protected, and out of untrusted jobs.
- Remediate advisories promptly and document temporary mitigations with an expiry.
- Preserve enough build metadata to identify and rebuild the deployed artifact.

## Key takeaways

- The supply chain includes direct and transitive packages, plugins, scripts, tools, CI, and deployment inputs.
- A lock file pins a resolved dependency tree for reproducible installs; it is not a safety certificate.
- Use Composer audit and validation regularly, then investigate and remediate findings.
- Review update diffs and code that can run during installation.
- Build with least privilege and minimal secrets, and deploy reviewed artifacts.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [132 - Managing secrets and production configuration](132-managing-secrets-and-production-configuration.md) | [Q&A](../qnas/133-dependency-security-and-software-supply-chains.md) | [Example](../examples/133-dependency-security-and-software-supply-chains.md) | [134 - HTTPS, security headers, and secure transport](134-https-security-headers-and-secure-transport.md) |