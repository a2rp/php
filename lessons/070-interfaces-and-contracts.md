# Interfaces and contracts

A PHP interface describes public behavior that a class promises to provide. The interface names the operations; the implementing class supplies their behavior.

Code that depends on an interface can work with different implementations without needing to know each implementation's internal details. This makes interfaces useful for interchangeable services, clear method parameters, and testing.

## What you will learn

- declare an interface and implement it in a class
- use an interface as a parameter, property, or return type
- provide multiple implementations of one contract
- implement several interfaces or extend an interface
- compare interfaces with abstract classes and traits
- account for interface constants and properties by PHP version

## Declare an interface

Use the <code>interface</code> keyword to declare a contract:

~~~php
interface NotificationSender
{
    public function send(string $recipient, string $message): bool;
}
~~~

An interface method declares the public operation and its signature. It does not contain the implementation.

A class uses <code>implements</code> to promise that it follows the contract:

~~~php
class EmailSender implements NotificationSender
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Email to ' . $recipient . ': ' . $message . PHP_EOL;
        return true;
    }
}
~~~

Every method required by the interface must be implemented with a compatible signature. If a class does not implement all required methods, PHP will reject it as a concrete class. A class that is intentionally incomplete must be declared abstract.

Interface methods are public. Implementations must also be public; they cannot reduce the visibility.

## Use an interface as a type

Use the interface type wherever the caller needs the promised behavior:

~~~php
function alertUser(
    NotificationSender $sender,
    string $recipient,
    string $message
): bool {
    return $sender->send($recipient, $message);
}

$sender = new EmailSender();

alertUser($sender, 'reader@example.test', 'Your report is ready.');
~~~

The function depends on the contract, not specifically on <code>EmailSender</code>. A second class can implement the same interface:

~~~php
class SmsSender implements NotificationSender
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Text to ' . $recipient . ': ' . $message . PHP_EOL;
        return true;
    }
}

alertUser(new SmsSender(), '+15550102030', 'Your report is ready.');
~~~

Both implementations can be supplied to <code>alertUser()</code> because both promise the <code>NotificationSender</code> behavior.

## Constructor injection with an interface

An object can receive an interface-typed dependency through its constructor:

~~~php
final class AlertService
{
    public function __construct(
        private NotificationSender $sender
    ) {
    }

    public function notify(string $recipient, string $message): bool
    {
        return $this->sender->send($recipient, $message);
    }
}

$service = new AlertService(new EmailSender());
$service->notify('reader@example.test', 'Your report is ready.');
~~~

<code>AlertService</code> does not need to know how email or text messages are sent. The caller selects the implementation when it creates the service. This applies the composition and dependency-injection ideas from [Chapter 068](068-composition-and-object-collaboration.md).

## Interface contracts and signatures

An implementation must remain compatible with the interface signature:

~~~php
interface PriceProvider
{
    public function priceFor(string $productCode): float;
}

class CatalogPriceProvider implements PriceProvider
{
    public function priceFor(string $productCode): float
    {
        return 19.99;
    }
}
~~~

Do not remove a required parameter, narrow visibility, or choose incompatible parameter and return types. PHP's normal variance rules apply.

Since PHP 8.0, callers can use named arguments. Keep implementation parameter names the same as the interface names so callers can safely use those names:

~~~php
$provider->priceFor(productCode: 'BOOK-1');
~~~

Renaming a parameter does not necessarily violate PHP's signature compatibility rules, but it can break callers that use named arguments. Treat parameter names as part of the practical public API.

## Implement multiple interfaces

A class can implement more than one interface:

~~~php
interface NotificationSender
{
    public function send(string $recipient, string $message): bool;
}

interface HealthCheck
{
    public function isAvailable(): bool;
}

class EmailSender implements NotificationSender, HealthCheck
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Email to ' . $recipient . ': ' . $message . PHP_EOL;
        return true;
    }

    public function isAvailable(): bool
    {
        return true;
    }
}
~~~

Each interface represents a capability. Keep contracts focused so classes do not have to promise unrelated behavior they cannot support.

If multiple interfaces declare a method with the same name, the class must provide one implementation whose signature is compatible with all of them.

## Extend an interface

An interface can extend one or more interfaces. A class implementing the child interface must fulfill the methods inherited from every parent interface, as well as the child interface's own methods:

~~~php
interface Sender
{
    public function send(string $recipient, string $message): bool;
}

interface AuditableSender extends Sender
{
    public function lastSentAt(): ?DateTimeImmutable;
}

class LoggedEmailSender implements AuditableSender
{
    private ?DateTimeImmutable $lastSentAt = null;

    public function send(string $recipient, string $message): bool
    {
        echo 'Email to ' . $recipient . ': ' . $message . PHP_EOL;
        $this->lastSentAt = new DateTimeImmutable();

        return true;
    }

    public function lastSentAt(): ?DateTimeImmutable
    {
        return $this->lastSentAt;
    }
}
~~~

A class can extend only one class, but it can implement multiple interfaces. An interface can extend multiple interfaces.

## Interface constants

Interfaces can declare constants:

~~~php
interface ResponseCode
{
    public const SUCCESS = 200;
}
~~~

The constant can be read through the interface name:

~~~php
echo ResponseCode::SUCCESS;
~~~

Interface constants are not a place for instance state or configurable values. Constant inheritance and overriding behavior changed in PHP 8.1; avoid relying on an override when supporting earlier versions.

## Interface properties in PHP 8.4+

PHP 8.4 allows interfaces to require public property access through property hooks:

~~~php
interface HasLabel
{
    public string $label { get; }
}

class ProductLabel implements HasLabel
{
    public string $label = 'Product';
}
~~~

The contract above requires the property to be publicly readable. It does not require it to be publicly writable.

An interface can require public reading, public writing, or both. A class may fulfill the requirement with a regular public property or with suitable property hooks. A settable property requirement cannot be met by a readonly property.

For projects that support PHP versions before 8.4, express property access as methods instead:

~~~php
interface HasLabel
{
    public function label(): string;
}
~~~

## Interfaces and abstract classes

| Feature | Interface | Abstract class |
| --- | --- | --- |
| Main role | Declare a behavior contract | Share implementation and require selected behavior |
| Implementations | Class uses <code>implements</code> | Class uses <code>extends</code> |
| Number a class can use | Multiple interfaces | One parent class |
| Shared instance state | No ordinary instance properties before PHP 8.4 | Can declare properties |
| Implemented methods | No method bodies | Can include concrete methods |
| Property requirements | PHP 8.4+ supports property access requirements | PHP 8.4+ supports abstract properties |

Use an interface when different or unrelated classes should promise the same behavior. Use an abstract class when related classes also need shared state or implementation. They can be used together: an abstract class may implement an interface and provide part of its behavior.

## Interfaces and traits

An interface requires behavior but does not provide a reusable method body. A trait provides methods and properties that classes can reuse, but it is not a type contract. Traits are covered in [Traits and code reuse](071-traits-and-code-reuse.md).

A class may use both: implement an interface to promise behavior and use a trait to share an implementation.

## Constructors in interfaces

PHP allows constructor declarations in interfaces, but doing so is generally discouraged. It forces every implementation to accept the same construction shape even when different implementations need different setup.

If callers need to create interchangeable objects, use a separate factory contract or have the composition root create the objects. Keep the behavior interface focused on what clients need to do with the object.

## Check an interface at runtime

Use <code>instanceof</code> to check whether an object implements a contract:

~~~php
if ($sender instanceof NotificationSender) {
    echo 'This object can send notifications.';
}
~~~

Prefer an interface type declaration when a function requires a capability. Use a runtime check when the code is deliberately handling several kinds of values.

## Common mistakes

- **Using <code>extends</code> on a class to implement an interface:** use <code>implements</code>.
- **Forgetting a required method:** implement every method or make the class abstract.
- **Making an implementation method protected:** interface methods are public.
- **Depending on a concrete class unnecessarily:** type against the behavior the caller needs.
- **Creating one huge interface:** split unrelated capabilities into smaller contracts.
- **Putting implementation code in an interface:** use an abstract class or trait for shared method bodies.
- **Treating an interface as an object:** instantiate a concrete implementation, not the interface.
- **Changing parameter names casually:** named-argument callers can depend on them.
- **Using property contracts on older PHP:** interface properties require PHP 8.4; use methods for broader compatibility.
- **Putting configuration or mutable state in constants:** use an object or configuration value when data must vary.

## Key takeaways

- An interface is a public behavior contract.
- A class implements an interface and must provide compatible public methods.
- Type against an interface to accept any suitable implementation.
- Classes can implement multiple interfaces, and interfaces can extend multiple interfaces.
- Keep interfaces small and focused on client needs.
- PHP 8.4 adds interface property requirements; methods remain the portable alternative.
- Interface constants' inheritance behavior changed in PHP 8.1.
- Use interfaces for contracts, abstract classes for shared base behavior, and traits for code reuse.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [069 - Abstract classes and final declarations](069-abstract-classes-and-final-declarations.md) | [Q&A](../qnas/070-interfaces-and-contracts.md) | [Example](../examples/070-interfaces-and-contracts.md) | [071 - Traits and code reuse](071-traits-and-code-reuse.md) |
