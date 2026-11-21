# Abstract classes and final declarations

Abstract classes let related classes share state and implemented behavior while requiring child classes to provide selected details. Final declarations limit which parts of a class hierarchy may be extended or overridden.

Use an abstract class when related subclasses share a meaningful base implementation. Use final when a class or member must not be specialized further.

## What you will learn

- understand why an abstract class cannot be instantiated
- declare and implement abstract methods
- combine required methods with shared implementation
- use final classes, methods, constants, and properties
- choose sensible extension points
- recognize PHP version requirements for newer features

## Abstract classes

An abstract class describes a base that is intentionally incomplete. It can hold properties, a constructor, and ordinary methods, but it cannot be instantiated directly.

~~~php
abstract class Message
{
    public function __construct(
        protected string $recipient
    ) {
    }

    abstract public function deliver(string $body): bool;

    public function recipient(): string
    {
        return $this->recipient;
    }
}
~~~

The abstract method declares a required operation without a method body. A concrete child must implement it:

~~~php
class EmailMessage extends Message
{
    public function deliver(string $body): bool
    {
        echo 'Email to ' . $this->recipient . ': ' . $body . PHP_EOL;
        return true;
    }
}

$email = new EmailMessage('reader@example.test');
$email->deliver('Your report is ready.');
~~~

The parent provides shared recipient storage and a getter. The child supplies delivery behavior. Creating an instance of Message directly is invalid because its required behavior is incomplete.

## Abstract methods

An abstract method has a signature but no implementation:

~~~php
abstract protected function formatBody(string $value): string;
~~~

Its declaration specifies the name, parameters, return type, and visibility that child classes must satisfy. The class containing an abstract method must itself be declared abstract.

~~~php
abstract class Formatter
{
    abstract protected function formatBody(string $value): string;

    public function format(string $value): string
    {
        return $this->formatBody(trim($value));
    }
}

class UppercaseFormatter extends Formatter
{
    protected function formatBody(string $value): string
    {
        return strtoupper($value);
    }
}

echo (new UppercaseFormatter())->format(' hello ');
// HELLO
~~~

A concrete child must implement every inherited abstract method before it can be instantiated.

### Visibility and compatible signatures

Abstract methods can be public or protected, not private: a child must be able to implement them. A child may widen a protected method to public, but cannot make a public method protected.

The implementation must be compatible with the declared signature. Keep parameter and return types aligned with the promise made by the parent. The inheritance rules from the previous chapter still apply.

## Shared implementation and template methods

An abstract base can provide a high-level workflow while requiring subclasses to implement particular steps. This is often called a **template method**:

~~~php
abstract class DataImporter
{
    final public function import(string $source): array
    {
        $records = $this->readRecords($source);

        return $this->normalizeRecords($records);
    }

    abstract protected function readRecords(string $source): array;

    abstract protected function normalizeRecords(array $records): array;
}

class DemoImporter extends DataImporter
{
    protected function readRecords(string $source): array
    {
        return [
            ['name' => '  Asha  '],
            ['name' => '  Leo  '],
        ];
    }

    protected function normalizeRecords(array $records): array
    {
        foreach ($records as &$record) {
            $record['name'] = trim($record['name']);
        }
        unset($record);

        return $records;
    }
}

$importer = new DemoImporter();
$records = $importer->import('memory://people');

echo $records[0]['name'];
// Asha
~~~

The parent fixes the order of steps in <code>import()</code>. Each concrete importer supplies the reading and normalization details. The method is final so a child cannot change the workflow order.

Use this pattern only when subclasses genuinely share the same workflow. If implementations need substantially different processes, composition may fit better.

## Abstract properties in PHP 8.4+

PHP 8.4 introduced abstract properties using property hooks. They can require accessor behavior from a child:

~~~php
abstract class ReadableRecord
{
    abstract public string $label {
        get;
    }
}

class NamedRecord extends ReadableRecord
{
    public string $label = 'Customer';
}
~~~

This feature requires PHP 8.4 or later. On earlier versions, use an abstract getter method when a base class needs to require readable data:

~~~php
abstract class ReadableRecord
{
    abstract public function label(): string;
}
~~~

## Final classes

A final class cannot be extended:

~~~php
final class ConfigurationLoader
{
    public function load(): array
    {
        return [];
    }
}
~~~

Attempting to extend <code>ConfigurationLoader</code> causes an error. A final class can still be instantiated and used normally.

Final classes are useful when subclassing would violate invariants or when the class is deliberately not an extension point. Do not mark every class final automatically; decide whether extension is part of its intended design.

A class cannot be both abstract and final: abstract requires subclasses to complete it, while final prohibits subclasses.

## Final methods

A final method cannot be overridden by a child class:

~~~php
abstract class Importer
{
    final public function import(string $source): array
    {
        return $this->read($source);
    }

    abstract protected function read(string $source): array;
}

class ArrayImporter extends Importer
{
    protected function read(string $source): array
    {
        return ['source' => $source];
    }
}
~~~

The child must implement <code>read()</code>, but it cannot replace <code>import()</code>. Use a final method to protect behavior that subclasses rely on or must not bypass. Keep intentionally variable behavior in separate overridable or abstract methods.

A private method is not inherited as an extension point. In PHP 8 and later, private methods may not be declared final except for a constructor.

## Final class constants

Since PHP 8.1, a class constant can be declared final so a child cannot redefine it:

~~~php
class ApiDefaults
{
    final public const VERSION = 'v1';
}
~~~

Before PHP 8.1, class constants could not be declared final.

## Final properties in PHP 8.4+

PHP 8.4 introduced final properties. A child class cannot redeclare one:

~~~php
class BaseRecord
{
    final protected string $table = 'records';
}
~~~

Do not use final properties if the project supports PHP versions before 8.4. Final restricts overriding; it does not make an object's value immutable. Use a readonly property when the goal is to prevent reassignment. Readonly and final solve different problems.

## Abstract and final together

Abstract and final describe opposite constraints:

- **abstract** requires a subclass to provide missing behavior
- **final** prevents a subclass from extending or replacing behavior

A method cannot be both abstract and final. One abstract class can, however, contain abstract methods as extension points and final methods for fixed behavior, as the importer example shows.

## Abstract classes and interfaces

An abstract class can provide shared implementation and state while requiring subclasses to complete selected operations. An interface declares a contract a class can implement without serving as its parent class for shared implementation.

PHP permits a class to extend only one class but implement multiple interfaces. Choose based on the relationship and reuse needed. The next chapter covers [Interfaces and contracts](070-interfaces-and-contracts.md).

## Common mistakes

- **Instantiating an abstract class:** instantiate a concrete child that implements all required methods.
- **Leaving an abstract method unimplemented:** the child must implement it before it can be instantiated.
- **Giving an abstract method a body:** abstract declarations specify a contract, not an implementation.
- **Making an abstract method private:** the child must be able to implement it.
- **Marking a class final without a reason:** final removes subclassing as an extension option.
- **Marking every method final:** protect only behavior subclasses must not replace.
- **Declaring both abstract and final:** the constraints conflict.
- **Using newer features on older PHP versions:** final constants require PHP 8.1; abstract and final properties require PHP 8.4.
- **Using a base class only to share unrelated code:** composition or a focused utility may better represent the design.

## Key takeaways

- Abstract classes cannot be instantiated.
- They can combine shared implementation with required abstract methods.
- Concrete subclasses must implement inherited abstract methods with compatible signatures.
- Abstract methods cannot be private.
- A final class cannot be extended; a final method cannot be overridden.
- Final class constants require PHP 8.1 or later.
- Abstract and final properties require PHP 8.4 or later.
- Use abstract and final deliberately to make extension points clear.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [068 - Composition and object collaboration](068-composition-and-object-collaboration.md) | [Q&A](../qnas/069-abstract-classes-and-final-declarations.md) | [Example](../examples/069-abstract-classes-and-final-declarations.md) | [070 - Interfaces and contracts](070-interfaces-and-contracts.md) |
