# Formatting, linting, and code quality tools

## Why teams use these tools

A consistent development workflow catches different problems at different stages. PHP projects commonly use:

- **PHP linting** to check whether source files have valid PHP syntax.
- **Coding-standard tools** to report naming, spacing, and other configured rule violations.
- **Formatters** to apply a consistent layout automatically.
- **Refactoring tools** to suggest or apply structured code transformations.
- **Static analyzers and tests** to check types and behavior.

These tools have different jobs. A formatter does not prove a function returns the right result. A syntax check does not verify that a method exists. Tests do not automatically enforce one coding style.

## PHP syntax linting

PHP's command-line linter checks syntax without executing the file:

~~~sh
php -l src/CheckoutService.php
~~~

A successful result means that PHP could parse that file. It does not prove that the program behaves correctly, that all referenced services are available, or that every file in the project has been checked.

On Windows PowerShell, lint every PHP file in a directory tree with:

~~~powershell
Get-ChildItem -Recurse -Filter *.php src, tests |
    ForEach-Object { php -l $_.FullName }
~~~

For a small project, the direct command is useful. In a larger project, use a maintained lint script or CI action that checks all project PHP files and returns a failing exit code if any file has invalid syntax.

## Coding standards with PHP_CodeSniffer

PHP_CodeSniffer has two main commands:

- **phpcs** checks files against a selected coding standard and reports violations.
- **phpcbf** automatically fixes a subset of the reported violations.

Install it as a development dependency:

~~~sh
composer require --dev squizlabs/php_codesniffer
~~~

A project ruleset can be saved as **phpcs.xml.dist** at the repository root:

~~~xml
<?xml version="1.0"?>
<ruleset name="Project">
    <description>Shared coding rules for this project.</description>

    <file>src</file>
    <file>tests</file>

    <exclude-pattern>*/vendor/*</exclude-pattern>

    <rule ref="PSR12"/>
</ruleset>
~~~

Run the checker and fixer:

~~~sh
vendor/bin/phpcs
vendor/bin/phpcbf
~~~

When a ruleset file is present, PHP_CodeSniffer can discover and use it automatically. Review the diff after running the fixer: it fixes many mechanical violations, but it does not decide how every code-quality concern should be handled.

On Windows, Composer also provides command shims such as **vendor/bin/phpcs.bat** and **vendor/bin/phpcbf.bat**.

## Formatting with PHP CS Fixer

PHP CS Fixer applies configured formatting rules to PHP files. It can be used as an alternative formatter or alongside a clearly defined coding-standard check.

Install it as a development dependency:

~~~sh
composer require --dev friendsofphp/php-cs-fixer
~~~

Preview changes without writing them:

~~~sh
vendor/bin/php-cs-fixer fix --dry-run --diff
~~~

Apply the configured fixes:

~~~sh
vendor/bin/php-cs-fixer fix
~~~

The dry run is useful in CI because it reports files that need formatting without modifying the working tree. A formatter follows its configured rules; the project still needs to choose and commit a configuration so everyone uses the same conventions.

Do not run two formatters with conflicting rules. They can undo each other's changes and create repeated diffs. Choose one formatter, or configure each tool so their responsibilities do not overlap.

## Automated refactoring with Rector

Rector applies configured, syntax-aware transformations such as code upgrades and repetitive refactors. It can modify files, so preview its proposed changes before applying them.

Install it as a development dependency:

~~~sh
composer require --dev rector/rector
~~~

Preview the configured changes:

~~~sh
vendor/bin/rector process --dry-run
~~~

After reviewing the diff, apply the transformations:

~~~sh
vendor/bin/rector process
~~~

A dry run makes no file changes. An applied refactor still needs review and appropriate tests: a structurally valid transformation can change behavior, and the project must verify the result against its own requirements.

## Put common commands in Composer scripts

Composer scripts give the team short, shared commands. Add the relevant entries to the existing scripts object in **composer.json**:

~~~json
{
    "scripts": {
        "style:check": "phpcs",
        "style:fix": "phpcbf",
        "format:check": "php-cs-fixer fix --dry-run --diff",
        "format:fix": "php-cs-fixer fix",
        "refactor:check": "rector process --dry-run"
    }
}
~~~

Then run the checks from the project root:

~~~sh
composer style:check
composer format:check
composer refactor:check
~~~

Use the fix commands intentionally during development, inspect the resulting diff, and keep the checks in CI. Do not configure CI to silently rewrite a contributor's branch unless the team has deliberately chosen that workflow.

## A practical quality workflow

A useful local and CI sequence is:

1. Run PHP syntax checks.
2. Run the coding-standard check and formatter check.
3. Review or apply any suggested formatting fixes.
4. Run static analysis.
5. Run automated tests.
6. Use Rector in dry-run mode when reviewing possible refactors.
7. Inspect and test any refactor before committing it.

The exact order can vary by project. The important part is to keep the commands reproducible and to distinguish checks from commands that edit files.

## What these tools do not guarantee

- A clean syntax check does not prove correct behavior.
- A clean style report does not prove safe or maintainable design.
- A formatter does not understand every project-specific design decision.
- A refactoring tool does not replace code review or tests.
- A passing test suite does not automatically enforce formatting or static-analysis rules.

Use the appropriate tool for each concern. Keep configuration in the repository, keep third-party code out of project checks, and review automated changes before committing them.

## Key takeaways

- PHP linting checks syntax without executing application code.
- PHP_CodeSniffer reports coding-standard violations; PHP_Code Beautifier and Fixer can fix many of them.
- PHP CS Fixer applies configured formatting rules and supports a no-write preview.
- Rector performs configured code transformations; preview and review those changes.
- Keep formatting rules and commands consistent across the team and CI.
- Run tests and static analysis in addition to style and syntax checks.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [121 - Static analysis with PHPStan and Psalm](121-static-analysis-with-phpstan-and-psalm.md) | [Q&A](../qnas/122-formatting-linting-and-code-quality-tools.md) | [Example](../examples/122-formatting-linting-and-code-quality-tools.md) | [123 - Secure coding and threat modeling](123-secure-coding-and-threat-modeling.md) |