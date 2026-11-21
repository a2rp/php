# Abstract classes and final declarations - Q&A

Use these questions to review abstract classes, required subclass behavior, and restrictions created by final declarations. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. Can an abstract class be instantiated directly?

A. Yes, if all its methods are public
B. Yes, if it has a constructor
C. No, instantiate a concrete child instead
D. Only when it has no properties

**Answer: C.** An abstract class is intentionally incomplete. A concrete child must implement all inherited abstract methods before it can be instantiated.

### 2. What does an abstract method declaration provide?

A. A method body that every child inherits unchanged
B. A required method signature without an implementation
C. A private helper that children cannot see
D. A constant value for all child classes

**Answer: B.** An abstract method states which operation a concrete subclass must provide, including its compatible signature.

### 3. What must be true of a class containing an abstract method?

A. It must be final
B. It must be declared abstract
C. It must have a private constructor
D. It must implement an interface

**Answer: B.** Any class containing an abstract method must itself be abstract.

### 4. Which visibility can an abstract method have?

A. Private only
B. Public or protected
C. Private or protected
D. Static only

**Answer: B.** A child needs access to implement an abstract method, so it cannot be private.

### 5. Which statement about an abstract class is correct?

A. It can contain only abstract methods
B. It can contain shared properties and implemented methods
C. It cannot have a constructor
D. It cannot be extended

**Answer: B.** Abstract classes can provide state and common behavior alongside required abstract operations.

### 6. What does a final class prevent?

A. Instantiating the class
B. Calling its methods
C. Extending the class
D. Declaring properties

**Answer: C.** A final class can be instantiated normally, but no class may extend it.

### 7. What does a final method prevent?

A. Calling the method from a child
B. Overriding the method in a child class
C. Defining the method in the parent class
D. Returning a value

**Answer: B.** A child inherits and can call a final method, but it cannot replace its implementation.

### 8. Which feature arrived in PHP 8.1?

A. Typed properties
B. Constructor property promotion
C. Final class constants
D. Final properties

**Answer: C.** Final class constants were introduced in PHP 8.1. Typed properties arrived in 7.4, constructor property promotion in 8.0, and final properties in 8.4.

### 9. Which features require PHP 8.4 or later?

A. Abstract properties with property hooks and final properties
B. Abstract classes and final classes
C. Abstract methods and final methods
D. Class constants and constructors

**Answer: A.** PHP 8.4 added abstract properties using property hooks and final properties.

### 10. Can a class be both abstract and final?

A. Yes, if it has a public constructor
B. Yes, if all methods are static
C. No, the constraints conflict
D. Only in PHP 8.4+

**Answer: C.** Abstract requires a subclass to complete the class; final prohibits subclassing.

### 11. Can one method be both abstract and final?

A. Yes, if it has no return type
B. Yes, if it is protected
C. No, one requires overriding and the other prohibits it
D. Only when it is static

**Answer: C.** A concrete child must implement an abstract method, but a final method cannot be overridden.

### 12. How many parent classes can a PHP class extend?

A. One
B. Two
C. Any number
D. None

**Answer: A.** A PHP class extends one class. It can implement multiple interfaces.

## Short-answer questions

### 13. Why would you create an abstract class?

**Answer:** To share state or implemented behavior among related subclasses while requiring each concrete subclass to provide selected operations.

**Explanation:** It is useful when the classes share a meaningful base, not merely because some code happens to look similar.

### 14. What makes a child class concrete?

**Answer:** It implements all inherited abstract methods with compatible signatures.

**Explanation:** Until the abstract requirements are satisfied, PHP will not allow the class to be instantiated.

### 15. Can an abstract class contain a constructor and ordinary methods?

**Answer:** Yes.

**Explanation:** Abstract describes whether the class may be instantiated and whether it contains incomplete behavior. It does not prevent the class from initializing shared state or implementing common methods.

### 16. Why can't an abstract method be private?

**Answer:** A child must be able to implement it, but a private method is inaccessible to child classes.

**Explanation:** Use protected when the operation is an implementation detail for subclasses, or public when callers also need it.

### 17. Can a child widen a protected abstract method to public?

**Answer:** Yes, if the rest of the method signature remains compatible.

**Explanation:** Widening visibility makes the operation more accessible. Narrowing a public contract to protected would break callers.

### 18. What is a template method?

**Answer:** It is a high-level method that defines the order of an algorithm while calling overridable or abstract steps for subclass-specific behavior.

**Explanation:** A base importer can keep its import sequence fixed while child classes decide how to read and normalize records.

### 19. Why might a template method be declared final?

**Answer:** To preserve the workflow order and prevent subclasses from bypassing required steps.

**Explanation:** The base class can expose deliberate extension points as protected abstract methods while keeping the public algorithm stable.

### 20. When should a class be declared final?

**Answer:** When subclassing would break an invariant or is deliberately not part of the class's extension contract.

**Explanation:** Final removes a future customization option, so it should be a design decision rather than a default added everywhere.

### 21. When should a method be declared final?

**Answer:** When child classes must not replace a behavior that callers or the base class rely on.

**Explanation:** Keep methods that are intended for variation non-final and make the supported extension points explicit.

### 22. Does a final class have to be static or abstract?

**Answer:** No. It can be instantiated and used like an ordinary class.

**Explanation:** Final restricts inheritance; it does not prevent object creation or method calls.

### 23. What is the difference between a readonly property and a final property?

**Answer:** Readonly prevents reassignment of a property's value after initialization; final prevents a child class from redeclaring that property.

**Explanation:** Readonly concerns instance state. Final concerns inheritance. A final property is not automatically immutable.

### 24. What is the difference between an abstract property and an abstract getter method?

**Answer:** An abstract property, available with property hooks in PHP 8.4+, declares required accessor behavior. An abstract getter method declares a required method.

**Explanation:** The method form works on older supported PHP versions and is often the simpler choice when broad compatibility is needed.

### 25. How does an abstract class differ from an interface?

**Answer:** An abstract class can provide shared state and implemented behavior; an interface declares a contract that implementing classes agree to follow.

**Explanation:** A class extends only one class but can implement multiple interfaces. Use the relationship and reuse needs to decide; interfaces are detailed in the next chapter.

## Code-reading and design questions

### 26. What happens here?

~~~php
abstract class Shape
{
    abstract public function area(): float;
}

$shape = new Shape();
~~~

**Answer:** PHP reports an error because Shape is abstract and cannot be instantiated.

**Explanation:** Create a concrete subclass, such as Circle, that implements area(), then instantiate that subclass.

### 27. What must Circle implement?

~~~php
abstract class Shape
{
    abstract public function area(): float;
}

class Circle extends Shape
{
}
~~~

**Answer:** Circle must implement public function area(): float before it can be instantiated.

**Explanation:** It inherits an abstract method with a public contract. Leaving it unimplemented keeps Circle abstract in effect, so PHP rejects a concrete declaration that does not fulfill the requirement.

### 28. Is this a valid abstract method declaration?

~~~php
abstract class Parser
{
    abstract private function parse(string $input): array;
}
~~~

**Answer:** No.

**Explanation:** An abstract method cannot be private because child classes must implement it. Declare it protected or public.

### 29. What does this code print?

~~~php
abstract class Formatter
{
    abstract protected function body(string $text): string;

    public function format(string $text): string
    {
        return $this->body(trim($text));
    }
}

class UppercaseFormatter extends Formatter
{
    protected function body(string $text): string
    {
        return strtoupper($text);
    }
}

echo (new UppercaseFormatter())->format(' hello ');
~~~

**Answer:** HELLO

**Explanation:** The inherited format() method trims the input, then dispatches to the subclass implementation of body().

### 30. What is prohibited in this example?

~~~php
class BaseLogger
{
    final public function write(string $message): void
    {
        echo $message;
    }
}

class FileLogger extends BaseLogger
{
    public function write(string $message): void
    {
        // Save the message to a file.
    }
}
~~~

**Answer:** FileLogger cannot override write() because the parent method is final.

**Explanation:** The child may inherit and call write(), but PHP rejects a replacement method with that name.

### 31. Can this child class exist?

~~~php
final class TokenParser
{
}

class CustomTokenParser extends TokenParser
{
}
~~~

**Answer:** No.

**Explanation:** A final class cannot be extended. Instantiate TokenParser directly or redesign it if customization is a requirement.

### 32. Why is the import method final while readRecords() is abstract?

**Answer:** The base class wants to preserve the shared workflow but allow each importer to define how records are read.

**Explanation:** The final method fixes the sequence; the abstract method is the supported extension point.

### 33. What is the error in trying to declare an abstract final method?

**Answer:** The requirements contradict each other.

**Explanation:** Abstract requires a child implementation. Final forbids a child override. Choose one behavior for the method.

### 34. Should a final method be used for every method in an abstract class?

**Answer:** No.

**Explanation:** Only mark methods final when replacing them would violate a contract or invariant. Other methods can remain extension points.

## Coding questions

### 35. Create an abstract notification class with a required send operation.

**Answer:**

~~~php
abstract class Notification
{
    public function __construct(
        protected string $recipient
    ) {
    }

    abstract public function send(string $message): bool;

    public function recipient(): string
    {
        return $this->recipient;
    }
}
~~~

**Explanation:** The base stores shared recipient data and requires concrete notification types to implement send().

### 36. Implement the notification as an email message.

**Answer:**

~~~php
class EmailNotification extends Notification
{
    public function send(string $message): bool
    {
        echo 'Email to ' . $this->recipient . ': ' . $message . PHP_EOL;
        return true;
    }
}
~~~

**Explanation:** The child fulfills the public abstract method with the same parameter and return type.

### 37. Create a final class that cannot be subclassed.

**Answer:**

~~~php
final class SystemClock
{
    public function now(): DateTimeImmutable
    {
        return new DateTimeImmutable();
    }
}
~~~

**Explanation:** SystemClock remains instantiable, but PHP prevents another class from extending it.

### 38. Add a final class constant, assuming PHP 8.1 or later.

**Answer:**

~~~php
class ApiDefaults
{
    final public const VERSION = 'v1';
}
~~~

**Explanation:** A child class cannot redefine this constant. Final class constants require PHP 8.1 or later.

### 39. On PHP 8.4+, how can a base class require a readable property?

**Answer:**

~~~php
abstract class Labeled
{
    abstract public string $label {
        get;
    }
}

class ProductLabel extends Labeled
{
    public string $label = 'Product';
}
~~~

**Explanation:** The abstract property specifies the getter requirement. This property-hook syntax requires PHP 8.4 or later; an abstract getter method is a compatible alternative for older versions.

### 40. How can you keep an import workflow fixed while letting child classes customize reading?

**Answer:**

~~~php
abstract class Importer
{
    final public function import(string $source): array
    {
        return $this->read($source);
    }

    abstract protected function read(string $source): array;
}
~~~

**Explanation:** The final public method preserves the algorithm's entry point, while the protected abstract method defines the subclass extension point.

## Key takeaways

- Abstract classes cannot be instantiated; concrete children must complete their abstract methods.
- Abstract classes may contain shared state and implemented behavior.
- Abstract methods cannot be private and must be implemented compatibly.
- Final classes cannot be extended, and final methods cannot be overridden.
- Final constants require PHP 8.1; abstract and final properties require PHP 8.4.
- A template method can keep an algorithm stable while exposing deliberate extension points.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [068 - Composition and object collaboration](../lessons/068-composition-and-object-collaboration.md) | [Lesson](../lessons/069-abstract-classes-and-final-declarations.md) | [Example](../examples/069-abstract-classes-and-final-declarations.md) | [070 - Interfaces and contracts](../lessons/070-interfaces-and-contracts.md) |
