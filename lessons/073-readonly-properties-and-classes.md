# Readonly properties and classes

A readonly property can be initialized once and then cannot be reassigned. Readonly properties are useful for values that should remain stable after an object is created, such as an identifier, a validated amount, or a timestamp.

A readonly class applies this rule to every declared instance property in the class. Readonly helps make an object’s own property assignments predictable, but it does not automatically make every object stored inside it immutable.

## What you will learn

- declare and initialize a readonly property
- understand the one-time initialization rule
- recognize writes that PHP rejects
- distinguish readonly from deep immutability
- use a readonly class for value objects and data-transfer objects
- account for changes introduced across PHP versions
- choose between readonly state and mutable properties

## Why readonly can help

A regular property can be changed by any code that has permission to write it:

~~~php
class User
{
    public function __construct(public string $email)
    {
    }
}

$user = new User('lee@example.test');
$user->email = 'other@example.test';
~~~

That may be useful for mutable state, but it can make an object harder to reason about. If an email is intended to be fixed for the lifetime of this object, readonly expresses that rule in the language.

## Declare a readonly property

Readonly properties were introduced in PHP 8.1. They must have a type:

~~~php
class UserIdentity
{
    public function __construct(
        public readonly int $id,
        public readonly string $email,
    ) {
    }
}

$user = new UserIdentity(42, 'lee@example.test');

echo $user->id;
echo $user->email;
~~~

The constructor initializes the promoted properties while the object is being created. You can also declare a property separately and assign it in the declaring class:

~~~php
class ApiToken
{
    public readonly string $value;

    public function __construct(string $value)
    {
        $this->value = $value;
    }
}
~~~

A readonly property without an initial value is uninitialized until the class assigns it. Reading it before initialization throws an <code>Error</code>. Make sure every valid construction path initializes every readonly property that callers may read.

## A property can be initialized only once

The first permitted assignment initializes a readonly property. After that assignment, another assignment fails, even if it uses the same value:

~~~php
class Build
{
    public function __construct(public readonly string $version)
    {
    }
}

$build = new Build('1.0');
// $build->version = '1.0'; // Error: the property is already initialized
~~~

The assignment must happen from a scope allowed to initialize that property. For properties declared in a class, the declaring class is the clearest place to perform initialization, usually in its constructor. PHP 8.4 changed the default setter visibility; the version notes below explain that detail.

Readonly is not a promise that the property has a value at declaration time. Before initialization, it has an uninitialized state. A nullable type such as <code>?string</code> does not by itself initialize the property to <code>null</code>.

## Operations PHP rejects

After initialization, PHP rejects operations that would change the property:

~~~php
class Counter
{
    public function __construct(public readonly int $value)
    {
    }

    public function invalidChanges(): void
    {
        // $this->value = 10;
        // $this->value++;
        // $this->value += 1;
        // unset($this->value);
    }
}
~~~

Readonly array properties cannot be changed indirectly either. These operations attempt to modify the property’s value:

~~~php
class SearchResult
{
    public function __construct(public readonly array $items)
    {
    }
}

$result = new SearchResult(['first']);

// All of these are invalid after initialization:
// $result->items[] = 'second';
// $result->items[0] = 'changed';
// unset($result->items[0]);
~~~

A readonly property must also be initialized by direct assignment. Passing an uninitialized property to code that fills an argument by reference is not a valid way to initialize it.

## Readonly is shallow

Readonly prevents replacing the property value. It does not freeze the internal state of an object stored in that property:

~~~php
class MetadataHolder
{
    public function __construct(public readonly object $metadata)
    {
    }
}

$metadata = (object) ['views' => 1];
$holder = new MetadataHolder($metadata);

$holder->metadata->views++; // The referenced object can still change.
// $holder->metadata = new stdClass(); // Replacing the property is invalid.
~~~

The property still points to the same object, but that object’s own public state changed. Arrays behave differently: a readonly array property cannot have its elements changed through that property.

For stronger immutability, use immutable nested objects such as <code>DateTimeImmutable</code>, avoid exposing mutable objects, or make defensive copies where appropriate. Readonly is a useful language-level guard, not a universal deep-freeze mechanism.

## Default values and nullable properties

Before PHP 8.6, an individually declared readonly property could not have a property default value. Initialize it in the constructor instead:

~~~php
class Pagination
{
    public readonly int $page;

    public function __construct(int $page = 1)
    {
        $this->page = $page;
    }
}
~~~

A default value on a promoted constructor parameter is a parameter default. The constructor still initializes the promoted property for each new object.

As of PHP 8.6, a readonly property may declare a default value. That default initializes the property as each object is created, so the constructor cannot assign to it again:

~~~php
class FeatureFlag
{
    public readonly bool $enabled = false;
}
~~~

Do not rely on PHP 8.6 property defaults if the project supports an older runtime. A typed nullable property declared without a default is still uninitialized until it is explicitly assigned, including when the desired value is <code>null</code>.

## Readonly classes

Readonly classes were introduced in PHP 8.2. The modifier makes all declared instance properties readonly:

~~~php
readonly class Money
{
    public function __construct(
        public int $minorUnits,
        public string $currency,
    ) {
    }
}

$price = new Money(3999, 'USD');
echo $price->currency;
~~~

With a readonly class, you do not need to repeat <code>readonly</code> on each property. A class-level declaration can make a data object’s intent easier to see.

A readonly class has important restrictions:

- Every declared instance property must have a type.
- Static properties are not allowed.
- Dynamic properties cannot be added.
- The <code>#[AllowDynamicProperties]</code> attribute cannot enable dynamic properties on it.
- A class extending a readonly class must also be readonly.
- A readonly class cannot extend a non-readonly class.

Class constants are not properties and can still be declared. Methods can read the properties and return new objects, but cannot reassign an initialized property.

## Use a new object for changed values

When a value object should appear to change, create a replacement object instead of mutating the original:

~~~php
readonly class Point
{
    public function __construct(
        public int $x,
        public int $y,
    ) {
    }

    public function withX(int $x): self
    {
        return new self($x, $this->y);
    }
}

$first = new Point(2, 5);
$second = $first->withX(9);

echo $first->x;  // 2
echo $second->x; // 9
~~~

This copy-style method preserves the original object and returns a new value with the requested change. It is often easier to test and reason about than hidden mutation.

## Cloning and readonly properties

As of PHP 8.3, a class can reinitialize its readonly properties while its <code>__clone()</code> method runs. This can support specialized copy behavior, but it does not make ordinary post-construction writes legal.

For example, a clone method can assign a new value to an uninitialized readonly property on the clone. Treat this as a deliberate, version-specific design; ordinary value objects can often use a method that constructs a fresh instance instead.

## Property setter visibility in PHP 8.4

Readonly properties have restricted write access. Before PHP 8.4, their implicit setter visibility was private to the declaring class. As of PHP 8.4, it is implicitly <code>protected(set)</code>, so a child class may initialize an inherited readonly property from its scope when that is otherwise permitted.

This change affects inheritance and initialization design. If a library needs stable behavior across multiple PHP versions, keep readonly initialization in the class that declares the property, or use explicit asymmetric visibility where the supported versions allow it. Do not assume that code using inherited readonly properties behaves identically on PHP 8.3 and PHP 8.4.

## Version notes

| PHP version | Readonly feature |
| --- | --- |
| PHP 8.1 | Readonly typed properties |
| PHP 8.2 | Readonly classes |
| PHP 8.3 | Readonly properties may be reinitialized during <code>__clone()</code> |
| PHP 8.4 | Implicit setter visibility changes to <code>protected(set)</code> |
| PHP 8.6 | Readonly properties may declare default values |

Choose the syntax based on the project’s minimum supported PHP version. Code that uses only typed readonly properties needs PHP 8.1 or later. A readonly class needs PHP 8.2 or later.

## When to use readonly

Readonly is a good fit when a value belongs to an object and should be fixed after construction:

- identifiers assigned when a record is loaded
- validated request or response data
- value objects such as money, coordinates, and date ranges
- configuration snapshots
- data-transfer objects between application layers

Use a normal property when the object represents state that must change, such as a job’s progress, a mutable form, or a cache that updates during the object’s lifetime. Do not use readonly only to make code look modern; choose it when it communicates a real invariant.

## Common mistakes

- **Forgetting the type:** readonly properties must be typed. Use <code>mixed</code> when an intentionally unrestricted type is needed.
- **Reading before initialization:** a nullable type is not automatically initialized to <code>null</code>.
- **Assigning twice:** even assigning the same value again is rejected.
- **Changing an array element:** readonly protects the array value, including its offsets.
- **Assuming deep immutability:** an object stored in a readonly property may still be mutable.
- **Declaring static state in a readonly class:** readonly classes cannot declare static properties.
- **Adding undeclared fields:** readonly classes do not allow dynamic properties.
- **Using readonly class syntax on PHP 8.1:** readonly classes require PHP 8.2.
- **Expecting identical inherited setter behavior:** PHP 8.4 changed implicit setter visibility.
- **Adding property defaults on older PHP versions:** individually declared readonly property defaults require PHP 8.6.

## Key takeaways

- A readonly property is typed, initialized once, and cannot be reassigned afterward.
- A readonly property without a value remains uninitialized until the class assigns it.
- Readonly array offsets cannot be changed through the property.
- Readonly does not freeze mutable objects stored in a property.
- A readonly class makes all of its declared instance properties readonly and forbids static and dynamic properties.
- Return a new object when a value object needs a different value.
- Check the minimum supported PHP version before using version-specific readonly behavior.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [072 - Class constants and constant expressions](072-class-constants-and-constant-expressions.md) | [Q&A](../qnas/073-readonly-properties-and-classes.md) | [Example](../examples/073-readonly-properties-and-classes.md) | [074 - Constructor property promotion](074-constructor-property-promotion.md) |
