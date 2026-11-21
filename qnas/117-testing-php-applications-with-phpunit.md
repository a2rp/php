# 117 - Testing PHP applications with PHPUnit: Q&A

## Multiple-choice questions

### 1. What is a useful purpose of an automated test?

- **A.** Check observable behavior under known conditions
- **B.** Prove that a program has no possible defects
- **C.** Replace every kind of manual review
- **D.** Guarantee that production data never changes

**Answer: A.** A test checks expected behavior for the cases it covers. **Explanation:** Tests provide evidence and regression protection, but cannot prove the absence of every defect.

### 2. What does Arrange, Act, Assert mean?

- **A.** Prepare the scenario, invoke the behavior, then check the result
- **B.** Install dependencies, deploy the app, and restart the server
- **C.** Mock every object, call a private method, and check internals
- **D.** Write production code, delete tests, and commit

**Answer: A.** The pattern gives a test a clear setup, action, and verification. **Explanation:** It helps readers see why the test exists and what outcome matters.

### 3. Which PHPUnit assertion checks the same scalar value and type?

- **A.** `assertSame()`
- **B.** `assertEquals()`
- **C.** `assertTrue()`
- **D.** `assertCount()`

**Answer: A.** `assertSame()` checks exact value and type for scalar values. **Explanation:** It distinguishes, for example, the integer `42` from the string `"42"`.

### 4. Where should PHPUnit normally be installed for a specific application project?

- **A.** As a Composer development dependency
- **B.** As a production extension in every PHP runtime
- **C.** As a global package only
- **D.** Inside the application's database

**Answer: A.** A project-local development dependency gives the project a controlled test-runner version. **Explanation:** The lock file helps collaborators and CI use the same version.

### 5. Which command runs a project's PHPUnit executable against the `tests` directory?

- **A.** `vendor/bin/phpunit tests`
- **B.** `php --tests tests`
- **C.** `composer extensions tests`
- **D.** `phpunit --install tests`

**Answer: A.** This runs the local PHPUnit binary against the specified directory. **Explanation:** A Composer script can wrap the same command for convenient and consistent use.

### 6. Which description best fits a unit test?

- **A.** A focused test of a small behavior with its collaborators controlled or simple
- **B.** A test that always requires a live production server
- **C.** A test that verifies the entire user journey through a browser
- **D.** A test that checks only that the application starts

**Answer: A.** Unit tests focus on small behavior and usually run quickly. **Explanation:** Integration and end-to-end tests cover broader component boundaries and flows.

### 7. When should a test call `expectException()`?

- **A.** Before invoking the operation expected to throw
- **B.** Only after catching every exception manually
- **C.** After the test has already ended
- **D.** Only in a production controller

**Answer: A.** Set the expected exception before the act step. **Explanation:** PHPUnit can then fail if the exception is not thrown or has the wrong type.

### 8. What is a test stub most useful for?

- **A.** Supplying controlled responses from a dependency
- **B.** Verifying every private method call in the system under test
- **C.** Replacing the test runner
- **D.** Setting up a production database

**Answer: A.** A stub controls indirect inputs, such as returning a chosen value or throwing an exception. **Explanation:** Use a mock when verifying a specific interaction is part of the contract.

### 9. When should a mock object be preferred over a stub?

- **A.** When a specific interaction with a dependency matters to the behavior
- **B.** Whenever any object appears in the test
- **C.** To make a test faster even when it adds no meaningful check
- **D.** To inspect a private property

**Answer: A.** Mocks can express expectations about calls and arguments. **Explanation:** If interaction does not matter, a stub or real simple object is usually clearer.

### 10. What does code coverage tell you?

- **A.** Which code ran while coverage was collected
- **B.** That every test assertion is correct
- **C.** That production cannot fail
- **D.** That no untested behavior exists

**Answer: A.** Coverage reports execution of lines or branches. **Explanation:** High coverage does not prove assertions are meaningful or that requirements are fully tested.

### 11. Which property makes an automated test suite more reliable?

- **A.** Tests give the same result regardless of order or repetition
- **B.** Each test depends on state created by the previous test
- **C.** Tests call live third-party services for every assertion
- **D.** Tests use real production customer data

**Answer: A.** Deterministic, isolated tests are easier to trust and diagnose. **Explanation:** Order dependencies and uncontrolled external state create flaky results.

### 12. Which statement about PHPUnit compatibility is correct?

- **A.** Choose a PHPUnit version compatible with the PHP versions the project supports
- **B.** PHPUnit's latest major always supports every older PHP version
- **C.** PHPUnit does not run under the CLI SAPI
- **D.** The PHPUnit version has no effect on the project's PHP requirements

**Answer: A.** PHPUnit major releases have their own PHP version requirements. **Explanation:** The development toolchain must work across the project's supported PHP matrix.

## Short-answer questions

### 13. What is the system under test?

**Answer:** It is the code or behavior that the test exercises. **Explanation:** The rest of the test arranges conditions, invokes the system under test, and checks the observable outcome.

### 14. How does an integration test differ from a unit test?

**Answer:** An integration test exercises multiple components working together, often across a real boundary such as a database or filesystem. **Explanation:** It verifies collaboration that a focused unit test may replace with a controlled dependency.

### 15. Why should tests normally assert public behavior instead of private implementation details?

**Answer:** Public behavior is the stable contract callers rely on, while internal structure can change during a safe refactor. **Explanation:** Tests coupled to private details often break even when users observe no behavior change.

### 16. What does `assertEquals()` check differently from `assertSame()`?

**Answer:** `assertEquals()` uses PHPUnit's value-comparison behavior, while `assertSame()` requires exact type and value for scalar checks. **Explanation:** Choose strict comparison when the contract distinguishes types.

### 17. What should a test do when the expected exception is not thrown?

**Answer:** It should fail. **Explanation:** `expectException()` makes the exception part of the behavior being verified rather than silently accepting a successful return.

### 18. What is a test fake?

**Answer:** A fake is a lightweight working implementation used instead of a production component, such as an in-memory repository. **Explanation:** Unlike a stub that returns configured values, a fake can implement a small amount of real behavior.

### 19. Why should tests avoid using production data or services?

**Answer:** Tests can change state, send real messages, incur charges, or expose private data. **Explanation:** Use isolated test infrastructure and fake or stub external boundaries where appropriate.

### 20. Why should a test not rely on another test having run first?

**Answer:** Test order may vary, and isolated execution should produce the same result. **Explanation:** Shared state and order dependence create failures that are hard to reproduce.

### 21. What does a nonzero PHPUnit process exit status allow CI to do?

**Answer:** It signals that the run failed so CI can mark the job as failed and report the problem. **Explanation:** Automated pipelines need a reliable machine-readable result, not just output text.

### 22. When can `setUp()` be useful?

**Answer:** It can create per-test state or resources needed by each test. **Explanation:** Keep it small and specific; large shared fixtures can hide each test's actual inputs.

### 23. What is a data provider useful for?

**Answer:** It supplies several input and expected-result cases to the same focused test. **Explanation:** Use it when cases share one behavior and stay readable; otherwise separate test methods may be clearer.

### 24. Why is test coverage not a quality score by itself?

**Answer:** Coverage measures what ran, not whether tests checked the right results. **Explanation:** Code may be executed without a useful assertion, so test intent and behavior still need review.

## Scenario questions

### 25. A test asserts that `42` is the expected result, but the method returns the string `"42"`. Which assertion makes the type mismatch visible?

**Answer:** Use `assertSame(42, $actual)`. **Explanation:** A strict assertion distinguishes an integer from a numeric string.

### 26. A test needs the payment gateway to return a declined result so it can check the application's response. Should it use a stub or mock?

**Answer:** Use a stub that returns the declined result. **Explanation:** The test needs to control an indirect input; it does not need to verify an interaction unless the gateway call itself is part of the behavior under test.

### 27. A notification must be sent exactly once after an order is accepted. What kind of test double can verify that interaction?

**Answer:** Use a mock for the notification boundary with an expectation of one call and the relevant arguments. **Explanation:** The interaction count and payload are part of the behavior being checked.

### 28. One test fails only when the full suite runs, but passes by itself. What should you investigate?

**Answer:** Look for shared mutable state, leaked files or environment changes, order-dependent setup, and tests that fail to clean up. **Explanation:** Each test should establish its own required state and work in any execution order.

### 29. A test calls a live shipping-rate API and fails whenever the provider is unavailable. How can the unit test become more reliable?

**Answer:** Replace the API boundary with a stub or fake for unit tests, and keep a small number of separately controlled integration tests for the real boundary. **Explanation:** A unit test should not require the public network to verify local business behavior.

### 30. A test expects `InvalidArgumentException`, but calls `expectException()` after the invalid operation. What is wrong?

**Answer:** The expectation was configured too late. Move `expectException()` before the call. **Explanation:** The test runner must know which exception to expect before the operation throws it.

### 31. The team reports 100% line coverage, but a calculation still returns the wrong total. How can that happen?

**Answer:** The tests may execute every line without asserting the correct result for the relevant inputs. **Explanation:** Coverage measures execution, not assertion quality or requirement completeness.

### 32. A project supports PHP 8.3, but its tool update selects a PHPUnit major that requires PHP 8.4. What should the team do?

**Answer:** Select a PHPUnit release compatible with the project's minimum PHP version or raise the project's supported PHP minimum deliberately. **Explanation:** Development tools and test runners must be runnable in every environment where the suite is expected to run.

### 33. A CI job prints a PHPUnit failure but still reports success. What should be checked?

**Answer:** Check that CI runs the test command directly or propagates its exit status and does not mask failures with shell logic. **Explanation:** The pipeline must treat a nonzero runner exit as a failed job.

### 34. A test mutates a shared fixture, and later tests unexpectedly see the changed data. What is a better design?

**Answer:** Give each test fresh state or explicitly reset the fixture between tests. **Explanation:** Shared mutable data makes outcomes depend on execution order.

### 35. A test fails because it inspects a private property even though the public method returns the expected result. What should be changed?

**Answer:** Assert the public result or other observable contract and remove the private-state assertion unless that state is itself a documented contract. **Explanation:** Tests should protect behavior rather than freeze implementation structure.

### 36. The PHPUnit run succeeds locally but fails on the CI runner because an extension is missing. What should the team add?

**Answer:** Declare the required extension in project platform requirements and configure CI to install or enable it for the PHP runtime used to run tests. **Explanation:** A local toolchain may have capabilities that a clean CI environment does not.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [116 - PHP extensions and the extension ecosystem](../lessons/116-php-extensions-and-the-extension-ecosystem.md) | [Lesson](../lessons/117-testing-php-applications-with-phpunit.md) | [Example](../examples/117-testing-php-applications-with-phpunit.md) | [118 - Unit tests, fixtures, and data providers](../lessons/118-unit-tests-fixtures-and-data-providers.md) |
