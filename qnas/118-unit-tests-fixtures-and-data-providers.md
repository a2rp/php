# 118 - Unit tests, fixtures, and data providers: Q&A

## Multiple-choice questions

### 1. What is a test fixture?

- **A.** The state and objects a test needs to run
- **B.** A PHPUnit version number
- **C.** A production database backup
- **D.** A report that lists private methods

**Answer: A.** A fixture includes the inputs, system under test, and any collaborators needed by a test. **Explanation:** Keeping it small and visible helps readers understand the scenario.

### 2. When does PHPUnit normally call `setUp()`?

- **A.** Before each test method on that test's fresh test-case instance
- **B.** Once for the whole application process
- **C.** Only after a test fails
- **D.** Once for every data provider class

**Answer: A.** `setUp()` prepares per-test state before the test method runs. **Explanation:** Each test can start with a fresh fixture rather than inherit mutations from another case.

### 3. What is `tearDown()` primarily for?

- **A.** Releasing resources a test explicitly acquired
- **B.** Making assertions before the test runs
- **C.** Reordering the test suite
- **D.** Returning data-provider values

**Answer: A.** Use `tearDown()` for cleanup such as removing a test-owned temporary file. **Explanation:** Simple local objects usually do not need explicit teardown code.

### 4. What is one risk of `setUpBeforeClass()`?

- **A.** Shared mutable state can couple tests or make their order matter
- **B.** It always runs after every test method
- **C.** It prevents the test class from using assertions
- **D.** It creates a separate PHP process for each assertion

**Answer: A.** Class-level setup shares a resource across tests. **Explanation:** Use it only when the cost or lifecycle justifies sharing, and reset state as needed.

### 5. Which PHPUnit attribute connects a test to a provider method in the same class?

- **A.** `#[DataProvider('cases')]`
- **B.** `#[Fixture('cases')]`
- **C.** `#[SetUp('cases')]`
- **D.** `#[TestData('cases')]`

**Answer: A.** `DataProvider` names a static provider method. **Explanation:** Each provided data set invokes the test separately.

### 6. Which signature is appropriate for a PHPUnit data provider method?

- **A.** A public static method returning iterable data sets
- **B.** A private instance method reading `$this->fixture`
- **C.** A protected method that calls assertions
- **D.** A test method that returns a database connection

**Answer: A.** A provider is public and static and yields arrays of test arguments. **Explanation:** It runs before per-test setup and cannot use the current test instance.

### 7. How is one inner array from a data provider normally used?

- **A.** As the arguments for one invocation of the test method
- **B.** As the output file for the test report
- **C.** As a database transaction
- **D.** As PHPUnit configuration

**Answer: A.** Each inner array supplies the test method's arguments in order. **Explanation:** Three data sets produce three executions of that test method.

### 8. What is the benefit of naming provider data sets?

- **A.** The failure report can identify a scenario such as `empty cart`
- **B.** PHPUnit skips all other tests automatically
- **C.** The provider can access `$this`
- **D.** It makes the data set run only once per test class

**Answer: A.** Named data sets make failures easier to identify and filter. **Explanation:** A meaningful scenario name is more useful than a numeric case index.

### 9. What should a provider usually return?

- **A.** Simple deterministic values, immutable value objects, or test stubs arranged as data sets
- **B.** A live database connection for every case
- **C.** A mock object for each test method
- **D.** A production service container

**Answer: A.** Providers should describe data, not perform fixture construction or external setup. **Explanation:** Keep resource creation in test setup or the test body.

### 10. When are separate test methods clearer than a data provider?

- **A.** When cases need different setup, assertions, or explanations
- **B.** Whenever two cases share a behavior
- **C.** Only when there is no assertion
- **D.** Never; all cases should use providers

**Answer: A.** Different test structures are easier to explain in separate methods. **Explanation:** Providers are for the same behavior with varying data, not a way to hide unrelated tests.

### 11. Which command can help run a single named test or provider case?

- **A.** `vendor/bin/phpunit --filter`
- **B.** `composer --reset-fixtures`
- **C.** `php --data-provider`
- **D.** `vendor/bin/phpunit --setUp`

**Answer: A.** PHPUnit's `--filter` option selects tests by name and can target data sets. **Explanation:** A concise case name helps isolate a failing scenario.

### 12. Which is usually a better default than passing one test's object to another test as a dependency?

- **A.** Create the needed fixture in the test or `setUp()`
- **B.** Require the tests to run in file order
- **C.** Store the object in a global variable
- **D.** Use a static property for every fixture

**Answer: A.** Independent fixture creation makes each test easier to run and debug alone. **Explanation:** Test dependencies can make consumers depend on a producer's execution and state.

## Short-answer questions

### 13. What does a small fixture make easier?

**Answer:** It makes the test's scenario and relevant inputs easy to see. **Explanation:** Readers can follow the behavior without tracing a large hidden object graph.

### 14. What is the normal purpose of `setUp()`?

**Answer:** Prepare state needed by one test method. **Explanation:** PHPUnit invokes it for each test, allowing the class to begin from a fresh fixture.

### 15. When is `tearDown()` appropriate?

**Answer:** When the test must release a resource it explicitly acquired, such as a temporary file. **Explanation:** Do not add cleanup hooks for ordinary values PHP can release automatically.

### 16. How does `setUpBeforeClass()` differ from `setUp()`?

**Answer:** `setUpBeforeClass()` runs once for a test class, while `setUp()` prepares state for each test. **Explanation:** Class-level state is shared and can create coupling or order dependence.

### 17. Why should a data provider not read `$this->cart`?

**Answer:** The provider is static and is evaluated before per-test setup and the test instance's fixture is available. **Explanation:** It should supply input data rather than depend on instance state.

### 18. What is a named data set?

**Answer:** A provider case associated with a descriptive string key, such as `empty cart`. **Explanation:** PHPUnit can include that name in test output to identify a failure.

### 19. When is a provider useful?

**Answer:** When the same test logic and assertions should run against several input and expected-result combinations. **Explanation:** Each case is independently reported, while the shared behavior stays in one test body.

### 20. Why should providers avoid database connections and filesystem setup?

**Answer:** Providers run before the per-test lifecycle and should not hide resource work in data enumeration. **Explanation:** Resource setup belongs in explicit fixture setup, and each case needs isolated cleanup.

### 21. Can a PHPUnit mock object be created in a data provider?

**Answer:** No; create it in `setUp()` or in the test method instead. **Explanation:** Provider data should remain simple and available before each test's lifecycle hooks.

### 22. How many test executions result from three cases in one provider?

**Answer:** Three executions of the associated test method. **Explanation:** PHPUnit reports and counts each data set separately.

### 23. Why should test-to-test dependencies be used sparingly?

**Answer:** A dependent test can be hard to run or understand on its own and may be skipped when the producer fails. **Explanation:** Creating a fresh fixture in the test or `setUp()` is usually more direct.

### 24. What kinds of data are suitable for provider cases?

**Answer:** Prefer scalar values, immutable value objects, and test stubs. **Explanation:** Avoid large object graphs, services, mutable shared fixtures, and mocks.

## Scenario questions

### 25. A test class has five methods that all need a fresh `Cart`. Where should it create the cart?

**Answer:** Create a new cart in `setUp()` and store it in a test-case property. **Explanation:** Each method receives a fresh cart, so one test's mutations do not leak into another.

### 26. One test creates a temporary file and must remove it even if an assertion fails. Where should cleanup go?

**Answer:** Track the test-owned path and remove it in `tearDown()`, with cleanup guarded so it is safe if the file was not created. **Explanation:** External resources need explicit lifecycle management.

### 27. A provider accesses `$this->database` and fails before the first `setUp()` call. What is the design mistake?

**Answer:** The static provider depends on per-test instance state. Return simple case data from the provider and create the database fixture in setup or the test body. **Explanation:** Providers execute before per-test fixtures are prepared.

### 28. A test has three input sets for the same calculation and identical assertion logic. What structure fits?

**Answer:** A named data provider connected with `#[DataProvider('...')]`. **Explanation:** It keeps one behavior assertion while each meaningful input set appears as a separate test.

### 29. Four scenarios require different setup and verify unrelated outcomes. Should they be bundled into one provider?

**Answer:** No. Use separate test methods or separate providers aligned to each behavior. **Explanation:** A provider is clearest when all cases share the same test structure.

### 30. A provider returns three unnamed arrays and the second case fails. How can the failure report be easier to interpret?

**Answer:** Add descriptive string keys to the data sets, such as `zero quantity`, `one item`, and `multiple lines`. **Explanation:** Named cases expose the scenario in runner output and filtering.

### 31. A mutable object is created once in a provider and passed to several test cases. One case changes it and another now fails. What should change?

**Answer:** Provide immutable/scalar data and create a fresh mutable object inside each test or `setUp()`. **Explanation:** Shared mutable provider values make cases order-dependent.

### 32. A test class shares a database fixture through `setUpBeforeClass()`, and rows from one test appear in another. What is the issue?

**Answer:** The class-level resource is shared without adequate reset or isolation. Use per-test transactions, isolated data, or fresh fixture setup and teardown. **Explanation:** Class-level hooks run once, so mutable state can leak across tests.

### 33. One test needs an object, while every other test does not. Should it be created in `setUp()`?

**Answer:** Usually create it inside the one test that needs it. **Explanation:** Per-test setup is best for small state shared by the class; one-off setup is clearest next to its use.

### 34. A named provider test fails only on the `negative price` case. What should a developer inspect first?

**Answer:** Inspect that named input set, the expected behavior, and the assertion failure details. **Explanation:** The case name narrows the defect without changing all the provider data.

### 35. A project uses test dependencies to pass a mutable cart from one test to the next. What is a simpler alternative?

**Answer:** Create a new cart in each test or use `setUp()` for a fresh cart. **Explanation:** Independent tests are easier to run in isolation and do not skip because a producer test failed.

### 36. A provider returns no cases because an environment-specific fixture is absent. What should the team decide?

**Answer:** Decide whether no cases is a valid skip condition or a broken test setup; make that behavior explicit in PHPUnit configuration or test design. **Explanation:** Empty providers normally indicate an error, while deliberate environment skips should be reported as skips rather than silent success.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [117 - Testing PHP applications with PHPUnit](../lessons/117-testing-php-applications-with-phpunit.md) | [Lesson](../lessons/118-unit-tests-fixtures-and-data-providers.md) | [Example](../examples/118-unit-tests-fixtures-and-data-providers.md) | [119 - Integration tests for databases and HTTP](../lessons/119-integration-tests-for-databases-and-http.md) |
