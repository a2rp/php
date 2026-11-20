# 067 - Inheritance and Method Overriding: Q&A

Review class inheritance, overriding, parent calls, and compatible method contracts. Every question has an answer and explanation immediately after it. See the [chapter lesson](../lessons/067-inheritance-and-method-overriding.md) and [complete example](../examples/067-inheritance-and-method-overriding.md).

## Multiple-choice questions

### 1. Which keyword declares that one class extends another?

A. implements
B. extends
C. inherits
D. parent

**Answer:** B. extends.

**Explanation:** A class declaration uses extends followed by the parent class name.

### 2. How many parent classes can a PHP class extend directly?

A. One
B. Two
C. Any number
D. None

**Answer:** A. One.

**Explanation:** A PHP class extends one class. A class may implement multiple interfaces, which is a separate mechanism.

### 3. Which parent members are ordinarily inherited and available to a child, subject to visibility?

A. Only private methods
B. Public and protected members
C. Only constants
D. Global variables

**Answer:** B. Public and protected members.

**Explanation:** A child inherits public and protected properties, methods, and constants. Private members remain accessible only in the class that declared them.

### 4. Can a child directly read a private property declared by its parent?

A. Yes, every inherited property is public to a child.
B. Yes, if the child constructor calls the parent.
C. No, not directly.
D. Only through parent::$property.

**Answer:** C. No, not directly.

**Explanation:** A parent private property is not visible to the child. The parent can expose a deliberate public or protected method if needed.

### 5. What is method overriding?

A. Defining a same-named local function inside a method
B. A child providing a compatible implementation of an inherited method
C. Calling a method twice
D. Declaring multiple constructors

**Answer:** B. A child providing a compatible implementation of an inherited method.

**Explanation:** The child method specializes inherited behavior while preserving the method contract expected by callers.

### 6. Which method runs when a child object overrides a method and the object is used through a parent-typed parameter?

A. Always the parent's version
B. The child's override for that runtime object
C. Both methods at the same time
D. Neither method

**Answer:** B. The child's override for that runtime object.

**Explanation:** PHP dispatches the method based on the object's runtime class. This is polymorphism.

### 7. What does parent::methodName() do inside a child method?

A. Calls a global function
B. Calls the parent class implementation
C. Creates a parent object
D. Changes method visibility

**Answer:** B. Calls the parent class implementation.

**Explanation:** The child can deliberately reuse parent behavior with the parent scope-resolution keyword.

### 8. If a child defines its own constructor, does PHP automatically call the parent's constructor?

A. Yes, always.
B. Only if the parent has typed properties.
C. No; the child must explicitly call parent::__construct() when parent initialization is needed.
D. Only when the child is final.

**Answer:** C. No; the child must explicitly call parent::__construct() when parent initialization is needed.

**Explanation:** A child-defined constructor replaces the inherited constructor call path. Parent initialization is not implicit.

### 9. A parent has a public method. Can a child override it as protected?

A. Yes, visibility is unrelated to overriding.
B. No, that would reduce accessibility.
C. Yes, if it calls parent::.
D. Only in PHP 8.4.

**Answer:** B. No, that would reduce accessibility.

**Explanation:** An override cannot make an inherited public or protected method less accessible.

### 10. Which statement about private parent methods is correct?

A. A child can directly call them.
B. They are normal override hooks.
C. They are not directly accessible to the child and are not normal override hooks.
D. They become public in every child.

**Answer:** C. They are not directly accessible to the child and are not normal override hooks.

**Explanation:** A same-named method in a child is a separate method in the child's scope.

### 11. What is a covariant return type?

A. A child method returns a more specific type than the parent method promises.
B. A child method accepts fewer parameter values.
C. A child method has no return type.
D. A child method widens a parameter type.

**Answer:** A. A child method returns a more specific type than the parent method promises.

**Explanation:** For example, returning Dog is compatible with a parent method that promises Animal, because Dog is an Animal.

### 12. What is a contravariant parameter type?

A. A child method accepts a broader type than the parent method's parameter.
B. A child method returns a broader type.
C. A child method removes an inherited property.
D. A child method becomes private.

**Answer:** A. A child method accepts a broader type than the parent method's parameter.

**Explanation:** A broader accepted parameter still allows every value that could be passed to the parent method.

### 13. Since which PHP version has full covariance and contravariance support been available?

A. PHP 5.6
B. PHP 7.2
C. PHP 7.4
D. PHP 8.4

**Answer:** C. PHP 7.4.

**Explanation:** PHP 7.4 added full covariance and contravariance support for method signatures.

### 14. What does the final keyword do when applied to a method?

A. It forces the method to be static.
B. It prevents child classes from overriding the method.
C. It makes the method private.
D. It deletes inherited methods.

**Answer:** B. It prevents child classes from overriding the method.

**Explanation:** final marks a behavior that must remain fixed in child classes.

### 15. When is inheritance usually a good design choice?

A. Whenever a class wants to reuse one convenient method
B. When the child is genuinely a specialized kind of the parent and honors its contract
C. Whenever two classes have a property with the same name
D. To avoid writing constructors

**Answer:** B. When the child is genuinely a specialized kind of the parent and honors its contract.

**Explanation:** Inheritance expresses a subtype relationship, not just code reuse.

## Short-answer questions

### 16. What does the extends keyword express?

**Answer:** It declares that a class is a child or subclass of one parent class.

**Explanation:** The child inherits accessible behavior and may add or override behavior while remaining a subtype of its parent.

### 17. What is polymorphism in the context of method overriding?

**Answer:** Code can use a parent type while PHP invokes the implementation belonging to the object's actual runtime class.

**Explanation:** Different child objects can respond differently to the same method call through a shared parent contract.

### 18. Which members can a child access from its parent by visibility?

**Answer:** Public and protected parent members are available to the child; private parent members are not directly available.

**Explanation:** Visibility controls access regardless of inheritance. Public members are generally available to callers too, while protected members are limited to the hierarchy.

### 19. Why might a parent keep a property private instead of protected?

**Answer:** To prevent children from depending directly on the parent's internal representation.

**Explanation:** The parent can offer a deliberate method for needed behavior while retaining control over how its state is stored.

### 20. When should an overriding method call parent::methodName()?

**Answer:** When the child is intended to preserve and extend or wrap the parent's implementation.

**Explanation:** An override can replace behavior completely when the contract permits it; parent:: is not required automatically.

### 21. What must a child do when it declares its own constructor but parent initialization is required?

**Answer:** Call parent::__construct() explicitly with the needed arguments.

**Explanation:** PHP does not implicitly call the parent constructor when the child defines its own constructor.

### 22. What can go wrong if the parent constructor is not called?

**Answer:** Required parent state may remain uninitialized and parent validation or setup may be skipped.

**Explanation:** A child should call the parent constructor when parent initialization forms part of a valid object.

### 23. Why must an override not reduce a public method to protected?

**Answer:** Existing callers that rely on the public method would lose access when given the child object.

**Explanation:** An override must preserve the parent's access contract. It may widen visibility but not narrow it.

### 24. Can a child use a parent's private method as an extension hook?

**Answer:** No.

**Explanation:** Private methods are accessible only within the class that defines them. Use a protected or public method when subclass customization is intended.

### 25. Explain covariance in one sentence.

**Answer:** Covariance allows an overriding child method to return a more specific type than its parent method's return type.

**Explanation:** The child still fulfills the parent's promise, because the returned child type is a subtype of the parent return type.

### 26. Explain contravariance in one sentence.

**Answer:** Contravariance allows an overriding child method to accept a broader parameter type than the parent method's parameter type.

**Explanation:** The child can accept every value callers were allowed to pass to the parent, plus possibly more.

### 27. Why is changing an override's parameter from AnimalFood to Food potentially valid when Food is the parent type of AnimalFood?

**Answer:** It broadens the accepted input type.

**Explanation:** The child still accepts all AnimalFood arguments allowed by the parent method, so it does not reject valid parent calls.

### 28. What is the purpose of final on a class?

**Answer:** It prevents any other class from extending that class.

**Explanation:** Use it when subclassing would break required behavior or when extension is intentionally not supported.

### 29. What is the difference between inheritance and composition?

**Answer:** Inheritance says one class is a specialized kind of another; composition says an object contains or collaborates with another object.

**Explanation:** Use composition when a class needs another object's behavior without claiming to be that kind of object.

### 30. Why can deep inheritance hierarchies be difficult to maintain?

**Answer:** Behavior and assumptions are spread across many parent and child classes, making changes harder to predict.

**Explanation:** A change in an ancestor can affect descendants in unexpected ways. Prefer small, intentional hierarchies.

## Scenarios and code review

### 31. Does this child inherit the public greeting method?

~~~php
class Person
{
    public function greeting(): string
    {
        return 'Hello';
    }
}

class Student extends Person
{
}
~~~

**Answer:** Yes.

**Explanation:** Student inherits the parent's public method because it does not override it.

### 32. Can this child read the parent's private identifier?

~~~php
class Record
{
    private string $id = 'R-1';
}

class UserRecord extends Record
{
    public function id(): string
    {
        return $this->id;
    }
}
~~~

**Answer:** No.

**Explanation:** The id property is private to Record. UserRecord cannot directly read it; Record would need to expose an appropriate method.

### 33. Which implementation runs for the second call?

~~~php
class Formatter
{
    public function label(): string
    {
        return 'Base';
    }
}

class HtmlFormatter extends Formatter
{
    public function label(): string
    {
        return 'HTML';
    }
}

function printLabel(Formatter $formatter): void
{
    echo $formatter->label();
}

printLabel(new Formatter());
printLabel(new HtmlFormatter());
~~~

**Answer:** The first call prints Base; the second prints HTML.

**Explanation:** The parameter accepts Formatter, but method dispatch uses the runtime object class and selects HtmlFormatter's override for the second call.

### 34. What does this method return?

~~~php
class BaseName
{
    public function name(): string
    {
        return 'Base';
    }
}

class CustomName extends BaseName
{
    public function name(): string
    {
        return parent::name() . ' + Custom';
    }
}

echo (new CustomName())->name();
~~~

**Answer:** Base + Custom.

**Explanation:** The override calls the parent's implementation and appends its own text.

### 35. Will this child constructor run the parent's initialization?

~~~php
class BaseTask
{
    protected string $id;

    public function __construct(string $id)
    {
        $this->id = $id;
    }
}

class EmailTask extends BaseTask
{
    public function __construct(string $id)
    {
        // No parent constructor call.
    }
}
~~~

**Answer:** No.

**Explanation:** A child-defined constructor does not automatically invoke the parent's constructor. Call parent::__construct($id) when the parent state is required.

### 36. Is this override compatible?

~~~php
class Page
{
    public function title(): string
    {
        return 'Page';
    }
}

class HomePage extends Page
{
    protected function title(): string
    {
        return 'Home';
    }
}
~~~

**Answer:** No.

**Explanation:** The child reduces the visibility from public to protected, so existing callers of Page::title() would not have the same access to HomePage.

### 37. Is this return-type override covariant?

~~~php
class Animal
{
}

class Dog extends Animal
{
}

class Shelter
{
    public function adopt(): Animal
    {
        return new Animal();
    }
}

class DogShelter extends Shelter
{
    public function adopt(): Dog
    {
        return new Dog();
    }
}
~~~

**Answer:** Yes, in PHP 7.4 and later.

**Explanation:** Dog is more specific than Animal and satisfies the parent's return promise.

### 38. Is this parameter change contravariant?

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

**Answer:** Yes, in PHP 7.4 and later.

**Explanation:** Food is broader than AnimalFood, so the child accepts all values the parent method allowed and more.

### 39. What happens when this final method is overridden?

~~~php
class BaseService
{
    final public function version(): string
    {
        return '1';
    }
}

class CustomService extends BaseService
{
    public function version(): string
    {
        return '2';
    }
}
~~~

**Answer:** PHP rejects the class because a final method cannot be overridden.

**Explanation:** final is an explicit restriction on child customization.

### 40. A Report class needs to format text using a Formatter object. Should Report extend Formatter?

**Answer:** Usually no. Report should contain or receive a Formatter and delegate formatting to it.

**Explanation:** A report is not necessarily a kind of formatter. This relationship is collaboration, so composition is a clearer model.

## Coding and applied questions

### 41. Create a parent class with a public method, then a child that inherits it and adds one method.

**Answer:**

~~~php
class Vehicle
{
    public function category(): string
    {
        return 'vehicle';
    }
}

class Bicycle extends Vehicle
{
    public function ringBell(): string
    {
        return 'Ring!';
    }
}

$bicycle = new Bicycle();
echo $bicycle->category() . PHP_EOL;
echo $bicycle->ringBell();
~~~

**Explanation:** Bicycle inherits category() and adds ringBell(). Both are public operations available on a Bicycle object.

### 42. Override a parent method and include the parent's result in the new result.

**Answer:**

~~~php
class Message
{
    public function text(): string
    {
        return 'Message';
    }
}

class WarningMessage extends Message
{
    public function text(): string
    {
        return parent::text() . ': please review';
    }
}

echo (new WarningMessage())->text();
~~~

**Explanation:** The child calls the parent implementation and extends its result.

### 43. Write a child constructor that initializes both parent and child state.

**Answer:**

~~~php
class Job
{
    public function __construct(
        protected string $id,
    ) {
    }

    public function id(): string
    {
        return $this->id;
    }
}

class EmailJob extends Job
{
    public function __construct(
        string $id,
        private string $recipient,
    ) {
        parent::__construct($id);
    }

    public function recipient(): string
    {
        return $this->recipient;
    }
}

$job = new EmailJob('job-1', 'learner@example.test');
echo $job->id() . ' ' . $job->recipient();
~~~

**Explanation:** The child explicitly passes the parent state to parent::__construct() and initializes its own promoted recipient property.

### 44. Write a parent method that returns Animal and a child override that returns Dog.

**Answer:**

~~~php
class Animal
{
}

class Dog extends Animal
{
}

class Shelter
{
    public function adopt(): Animal
    {
        return new Animal();
    }
}

class DogShelter extends Shelter
{
    public function adopt(): Dog
    {
        return new Dog();
    }
}
~~~

**Explanation:** This is a covariant return type. The child returns a more specific type that remains compatible with the parent contract.

### 45. Refactor this design so a Report uses a Formatter through composition instead of extending it.

~~~php
class Formatter
{
    public function format(string $text): string
    {
        return strtoupper($text);
    }
}

class Report extends Formatter
{
}
~~~

**Answer:**

~~~php
class Formatter
{
    public function format(string $text): string
    {
        return strtoupper($text);
    }
}

class Report
{
    public function __construct(
        private Formatter $formatter,
    ) {
    }

    public function render(string $text): string
    {
        return $this->formatter->format($text);
    }
}

$report = new Report(new Formatter());
echo $report->render('monthly results'); // MONTHLY RESULTS
~~~

**Explanation:** Report is not a kind of Formatter. It collaborates with a Formatter and delegates the formatting operation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [066 - Static properties and methods](../lessons/066-static-properties-and-methods.md) | [Lesson](../lessons/067-inheritance-and-method-overriding.md) | [Example](../examples/067-inheritance-and-method-overriding.md) | [068 - Composition and object collaboration](../lessons/068-composition-and-object-collaboration.md) |
