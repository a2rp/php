# Integration tests for databases and HTTP

Unit tests check a small piece of code in isolation. Integration tests check whether real parts of an application work together across a boundary, such as a repository talking to a database or an HTTP request passing through routing and middleware.

They catch defects that mocks cannot reveal: invalid SQL, a schema that does not match the application, an unregistered route, or a response header that is missing.

## Learning goals

- Distinguish unit, integration, and end-to-end tests.
- Test a PDO repository with a real database driver.
- Isolate test data and keep tests away from production services.
- Exercise HTTP behavior through the application request pipeline.
- Diagnose integration failures caused by drivers or configuration.

## Test layers

| Test type | What is real | Example question |
| --- | --- | --- |
| Unit | One class or function | Does a service reject an empty title? |
| Integration | Several parts at a boundary | Does a repository insert and retrieve a row through PDO? |
| End-to-end | A running application and client | Can a user submit a form and see the saved result? |

A service tested with a fake repository does not prove the real database accepts the SQL. A framework HTTP test that runs the request pipeline checks more than calling a controller method directly, even when it does not open a TCP connection. State which real boundary each test covers.

## Use a dedicated test environment

Never point tests at a production database. Give tests their own database name, credentials, configuration, and schema. Keep credentials in environment variables or a local ignored file; never commit live credentials.

A safe test environment is disposable, uses a known schema, starts with predictable data, and only cleans up resources owned by the test run. CI should receive test-only credentials from its secret store.

An in-memory SQLite database is convenient and uses a real SQL engine and PDO driver, but it does not behave exactly like MySQL or PostgreSQL. Syntax, column types, constraints, locking, collation, and transaction behavior can differ. Keep a focused suite against the production database engine when those differences matter.

An in-memory SQLite database belongs to one connection. A second connection creates a different database. Pass the same PDO connection to components that need to see the same test rows.

## Test a repository with PDO

This PHPUnit test creates a fresh SQLite database and schema for each test. It exercises a real PDO driver and SQL.

~~~php
<?php

declare(strict_types=1);

use PHPUnit\Framework\TestCase;

final class NoteRepositoryIntegrationTest extends TestCase
{
    private PDO $database;
    private NoteRepository $notes;

    protected function setUp(): void
    {
        if (!in_array('sqlite', PDO::getAvailableDrivers(), true)) {
            self::markTestSkipped('The PDO SQLite driver is not installed.');
        }

        $this->database = new PDO('sqlite::memory:', options: [
            PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
            PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC,
        ]);

        $this->database->exec(
            'CREATE TABLE notes (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                title TEXT NOT NULL CHECK (length(trim(title)) > 0)
            )',
        );

        $this->notes = new NoteRepository($this->database);
    }

    public function testItSavesAndFindsANote(): void
    {
        $id = $this->notes->create('Integration test');

        self::assertGreaterThan(0, $id);
        self::assertSame(
            ['id' => $id, 'title' => 'Integration test'],
            $this->notes->find($id),
        );
    }

    public function testItReturnsNullWhenANoteDoesNotExist(): void
    {
        self::assertNull($this->notes->find(999));
    }

    public function testTheDatabaseRejectsAnEmptyTitle(): void
    {
        $this->expectException(PDOException::class);

        $this->notes->create('');
    }
}
~~~

The last test expects the database or repository to reject an empty title. Decide which layer owns that rule. A schema CHECK constraint can be tested here; validation that belongs to a service should be tested at that layer.

A minimal repository is:

~~~php
<?php

declare(strict_types=1);

final class NoteRepository
{
    public function __construct(private PDO $database)
    {
    }

    public function create(string $title): int
    {
        $statement = $this->database->prepare(
            'INSERT INTO notes (title) VALUES (:title)',
        );
        $statement->execute(['title' => $title]);

        return (int) $this->database->lastInsertId();
    }

    public function find(int $id): ?array
    {
        $statement = $this->database->prepare(
            'SELECT id, title FROM notes WHERE id = :id',
        );
        $statement->execute(['id' => $id]);
        $note = $statement->fetch();

        return $note === false ? null : $note;
    }
}
~~~

Prepared statements bind data values separately from SQL text. Table and column names generally cannot be bound as ordinary values; choose variable identifiers from a fixed allowlist.

## Keep database tests isolated

Choose a strategy that matches the database engine and test behavior.

- A fresh in-memory database or temporary schema gives each test a clean start.
- A transaction that is rolled back is fast when all writes use the same connection and the engine supports the needed behavior.
- Unique test-owned rows with explicit cleanup help when application behavior commits its own transaction.

A transaction on one connection does not isolate writes from another connection, process, or asynchronous worker. Some engines implicitly commit schema changes. Do not assume rollback removes every test effect.

Avoid test-order dependencies and fixed auto-increment values. Assert the identifier returned by the repository or query using a unique value created in the test. Tests should run alone and in any order.
## Test HTTP behavior through the request pipeline

A controller unit test may call a method directly and skip route registration, middleware, request parsing, response conversion, and exception handling.

For HTTP integration, use the application test client or framework kernel to send a request through the real request pipeline. Verify observable behavior: status, content type, response body, redirects, cookies, validation errors, authorization, and persistent state.

A framework-specific test might look like this:

~~~php
public function testCreatingANoteReturnsAResourceResponse(): void
{
    $response = $this->postJson('/api/notes', [
        'title' => 'Integration test',
    ]);

    $response
        ->assertStatus(201)
        ->assertJsonPath('data.title', 'Integration test');

    self::assertDatabaseHas('notes', [
        'title' => 'Integration test',
    ]);
}
~~~

The methods depend on the framework. Send a request through its HTTP handling layer and check the response and persistent effect. The database must be test-only and safe to reset. Avoid assertions about private implementation details; an internal refactor should not break a test when the external behavior remains correct.

## When to start a local HTTP server

An in-process framework client is fast and avoids port management. A separate local server is useful when behavior depends on the server process, web SAPI, server configuration, headers, or a real network client.

PHP's built-in development server can support controlled local testing. Bind it to loopback, use a test-only document root and configuration, wait until it is ready before sending requests, and stop it during cleanup. Choose a port available in the test environment. Parallel workers should not all assume a fixed port is free.

The built-in server is for development and controlled testing, not production or public networks. For production-like behavior, use the deployment server and runtime configuration in an isolated test environment.

## Make tests repeatable

Integration tests require more setup than unit tests. Keep them reliable:

- Use predictable fixtures and do not rely on leftover data.
- Create the schema with the application's migrations.
- Keep third-party network calls out of the suite; replace those boundaries with controlled fakes or local services.
- Test against the production database engine when SQLite is not representative.
- Set finite network timeouts and report useful diagnostics.
- Keep test configuration separate from development and production.
- Run cleanup even when an assertion fails.
- Skip unavailable services only when that is an accepted policy, and make skips visible.
- Provision required services in CI so integration coverage does not silently disappear.
- Reserve integration tests for behavior that crosses a real boundary.

A test skipped on every developer machine and every CI run is not providing coverage. Document required services and provision them in CI.
## Common problems

### It passes locally but fails in CI

Check PHP extensions, PDO drivers, environment variables, schema setup, ports, and filesystem permissions. Do not silently switch to another database to hide the failure.

### Rows leak between tests

Check whether tests share a database, whether application code opens another connection, and whether a transaction was committed. Prefer fresh schemas, transactions on the same connection, or unique test-owned data.

### SQLite passes but the production database fails

The engines may accept different SQL or enforce different types and constraints. Keep fast SQLite tests, then add focused checks against the production engine for engine-specific queries and migrations.

### An HTTP test unexpectedly returns 404

Check route registration, method, base path, environment, and middleware. A direct controller test would not detect a missing route.

### A local server connection fails intermittently

Check process startup, readiness, port collisions, and cleanup. Fixed sleeps are unreliable because startup time varies. Poll for readiness until a deadline and include the server log in failure diagnostics.

## Key takeaways

- Integration tests verify real boundaries such as PDO-to-database behavior and request-to-response behavior.
- Use a dedicated, disposable test database.
- Isolation must account for multiple connections, processes, and database-specific transaction rules.
- SQLite is useful, but it does not replace testing a different production database engine.
- Use a framework request client for most HTTP pipeline tests; start a local server when server-process behavior matters.
- Assert observable responses and persisted state.
- Keep setup, cleanup, required services, and skipped tests visible.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [118 - Unit tests, fixtures, and data providers](118-unit-tests-fixtures-and-data-providers.md) | [Q&A](../qnas/119-integration-tests-for-databases-and-http.md) | [Example](../examples/119-integration-tests-for-databases-and-http.md) | [120 - Test doubles, stubs, and mocks](120-test-doubles-stubs-and-mocks.md) |
