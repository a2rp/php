# Composition and object collaboration

Composition builds an object from other objects that perform focused jobs. Those objects collaborate through public methods.

Inheritance describes an **"is-a"** relationship: a `SavingsAccount` is a kind of `BankAccount`. Composition describes a **"has-a"** or **"uses-a"** relationship: a `Report` has a formatter, or a `CheckoutService` uses a payment processor.

Use composition when an object needs another object's behavior but is not a specialized kind of that object.

## What you will learn

- distinguish inheritance from composition
- give an object a typed collaborator
- inject required dependencies through a constructor
- delegate work through public methods
- keep collaborators replaceable and responsibilities focused
- recognize common collaboration and dependency mistakes

## Start with the relationship

Suppose a report needs text formatted. A report is not a kind of formatter. Making `Report extends Formatter` would claim that every report can be used wherever a formatter is expected.

Instead, the report can use a formatter:

~~~php
<?php
declare(strict_types=1);

class Formatter
{
    public function format(string $text): string
    {
        return strtoupper(trim($text));
    }
}

class Report
{
    public function __construct(
        private Formatter $formatter
    ) {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}
~~~

The `Report` stores a reference to a `Formatter`. When `render()` is called, it delegates formatting to that collaborator.

## Constructor injection

Passing a dependency into a constructor is called **constructor injection**. The code that creates the objects chooses and supplies the collaborator:

~~~php
$formatter = new Formatter();
$report = new Report($formatter);

echo $report->render(' monthly results ');
// MONTHLY RESULTS
~~~

This makes a required dependency visible when a `Report` is created. The report cannot be constructed without a formatter.

The caller is sometimes called the **composition root**: the part of the program that assembles objects and connects their dependencies. In a small script, that may be the bottom of a file. Larger applications often centralize object creation in a bootstrap or container.

## Constructor property promotion

The example uses constructor property promotion, available in PHP 8.0 and later:

~~~php
public function __construct(
    private Formatter $formatter
) {
}
~~~

This declares the property and assigns the parameter to it in one step. The expanded form is useful when learning or when initialization needs more logic:

~~~php
class Report
{
    private Formatter $formatter;

    public function __construct(Formatter $formatter)
    {
        $this->formatter = $formatter;
    }
}
~~~

Both forms retain the collaborator in an object property. The expanded form works in PHP 7.4; typed properties were introduced in PHP 7.4.

A dependency that must not be reassigned after construction can use a readonly property in PHP 8.1 or later:

~~~php
public function __construct(
    private readonly Formatter $formatter
) {
}
~~~

Readonly prevents replacing the property with a different object. It does not make the collaborator object itself immutable.

## Collaborate through a clear public contract

A client should rely on a collaborator's public operations, not reach into its internal state:

~~~php
class Report
{
    public function __construct(
        private Formatter $formatter
    ) {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}
~~~

Here, `Report` knows that `Formatter` can format text. It does not need to know how formatting is implemented. Prefer a method that returns a result over exposing mutable internal fields.

## Give each object a focused responsibility

A class becomes hard to maintain when it creates, configures, formats, stores, sends, and logs everything itself. Composition lets focused objects cooperate:

- a `Report` decides what content belongs in a report
- a `Formatter` turns text into a chosen presentation
- a `Publisher` decides where or how finished content is delivered

Each collaborator should have a clear reason to change. Avoid splitting every tiny operation into a class; create a collaborator when it represents a meaningful responsibility or behavior you may want to replace.

## Replace a collaborator

Constructor injection makes it possible to provide a different implementation. Here, `Report` accepts a parent type and can receive specialized formatters:

~~~php
class PlainFormatter extends Formatter
{
    public function format(string $text): string
    {
        return trim($text);
    }
}

class UppercaseFormatter extends Formatter
{
    public function format(string $text): string
    {
        return strtoupper(parent::format($text));
    }
}

$plainReport = new Report(new PlainFormatter());
$uppercaseReport = new Report(new UppercaseFormatter());
~~~

Both objects can be given to `Report` because they extend `Formatter`. The second preserves the parent's formatting and applies its own behavior.

A shared interface can express a contract without requiring a shared implementation. Interfaces are covered in [Interfaces and contracts](069-interfaces-and-contracts.md).

Whether you use a parent class or an interface, the caller and collaborator need to agree on the public behavior the client uses. Chapter 067 covers compatible method overrides and substitutability.

## Collaborator chains

An object can collaborate with another object that itself has a dependency:

~~~php
class ReportService
{
    public function __construct(
        private Report $report
    ) {
    }

    public function create(string $title): string
    {
        return $this->report->render($title);
    }
}

$service = new ReportService(new Report(new Formatter()));
echo $service->create('monthly results');
~~~

This is a small object graph: the service uses the report, and the report uses the formatter. The code assembling them can see the full wiring.

Keep the chain understandable. If following one method call requires traversing many unrelated objects, simplify the design or clarify each responsibility.

## Shared objects and ownership

Assigning an object to another variable does not clone it. Both variables can refer to the same object:

~~~php
$first = new Formatter();
$second = $first;

var_dump($first === $second); // bool(true)
~~~

Passing a collaborator to a constructor also does not automatically copy it. Decide which part of the program is responsible for creating, configuring, and sharing that object. A dependency is not automatically owned exclusively just because it is stored in a property.

If multiple clients share a mutable object, changes made through one reference can be observed through another.

## Required and optional collaborators

Use a constructor parameter for behavior the object always needs:

~~~php
class Report
{
    public function __construct(
        private Formatter $formatter
    ) {
    }
}
~~~

Do not silently create a default dependency inside the class if callers should control which implementation is used. Hidden construction makes replacement harder and hides what the object needs.

An optional collaborator may be nullable or represented by a deliberate default, but make that choice explicit. Do not accept `null` for a dependency that is required and then fail later with an unclear error.

## Avoid circular dependencies

A circular dependency occurs when object A requires object B while object B also requires object A. Their constructors cannot be called in a straightforward order, and their responsibilities are often tangled.

~~~text
OrderService requires InvoiceService
InvoiceService requires OrderService
~~~

Move shared work into a third collaborator, or redesign the interaction so one object calls the other in one direction. Keep dependency arrows easy to follow.

## Composition and inheritance

| Question | Inheritance | Composition |
| --- | --- | --- |
| Relationship | "is a" | "has a" or "uses a" |
| Main mechanism | Extend a parent class | Store and call a collaborator |
| Reuse | Inherit behavior and specialize it | Delegate selected work |
| Change behavior | Override methods | Supply another collaborator |
| Typical fit | A genuine subtype | An object needing another object's service |

These are tools, not competing rules. A design can use inheritance for a genuine subtype and composition for its collaborators.

## Common mistakes

- **Extending a class just to reuse one method:** give the object a collaborator and delegate instead.
- **Constructing collaborators secretly:** pass required dependencies in so callers can choose and understand them.
- **Using concrete internal details:** collaborate through public methods.
- **Making one object do every job:** separate responsibilities that change for different reasons.
- **Over-fragmenting the design:** keep small, tightly related operations together.
- **Creating circular dependencies:** make collaboration directional or extract shared work.
- **Assuming a passed object was copied:** object assignment does not clone it.
- **Allowing invalid dependency states:** require essential collaborators when the object is constructed.

## Key takeaways

- Composition connects collaborating objects; it does not claim one is a subtype of another.
- Constructor injection makes required dependencies explicit.
- Typed properties document and constrain which collaborator an object accepts.
- Delegate work through a collaborator's public methods.
- The caller that assembles objects can choose which collaborator to provide.
- Focused responsibilities make objects easier to change and reuse.
- Avoid hidden dependencies and circular object graphs.
- Constructor property promotion requires PHP 8.0; typed properties require PHP 7.4.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [067 - Inheritance and method overriding](067-inheritance-and-method-overriding.md) | [Q&A](../qnas/068-composition-and-object-collaboration.md) | [Example](../examples/068-composition-and-object-collaboration.md) | [069 - Abstract classes and final declarations](069-abstract-classes-and-final-declarations.md) |
