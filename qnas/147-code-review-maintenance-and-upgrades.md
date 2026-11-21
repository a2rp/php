# Code review, maintenance, and upgrades — Q&A

Review safe change practices, dependency and PHP upgrades, compatibility, and recovery planning. Each answer follows its question.

## Multiple-choice questions

### 1. What should a reviewer understand before evaluating individual lines in a change?

- A. Only whether the formatting tool passes.
- B. The problem, expected behavior, callers, and important failure cases.
- C. The author's preferred editor.
- D. Whether every file was rewritten.
**Answer: B.** Understanding the goal and its behavior makes it possible to judge whether the implementation is correct and complete.

### 2. Which review comment is most useful?

- A. “Bad code.”
- B. “Rewrite this.”
- C. “This value can be missing in a request. Could we validate it here? Otherwise an invalid request can trigger a TypeError.”
- D. “I do not like this.”
**Answer: C.** It identifies a concrete risk, explains its impact, and suggests a practical next step.

### 3. For a PHP application, what is the purpose of committing composer.lock?

- A. It stores the exact resolved dependency versions so installs can reproduce the dependency set.
- B. It replaces composer.json.
- C. It records PHP source code.
- D. It automatically upgrades production packages.
**Answer: A.** Applications generally commit the lock file so installs use the reviewed dependency resolution rather than resolving a new set each time.

### 4. What does Composer's audit command check?

- A. Whether every application feature works correctly.
- B. Installed packages against dependency security policies and advisories.
- C. Whether the code is formatted.
- D. Whether database migrations can be rolled back.
**Answer: B.** An audit reports dependency policy and security findings; it does not replace application tests or review.

### 5. What is a sound first step when planning a PHP runtime upgrade?

- A. Upgrade production immediately and inspect errors later.
- B. Inventory the PHP versions, extensions, and processes used by the application.
- C. Delete the lock file.
- D. Change the version constraint without checking the code.
**Answer: B.** The application may use different runtimes for web requests, CLI commands, workers, and CI. The inventory establishes the real upgrade scope.

### 6. Why can a database migration make rollback difficult?

- A. PHP cannot connect to databases after a deployment.
- B. The new schema or data may no longer work with the old application code.
- C. Tests cannot run after migrations.
- D. Git cannot restore PHP files.
**Answer: B.** Code rollback does not reverse a schema or data change. Design migrations and deployment order so old and new code can coexist where needed.

## Short-answer questions

### 7. Why is code review not a substitute for automated tests?

**Answer:** Reviewers can miss defects, and review cannot exercise every path. Automated checks provide repeatable evidence for the behaviors and constraints they cover; human review adds context about intent, risk, and design.

### 8. What makes a change easier to review?

**Answer:** A focused purpose, a manageable diff, relevant tests or other evidence, updated documentation when needed, and an explanation of compatibility or deployment risks.

### 9. What is the difference between a blocking review comment and a suggestion?

**Answer:** A blocking comment identifies a correctness, security, compatibility, or data-loss concern that must be resolved before merge. A suggestion offers an improvement that is not required for the change to be safe.

### 10. How should a reviewer write constructive feedback?

**Answer:** Identify the specific code or behavior, explain the risk or impact, and suggest a concrete correction when possible. Keep the discussion about the change rather than the author's character.

### 11. What is technical debt?

**Answer:** It is a known future cost caused by a shortcut, design constraint, or deferred work. Record its affected area, current risk, conditions that make it more urgent, and a practical next step instead of hiding it or automatically rewriting everything.

### 12. Why should an application usually commit composer.lock?

**Answer:** The lock file records the resolved dependency versions, helping developers, CI, and deployment install the same dependency set. The composer.json file describes constraints and configuration; the lock file captures the chosen resolution.

### 13. What is the difference between Composer install and update for an application with a lock file?

**Answer:** Install uses the locked versions when available. Update resolves packages again within the declared constraints and changes the lock file, so its changes should be reviewed and checked.

### 14. Why does a passing test suite not prove a PHP upgrade is ready for production?

**Answer:** Tests may not cover every user flow, extension, process, or production setting. Verify the target PHP runtime across web, CLI, workers, scheduled tasks, CI, and deployment, then monitor a staged or controlled release.

## Review and maintenance scenarios

### 15. A pull request changes a public method's return value from a string to null in some cases. What should the reviewer ask?

**Answer:** Ask whether callers expect a string, how the new null outcome is handled, and whether the API's native type, PHPDoc, tests, and documentation are updated consistently. If this is a breaking change, the release and migration plan should account for callers.

### 16. A developer runs a broad dependency update to resolve one package conflict. What should be reviewed?

**Answer:** Identify the package and constraint causing the conflict, review the composer.json and composer.lock changes, inspect release notes and platform requirements, run the relevant checks, and decide whether the full set of updates is intentional. Do not hide an unexplained conflict inside a large dependency change.

### 17. Production PHP-FPM is upgraded, but queue workers and scheduled jobs still use the older PHP version. What is the risk?

**Answer:** The same application may behave differently across processes, or new code may use syntax and features unsupported by the old runtime. Inventory and verify every process environment as part of deployment.

### 18. A new application release adds a required database column and immediately removes the old one. What could go wrong?

**Answer:** Old application instances or workers may still read or write the old column during deployment, and rollback may fail. Use staged, backward-compatible migration steps, move traffic and data deliberately, and remove the old field only after old code no longer depends on it.

### 19. A security audit reports a vulnerable direct dependency. What is a responsible next step?

**Answer:** Confirm which installed package and versions are affected, review the advisory and available fixed versions, check compatibility and release notes, update the smallest appropriate set, run relevant checks, and deploy through the normal release process. An audit finding should be investigated rather than ignored or blindly patched.

### 20. A reviewer sees a large diff containing a feature, formatting changes, and unrelated refactoring. Why is that a problem?

**Answer:** The unrelated edits obscure the behavior change and make it harder to find defects. Separate the work into focused changes where practical, or clearly explain why the parts must be reviewed together.

### 21. A release has a destructive database migration. Is reverting the Git commit a sufficient rollback plan?

**Answer:** No. Reverting code does not restore deleted or transformed data. Prefer a reversible migration; otherwise plan and document a forward repair, backup or recovery approach, and the point after which rollback is no longer possible.

## Practical exercise

### 22. You are asked to upgrade a maintained PHP application and several dependencies. List a safe review and rollout plan.

**Answer:** First inventory PHP runtimes, extensions, dependencies, processes, and current tests. Choose a supported target and review PHP migration notes and package release notes. Check Composer constraints and update a small related dependency set; inspect the lock-file diff and audit results. Fix incompatibilities in focused changes, then run tests, static analysis, and critical flows on the target runtime. Verify CLI, web, workers, schedulers, and CI use the intended environment. Stage the release, observe errors and important health signals, and keep a recovery plan for both code and database changes.

### 23. A teammate's patch fixes a bug but has no test for the reported failure. What should you do in review?

**Answer:** Ask for a test that reproduces the failure and verifies the corrected behavior, unless the project has a clear reason that this case cannot be tested automatically. Also consider relevant edge cases and whether the change needs documentation.

**Explanation:** A focused regression test makes the intended behavior repeatable and helps prevent the same defect from returning. Review should still consider risks that the test does not cover.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [146 - PHPDoc, API documentation, and type annotations](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) | [Lesson](../lessons/147-code-review-maintenance-and-upgrades.md) | [Example](../examples/147-code-review-maintenance-and-upgrades.md) | [148 - Project: build a personal portfolio page](../lessons/148-project-build-a-personal-portfolio-page.md) |
