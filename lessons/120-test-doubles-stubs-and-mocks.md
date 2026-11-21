# Test doubles, stubs, and mocks

A unit often depends on other objects: a database repository, mail transport, clock, payment gateway, or HTTP client. Calling every real dependency in every unit test can make tests slow, unpredictable, or unsafe. A test double replaces a collaborator so a test can control the inputs to the unit or observe an important interaction.

PHPUnit can generate stubs and mock objects from interfaces and extendable classes. Use them deliberately: the goal is to test your code, not to recreate the dependency or mirror every private call.

## Learning goals

After this lesson, you should be able to:

- explain the difference between a stub and a mock;
- choose a test double based on the behavior under test;
- configure return values and exceptions in PHPUnit;
- verify important interactions with explicit expectations;
- recognize fake objects, spies, and dummies;
- reduce brittle tests by doubling stable interfaces.

## What is a test double?

Test double is a general term for a replacement used in a test. The names below describe different roles:

| Double | Purpose | Example |
| --- | --- | --- |
| Dummy | Fills a required parameter but is not used | An unused context object |
| Stub | Supplies controlled indirect input | A repository that returns a chosen record |
| Mock | Verifies expected calls and arguments | A mailer expected to send one message |
| Fake | A working but simplified implementation | An in-memory repository |
| Spy | Records calls so a test can inspect them later | A sender that stores messages in an array |

A test double does not need to reproduce the real dependency's full behavior. It should satisfy the contract that the system under test needs. PHPUnit focuses on creating stubs and mocks; a fake or spy is often a small handwritten class.

## Stub: control what a dependency returns

Use a stub when the test needs to control an indirect input and then assert the system under test's result. The call history of the dependency is not important.

For example, a recommendation service can use a catalog interface. The test controls the returned products instead of connecting to a real database:

~~~php
interface ProductCatalog
{
    public function featuredProducts(): array;
}

final class RecommendationService
{
    public function __construct(private ProductCatalog $catalog)
    {
    }

    public function titles(): array
    {
        return array_column($this->catalog->featuredProducts(), 'title');
    }
}
~~~

A PHPUnit test can configure the stub with the value needed for the scenario:

~~~php
$catalog = $this->createStub(ProductCatalog::class);
$catalog
    ->method('featuredProducts')
    ->willReturn([
        ['title' => 'Notebook'],
        ['title' => 'Pen'],
    ]);

$service = new RecommendationService($catalog);

self::assertSame(['Notebook', 'Pen'], $service->titles());
~~~

The assertion checks the service's result. It does not assert how many times the catalog method was called because that interaction is not part of this test's purpose.
## Configure stub behavior

A stub method can return a fixed value:

~~~php
$gateway
    ->method('balance')
    ->willReturn(1250);
~~~

It can also throw an exception so the test can exercise an error path:

~~~php
$gateway
    ->method('balance')
    ->willThrowException(new RuntimeException('Service unavailable'));
~~~

A callback can calculate a result from the arguments when a fixed return value is not enough:

~~~php
$rates
    ->method('forCurrency')
    ->willReturnCallback(
        static fn (string $currency): float => match ($currency) {
            'EUR' => 1.08,
            'GBP' => 1.27,
            default => throw new InvalidArgumentException('Unknown currency'),
        },
    );
~~~

Keep callback behavior small. If the callback starts implementing business rules, test those rules directly and make the double simpler.

PHPUnit can also return a sequence of values or map arguments to results. These options help with a small, focused scenario. When a test needs complex, stateful behavior, a small handwritten fake may be clearer than a long chain of mock configuration.

## Mock: verify an important interaction

Use a mock when communication with a collaborator is itself part of the behavior. A notification service should send a message to the correct recipient when an account is created. The test can assert that the sender is called once with the expected arguments.

~~~php
interface MessageSender
{
    public function send(string $recipient, string $subject, string $body): void;
}

final class WelcomeService
{
    public function __construct(private MessageSender $sender)
    {
    }

    public function welcome(string $email, string $name): void
    {
        $this->sender->send(
            $email,
            'Welcome',
            'Hello ' . $name,
        );
    }
}
~~~

~~~php
$sender = $this->createMock(MessageSender::class);

$sender
    ->expects($this->once())
    ->method('send')
    ->with(
        'ada@example.test',
        'Welcome',
        'Hello Ada',
    );

$service = new WelcomeService($sender);
$service->welcome('ada@example.test', 'Ada');
~~~

The test passes if the expected call occurs once with those arguments. It fails if the call is missing, repeated, or uses different values. The example verifies a meaningful external effect, not a private helper call.

## Choose call-count expectations carefully

Use an exact expectation when the number of calls matters:

- **once()** means exactly one call.
- **exactly(2)** means exactly two calls.
- **never()** means the method must not be called.
- Broader expectations such as **atLeastOnce()** and **atMost()** can hide unclear intent. Prefer exact counts when the count is meaningful.

If the call count does not matter, use a stub and assert the result instead of creating a mock with a permissive expectation. A mock without a meaningful expectation is usually the wrong kind of double. Current PHPUnit versions also report notices for mock objects that have no expectations configured.

A test should not fail because an implementation performed an irrelevant extra read or called a helper in a different order. Assert a call only when it represents an externally important effect or contract.
## Fakes, spies, and handwritten doubles

A fake has a working simplified implementation. For example, an in-memory repository can implement the same interface as a database repository and store records in an array. It is useful when several tests need realistic behavior without a database, but it must remain simple and should not become a second implementation of the application.

A spy records calls and lets a test inspect them afterward. PHPUnit mocks usually make the expectation before the action; a handwritten spy is useful when a test needs to inspect a sequence of calls without setting a strict expectation up front.

A dummy only satisfies a required parameter that the code under test does not use. Do not add behavior to a dummy unless a test needs it.

## Design code so it can be tested

Depend on a small interface at a boundary, then inject the implementation. The unit test can pass a stub or mock while production code passes the real adapter.

~~~php
interface Clock
{
    public function now(): DateTimeImmutable;
}

final class ExpirationService
{
    public function __construct(private Clock $clock)
    {
    }

    public function isExpired(DateTimeImmutable $expiresAt): bool
    {
        return $this->clock->now() >= $expiresAt;
    }
}
~~~

A test can stub the clock to return a fixed time. This avoids waiting for real time to pass and makes the boundary case repeatable. The same design works for random number generators, queues, file storage, and external APIs.

## Know PHPUnit's limits

PHPUnit generates doubles for interfaces and extendable classes. A final class cannot be doubled. Final, private, and static methods cannot be replaced as ordinary doubled methods; private and final methods keep their real behavior, and static calls are not a useful injectable boundary. Enums are final and cannot be doubled.

Prefer doubling an interface instead of removing final declarations from production classes just to make PHPUnit generate a double. If a concrete final class hides an external side effect, introduce a narrow interface around the side effect.

Some PHPUnit features vary by version. Use APIs supported by the version installed in the project. Prefer explicit expectation matchers such as once(), exactly(), and never(); the broad any() expectation is deprecated in recent PHPUnit versions. A configured stub is clearer when invocation count does not matter.
## Common over-mocking problems

### The test configures every method on a dependency

Configure only the behavior needed for the scenario. A double with a long setup script is hard to understand and may indicate that the unit has too many responsibilities.

### The test asserts private call order

Verify order only when it is part of the collaborator contract, such as begin, write, commit. Do not tie ordinary business tests to incidental helper calls.

### The test mocks simple data objects

Use real value objects and immutable data in most tests. They are deterministic and usually cheaper to construct than a double.

### A database integration test replaces PDO with a mock

That test cannot prove SQL works. Use a real disposable database for repository integration tests; use a stubbed repository when testing a service's decisions.

### A final class cannot be doubled

Do not weaken a useful production design only for a test. Add an interface at the external boundary or test the real class when it is cheap and deterministic.
## A quick decision guide

| Test need | Prefer |
| --- | --- |
| Control a dependency's returned value | Stub |
| Simulate an exception from a dependency | Stub |
| Verify a meaningful call, arguments, or exact count | Mock |
| Reuse lightweight working behavior across tests | Fake |
| Inspect calls after execution | Spy |
| Satisfy an unused parameter | Dummy |
| Verify SQL or HTTP integration | A real test service |

## Key takeaways

- Stubs control inputs; mocks verify important interactions.
- Use a stub when the result matters and the dependency call itself does not.
- Use a mock for meaningful side effects, arguments, and exact call counts.
- Prefer interfaces and dependency injection at external boundaries.
- Keep doubles small and use real value objects where practical.
- PHPUnit cannot ordinarily double final classes or final, private, and static methods.
- Avoid permissive expectations and tests tied to internal details.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [119 - Integration tests for databases and HTTP](119-integration-tests-for-databases-and-http.md) | [Q&A](../qnas/120-test-doubles-stubs-and-mocks.md) | [Example](../examples/120-test-doubles-stubs-and-mocks.md) | [121 - Static analysis with PHPStan and Psalm](121-static-analysis-with-phpstan-and-psalm.md) |
