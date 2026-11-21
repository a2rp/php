# 117 - Testing PHP applications with PHPUnit

## Goals

After this lesson, you should be able to:

- Describe what a test verifies and distinguish unit, integration, and end-to-end tests.
- Install PHPUnit as a project development dependency and run it from the command line.
- Write a focused test using Arrange, Act, Assert and an appropriate assertion.
- Verify exceptions and invalid-input behavior.
- Keep tests deterministic, isolated, readable, and independent of execution order.
- Use stubs and mocks only when a dependency boundary needs control or observation.
- Interpret test-runner failures and make a test suite part of continuous integration.

## What a test does

A test exercises code under known conditions and checks observable behavior against an expectation. A useful test can show which behavior is wrong when it fails. Tests help preserve working behavior as code changes, catch regressions, and make assumptions executable. They do not prove that a program has no defects; they provide evidence about the cases the suite covers.

The code a test exercises is its system under test (SUT). A test normally arranges inputs and dependencies, invokes one behavior, and asserts the result or a visible side effect. PHPUnit is both a PHP testing framework and a command-line test runner.

## Test levels

The terms vary a little between teams, but these levels describe useful differences in scope:

| Level | Typical scope | Main trade-off |
| --- | --- | --- |
| Unit test | A small unit of behavior with collaborators controlled or kept simple | Fast and focused, but does not verify external integrations |
| Integration test | Several real components working together, such as application code and a test database | Verifies boundaries, but needs controlled infrastructure and setup |
| End-to-end test | A user-visible flow through the running application and its external boundaries | Gives broad confidence, but is slower and more sensitive to environment changes |

Most tests should give quick feedback about individual behaviors. Add integration and end-to-end tests for important boundaries and flows that isolated tests cannot verify. A unit test that replaces every collaborator may only confirm the mock setup; test real value objects and simple domain logic directly when practical.

## Arrange, Act, Assert

A test can be organized in three simple steps:

1. **Arrange:** create the object and prepare its inputs or collaborators.
2. **Act:** perform the operation being tested.
3. **Assert:** compare the observed result or effect with the expected behavior.

For example, a price calculation test arranges a calculator and input values, calls the calculation method, then asserts the exact cents returned. Keep each test focused on one behavior and avoid unrelated setup that makes the reason for failure hard to see.

## Install PHPUnit for the project

A test runner is part of the project's development toolchain, so install it as a development dependency rather than as an application runtime dependency:

~~~sh
composer require --dev phpunit/phpunit
~~~

Composer selects a PHPUnit release compatible with the PHP runtime and the project's dependency constraints. Commit `composer.json` and `composer.lock` so collaborators and CI use a reproducible tool version. Run PHPUnit through the project's local binary, usually `vendor/bin/phpunit`, or define a Composer script for it.

Match the PHPUnit major version to the PHP versions the project supports. At the time of writing, PHPUnit 13 requires PHP 8.4, and PHPUnit 12 requires PHP 8.3. A project that supports an older PHP version should choose a compatible PHPUnit release rather than forcing the latest major into every environment.

## How PHPUnit finds and runs tests

A PHPUnit test class extends `PHPUnit\Framework\TestCase`. A test method commonly starts with `test`, and a test file conventionally ends in `Test.php`. Point the runner at a test file or directory:

~~~sh
vendor/bin/phpunit tests
~~~

A project can also define a Composer script such as `"test": "phpunit tests"` and run `composer test`. Use a `phpunit.xml` configuration file when the project needs shared suite definitions, bootstrap settings, source paths for coverage, or other runner options. Keep the configuration aligned with the PHPUnit version installed by the lock file.

The runner reports tests, assertions, failures, errors, skipped tests, and warnings or deprecations depending on the situation and version. A failing assertion means the observed behavior did not match its expectation. An error usually means execution could not complete normally, such as an unexpected exception. Read the first failure and its file/line before changing production code or the expectation.

## Assertions should express intent

An assertion states what must be true for the test to pass. Use an assertion that communicates the contract:

- `assertSame($expected, $actual)` checks exact identity of type and value for scalar values, such as the integer `42` versus the string `"42"`.
- `assertEquals($expected, $actual)` checks value equality with PHPUnit's comparison behavior; use it only when that looser comparison is intended.
- `assertTrue()` and `assertFalse()` check a boolean outcome.
- `assertNull()` and `assertNotNull()` check nullability.
- `assertCount()` checks the number of items in a collection.

Prefer an assertion that reveals the expected behavior over a vague assertion. One well-chosen assertion is often clearer than a series of checks about private implementation details.

## Test errors and exceptions

Invalid input and failure paths are part of a public contract. PHPUnit can expect a particular exception:

~~~php
$this->expectException(InvalidArgumentException::class);

$calculator->finalPriceInCents(-1, 10);
~~~

Set the expectation before invoking the operation. The test passes only when the expected exception is thrown; if no exception is thrown or a different exception occurs, the test fails. Add message or code assertions only when those details are part of the behavior callers rely on.

Do not catch an exception in the test and then assert nothing. An unexpected exception should naturally fail the test and retain a useful stack trace.

## Verify behavior, not private implementation

Tests are easier to maintain when they exercise public behavior and stable contracts. Avoid asserting private property values, calling private methods through reflection, or requiring a particular internal sequence of operations unless that sequence itself is contractually important.

When code depends on an external service or an expensive collaborator, a test double can make one boundary predictable:

- A **stub** supplies controlled responses or throws a chosen exception. Use it to control indirect inputs to the system under test.
- A **mock** also checks interactions, such as whether a method was called with a particular argument. Use it when that communication is part of the behavior being tested.
- A **fake** is a lightweight working implementation, such as an in-memory repository, used in place of a production component.

Prefer doubles at boundaries such as a mailer, payment gateway, or repository. Do not mock simple value objects or internal methods just to make tests pass. Too many interaction expectations can couple a test to implementation details and make harmless refactoring expensive.

## Deterministic and isolated tests

A test should produce the same result regardless of when or how often it runs. Avoid depending on the real clock, random values, public network services, shared databases, user-specific files, or another test's state. When a behavior depends on time or randomness, pass a clock or generator through a dependency so the test can control it.

Tests should clean up the state they create. Use a dedicated test database or isolated schema for integration tests; never point destructive test setup at production data. Do not rely on test execution order. If one test passes alone but fails in the suite, look for shared mutable state, leaked files, environment changes, or order-dependent setup.

`setUp()` and `tearDown()` can prepare and release per-test resources. Keep fixture setup small. Large shared fixtures often hide which data a test needs and can make failures difficult to understand. The next lesson covers fixtures and data providers in more detail.

## Data providers and focused cases

When the same behavior needs several independent input/output cases, a data provider can reduce repetitive test code. Keep each provided case easy to identify and understand. For a small number of meaningful cases, separate test methods can be clearer than a complicated provider.

Use descriptive test names such as `testAppliesPercentageDiscountAndRoundsToCents`. The name should help a reader understand the behavior that failed without needing to decode a line number.

## Coverage is not correctness

Code coverage reports which lines or branches ran while a coverage-enabled test suite executed. High coverage does not prove that assertions are meaningful or that the expected behavior is correct. A test can execute every line and still fail to detect a wrong result if it makes no useful assertion.

Start by testing important behavior, boundaries, and failure cases. Add coverage collection when it helps find untested code, using a compatible coverage driver such as PCOV or Xdebug and the configuration required by the installed PHPUnit release. Treat coverage as a map for investigation, not a score that replaces judgment.

## Run tests in continuous integration

A continuous integration job should install the project's locked dependencies and run the same test command used locally. A failing test should produce a nonzero process exit status so the job can block a change or report failure. If the project supports multiple PHP versions, run an appropriate matrix and install a compatible PHPUnit version for each supported runtime.

Keep test configuration separate from production configuration. Use test credentials and isolated test services. Do not let tests send real email, make real payments, or mutate production data.

## A practical review checklist

Before considering a test useful, ask:

1. Is it clear what behavior is under test?
2. Does it exercise the public contract rather than private details?
3. Does the assertion check the exact outcome that matters?
4. Does the test cover a normal case and relevant invalid or failure cases?
5. Can it run repeatedly and in any order without changing its result?
6. Are external services replaced or isolated where appropriate?
7. Does the installed PHPUnit version support every PHP version the project claims to support?
8. Does CI run the same test command against an isolated environment?

## Key takeaways

- Tests exercise behavior under known conditions and provide evidence, not a proof that software is defect-free.
- Use unit, integration, and end-to-end tests for different scopes and trade-offs.
- Arrange, Act, Assert keeps a test easy to understand.
- Choose assertions that express the contract; use strict `assertSame()` when exact type and value matter.
- Test public behavior, invalid input, and important failures without over-mocking internals.
- Keep tests deterministic, isolated, and independent of execution order.
- Install PHPUnit as a project development dependency and match its version to the PHP support range.
- Coverage shows execution, not whether tests assert the right behavior.
- Run the project test command in CI against isolated services and data.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [116 - PHP extensions and the extension ecosystem](116-php-extensions-and-the-extension-ecosystem.md) | [Q&A](../qnas/117-testing-php-applications-with-phpunit.md) | [Example](../examples/117-testing-php-applications-with-phpunit.md) | [118 - Unit tests, fixtures, and data providers](118-unit-tests-fixtures-and-data-providers.md) |
