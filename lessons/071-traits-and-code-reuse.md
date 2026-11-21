# Traits and code reuse

A trait packages methods and related members so multiple classes can reuse them without sharing a parent class. A class includes a trait with the <code>use</code> statement.

Traits help when independent classes need the same small, focused behavior but do not form a genuine "is-a" hierarchy. A trait is not a class: it cannot be instantiated on its own.

## What you will learn

- define and include a trait
- reuse multiple traits in one class
- resolve method name conflicts with <code>insteadof</code> and <code>as</code>
- change an imported method's visibility or provide an alias
- compose traits from other traits
- define requirements with abstract trait methods
- understand trait properties, constants, and PHP version notes
- choose between a trait, interface, abstract class, and composition

## Define and use a trait

A trait declares reusable methods:

~~~php
trait Timestamped
{
    public function currentTimestamp(): string
    {
        return date('Y-m-d H:i:s');
    }
}
~~~

A class includes the trait with <code>use</code> inside its class body:

~~~php
class Article
{
    use Timestamped;

    public function title(): string
    {
        return 'Learning PHP';
    }
}

$article = new Article();

echo $article->title() . PHP_EOL;
echo $article->currentTimestamp();
~~~

The trait's method becomes available as a method of the class using it. There is no separate Timestamped object. This combines behavior with the class without creating a parent-child relationship.

## Reuse behavior across unrelated classes

Two classes in different parts of a hierarchy can use the same trait:

~~~php
trait HasDisplayName
{
    public function displayName(): string
    {
        return trim($this->name);
    }
}

class Customer
{
    use HasDisplayName;

    public function __construct(
        private string $name
    ) {
    }
}

class Team
{
    use HasDisplayName;

    public function __construct(
        private string $name
    ) {
    }
}

echo (new Customer('  Asha  '))->displayName();
echo (new Team('  Platform  '))->displayName();
~~~

The trait expects the using class to have a <code>$name</code> property. That expectation is a dependency. Make such requirements clear, or declare an abstract method in the trait when possible so PHP can check that the class provides it.

Do not use a trait merely to avoid writing a short method twice. Reuse it when it represents a coherent behavior and its assumptions fit every using class.

## Use multiple traits

A class can include multiple traits:

~~~php
trait CanPublish
{
    public function publish(): string
    {
        return 'Published';
    }
}

trait CanArchive
{
    public function archive(): string
    {
        return 'Archived';
    }
}

class BlogPost
{
    use CanPublish, CanArchive;
}

$post = new BlogPost();
echo $post->publish();
echo $post->archive();
~~~

Each trait should have a focused purpose. A class that accumulates many unrelated traits can become difficult to understand, much like a class with too many responsibilities.

## Resolve method name conflicts

If two traits provide methods with the same name, PHP requires an explicit choice:

~~~php
trait FriendlyGreeting
{
    public function greet(): string
    {
        return 'Hi!';
    }
}

trait FormalGreeting
{
    public function greet(): string
    {
        return 'Good day.';
    }
}

class Greeter
{
    use FriendlyGreeting, FormalGreeting {
        FormalGreeting::greet insteadof FriendlyGreeting;
        FriendlyGreeting::greet as casualGreet;
    }
}

$greeter = new Greeter();

echo $greeter->greet();       // Good day.
echo $greeter->casualGreet(); // Hi!
~~~

The <code>insteadof</code> operator selects which implementation wins for the conflicting method name. The <code>as</code> operator adds an alias to a trait method. It does not remove or rename the original method.

## Change method visibility

The <code>as</code> operator can change the visibility of an imported method:

~~~php
trait HasInternalId
{
    public function createInternalId(): string
    {
        return 'id-123';
    }
}

class Record
{
    use HasInternalId {
        createInternalId as protected;
    }

    public function idForStorage(): string
    {
        return $this->createInternalId();
    }
}
~~~

Here, the method is available inside Record and its subclasses, but external callers cannot call it directly.

You can combine an alias and a visibility change:

~~~php
class AnotherRecord
{
    use HasInternalId {
        createInternalId as protected makeInternalId;
    }

    public function idForStorage(): string
    {
        return $this->makeInternalId();
    }
}
~~~

The alias is a second name for the imported method, with protected visibility.

## Understand method precedence

For methods with the same name, PHP uses this precedence:

1. A method declared directly in the class wins over the trait method.
2. A trait method wins over an inherited method from the parent class.

~~~php
class BasePage
{
    public function heading(): string
    {
        return 'Base heading';
    }
}

trait AddsPageHeading
{
    public function heading(): string
    {
        return 'Trait heading';
    }
}

class HomePage extends BasePage
{
    use AddsPageHeading;
}

class CustomHomePage extends BasePage
{
    use AddsPageHeading;

    public function heading(): string
    {
        return 'Class heading';
    }
}

echo (new HomePage())->heading();       // Trait heading
echo (new CustomHomePage())->heading(); // Class heading
~~~

Do not rely on accidental precedence to hide confusing behavior. Use explicit conflict rules and clear method names when multiple sources provide similar methods.

## Compose traits from traits

A trait can include other traits:

~~~php
trait HasCreatedAt
{
    public function createdAt(): string
    {
        return date('Y-m-d');
    }
}

trait HasUpdatedAt
{
    public function updatedAt(): string
    {
        return date('Y-m-d');
    }
}

trait HasTimestamps
{
    use HasCreatedAt, HasUpdatedAt;
}

class Comment
{
    use HasTimestamps;
}
~~~

This can group related behavior for reuse. If nested traits bring in conflicting methods, resolve the conflicts where the traits are composed.

## Require a method from the using class

A trait can declare an abstract method. The class using the trait must provide a compatible implementation:

~~~php
trait HasGreeting
{
    abstract protected function greetingName(): string;

    public function greeting(): string
    {
        return 'Hello, ' . $this->greetingName();
    }
}

class Member
{
    use HasGreeting;

    public function __construct(
        private string $name
    ) {
    }

    protected function greetingName(): string
    {
        return $this->name;
    }
}

echo (new Member('Asha'))->greeting();
// Hello, Asha
~~~

The requirement makes the trait's dependency explicit. The using class supplies the class-specific detail.

## Trait properties

Traits may declare properties. When a trait is used, those properties become part of the class:

~~~php
trait TracksViews
{
    private int $views = 0;

    public function recordView(): void
    {
        $this->views++;
    }

    public function viewCount(): int
    {
        return $this->views;
    }
}
~~~

Take care when the using class or another trait declares a property with the same name. Conflicting property definitions can make trait composition invalid. Keep trait state small, clearly named, and consistent with the classes that use it.

Trait and parent class property/constant composition precedence changed in PHP 8.5. Avoid defining same-named properties or constants in a trait and its parent class; use distinct names instead of relying on version-specific resolution.

## Trait constants

Traits can define constants as of PHP 8.2:

~~~php
trait JsonDefaults
{
    public const CONTENT_TYPE = 'application/json';
}
~~~

A class using the trait can access the constant through the class:

~~~php
class JsonResponse
{
    use JsonDefaults;
}

echo JsonResponse::CONTENT_TYPE;
~~~

For PHP versions before 8.2, define the constant in the class or use another compatible design.

## Static trait members

Traits can contain static methods and properties. Access static members through the class that uses the trait, not through the trait name. Direct access through a trait name has been deprecated since PHP 8.1.

~~~php
trait HasVersion
{
    public static function version(): string
    {
        return '1.0';
    }
}

class ApiClient
{
    use HasVersion;
}

echo ApiClient::version();
~~~

Static property behavior when a trait is reused within an inheritance hierarchy changed in PHP 8.3. Avoid using trait static properties for shared mutable state unless the target PHP versions and intended sharing behavior are well understood.

## Final methods imported from traits in PHP 8.3+

PHP 8.3 added syntax for marking a method imported from a trait as final:

~~~php
trait StableIdentifier
{
    public function identifier(): string
    {
        return 'record-1';
    }
}

class Record
{
    use StableIdentifier {
        StableIdentifier::identifier as final;
    }
}
~~~

A child of Record cannot override the imported identifier method. The class using the trait itself can still define or override that method. This adaptation syntax requires PHP 8.3 or later.

## Traits, interfaces, abstract classes, and composition

| Tool | What it provides | Typical use |
| --- | --- | --- |
| Trait | Reusable methods and possibly properties | Share a focused implementation across classes |
| Interface | A public behavior contract | Allow interchangeable implementations |
| Abstract class | Shared state or behavior plus required subclass methods | Build related classes on a common base |
| Composition | One object collaborates with another object | Delegate a separate responsibility |

A class may use a trait and implement an interface at the same time. The interface says what behavior the class promises; the trait may provide some of that behavior. A trait itself does not create an interface type and cannot be instantiated.

Prefer composition when the reused behavior is really a service with its own state or lifecycle. Prefer an interface when callers need a contract but should not inherit an implementation.

## Common mistakes

- **Trying to instantiate a trait:** instantiate a class that uses it.
- **Using a trait as a substitute for every abstraction:** choose the mechanism that matches the relationship.
- **Ignoring hidden trait assumptions:** declare required operations with abstract methods or document dependencies.
- **Combining conflicting methods without adaptation:** select a method with <code>insteadof</code> and add aliases with <code>as</code>.
- **Assuming an alias removes the original name:** an alias adds another name; it does not rename the method.
- **Adding unrelated behavior to one trait:** keep traits cohesive.
- **Duplicating state across many traits:** stateful traits can create confusing interactions.
- **Accessing static members through the trait name:** access them through a class that uses the trait.
- **Using newer trait features on older PHP:** constants require PHP 8.2; final method adaptation requires PHP 8.3.
- **Depending on trait and parent members with the same names:** avoid version-sensitive collisions.

## Key takeaways

- Traits reuse implementation across classes without requiring a shared parent class.
- A trait is included with <code>use</code> and cannot be instantiated.
- Classes can use multiple traits and traits can include other traits.
- Resolve method conflicts with <code>insteadof</code>; use <code>as</code> for aliases or visibility changes.
- A class method overrides an imported trait method, and a trait method overrides an inherited parent method.
- Abstract trait methods declare requirements for the class using the trait.
- Trait constants require PHP 8.2; final method adaptation requires PHP 8.3.
- Use traits for cohesive code reuse, interfaces for contracts, and composition for collaborating services.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [070 - Interfaces and contracts](070-interfaces-and-contracts.md) | [Q&A](../qnas/071-traits-and-code-reuse.md) | [Example](../examples/071-traits-and-code-reuse.md) | [072 - Class constants and constant expressions](072-class-constants-and-constant-expressions.md) |
