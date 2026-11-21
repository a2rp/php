# 118 - Unit tests, fixtures, and data providers

## Goals

After this lesson, you should be able to:

- Build a focused unit test around one behavior and a small system under test.
- Create a fresh fixture for each test with `setUp()` and clean up resources with `tearDown()` when needed.
- Decide whether shared setup belongs in a test method, per-test hook, or class-level hook.
- Use PHPUnit data providers to run the same test logic against distinct input and expected-result cases.
- Name and filter data sets so failures point to a useful scenario.
- Keep provider data simple, deterministic, and independent of the test case instance.
- Avoid shared mutable fixtures and unnecessary dependencies between tests.

## Unit tests and fixtures

A unit test focuses on a small behavior, usually by exercising one class or function while keeping collaborators simple. The **fixture** is the state and objects needed to run the test: inputs, the system under test, and any controlled collaborators.

A good fixture is small and explicit. When readers can see the setup next to the action and assertion, they can understand what the test proves. A very large fixture can obscure the behavior and make failures harder to diagnose.

## Start with one behavior per test

A focused test follows Arrange, Act, Assert:

1. **Arrange:** create the object and provide the values or collaborators needed for this case.
2. **Act:** call the public operation under test.
3. **Assert:** compare the result or observable effect with what should happen.

For example, a cart total test should add a known set of items, call `totalInCents()`, and assert the exact total. Invalid input should usually have its own test with an expected exception. Separate tests make it clear which behavior failed.

Test names should describe behavior, such as `testRejectsAQuantityBelowOne`. Avoid names such as `testMethod2`; a useful failure report should explain the case before anyone opens the source file.

## Use `setUp()` for fresh per-test fixtures

PHPUnit creates a fresh test-case instance for each test method and runs `setUp()` before that test. Use it for small setup shared by several tests in the class:

~~~php
protected function setUp(): void
{
    $this->cart = new Cart();
}
~~~

Each test starts with its own cart. A mutation in one test cannot silently affect the next test. If only one test needs an object, creating it inside that test can be clearer than adding a class property and setup hook.

`tearDown()` runs after a test and is useful when that test owns a resource that must be released, such as a temporary file or a manually started process:

~~~php
protected function tearDown(): void
{
    if (is_file($this->temporaryFile)) {
        unlink($this->temporaryFile);
    }
}
~~~

Do not add teardown code when PHP can release a simple local object automatically. Keep cleanup explicit and make it safe when setup or the test fails partway through. Prefer dedicated temporary directories and test databases over shared or production resources.

## Class-level setup is for expensive shared resources

`setUpBeforeClass()` and `tearDownAfterClass()` run at the class level rather than once per test. They can be useful for an expensive resource that is safe for tests in that class to share, but shared mutable state can introduce order dependence. Start with a fresh per-test fixture. Use class-level state only when the cost or resource lifecycle justifies the extra coupling, and make sure the shared resource is reset between cases.

## Repeated cases with a data provider

A data provider supplies several input sets to one test method. PHPUnit runs the method once for each set, and each set is reported as its own test. This is useful when the cases have the same behavior and assertion structure but different inputs or expected results.

In PHPUnit 13, connect a public static provider method to the test with `#[DataProvider('providerMethodName')]`:

~~~php
use PHPUnit\\Framework\\Attributes\\DataProvider;
use PHPUnit\\Framework\\TestCase;

final class AdditionTest extends TestCase
{
    public static function additionCases(): array
    {
        return [
            'adding zeros' => [0, 0, 0],
            'adding one to zero' => [0, 1, 1],
            'adding positive values' => [4, 7, 11],
        ];
    }

    #[DataProvider('additionCases')]
    public function testAddsTwoNumbers(int $left, int $right, int $expected): void
    {
        $this->assertSame($expected, $left + $right);
    }
}
~~~

Every inner array supplies the arguments for one invocation of the test method, in order. The string keys name the cases. If the `positive values` case fails, the runner can report that scenario instead of only showing a numeric data-set index.

A provider method must be `public` and `static`, and return an iterable of arrays. Its name must not start with `test`. Providers are evaluated before `setUp()` and before PHPUnit constructs the individual test execution state, so they cannot read `$this` or a fixture property. Keep providers declarative: provide scalar values, immutable value objects, or test stubs. Do not open files, connect to a database, create mocks, or build a large service graph in a provider.

## When a provider helps

Use a provider when:

- The same test logic applies to many inputs and expected values.
- Each case should appear separately in the test output.
- Adding a scenario should mostly mean adding data, not duplicating a test body.

Use separate methods when cases require different setup, different assertions, or different explanations. Do not combine unrelated behaviors into one data provider just to reduce the number of methods. Repetition is not a problem when separate tests communicate different contracts more clearly.

## Naming, filtering, and diagnosing cases

Give data sets meaningful names. Names should describe the condition or edge case, such as `empty cart`, `one item`, or `rejects negative price`. Avoid names that repeat the method name or merely number the scenarios.

The PHPUnit `--filter` option can select a test or a named data set while debugging. A data-provider case appears as a separate test, so a concise name helps reproduce and diagnose the failing input. Check the runner's reported data-set name and arguments before changing the expected value.

## Fixtures should not hide behavior

A fixture should help explain the test, not hide its subject. Consider these choices:

- Create one-off values directly in the test when they are simple.
- Use a small object factory when repeated setup has meaningful defaults.
- Use `setUp()` for a small object that every test in the class needs.
- Use a fake or stub for a boundary that needs predictable behavior.
- Use a dedicated integration fixture for a database or filesystem boundary.

Avoid one enormous fixture shared by many test classes. It can make individual cases depend on unrelated defaults and turn a small change into a suite-wide failure. Keep test data close enough to the test that a reader can understand what it means.

## Prefer independent tests over test dependencies

A test should normally create the fixture it needs rather than depend on another test to return an object. Dependencies make a test harder to run or debug alone, and a failed producer can cause dependent tests to be skipped. Reusing a fixture through `setUp()` or a small factory is usually simpler. Use PHPUnit test dependencies only when the shared result is genuinely expensive and the relationship is intentional.

## Avoid common provider and fixture mistakes

- Do not read a test-case property inside a static provider; the provider runs before per-test setup.
- Do not put database connections, services, or large mutable objects into provider data.
- Do not create a mock in a provider; create it during test setup or in the test body.
- Do not use one mutable object instance for several data sets if one case can change another.
- Do not use a provider when each case needs a different test structure.
- Do not make assertions in `setUp()`; keep behavior checks in test methods so failures identify a scenario.
- Do not write teardown that deletes paths or records outside a test-owned, isolated area.
- Do not make every property static just to share setup.

## A practical review checklist

1. Is each test focused on one behavior?
2. Does the fixture make the scenario easy to understand?
3. Does each test receive fresh mutable state?
4. Is cleanup present only for resources that require it?
5. Would a data provider make the cases clearer and separately reportable?
6. Is the provider public, static, deterministic, and limited to simple data?
7. Are data sets named after meaningful scenarios?
8. Can each test run by itself and in any order?
9. Are expensive shared fixtures necessary, reset, and isolated?

## Key takeaways

- A fixture is the state and objects a test needs; keep it small and visible.
- `setUp()` gives each test a fresh per-test fixture; `tearDown()` releases resources that need explicit cleanup.
- Class-level hooks share resources and can create coupling, so use them deliberately.
- A PHPUnit data provider is a public static method returning iterable arrays for separate test invocations.
- Providers run before per-test setup and cannot use `$this`; keep them free of resource creation and large object graphs.
- Named data sets make failures easier to understand and filter.
- Use providers for the same behavior with different data; write separate methods when cases have different structure.
- Independent tests are usually clearer than test-to-test dependencies.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [117 - Testing PHP applications with PHPUnit](117-testing-php-applications-with-phpunit.md) | [Q&A](../qnas/118-unit-tests-fixtures-and-data-providers.md) | [Example](../examples/118-unit-tests-fixtures-and-data-providers.md) | [119 - Integration tests for databases and HTTP](119-integration-tests-for-databases-and-http.md) |
