# Formatting, linting, and code quality tools: Questions and answers

## Tool roles

### 1. What does PHP syntax linting check?

**Answer:** Whether PHP can parse a file's syntax. **Explanation:** The linter does not execute the application or prove that the code behaves correctly.

### 2. What command checks the syntax of one PHP file?

**Answer:** Run php -l followed by the file path. **Explanation:** For example, php -l src/CheckoutService.php checks that file without executing it.

### 3. Does a successful PHP syntax check prove that every class and method call is valid?

**Answer:** No. **Explanation:** Syntax linting checks parsing. Static analysis and runtime tests address other concerns such as types, symbols, and behavior.

### 4. What does PHP_CodeSniffer do?

**Answer:** It checks PHP files against a selected coding standard and reports violations. **Explanation:** A ruleset determines which conventions and checks apply.

### 5. What does the phpcbf command do?

**Answer:** It automatically fixes a subset of coding-standard violations detected by PHP_CodeSniffer. **Explanation:** It cannot make every design decision or repair every reported issue, so review its diff.

### 6. What does PHP CS Fixer do?

**Answer:** It applies configured formatting and coding-style rules to PHP files. **Explanation:** It can preview changes without writing them and apply them when explicitly run in fix mode.

### 7. How is Rector different from a formatter?

**Answer:** Rector applies configured code transformations and refactors; a formatter applies layout and style rules. **Explanation:** Refactoring can change program behavior, so review and test Rector's changes.

### 8. Which tool pairing is correct?

A. php -l formats code; PHPCS runs tests
B. PHPCS reports coding-standard violations; phpcbf can fix some
C. Rector validates all business rules
D. A formatter proves a method returns the right value

**Answer:** B. **Explanation:** Each tool has a distinct role. Formatting, syntax checking, testing, and refactoring are not interchangeable.

## Coding standards and formatting

### 9. What is a coding standard?

**Answer:** A set of rules for how code should be written and organized. **Explanation:** A project might use an existing standard such as PSR-12 or a committed custom ruleset.

### 10. Why should a team commit its coding-standard configuration?

**Answer:** So developers and CI apply the same rules. **Explanation:** Without shared configuration, different machines can produce conflicting reports and formatting diffs.

### 11. What is the purpose of phpcs.xml.dist?

**Answer:** It can define the files and coding-standard rules PHP_CodeSniffer should check. **Explanation:** PHP_CodeSniffer can discover a project ruleset file and use it consistently.

### 12. Should third-party files in vendor usually be included in project style checks?

**Answer:** No. **Explanation:** The project does not own those files, and changing their style can create noisy or unsafe diffs.

### 13. Is it useful to run two formatters with overlapping, conflicting rules?

**Answer:** Usually not. **Explanation:** They can undo each other's changes and create recurring diffs. Choose one formatter or clearly separate their responsibilities.

### 14. What is a formatter's dry run for?

**Answer:** Showing whether files need formatting without changing them. **Explanation:** A dry run is useful in CI because it can report style drift while leaving the contributor's files untouched.

### 15. What should you do after a formatter or phpcbf modifies files?

**Answer:** Inspect the diff and run relevant checks. **Explanation:** Automated tools make mechanical changes, but the developer remains responsible for reviewing the result.

## Refactoring and package scripts

### 16. What is Rector commonly used for?

**Answer:** Applying configured, syntax-aware refactors and code upgrades. **Explanation:** It can automate repetitive transformations, but it cannot decide whether every change fits the application's requirements.

### 17. How can you preview Rector's proposed changes?

**Answer:** Run Rector with process --dry-run. **Explanation:** The dry run shows proposed changes without writing them to source files.

### 18. What should happen before committing an applied Rector refactor?

**Answer:** Review the diff and run appropriate analysis and tests. **Explanation:** A transformation that parses correctly can still change behavior.

### 19. Why define Composer scripts for quality commands?

**Answer:** They give the team short, consistent commands to run locally and in CI. **Explanation:** Shared scripts reduce differences in command options and paths.

### 20. Which commands should normally run in CI?

**Answer:** Read-only checks such as lint, coding-standard checks, formatter dry runs, static analysis, and tests. **Explanation:** CI should report problems consistently; it should not silently rewrite a contributor's branch unless the team deliberately chooses that workflow.

### 21. Is the formatter's fix command the same as its check command?

**Answer:** No. **Explanation:** A check or dry-run mode reports changes without writing them; a fix command edits files.

## Workflow and limits

### 22. What is a practical order for quality checks?

**Answer:** Syntax checks, style and format checks, static analysis, then tests; use refactoring previews when needed. **Explanation:** The exact order may vary, but checks should be reproducible and edits should remain explicit.

### 23. Does a clean style report prove the code is correct?

**Answer:** No. **Explanation:** Style tools check configured conventions, not all program behavior, security properties, or business requirements.

### 24. Does a passing test suite prove that all files follow the team's formatting rules?

**Answer:** No. **Explanation:** Tests verify selected behavior. A separate style check enforces coding conventions.

### 25. Does php -l execute a PHP script?

**Answer:** No. **Explanation:** The -l option checks syntax only and does not execute the file.

### 26. A formatter changes quotes, indentation, and blank lines. Is that a refactor?

**Answer:** Usually this is a formatting change. **Explanation:** Formatting aims to preserve behavior, but you should still review diffs and avoid risky rules unless their effects are understood.

### 27. What should you do if PHPCS reports a violation that phpcbf cannot fix?

**Answer:** Read the sniff and fix the code manually, or make a narrow documented exception if there is a valid reason. **Explanation:** A non-fixable report still needs an explicit decision.

### 28. Why use the same tool versions locally and in CI?

**Answer:** To make quality results reproducible. **Explanation:** A committed lock file and configuration help keep the tools and rules consistent.

### 29. What is the difference between a quality check and a quality fix?

**Answer:** A check reports a condition; a fix changes files. **Explanation:** Run fixes intentionally, inspect the changes, and keep CI focused on checks.

### 30. Why should a team avoid running code formatters over generated or vendor files?

**Answer:** Those files are not hand-maintained project source and may be overwritten or create unrelated diffs. **Explanation:** Configure the tools to target code the project owns.

### 31. What should happen if two tools disagree about code style?

**Answer:** Review the configuration and choose a clear source of truth. **Explanation:** Conflicting rules can cause repeated changes and make code review harder.

### 32. Can Rector replace tests?

**Answer:** No. **Explanation:** Rector automates transformations; tests help verify that the transformed program still behaves as intended.

### 33. Can static analysis replace PHP syntax linting?

**Answer:** It can detect many syntax problems while analyzing files, but the tools serve different workflow roles. **Explanation:** A direct syntax check is a quick parser check; static analysis also evaluates types and code paths.

### 34. How should a project introduce automated formatting to an existing codebase?

**Answer:** Agree on rules, preview the changes, review the resulting diff, and apply formatting in a deliberate change. **Explanation:** A large unreviewed formatting sweep can obscure functional changes.

### 35. What is the main lesson about PHP quality tools?

**Answer:** Choose each tool for the problem it solves, share configuration, review edits, and run checks alongside tests and static analysis. **Explanation:** No single tool guarantees code quality by itself.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [121 - Static analysis with PHPStan and Psalm](../lessons/121-static-analysis-with-phpstan-and-psalm.md) | [Lesson](../lessons/122-formatting-linting-and-code-quality-tools.md) | [Example](../examples/122-formatting-linting-and-code-quality-tools.md) | [123 - Secure coding and threat modeling](../lessons/123-secure-coding-and-threat-modeling.md) |