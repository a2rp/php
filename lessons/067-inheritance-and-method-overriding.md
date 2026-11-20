# 067 - Inheritance and Method Overriding

Inheritance lets a child class reuse and specialize behavior from a parent class. Method overriding lets a child provide its own implementation of an inherited method while keeping a compatible public contract.

## Learning goals

By the end of this chapter, you should be able to:

- Extend a class with the extends keyword.
- Explain which parent members a child inherits and can access.
- Override an inherited method and understand runtime method dispatch.
- Call a parent implementation with parent::.
- Initialize parent state by explicitly calling the parent constructor when needed.
- Keep method visibility and type declarations compatible.
- Recognize covariant return types and contravariant parameter types.
- Choose inheritance only when the child is genuinely a specialized kind of the parent.

## What inheritance means

A class can extend one parent class. The child inherits the parent's public and protected properties, methods, and constants, and may add or specialize behavior. A child does not directly inherit access to the parent's private members.

~~~php
class Person
{
    public function introduce(): string
    {
        return 'I am a person.';
    }
}

class Teacher extends Person
{
    public function teach(): string
    {
        return 'I teach a class.';
    }
}

$teacher = new Teacher();

echo $teacher->introduce() . PHP_EOL;
echo $teacher->teach() . PHP_EOL;
~~~

Teacher can use the inherited introduce() method and also defines its own teach() method.

A child class can extend only one class. PHP uses interfaces for types that need to represent multiple capabilities; implementing interfaces is covered separately.

## When inheritance is a good fit

Inheritance is useful when a child really is a specialized kind of its parent and should be usable anywhere that parent is expected.

For example, a SavingsAccount can be a kind of BankAccount if it follows the BankAccount contract. A class should not extend another merely to borrow one convenient method.

Good inheritance can:

- Reuse stable behavior shared by related classes.
- Let a child add or specialize behavior.
- Allow code to accept a parent type while working with different child implementations.

Inheritance can also create tight coupling. Changes to a parent's protected state or method behavior may affect every child. Keep hierarchies small and intentional.

## Inherited visibility

Public and protected parent members are inherited by a child. Private parent members remain private to the class that declares them.

~~~php
class Document
{
    private string $internalId;
    protected string $title;

    public function __construct(string $internalId, string $title)
    {
        $this->internalId = $internalId;
        $this->title = $title;
    }

    public function title(): string
    {
        return $this->title;
    }
}

class Report extends Document
{
    public function heading(): string
    {
        return 'Report: ' . $this->title;
    }
}
~~~

Report can read the protected title. It cannot directly read Document's private internalId. The parent may provide a public or protected method when subclasses need a supported way to use that value.

Keep parent fields private when children do not need direct access. Use protected only for a deliberate extension point because every protected member couples subclasses to the parent's implementation.

## Overriding a method

A child overrides an inherited method by declaring a method with the same name and a compatible signature. Calls on a child object use the child implementation, including calls made through code that accepts the parent type.

~~~php
class Notification
{
    public function message(): string
    {
        return 'You have a notification.';
    }
}

class EmailNotification extends Notification
{
    public function message(): string
    {
        return 'You have a new email.';
    }
}

function displayNotification(Notification $notification): void
{
    echo $notification->message() . PHP_EOL;
}

displayNotification(new Notification());
displayNotification(new EmailNotification());
~~~

The function accepts a Notification. When it receives an EmailNotification, PHP dispatches message() to the child's override at runtime. This behavior is called polymorphism.

A child can also inherit other methods without overriding them. EmailNotification still has every accessible public or protected method it inherits from Notification.

## Calling the parent implementation

An override can replace parent behavior completely or build on it by calling parent::methodName().

~~~php
class BasicLogger
{
    public function format(string $message): string
    {
        return '[LOG] ' . $message;
    }
}

class TimestampLogger extends BasicLogger
{
    public function format(string $message): string
    {
        return date('H:i:s') . ' ' . parent::format($message);
    }
}

$logger = new TimestampLogger();
echo $logger->format('Cache refreshed');
~~~

The child adds a timestamp and reuses the parent's formatting. Use parent:: when preserving the parent implementation is part of the intended behavior.

A child may choose not to call the parent method if it is intentionally replacing that behavior and the contract allows it.

## Constructors in an inheritance hierarchy

If a child does not declare a constructor, it can inherit the parent's constructor. If the child declares its own constructor, PHP does not automatically call the parent constructor.

Call parent::__construct() explicitly when the parent's constructor establishes required state:

~~~php
class Job
{
    public function __construct(
        protected string $jobId,
    ) {
        if (trim($this->jobId) === '') {
            throw new InvalidArgumentException('Job ID cannot be empty.');
        }
    }

    public function id(): string
    {
        return $this->jobId;
    }
}

class EmailJob extends Job
{
    public function __construct(
        string $jobId,
        private string $recipient,
    ) {
        parent::__construct($jobId);

        if (!filter_var($this->recipient, FILTER_VALIDATE_EMAIL)) {
            throw new InvalidArgumentException('Recipient must be a valid email.');
        }
    }

    public function recipient(): string
    {
        return $this->recipient;
    }
}

$email = new EmailJob('job-17', 'learner@example.test');
echo $email->id() . ': ' . $email->recipient();
~~~

The parent constructor validates and stores the job ID. The child constructor calls it, then validates the recipient. Without the parent call, the parent initialization would not run.

The parent constructor should be called in a deliberate order. If parent initialization must happen before child behavior, call it before using the inherited state.

## Compatible method signatures

An override must remain compatible with the method it replaces. In practical terms:

- Keep the method name and compatible parameter structure.
- Do not reduce public or protected visibility.
- Keep the return value compatible with what callers expect.
- Do not add new required parameters that callers of the parent method would not supply.

~~~php
class Exporter
{
    public function export(string $format): string
    {
        return 'Exporting as ' . $format;
    }
}

class JsonExporter extends Exporter
{
    public function export(string $format): string
    {
        return 'JSON export';
    }
}
~~~

This child preserves the method signature. It can change the implementation while honoring the parent method's contract.

This override is invalid because it narrows a public method to protected:

~~~php
class RestrictedExporter extends Exporter
{
    // Invalid: this reduces the inherited method's visibility.
    protected function export(string $format): string
    {
        return 'Restricted export';
    }
}
~~~

A protected method may be widened to public, but a public method cannot be narrowed to protected or private. Constructors have special visibility rules; ordinary overrides should preserve the parent's accessibility.

## Covariant returns and contravariant parameters

Since PHP 7.4, method overrides support full covariance for return types and contravariance for parameter types.

- **Covariance:** a child method may return a more specific type than the parent promises.
- **Contravariance:** a child method may accept a less specific, broader parameter type than the parent requires.

~~~php
class Animal
{
}

class Dog extends Animal
{
}

class Shelter
{
    public function adopt(string $name): Animal
    {
        return new Animal();
    }

    public function feed(Animal $animal): void
    {
        echo 'Feeding an animal.' . PHP_EOL;
    }
}

class DogShelter extends Shelter
{
    // A Dog is a more specific return type than Animal.
    public function adopt(string $name): Dog
    {
        return new Dog();
    }

    // Animal is broader than Dog, so this accepts at least what the parent accepts.
    public function feed(Animal $animal): void
    {
        echo 'Feeding a dog shelter animal.' . PHP_EOL;
    }
}
~~~

A more visible illustration of contravariance is a parent that accepts a specific class and a child that accepts its broader parent type:

~~~php
class Food
{
}

class AnimalFood extends Food
{
}

class Feeder
{
    public function feed(AnimalFood $food): void
    {
    }
}

class FlexibleFeeder extends Feeder
{
    public function feed(Food $food): void
    {
    }
}
~~~

FlexibleFeeder can still accept every AnimalFood value that callers were allowed to pass to Feeder, and it accepts additional Food values. The broader parameter is compatible.

Do not memorize variance as permission to change signatures arbitrarily. PHP checks type compatibility, and callers still rely on the parent's documented contract.

## Private parent methods are not override hooks

A private method is not available to the child and is not a normal override point. A child can declare a same-named method, but that is a separate method in the child's scope.

If child classes are intended to customize a method, declare it protected or public with a clear contract.

## Preventing extension and overriding

The final keyword can prevent a class from being extended or a method from being overridden:

~~~php
class Payment
{
    final public function currencyCode(): string
    {
        return 'USD';
    }
}

final class OneTimeToken
{
}
~~~

No child can override Payment::currencyCode(), and OneTimeToken cannot be extended. Use final when the class or behavior must remain fixed for correctness or security. The next language-reference chapter covers abstract classes and final declarations in more detail.

## Inheritance versus composition

If a class only needs another object's behavior, composition may be a better fit than inheritance. A class can hold another object and delegate work to it without claiming to be the same kind of thing.

For example, a Report may use a Formatter. The report is not necessarily a kind of formatter; it collaborates with one. The next chapter studies composition and object collaboration.

Prefer inheritance for a true subtype relationship. Prefer composition when one object uses or delegates to another.

## Common mistakes

- **Extending only to reuse one method:** the child inherits a wider contract and coupling than it needs.
- **Assuming private parent fields are available:** children cannot directly access them.
- **Forgetting parent constructor initialization:** a child constructor does not implicitly call it.
- **Reducing method visibility in an override:** public and protected contracts cannot be narrowed.
- **Adding required parameters to an override:** existing callers of the parent method may not provide them.
- **Calling parent:: automatically in every override:** call it only when the parent behavior should be preserved.
- **Using protected everywhere:** protected fields and methods make subclasses dependent on implementation details.
- **Creating deep hierarchies:** changes become harder to predict as behavior is spread across many ancestors.
- **Using inheritance where collaboration is intended:** consider composition when the relationship is not genuinely "is a".

## Key takeaways

- A PHP class extends one parent class using extends.
- Public and protected members are inherited; a parent's private members are not directly available to a child.
- Overridden methods are dispatched according to the runtime object.
- Use parent::method() to deliberately reuse parent behavior.
- A child-defined constructor must call parent::__construct() when parent initialization is required.
- Overrides must preserve compatible visibility and method contracts.
- PHP 7.4+ supports covariant returns and contravariant parameters.
- Use inheritance for a real subtype relationship; use composition for collaboration.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [066 - Static properties and methods](066-static-properties-and-methods.md) | [Q&A](../qnas/067-inheritance-and-method-overriding.md) | [Example](../examples/067-inheritance-and-method-overriding.md) | [068 - Composition and object collaboration](068-composition-and-object-collaboration.md) |
