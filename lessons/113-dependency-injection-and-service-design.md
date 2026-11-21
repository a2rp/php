# 113 - Dependency injection and service design

## Goals

After this lesson, you should be able to:

- Explain a dependency and dependency injection in plain terms.
- Use constructor injection for required collaborators.
- Decide when a class should depend on an interface or a concrete class.
- Wire an application's objects explicitly at its composition root.
- Explain what a dependency injection container does and what PSR-11 does not prescribe.
- Design focused services with visible dependencies and deliberate side effects.
- Recognize service-locator, over-abstraction, and oversized-constructor problems.
- Use test doubles to check behavior without replacing every collaborator with a mock.

## What is a dependency?

A dependency is an object or value that a class needs to do its work. An order receipt service might need an order repository to find an order and a sender to deliver the receipt. Those collaborators are its dependencies.

A class is coupled to the way it obtains a dependency as well as to the dependency's behavior. If a class creates its own database connection, SMTP client, clock, and logger, those choices are hidden inside its behavior. Replacing or testing them becomes harder.

## Dependency injection

Dependency injection means supplying a class with the collaborators it needs from outside that class. The class can then focus on its responsibility instead of constructing every service it uses.

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

final class SendOrderReceipt
{
    public function __construct(
        private readonly OrderRepository $orders,
        private readonly ReceiptSender $sender,
    ) {
    }

    public function send(string $orderNumber): void
    {
        // Find the order and ask the sender to deliver its receipt.
    }
}
~~~

The constructor makes two required collaborators visible. The class does not need to know whether the repository uses a database or whether the sender uses an SMTP provider, a queue, or a test double.

### Constructor injection is the usual choice for required dependencies

Pass a required collaborator through the constructor and store it in a property. If a class cannot do its job without a dependency, make that dependency required rather than silently creating a fallback inside the method.

Constructor property promotion, available in PHP 8.0 and newer, is a concise way to declare and assign a constructor dependency. A regular property assignment is also valid and may be clearer for more involved setup.

~~~php
<?php

final class OrderLookup
{
    private OrderRepository $orders;

    public function __construct(OrderRepository $orders)
    {
        $this->orders = $orders;
    }
}
~~~

### Other injection styles

- **Constructor injection:** dependencies required for the object's valid operation. This is usually the clearest default.
- **Method injection:** a one-off collaborator or value needed only for a particular operation. Prefer passing ordinary operation data as method arguments; use a collaborator argument when the behavior itself varies per call.
- **Setter or property injection:** optional or late-configured collaborators. It can leave an object only partly configured, so avoid it for required dependencies.

There is no need to use every style. Consistent constructor injection keeps an object's requirements visible at creation time.

## Depend on a useful contract

An application service can depend on a small interface when that interface represents a meaningful boundary, such as persistence or message delivery.

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

interface ReceiptSender
{
    public function send(string $recipient, string $subject, string $body): void;
}
~~~

An infrastructure adapter can implement the contract:

~~~php
<?php

declare(strict_types=1);

namespace App\Infrastructure\Mail;

use App\Application\Order\ReceiptSender;

final class SmtpReceiptSender implements ReceiptSender
{
    public function __construct(private readonly MailClient $client)
    {
    }

    public function send(string $recipient, string $subject, string $body): void
    {
        $this->client->send($recipient, $subject, $body);
    }
}
~~~

The interface lets application code ask for the behavior it needs without depending on a particular mail implementation. It can also allow a recording sender in a focused test.

An interface is not valuable solely because a class exists. If an implementation is stable, local to one module, and has no meaningful alternate behavior or boundary, depending directly on the concrete class can be simpler. Add abstractions when they protect a real seam, support a real alternative, or clarify ownership.

## Manual wiring and the composition root

Some part of the application must choose the concrete objects and connect them. That assembly boundary is often called the composition root. It is commonly the public entry point, command bootstrap, or framework configuration.

~~~php
<?php

declare(strict_types=1);

use App\Application\Order\SendOrderReceipt;
use App\Infrastructure\Mail\SmtpReceiptSender;
use App\Infrastructure\Persistence\PdoOrderRepository;

$orders = new PdoOrderRepository($pdo);
$sender = new SmtpReceiptSender($mailClient);
$sendOrderReceipt = new SendOrderReceipt($orders, $sender);
~~~

The use case receives the chosen implementations. The bootstrap knows concrete classes; the application service does not need to. For a small project, explicit `new` calls at the composition root are often all the wiring needed.

Using `new` is not automatically a design problem. It is normal to create value objects, domain objects, and short-lived data objects close to where they are used. The concern is hiding the construction of long-lived or replaceable collaborators deep inside a class that should receive them.

## Dependency injection container

As an application grows, manually constructing a long dependency graph in every entry point can become repetitive. A dependency injection container can keep construction recipes in one place, resolve constructor dependencies, and manage service lifetimes according to its configuration.

A container is an assembly tool. It does not make a service's design good by itself, and it does not eliminate the need to understand dependencies. Whether the container builds objects eagerly, lazily, once, or per request depends on the container and its configuration.

PSR-11 defines a common interface for retrieving container entries with `get()` and checking for entries with `has()`. It does not define a universal configuration format, autowiring algorithm, registration API, or service lifetime policy.

### Avoid using the container as a service locator

A service locator is an object passed into application code so that the code can request arbitrary dependencies by string identifier while it runs.

~~~php
<?php

final class SendOrderReceipt
{
    public function __construct(private ContainerInterface $container)
    {
    }

    public function send(string $orderNumber): void
    {
        $orders = $this->container->get('orders');
        $sender = $this->container->get('receipt_sender');
        // Work with dependencies that were hidden from the constructor contract.
    }
}
~~~

This hides the real requirements from the class signature. It also moves missing-service errors from application startup or object creation to the point where a code path tries to retrieve the service. Prefer injecting the specific repository and sender the class needs.

A container may still be used at the composition root to build the graph. Keep container lookups at that boundary instead of spreading container access through domain and application code.

## Design focused services

A service is an object that performs a behavior or coordinates a use case. The term can mean different things in different frameworks, so name the responsibility clearly rather than relying on the word “service” alone.

A focused service usually has:

- A clear purpose described by its name and public methods.
- Required dependencies visible in its constructor.
- Small, understandable inputs and outputs.
- Business rules in the domain or application responsibility that owns them.
- External side effects behind an explicit collaborator.
- No hidden dependency on global state, request globals, or a container.

For example, `SendOrderReceipt` describes an action. A `Manager` class with unrelated methods for mail, billing, caching, and user registration does not communicate one responsibility.

### Separate services from data and domain objects

Not every object is a service. A value object represents a value, and an entity represents an identity and associated state. They can enforce their own rules and can often be created directly. A service coordinates behavior that does not naturally belong to one entity or that needs external collaborators.

Keep the distinction useful, not ceremonial. A small project may have a few straightforward classes rather than a formal class for every noun.

### Keep side effects visible

Database writes, emails, filesystem operations, and network calls are side effects. Put them behind collaborators that can be configured at the edge of the application. This makes it clear when an operation can fail or affect something outside the process.

A use case can coordinate a side effect, but it should not hide provider-specific setup inside a business rule. An adapter can translate a simple application contract into a provider's API.

### Avoid too many dependencies in one service

A constructor with many unrelated collaborators can indicate that a class owns too many responsibilities. Do not fix the count by hiding dependencies in a container or bundling unrelated services into a generic `Helper` object. Group related behavior into cohesive services, and pass only what each service uses.

A larger constructor is not automatically wrong; inspect the responsibilities and dependency relationships before changing it.

### Treat configuration as data

Strings such as sender addresses, timeouts, feature flags, and endpoint names are configuration values, not services. Read configuration at the application boundary and pass the relevant values or a cohesive settings object to the component that needs them.

Avoid having each service read environment variables or parse configuration files on its own. Centralized configuration makes defaults, validation, and deployment differences easier to review.

## Dependency injection and testing

When dependencies are passed in, a focused test can supply a small fake or recording implementation. For example, a recording `ReceiptSender` can save the recipient and message in memory. A test can call the use case and assert what the use case asked the sender to do without contacting a mail provider.

Use the smallest test double that makes the behavior clear:

- A **fake** has a working but simplified implementation, such as an in-memory repository.
- A **stub** returns a prepared value so a code path can be exercised.
- A **spy** records calls for later inspection.
- A **mock** is configured with expected interactions and can fail when those expectations are not met.

A mock is not required for every dependency. Real value objects and simple in-memory collaborators can make tests more realistic and easier to read. Test the behavior that matters, not merely that every method was called in one exact order.

## Object lifetime and shared state

A container may reuse an object or create a new one depending on configuration. A reusable service should generally avoid storing request-specific mutable state in its properties. If a shared service stores one user's request data, that data can leak into another operation in long-running processes or workers.

Pass operation-specific values to methods, and keep state in the appropriate request, domain entity, or persistence layer. Understand the lifetime rules of the framework or container being used.

## Common mistakes

### Constructing infrastructure inside a use case

A use case that creates a PDO connection or SMTP client hides its requirements and locks itself to one implementation. Inject the collaborator and construct its implementation at the composition root.

### Injecting a container everywhere

This makes every class capable of retrieving arbitrary services and hides the actual dependency list. Inject the specific collaborator each class needs.

### Creating an interface for every class

Interfaces add value when they express a useful boundary or substitution. Otherwise, they can create more files and indirection without improving design.

### Using optional dependencies to hide required behavior

A nullable sender with a silent default can make an operation appear successful even though no receipt was sent. Make required behavior required; model genuinely optional behavior explicitly.

### Building a generic service with unrelated responsibilities

A `UtilityService` or `Manager` with many unrelated operations is difficult to understand and change. Split behavior by cohesive responsibility when the code demonstrates that need.

### Using static global access instead of wiring

Global service access hides dependencies and can make tests influence each other. Keep application-wide choices at the composition root and pass collaborators explicitly.

### Confusing dependency injection with a container

Dependency injection is the act of supplying dependencies from outside. A container can automate construction, but manual constructor wiring is also dependency injection.

## A practical design process

1. State what the class is responsible for in one short sentence.
2. List the external collaborators it needs to fulfill that responsibility.
3. Make required collaborators explicit in the constructor.
4. Depend on a contract only where it protects a meaningful boundary.
5. Keep business decisions in the application or domain responsibility that owns them.
6. Put construction and implementation choices in the composition root or container configuration.
7. Pass a small fake or in-memory collaborator when a focused test needs one.
8. Review object lifetime and avoid request-specific mutable state in shared services.
9. Add a container only when manual assembly becomes repetitive enough to justify it.

## Key takeaways

- Dependency injection supplies collaborators from outside a class; a container is optional tooling that can automate object construction.
- Constructor injection is a clear default for required dependencies.
- Use interfaces when they define useful boundaries, not automatically for every class.
- Keep concrete object selection in the composition root or container configuration.
- Avoid service-locator lookups inside application and domain code because they hide dependencies.
- Keep services focused, side effects visible, and configuration separate from behavior.
- `new` is appropriate for values and domain objects; hidden construction of replaceable collaborators is the concern.
- Test doubles are useful when they clarify behavior, but not every dependency needs a mock.
- Understand the lifetime and shared-state behavior of the framework or container in use.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [112 - Routing, controllers, and middleware](112-routing-controllers-and-middleware.md) | [Q&A](../qnas/113-dependency-injection-and-service-design.md) | [Example](../examples/113-dependency-injection-and-service-design.md) | [114 - Application configuration and logging](114-application-configuration-and-logging.md) |
