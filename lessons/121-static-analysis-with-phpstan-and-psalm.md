# Static analysis with PHPStan and Psalm

## What static analysis does

Static analysis examines source code without running the application. It uses PHP syntax, native type declarations, PHPDoc, and control-flow information to find code that may be incorrect.

For example, an analyzer can report a method call on a value that might be null, a misspelled method, a value returned with the wrong type, or an array key that is not guaranteed to exist.

Static analysis complements tests:

- **Static analysis** checks code paths and type assumptions that can be understood from the source.
- **Tests** execute selected behavior and verify runtime results.
- **Both** help catch different classes of defects. Neither proves that the application meets every business requirement.

PHPStan and Psalm are separate tools with overlapping capabilities. Both understand PHP types and PHPDoc, but their configuration formats, issue names, strictness settings, and some type features differ.

## Why type information matters

Native types describe the broad contract PHP itself can enforce:

~~~php
function findUser(int $id): ?User
{
    // ...
}
~~~

PHPDoc can describe useful details that native PHP type declarations cannot express directly, such as a list of strings, a generic collection, or an array with named keys:

~~~php
/**
 * @param list<string> $roles
 * @return array{id: positive-int, email: non-empty-string}
 */
function userSummary(array $roles): array
{
    // ...
}
~~~

More precise types give an analyzer more information. They also help people understand how a function should be called and what it returns.

Avoid using **mixed** as a shortcut when the input is known to have a more specific shape. When a value truly starts as unknown, accept it as mixed, validate it, and return a precise type.

## Install the analyzers

Install one or both as development dependencies with Composer:

~~~sh
composer require --dev phpstan/phpstan vimeo/psalm
~~~

Choose package versions compatible with the PHP version and dependencies used by the project. Composer puts the command-line tools in the project's vendor/bin directory.

Run an analysis from the project root:

~~~sh
vendor/bin/phpstan analyse src tests
vendor/bin/psalm
~~~

On Windows, Composer also provides .bat command shims:

~~~powershell
vendor/bin/phpstan.bat analyse src tests
vendor/bin/psalm.bat
~~~

Run only the analyzer or analyzers your project has installed. A project can use either tool independently; it does not need both.

## Configure the project scope

A PHPStan configuration file can keep the analyzed paths and rule level consistent.

### File: phpstan.neon

~~~neon
parameters:
    level: 6
    paths:
        - src
        - tests
~~~

PHPStan has cumulative rule levels from 0 through 10. A higher level enables stricter checks. Start at a level the project can maintain, fix its findings, and increase it as the code becomes more precise. The levels are not equivalent to Psalm's error levels.

Psalm can generate a starting configuration by scanning the project:

~~~sh
vendor/bin/psalm --init
~~~

A compact configuration can specify which project files to inspect:

### File: psalm.xml

~~~xml
<?xml version="1.0"?>
<psalm errorLevel="4">
    <projectFiles>
        <directory name="src" />
        <directory name="tests" />
    </projectFiles>
</psalm>
~~~

Review the generated or written configuration rather than assuming that two analyzers use the same defaults. Keep third-party dependencies under vendor out of the project paths; the analyzers use installed dependency metadata to understand symbols without treating vendor code as your own source.

## Validate unknown input and return a precise shape

Consider data decoded from an external request. The value can be anything until the application checks it. This function accepts an unknown value, validates it, and returns an array shape with a non-empty email and a boolean option.

~~~php
<?php

declare(strict_types=1);

/**
 * @return array{email: non-empty-string, marketing: bool}
 */
function parseSignupInput(mixed $input): array
{
    if (!is_array($input)) {
        throw new InvalidArgumentException('Expected an object-like array.');
    }

    $email = $input['email'] ?? null;

    if (!is_string($email)) {
        throw new InvalidArgumentException('Email must be a string.');
    }

    $email = trim($email);

    if ($email === '') {
        throw new InvalidArgumentException('Email cannot be empty.');
    }

    $marketing = $input['marketing'] ?? false;

    if (!is_bool($marketing)) {
        throw new InvalidArgumentException('Marketing must be a boolean.');
    }

    return [
        'email' => $email,
        'marketing' => $marketing,
    ];
}
~~~

The native parameter type says that input can be any PHP value. Runtime checks establish that it is an array with the required values. The return PHPDoc documents the validated result for callers and analyzers.

A type annotation does not validate untrusted data at runtime. Keep the checks even if the annotation makes the code easier to analyze.

## Read and act on findings

When an analyzer reports an issue:

1. Read the full message, file, line, and inferred type.
2. Check the surrounding code and determine whether the report reveals a real defect.
3. Fix the implementation or add the missing runtime validation.
4. If the code is correct, improve the type declaration or PHPDoc so it accurately describes the behavior.
5. Configure a framework bootstrap, extension, or stub only when the analyzer genuinely lacks information.
6. Suppress a finding only when there is a specific, documented reason and the suppression is as narrow as possible.

Do not add a broad ignore rule just to make a command pass. An incorrect ignore can hide future defects in unrelated code.

## Adopt analysis incrementally with a baseline

A mature codebase may already have many findings. A baseline records known findings so a team can start using the analyzer while preventing the list from growing.

For PHPStan, generate a baseline after choosing paths and a rule level:

~~~sh
vendor/bin/phpstan analyse --generate-baseline
~~~

For Psalm, generate a baseline file:

~~~sh
vendor/bin/psalm --set-baseline=psalm-baseline.xml
~~~

Commit the baseline so local development and continuous integration use the same state. A baseline is a migration aid, not a permanent substitute for fixes. Review it, avoid adding new entries casually, and remove entries when the underlying issue is fixed.

A useful progression is:

1. Analyze the code you own.
2. Fix clear errors first.
3. Create a baseline only for remaining known issues.
4. Run analysis in continuous integration.
5. Stop new issues from being introduced.
6. Reduce old baseline findings and raise strictness over time.

## Add analysis to continuous integration

Run the same commands in a local shell and in CI:

~~~sh
vendor/bin/phpstan analyse
vendor/bin/psalm
~~~

Configure the paths and settings in committed configuration files. An analyzer normally exits unsuccessfully when it finds an unignored issue, so a CI job can prevent a change from merging until the report is addressed.

Keep analyzer versions and configuration consistent across developer machines and CI, usually by committing Composer's lock file and installing dependencies from it.

## What static analysis cannot guarantee

Static analysis reasons from the information available to it. Dynamic properties, reflection, magic methods, framework conventions, generated code, or incomplete third-party metadata may require configuration or extensions.

An analyzer also cannot automatically prove that:

- a business rule is the right one;
- a database or remote service will behave as expected;
- every runtime input has been tested;
- a user interface or workflow is useful;
- a reported security flow is exploitable in every deployment context.

Use runtime validation for external data, tests for behavior, and review for requirements and design. Treat analyzer output as useful evidence, not as a replacement for those practices.

## Key takeaways

- Static analysis inspects code without executing it.
- PHPStan and Psalm have overlapping features, but their settings and reports are not interchangeable.
- Native types and precise PHPDoc make analysis more useful.
- Validate unknown input at runtime before returning a precise type.
- Fix findings where possible; use narrow, explained suppressions only when needed.
- A baseline supports gradual adoption and should shrink over time.
- Run analysis locally and in CI, alongside tests.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [120 - Test doubles, stubs, and mocks](120-test-doubles-stubs-and-mocks.md) | [Q&A](../qnas/121-static-analysis-with-phpstan-and-psalm.md) | [Example](../examples/121-static-analysis-with-phpstan-and-psalm.md) | [122 - Formatting, linting, and code quality tools](122-formatting-linting-and-code-quality-tools.md) |