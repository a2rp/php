# Code review, maintenance, and upgrades

Reliable software is maintained through small, deliberate changes. Code review helps people find correctness, security, compatibility, and clarity problems before a change reaches users. Maintenance keeps the codebase understandable and supported. Upgrades keep the application, its dependencies, and its runtime within a safe support window.

These practices work together: a good review considers how a change will be operated and maintained, while a good maintenance plan makes future reviews and upgrades less risky.

## Learning goals

After this chapter, you should be able to:

- Prepare a focused change that is straightforward to review.
- Review behavior, edge cases, security, compatibility, tests, and operational impact.
- Give constructive, specific feedback and respond to review comments.
- Distinguish a behavior-preserving refactor from a behavior change.
- Maintain dependencies and the PHP runtime using a repeatable process.
- Use tests, static analysis, security checks, and deployment safeguards during upgrades.
- Plan a rollback or recovery path before a risky release.

## What code review is for

Code review is a structured check of a proposed change by one or more people before it is merged. It can catch defects and improve shared understanding of the system. It is also a chance to discuss whether the change fits the project's conventions and long-term design.

A review is not a substitute for automated checks, design work, or testing. Reviewers can miss defects, and passing tests cannot prove that every requirement is met. Use human review and automated feedback together.

### Keep changes reviewable

A focused change normally has:

- One clear purpose, stated in the pull request or change description.
- A small enough diff to understand without losing context.
- Tests or other evidence for changed behavior.
- Updated documentation or configuration when the behavior or setup changes.
- No unrelated formatting or cleanup that obscures the important edits.
- A description of compatibility, data, security, or deployment risks.

If a task requires broad refactoring, split it into safe steps when possible. First make behavior-preserving changes; then make the functional change in a separate, easier-to-review patch.

## A practical review checklist

### 1. Understand the intended behavior

Before reading individual lines, identify:

- What problem is being solved?
- Who or what calls the changed code?
- What should happen on success?
- What should happen for missing, invalid, or unexpected input?
- Are there existing requirements or public API promises that must remain true?

Compare the implementation with the stated goal. A patch can be well-written and still solve the wrong problem.

### 2. Check correctness and edge cases

Follow the data from entry to output. Look for:

- Incorrect conditions, ordering, or calculations.
- Empty collections, missing fields, boundary values, and null values.
- Duplicate requests, retries, and partially completed operations.
- Exceptions that are swallowed, exposed to users, or left undocumented.
- Changes that work for the example but fail on realistic data.
- Assumptions about array shapes, database rows, time zones, encoding, or locale.

Ask whether the code behaves correctly both on the expected path and when something goes wrong.

### 3. Check security and privacy

Consider:

- Is untrusted input validated and safely encoded for its destination?
- Are database values passed through parameterized queries?
- Are authorization checks performed on the server for every protected action?
- Can output expose credentials, personal information, internal paths, or stack traces?
- Are file uploads, redirects, deserialization, shell commands, and external URLs handled safely?
- Does the patch add a dependency or permission that is broader than necessary?
- Are secrets kept out of source code, logs, sample configuration, and test fixtures?

A type declaration or PHPDoc annotation does not replace validation or authorization.

### 4. Check maintainability

Ask whether:

- Names reveal the purpose of functions, values, and classes.
- Each function has a clear responsibility.
- The code follows nearby conventions without copying a known defect.
- Complex logic has useful structure and comments explain why, not merely what.
- Error handling preserves enough context for diagnosis without leaking sensitive details.
- The change duplicates existing behavior that should have one source of truth.
- New abstractions make the use case simpler rather than harder to follow.

Prefer the simplest implementation that clearly satisfies the requirement.

### 5. Check tests and automated feedback

For a change that alters behavior, look for tests covering:

- The expected behavior.
- Important invalid or boundary inputs.
- Error and authorization paths when relevant.
- Regressions in callers or integrations.

Also inspect the project's relevant automated checks, such as:

- Syntax and test suites.
- Static analysis and type checks.
- Formatting or coding-standard checks.
- Dependency vulnerability checks.
- Build, packaging, and deployment checks.

A green check is useful evidence, not a guarantee. Ask what the check actually covers and whether it ran on the intended PHP version and environment.

### 6. Check compatibility and operations

Before approval, consider:

- Does the code require a newer PHP version, extension, database feature, or dependency?
- Does a public method, route, command, event, or data format change?
- Does a database change work while old and new application versions may run at once?
- Are environment variables, worker processes, scheduled tasks, or web-server settings affected?
- Do logs and metrics provide enough information to diagnose failures?
- Can the release be rolled back safely, including its dependencies and data changes?

## Giving and receiving review feedback

Good review comments point to a specific risk and explain its impact. When possible, suggest a concrete alternative.

For example:

> This value comes from the request and can be missing. Could we validate it before passing it to this method? Otherwise this path can raise a TypeError for a normal invalid request.

Prefer comments that distinguish a required correction from a suggestion:

- **Blocking:** a correctness, security, compatibility, or data-loss risk that must be resolved before merge.
- **Suggestion:** a useful improvement that is not required for this change to be safe.
- **Question:** a request for context when the behavior or intent is unclear.

Discuss the code and its effect, not the author's character. Avoid vague comments such as “bad,” “wrong,” or “clean this up” without explaining the problem.

When receiving feedback:

1. Read the comment for the risk it describes.
2. Ask for context if the expected behavior is unclear.
3. Make the correction or explain the relevant constraint.
4. Update tests or documentation when the behavior changes.
5. Reply when a thread is resolved so the reviewer can verify the result.

A review conversation is complete when each important concern has a clear resolution, not merely when every comment has been marked done.

## Maintenance and technical debt

Maintenance is the planned work that keeps software safe to change. It includes fixing defects, simplifying confusing code, updating dependencies, improving observability, removing obsolete paths, and keeping documentation accurate.

Technical debt is a known future cost introduced by a shortcut or a constraint. It is not automatically a reason to rewrite code. Record debt with:

- The affected area.
- The current impact or risk.
- The conditions that would make it more urgent.
- A practical next step and owner when possible.

Refactor in small steps. Preserve behavior while improving structure, then make behavior changes separately when that separation reduces risk. Keep useful tests around the behavior being preserved.

## Dependency maintenance

Third-party packages can contain bugs, security advisories, abandoned projects, and new platform requirements. Maintain them deliberately instead of either ignoring updates or applying every update directly to production.

Composer provides commands that help inspect installed packages:

~~~sh
composer outdated
composer outdated --direct
composer audit
~~~

- **composer outdated** reports packages with updates available.
- **composer outdated --direct** narrows the review to direct dependencies.
- **composer audit** checks installed packages against dependency security policies and advisories.

Review what each command reports; do not treat a version number alone as proof that an update is safe.

### composer.json and composer.lock

The project’s **composer.json** declares dependency constraints and project configuration. The **composer.lock** records the exact resolved versions for an application. Commit the lock file for an application so developers and deployment environments can reproduce the same dependency set.

For a routine application install, **composer install** uses the lock file when it exists. **composer update** resolves versions again within the constraints and changes the lock file. Review that diff carefully; do not casually replace a reproducible install with an unreviewed update.

A deliberate dependency update usually follows this process:

1. Confirm the target release and read its changelog and upgrade notes.
2. Check PHP, extension, and peer-dependency requirements.
3. Update a small, related set of packages.
4. Review changes to composer.json and composer.lock.
5. Run the relevant automated checks on supported PHP versions.
6. Check application behavior in a representative environment.
7. Deploy through the project's normal release process and monitor it.

Do not use a broad update to hide an unexplained dependency conflict. Find which package or constraint causes the conflict and decide whether the project should upgrade, replace, or continue pinning that dependency.

## PHP runtime upgrades

PHP releases have finite support periods. Plan to move from an unsupported runtime before it becomes an emergency. Check the current PHP support schedule during planning; support dates change over time.

A runtime upgrade can affect:

- Syntax and language behavior.
- Deprecated or removed functions and features.
- Type errors and error handling.
- Extensions and their configuration.
- Composer platform requirements.
- Framework and package compatibility.
- PHP-FPM, CLI, queue workers, scheduled tasks, and local development.

Use the migration information for every PHP version crossed. Do not assume that code working under the local CLI proves the web server and background workers use that same runtime.

### A safe upgrade sequence

1. **Inventory the application.** Record the PHP versions used by local development, CI, web requests, command-line tasks, workers, and production.
2. **Choose a supported target.** Check the PHP support schedule and confirm that the deployment platform and required extensions provide it.
3. **Check declared constraints.** Review the Composer PHP requirement and package requirements. Change the project's declared constraint only when the code and toolchain support it.
4. **Read migration notes.** Review each version's migration guide between the current runtime and the target. Identify removed features, behavior changes, and new deprecations.
5. **Find risky usage.** Search the codebase and use static analysis or compatibility tooling already supported by the project. Treat automated reports as leads to investigate.
6. **Upgrade in controlled steps.** Resolve compatibility issues in small changes and avoid mixing unrelated refactors with the runtime migration.
7. **Run checks on the target.** Exercise tests, static analysis, Composer resolution, and critical application flows with the target PHP runtime and required extensions.
8. **Test the deployment path.** Verify CLI, PHP-FPM, workers, schedulers, containers, and build images use the intended runtime.
9. **Deploy with observation.** Release to a test or staged environment first when available. Inspect errors, deprecations, latency, queues, and important user flows.
10. **Keep a recovery plan.** Preserve the last known-good application artifact and compatible runtime/dependency set. Confirm how database changes affect rollback before deployment.

## Database and API changes need compatibility planning

An application can have multiple versions active during a deployment. A new server may run while old workers are still processing jobs, and a rollback may restore the old code after the database has changed.

For database changes, consider an expand-and-contract approach:

1. Add a new field or structure in a way old code can tolerate.
2. Deploy code that can work with both old and new representations.
3. Backfill or migrate data in a controlled, observable process.
4. Switch reads and writes to the new representation.
5. Remove the old structure only after old application versions no longer need it.

For an API or serialized message, consider older clients and queued messages that remain in circulation. Make additive, backward-compatible changes where possible; otherwise plan a version transition and communicate the migration.

## Release and rollback planning

Before a high-risk release, write down:

- The exact application artifact and dependency lock file being deployed.
- Runtime and extension requirements.
- Configuration and database changes.
- Health signals that indicate success or failure.
- Who can pause or roll back the release.
- What rollback means if data has already changed.

A code rollback cannot automatically undo a destructive database migration or reverse an external side effect. Prefer reversible migrations and idempotent jobs. When a release includes an irreversible step, plan a forward repair and make that constraint explicit.

## Common mistakes

- Reviewing only formatting while overlooking behavior and security.
- Merging a large mixed-purpose diff that is difficult to understand.
- Treating a passing test suite as proof that deployment is safe.
- Updating dependencies without reviewing the lock-file changes.
- Changing the PHP requirement without checking all environments and packages.
- Upgrading the web runtime but leaving CLI workers or schedulers on the old version.
- Combining a runtime migration with unrelated refactoring.
- Removing a database column before all deployed code has stopped using it.
- Assuming rollback can undo a data migration or external action.
- Leaving a known risk undocumented because it is inconvenient to fix immediately.

## Review and upgrade checklist

| Area | Questions to answer |
| --- | --- |
| Purpose | Is the change focused and does it meet the stated requirement? |
| Correctness | Are expected, invalid, and boundary cases handled? |
| Security | Are input, output, permissions, secrets, and data access safe? |
| Tests | Is there evidence for new behavior and likely regressions? |
| Compatibility | Are PHP, extensions, dependencies, APIs, and data formats compatible? |
| Dependencies | Are updates intentional, reviewed, and checked for advisories? |
| Operations | Do web, CLI, worker, and scheduled environments use the required setup? |
| Recovery | Is rollback or forward repair understood, including database effects? |
| Documentation | Are setup steps, contracts, and operational notes still accurate? |

## Key takeaways

- Review the purpose and risks of a change before focusing on individual lines.
- Use focused diffs, concrete feedback, and automated checks together.
- Keep maintenance and technical debt visible and manageable.
- Update dependencies and PHP deliberately; inspect changelogs, constraints, and lock-file changes.
- Test the target runtime across every environment that runs the application.
- Plan for deployment overlap, data compatibility, monitoring, and recovery.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [146 - PHPDoc, API documentation, and type annotations](146-phpdoc-api-documentation-and-type-annotations.md) | [Q&A](../qnas/147-code-review-maintenance-and-upgrades.md) | [Example](../examples/147-code-review-maintenance-and-upgrades.md) | [148 - Project: build a personal portfolio page](148-project-build-a-personal-portfolio-page.md) |
