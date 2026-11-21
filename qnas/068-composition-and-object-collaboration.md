# Composition and object collaboration - Q&A

Use these questions to review when objects should collaborate, how to supply dependencies, and how to keep object relationships clear. Every question is followed by its answer and explanation.

## Multiple-choice questions

### 1. What does composition mean in object-oriented programming?

A. One class inherits all methods from another
B. An object uses other objects to perform focused jobs
C. A function is declared inside another function
D. A property is made private

**Answer: B.** Composition connects objects that collaborate through their public behavior. It represents a "has-a" or "uses-a" relationship.

### 2. Which is a natural inheritance relationship?

A. A report uses a formatter
B. A checkout service uses a payment gateway
C. A savings account is a kind of bank account
D. A page uses a renderer

**Answer: C.** Inheritance models an "is-a" relationship. The other examples describe collaboration.

### 3. A Report needs a Formatter. Which design states that relationship directly?

A. Report extends Formatter
B. Report stores a Formatter property
C. Formatter is a global variable
D. Report copies all formatter methods

**Answer: B.** A report is not a formatter; it uses one. A collaborator property models that relationship.

### 4. What is constructor injection?

A. Creating dependencies as globals
B. Passing a dependency into an object's constructor
C. Calling a constructor from every method
D. Making every constructor private

**Answer: B.** Constructor injection supplies a dependency at object creation, making required collaborators explicit.

### 5. What is a key benefit of constructor injection?

A. It makes properties public
B. It hides which services an object needs
C. It makes dependencies visible and replaceable
D. It prevents the class from having methods

**Answer: C.** Callers can see and choose the required collaborators instead of relying on hidden construction.

### 6. In PHP 8.0+, what does this declaration do?

~~~php
public function __construct(private Formatter $formatter)
{
}
~~~

A. Declares a private property and assigns the constructor argument to it
B. Creates a new formatter automatically
C. Makes Formatter a parent class
D. Converts the formatter to a string

**Answer: A.** This is constructor property promotion. It declares the property and initializes it from the argument.

### 7. Which version introduced typed properties?

A. PHP 5.6
B. PHP 7.0
C. PHP 7.4
D. PHP 8.2

**Answer: C.** Typed properties were introduced in PHP 7.4. Constructor property promotion came in PHP 8.0.

### 8. What does a readonly promoted property guarantee?

A. The collaborator cannot change its own state
B. The property cannot be reassigned after initialization
C. The object is cloned before storage
D. The class has no other properties

**Answer: B.** Readonly prevents reassignment of that property; it does not make the collaborator itself immutable.

### 9. What is the composition root?

A. The parent class at the top of an inheritance tree
B. The part of the application that creates and connects objects
C. The first method in each class
D. A database table for dependencies

**Answer: B.** The composition root assembles the object graph by creating objects and passing their dependencies.

### 10. If $second = $first and $first is an object, what happens?

A. PHP always deep-clones the object
B. Both variables refer to the same object
C. $second becomes null
D. The object's class changes

**Answer: B.** Assignment does not clone an object. Both variables can access the same object.

## Short-answer questions

### 11. What is a collaborator?

**Answer:** A collaborator is another object that a class uses to perform some part of its work.

**Explanation:** Report is the client and Formatter is its collaborator. Report delegates formatting instead of implementing that behavior itself.

### 12. How do composition and inheritance differ?

**Answer:** Inheritance says one class is a specialized kind of another; composition says an object has or uses another object.

**Explanation:** A SavingsAccount may extend BankAccount. A Report should receive or store a formatter because it is not a formatter.

### 13. Why should a required collaborator usually be passed to the constructor?

**Answer:** The object cannot be created in an incomplete state, and callers can see what it needs.

**Explanation:** Constructing a dependency secretly inside the class hides that requirement and makes it harder to substitute another implementation.

### 14. What does delegation mean?

**Answer:** Delegation means asking a collaborator to perform work on the object's behalf.

**Explanation:** Report::render() can call Formatter::format() and return its result.

### 15. What is a dependency?

**Answer:** A dependency is an object or service another object needs to do its job.

**Explanation:** Report depends on Formatter because rendering calls the formatter.

### 16. Why are typed collaborator properties useful?

**Answer:** They document which kind of object is expected and let PHP check the value's type.

**Explanation:** A Formatter property rejects unrelated values instead of retaining invalid state until a later call.

### 17. Does constructor injection give a class exclusive ownership of a collaborator?

**Answer:** No.

**Explanation:** The caller may pass the same object to multiple clients. Storing it in a property does not clone it or prove exclusive ownership.

### 18. Why can sharing a mutable collaborator require care?

**Answer:** Changes through one reference may be visible through another reference to the same object.

**Explanation:** Shared mutable state can cause clients to affect one another. Define who can change it or avoid unnecessary sharing.

### 19. How can a circular dependency be improved?

**Answer:** Change the direction of collaboration or move shared work into a third collaborator.

**Explanation:** If OrderService and InvoiceService require each other, both might depend on a focused TaxCalculator instead.

### 20. Should every small operation become its own class?

**Answer:** No.

**Explanation:** Composition can separate meaningful responsibilities, but excessive splitting creates needless wiring and makes the object graph harder to follow.

### 21. How can a dependency be replaceable?

**Answer:** Accept it through a constructor using a suitable parent type or interface, then supply another compatible object.

**Explanation:** A clear contract lets a client use different implementations without changing its own code.

## Code-reading and design questions

### 22. What does this program print?

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
    public function __construct(private Formatter $formatter)
    {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}

$report = new Report(new Formatter());
echo $report->render('  status  ');
~~~

**Answer:** <code>STATUS</code>

**Explanation:** The report delegates to Formatter. The formatter trims the surrounding whitespace and converts the remaining text to uppercase.

### 23. Why is Report extending Formatter usually a poor fit?

**Answer:** It claims that a report is a kind of formatter.

**Explanation:** A report should not be usable everywhere a formatter is required. A Formatter property expresses the intended collaboration.

### 24. What is the issue with creating the formatter inside render()?

~~~php
class Report
{
    public function render(string $text): string
    {
        $formatter = new Formatter();
        return $formatter->format($text);
    }
}
~~~

**Answer:** The dependency is hidden and hard-coded.

**Explanation:** Callers cannot easily choose another formatter, and the class's requirement is not visible in its constructor. Inject the formatter instead.

### 25. Is this a safe representation for a required formatter?

~~~php
class Report
{
    private ?Formatter $formatter = null;

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}
~~~

**Answer:** No.

**Explanation:** The property can still be null, so render() may call a method on null. Make it non-nullable and initialize it in the constructor if it is required.

### 26. What does this code print?

~~~php
$formatter = new Formatter();
$anotherName = $formatter;
var_dump($formatter === $anotherName);
~~~

**Answer:** <code>bool(true)</code>

**Explanation:** Both variables refer to the same object. Assignment does not clone it.

### 27. Is it necessarily bad if ReportService calls Report, which calls Formatter?

**Answer:** No.

**Explanation:** A short chain can represent clear responsibilities. It is concerning when the chain is unnecessarily long or tangled.

### 28. How can a client be tested without its production collaborator?

**Answer:** Supply a small test-specific object that follows the same expected contract.

**Explanation:** Injection lets the caller choose the object. An interface or suitable parent type can make this replacement explicit.

### 29. Should a client directly change a collaborator's private fields?

**Answer:** No.

**Explanation:** Private state is an implementation detail. Ask the collaborator to perform work through a public method so it can preserve its own rules.

### 30. A class formats content, writes files, sends email, and logs analytics. What design concern does this raise?

**Answer:** It likely has too many responsibilities.

**Explanation:** Those behaviors may change for different reasons. Focused collaborators can handle formatting, publishing, notifications, and logging separately.

## Coding questions

### 31. Write a Formatter and inject it into a Report.

**Answer:**

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
    public function __construct(private Formatter $formatter)
    {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}

$report = new Report(new Formatter());
echo $report->render(' monthly results ');
~~~

**Explanation:** Report receives its required dependency in the constructor and delegates formatting through the public format() method.

### 32. Add a plain formatter that can be passed to the same report.

**Answer:**

~~~php
class PlainFormatter extends Formatter
{
    public function format(string $text): string
    {
        return trim($text);
    }
}

$report = new Report(new PlainFormatter());
echo $report->render(' monthly results '); // monthly results
~~~

**Explanation:** PlainFormatter is a Formatter, so it satisfies the constructor's declared type while providing different behavior.

### 33. Show object assembly outside the Report class.

**Answer:**

~~~php
$formatter = new Formatter();
$report = new Report($formatter);

echo $report->render('monthly results');
~~~

**Explanation:** The calling code acts as a small composition root. Report uses the object but does not decide how to create it.

### 34. How could a shared calculation be extracted from two mutually dependent services?

**Answer:** Create a focused collaborator such as TaxCalculator and inject it into both services.

**Explanation:** The services then depend on the calculator rather than depending on each other, removing the cycle.

### 35. What should you decide before making a collaborator optional?

**Answer:** Decide what the client should do when the collaborator is absent, and represent that state explicitly.

**Explanation:** If the behavior cannot proceed without it, keep it required. If absence is valid, use a nullable type or a deliberate default and handle that case.

## Key takeaways

- Use composition when an object needs another object's behavior.
- Inject required collaborators through a constructor.
- Delegate through clear public methods.
- Avoid hidden construction, invalid nullable dependencies, and circular references.
- Object assignment does not clone the object.
- Keep responsibilities and collaborator chains understandable.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [067 - Inheritance and method overriding](../lessons/067-inheritance-and-method-overriding.md) | [Lesson](../lessons/068-composition-and-object-collaboration.md) | [Example](../examples/068-composition-and-object-collaboration.md) | [069 - Abstract classes and final declarations](../lessons/069-abstract-classes-and-final-declarations.md) |
