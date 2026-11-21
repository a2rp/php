# Integration tests for databases and HTTP - Q&A

### 1. What does a database integration test verify?

**Answer:** It verifies that application code works with a real database driver and schema. **Explanation:** It can catch SQL and mapping problems that a fake repository cannot detect.

### 2. How is that different from a unit test of a service?

**Answer:** The unit test replaces dependencies and checks one unit of behavior; the integration test connects real components across a boundary. **Explanation:** Both are useful, but they answer different questions.

### 3. Why must tests never use the production database?

**Answer:** A test may insert, change, or delete data while setting up or cleaning up. **Explanation:** Use a dedicated disposable database with test-only credentials so test actions cannot harm real records.

### 4. Is SQLite in memory a real database for testing?

**Answer:** Yes. It uses a real SQLite engine and PDO driver, but it does not prove compatibility with MySQL or PostgreSQL. **Explanation:** Engine-specific syntax, types, constraints, and locking can differ.

### 5. Why might two components see different rows in an in-memory SQLite test?

**Answer:** They may have opened different PDO connections. **Explanation:** Each in-memory connection has its own database; share the same connection when components must share test data.

### 6. How can a test verify a repository actually inserted a row?

**Answer:** Call the repository, then read the row back through the repository or query the test database. **Explanation:** This checks both the write and read paths against the real schema.

### 7. What does PDO exception mode change in a test?

**Answer:** Database errors throw exceptions instead of being easy to overlook as return values. **Explanation:** Tests fail closer to the invalid query and provide clearer diagnostics.

### 8. Do prepared statements allow table names as parameters?

**Answer:** Usually no; parameter markers represent data values, not SQL identifiers. **Explanation:** If an identifier must vary, select it from an explicit allowlist.

### 9. Why can a test expecting an empty title to fail if a column is NOT NULL?

**Answer:** An empty string is a value, not NULL. **Explanation:** Add a suitable validation rule or schema CHECK constraint, then assert the behavior owned by that layer.

### 10. What is the benefit of creating a fresh schema for each test?

**Answer:** Each test begins with known database state. **Explanation:** This reduces order dependencies and makes failures easier to reproduce.
### 11. When is transaction rollback a useful isolation strategy?

**Answer:** When test writes use the same connection and the database supports the needed transactional behavior. **Explanation:** The test can discard its changes after each case.

### 12. Why may rollback fail to clean every side effect?

**Answer:** Work on another connection or process is outside that transaction, and some engines implicitly commit certain schema changes. **Explanation:** Choose isolation based on actual engine behavior.

### 13. Should tests rely on fixed auto-increment IDs?

**Answer:** No. Assert the ID returned by the application or find rows by unique test-owned data. **Explanation:** Other setup or engine behavior can change the sequence.

### 14. Why use the application's migrations in integration setup?

**Answer:** Migrations create the schema the application is expected to use. **Explanation:** Handwritten test-only schemas can drift and hide missing or incorrect migrations.

### 15. What should a test do if its PDO driver is unavailable?

**Answer:** Report a visible skip only when that environment is allowed to omit the test. **Explanation:** Required database coverage should be provisioned in CI rather than silently skipped everywhere.

### 16. How can SQLite tests miss a production database defect?

**Answer:** SQLite may accept different SQL or enforce different types, constraints, or transaction rules. **Explanation:** Add focused tests against the production engine for behaviors that depend on it.

### 17. Should a repository integration test call a third-party API?

**Answer:** Usually no. Keep the test focused on the database boundary and control unrelated external services. **Explanation:** External network dependency makes failures slow and unpredictable.

### 18. What should happen if a test changes a committed database record?

**Answer:** Use a disposable test database and restore or discard the test-owned state. **Explanation:** Never rely on cleanup against shared or production data.

### 19. Why should tests run independently and in any order?

**Answer:** Independent tests are easier to reproduce, parallelize, and debug. **Explanation:** A test that depends on a previous test can pass or fail based on execution order.

### 20. Which database behavior belongs in an integration test rather than a service unit test?

**Answer:** SQL execution, constraints, generated IDs, persistence, and row mapping. **Explanation:** These behaviors require the real database boundary.
### 21. What is an HTTP integration test meant to exercise?

**Answer:** A request through the application's HTTP handling pipeline, such as routing, middleware, validation, and response conversion. **Explanation:** Calling a controller method directly skips those connections.

### 22. When is an in-process framework test client enough?

**Answer:** When the goal is to verify application request and response behavior without depending on a real server process. **Explanation:** It is usually faster and avoids port management.

### 23. When should a test start a separate local HTTP server?

**Answer:** When behavior depends on the web SAPI, actual server configuration, process boundary, headers, or network client. **Explanation:** Use a controlled test environment and clean up the process.

### 24. Why should a local PHP development server bind to loopback for tests?

**Answer:** It keeps the test service local to the machine. **Explanation:** The built-in server is for controlled development and testing, not public use.

### 25. Why is a fixed sleep a poor way to wait for a test server?

**Answer:** Startup time varies, so a sleep can be too short or unnecessarily long. **Explanation:** Poll a readiness endpoint until a deadline and report useful server logs if it fails.

### 26. What should an HTTP request test usually assert?

**Answer:** The relevant status, headers, body, redirects, cookies, and persistent effects. **Explanation:** These are observable outcomes a client depends on.

### 27. Should an HTTP test assert which private helper a controller called?

**Answer:** Usually not. **Explanation:** Verify the response and state so internal refactors do not break a test without changing behavior.

### 28. A request test gets a 404, but direct controller tests pass. What should be checked?

**Answer:** Route registration, request method, base path, test environment, and middleware. **Explanation:** The request pipeline test can detect a missing route that direct calls bypass.

### 29. How can a test verify that an HTTP create request persisted the resource?

**Answer:** Check the response and query the dedicated test database for the created row. **Explanation:** This covers both the HTTP response and its database effect.

### 30. How should authentication or authorization be tested over HTTP?

**Answer:** Send requests for allowed and denied cases through the request pipeline and assert the resulting status and response. **Explanation:** This verifies middleware and endpoint behavior together.
### 31. What should CI do when a database service is required?

**Answer:** Provision the test database, apply the schema, and provide test-only credentials. **Explanation:** Required integration coverage should run predictably instead of being skipped in CI.

### 32. The database test passes locally but fails in CI. What should be compared first?

**Answer:** PHP version, loaded extensions, PDO drivers, environment variables, schema, permissions, and service availability. **Explanation:** Environment differences often explain the mismatch.

### 33. How should test data be cleaned if application code commits its own transaction?

**Answer:** Use a disposable database or remove only uniquely identifiable rows owned by that test. **Explanation:** A rollback cannot undo a transaction that the application already committed.

### 34. Why use finite timeouts for HTTP integration tests?

**Answer:** A stalled request should fail within a predictable period. **Explanation:** Timeouts prevent a broken server or dependency from hanging the full test run.
### 35. Is a suite that skips every database test still providing integration coverage?

**Answer:** No. **Explanation:** Make the required service available in CI and keep any accepted skips visible.

### 36. What belongs in a focused production-engine compatibility suite?

**Answer:** Queries, migrations, constraints, or transaction behavior that depend on the production database engine. **Explanation:** Keep the broad quick suite fast, then verify engine-specific behavior directly.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [118 - Unit tests, fixtures, and data providers](../lessons/118-unit-tests-fixtures-and-data-providers.md) | [Lesson](../lessons/119-integration-tests-for-databases-and-http.md) | [Example](../examples/119-integration-tests-for-databases-and-http.md) | [120 - Test doubles, stubs, and mocks](../lessons/120-test-doubles-stubs-and-mocks.md) |
