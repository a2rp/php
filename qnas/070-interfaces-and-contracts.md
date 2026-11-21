# Interfaces and contracts - Q&A

These questions review how PHP interfaces define behavior, how classes implement them, and how interface types help keep code flexible. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. What does a PHP interface primarily define?

A. A reusable method implementation
B. A public behavior contract
C. A database connection
D. An object's constructor arguments

**Answer: B.** An interface names public operations that implementing classes promise to provide. The interface does not supply the method bodies.

### 2. Which keyword connects a class to an interface?

A. <code>extends</code>
B. <code>uses</code>
C. <code>implements</code>
D. <code>inherits</code>

**Answer: C.** A class declares the interface contracts it follows with <code>implements</code>.

### 3. What happens if a concrete class omits a required interface method?

A. PHP creates an empty method automatically
B. PHP ignores the missing method
C. PHP rejects the incomplete concrete class
D. The interface supplies a default method body

**Answer: C.** Every required interface method must be implemented with a compatible signature, unless the class is declared abstract.

### 4. What visibility must interface methods have?

A. Private
B. Protected
C. Public
D. Any visibility

**Answer: C.** Interface methods are public contracts, and implementing methods cannot reduce that visibility.

### 5. What is a benefit of accepting an interface as a parameter type?

A. The function can use only one concrete class
B. The function can accept any object that implements the contract
C. The function can instantiate the interface
D. PHP skips type checking

**Answer: B.** The function depends on the required behavior rather than one specific implementation.

### 6. How many interfaces can a class implement?

A. One
B. Two
C. Any number
D. None

**Answer: C.** A class can implement multiple interfaces by listing them after <code>implements</code> and separating them with commas.

### 7. How many parent classes can a PHP class extend?

A. One
B. Two
C. Any number
D. It cannot extend a class

**Answer: A.** PHP supports one parent class, but a class can implement multiple interfaces.

### 8. Can an interface extend another interface?

A. No
B. Yes, and it can extend multiple interfaces
C. Only if the parent is a class
D. Only in PHP 8.4+

**Answer: B.** An interface can extend one or more interfaces and combines their method requirements.

### 9. Which PHP version added interface property requirements?

A. PHP 7.4
B. PHP 8.0
C. PHP 8.1
D. PHP 8.4

**Answer: D.** PHP 8.4 introduced interface properties using property hooks. Methods are the portable option for earlier PHP versions.

### 10. What is a reason to avoid declaring constructors in an interface?

A. PHP cannot parse constructors in interfaces
B. It forces implementations into a common construction shape and reduces flexibility
C. Constructors cannot have parameters
D. Interfaces cannot be used with objects

**Answer: B.** PHP supports interface constructors, but requiring the same constructor across implementations can constrain their setup unnecessarily.

### 11. What is true about interface constants?

A. They are instance properties
B. Interfaces can declare constants; inheritance behavior changed in PHP 8.1
C. They can be reassigned after object creation
D. They can contain closures

**Answer: B.** Interface constants work like class constants. Overriding behavior differs before and from PHP 8.1.

### 12. What should implementing classes generally do with interface parameter names?

A. Rename them to describe local preferences
B. Keep the same names because PHP 8+ supports named arguments
C. Remove all parameter names
D. Use names that match the class name

**Answer: B.** Named-argument callers can use the parameter names declared by the interface, so implementations should preserve them.

## Short-answer questions

### 13. What is an interface contract?

**Answer:** It is a promise that an implementing class provides specified public behavior.

**Explanation:** A caller can rely on the methods in the contract without depending on the object's internal implementation.

### 14. Does an interface provide a method body?

**Answer:** No.

**Explanation:** An interface declares method signatures. Use an abstract class or trait when shared method implementations are needed.

### 15. Why might two unrelated classes implement the same interface?

**Answer:** They may provide the same capability in different ways.

**Explanation:** An email sender and an SMS sender can both implement a notification-sending contract even though one is not a subtype of the other.

### 16. Why type a service against an interface instead of a concrete class?

**Answer:** It allows the service to accept any implementation that meets the needed contract.

**Explanation:** This reduces coupling and lets the composition root choose an implementation without changing the service.

### 17. Can an interface itself be instantiated?

**Answer:** No.

**Explanation:** An interface is a contract, not a concrete implementation. Instantiate a class that implements it.

### 18. What does a class promise when it writes <code>implements NotificationSender</code>?

**Answer:** It promises to provide all required public methods with compatible signatures.

**Explanation:** PHP checks this contract when the class is declared.

### 19. Can an interface declare a protected method?

**Answer:** No.

**Explanation:** Interface methods must be public because they define behavior that callers may use through the interface type.

### 20. Can an abstract class implement only part of an interface?

**Answer:** Yes, if the abstract class itself remains abstract.

**Explanation:** A later concrete subclass must implement the remaining interface methods before it can be instantiated.

### 21. What is interface segregation?

**Answer:** It is the practice of keeping interfaces focused on small, related capabilities.

**Explanation:** A class should not be forced to promise unrelated behavior that its clients do not need.

### 22. What is the difference between an interface and a trait?

**Answer:** An interface declares a contract; a trait provides reusable method or property implementation.

**Explanation:** A class can implement an interface and use a trait at the same time. A trait does not make the class an instance of an interface.

### 23. Why is a constructor contract often better handled separately?

**Answer:** Different implementations may need different setup arguments even if they provide the same behavior after creation.

**Explanation:** A factory or composition root can create the implementation, while the behavior interface stays focused on what clients call.

### 24. What is the purpose of <code>instanceof</code> with an interface?

**Answer:** It checks whether an object implements that interface.

**Explanation:** This can be useful for runtime branching. When a function always requires the capability, prefer declaring the interface as its parameter type.

### 25. Are interface properties supported on every PHP version?

**Answer:** No. Interface property contracts were added in PHP 8.4.

**Explanation:** For earlier versions, represent required access with public methods such as <code>label(): string</code>.

## Code-reading and design questions

### 26. What does this program print?

~~~php
<?php
interface Greeter
{
    public function greet(string $name): string;
}

class FriendlyGreeter implements Greeter
{
    public function greet(string $name): string
    {
        return 'Hello, ' . $name;
    }
}

function welcome(Greeter $greeter): void
{
    echo $greeter->greet('Sam');
}

welcome(new FriendlyGreeter());
~~~

**Answer:** <code>Hello, Sam</code>

**Explanation:** The function accepts the interface type. FriendlyGreeter implements it and provides the method called by the function.

### 27. What is missing here?

~~~php
interface CacheStore
{
    public function get(string $key): ?string;
    public function put(string $key, string $value): void;
}

class MemoryCache implements CacheStore
{
    public function get(string $key): ?string
    {
        return null;
    }
}
~~~

**Answer:** MemoryCache is missing the public <code>put()</code> implementation.

**Explanation:** A concrete class must implement every method required by the interface.

### 28. Is this implementation compatible?

~~~php
interface NameProvider
{
    public function name(): string;
}

class UserNameProvider implements NameProvider
{
    protected function name(): string
    {
        return 'Asha';
    }
}
~~~

**Answer:** No.

**Explanation:** The interface method is public. The implementation cannot narrow its visibility to protected.

### 29. Why is this service more flexible than one typed specifically to EmailSender?

~~~php
class AlertService
{
    public function __construct(
        private NotificationSender $sender
    ) {
    }
}
~~~

**Answer:** It can accept any object implementing NotificationSender.

**Explanation:** The service relies only on the sending contract. EmailSender, SmsSender, or another valid implementation can be injected.

### 30. What is a problem with this interface?

~~~php
interface EverythingService
{
    public function sendEmail(string $address): void;
    public function saveReport(string $path): void;
    public function calculateTax(float $amount): float;
    public function drawChart(array $data): string;
}
~~~

**Answer:** It combines unrelated capabilities into one oversized contract.

**Explanation:** Split it into focused interfaces so a class only implements the behavior it actually supports.

### 31. What does the interface extension below require from a concrete implementation?

~~~php
interface Reader
{
    public function read(): string;
}

interface ClosableReader extends Reader
{
    public function close(): void;
}
~~~

**Answer:** A concrete class implementing ClosableReader must implement both <code>read()</code> and <code>close()</code>.

**Explanation:** The child interface inherits the methods of Reader and adds its own method.

### 32. If a class implements Readable and Writable, can both interfaces declare the same method name?

**Answer:** Yes, if one implementation can satisfy both compatible signatures.

**Explanation:** PHP checks compatibility across all implemented contracts. Conflicting signatures may make the class declaration invalid.

### 33. Why should method parameter names be kept aligned between an interface and its implementation?

**Answer:** PHP 8+ named arguments allow callers to use parameter names from the interface declaration.

**Explanation:** Changing only the implementation's parameter name can break a call such as <code>send(message: 'Hi')</code> even when positional calls still work.

### 34. What is the contract in this PHP 8.4+ interface?

~~~php
interface HasLabel
{
    public string $label { get; }
}
~~~

**Answer:** Implementing objects must expose a publicly readable string property named <code>label</code>.

**Explanation:** The contract requires public reading, but does not require the property to be publicly writable. This syntax requires PHP 8.4 or later.

### 35. Why might a class implement two small interfaces instead of one large interface?

**Answer:** It can promise only the capabilities it actually supports.

**Explanation:** Smaller contracts reduce unnecessary coupling and make implementations easier to reuse.

## Coding questions

### 36. Declare a notification sender contract and implement it for email.

**Answer:**

~~~php
interface NotificationSender
{
    public function send(string $recipient, string $message): bool;
}

class EmailSender implements NotificationSender
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Email to ' . $recipient . ': ' . $message;
        return true;
    }
}
~~~

**Explanation:** The interface defines the public method shape. EmailSender supplies the actual behavior.

### 37. Write a function that accepts any NotificationSender.

**Answer:**

~~~php
function notify(
    NotificationSender $sender,
    string $recipient,
    string $message
): bool {
    return $sender->send($recipient, $message);
}
~~~

**Explanation:** The function accepts implementations by contract, so it can work with different sender classes.

### 38. Make a service use constructor injection with an interface type.

**Answer:**

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
~~~

**Explanation:** The service receives a required capability and does not select or construct a specific sender itself.

### 39. Implement two interfaces on one class.

**Answer:**

~~~php
interface Sender
{
    public function send(string $message): bool;
}

interface HealthCheck
{
    public function isAvailable(): bool;
}

class QueueSender implements Sender, HealthCheck
{
    public function send(string $message): bool
    {
        return true;
    }

    public function isAvailable(): bool
    {
        return true;
    }
}
~~~

**Explanation:** QueueSender promises both capabilities and implements every method required by both interfaces.

### 40. Create an interface contract for a value that can provide a label, compatible with PHP before 8.4.

**Answer:**

~~~php
interface HasLabel
{
    public function label(): string;
}

class Product implements HasLabel
{
    public function label(): string
    {
        return 'Product';
    }
}
~~~

**Explanation:** A method contract works on older PHP versions and avoids PHP 8.4 interface property syntax.

## Key takeaways

- Interfaces define public behavior contracts without supplying method bodies.
- Classes use <code>implements</code> and must provide every required method.
- Interface type declarations allow different implementations to be substituted.
- Classes can implement many interfaces; interfaces can extend many interfaces.
- Keep contracts small and focused, and preserve parameter names for named-argument callers.
- PHP 8.4 added interface property contracts; methods are more portable.
- Interface constants' inheritance behavior changed in PHP 8.1.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [069 - Abstract classes and final declarations](../lessons/069-abstract-classes-and-final-declarations.md) | [Lesson](../lessons/070-interfaces-and-contracts.md) | [Example](../examples/070-interfaces-and-contracts.md) | [071 - Traits and code reuse](../lessons/071-traits-and-code-reuse.md) |
