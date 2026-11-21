# Traits and code reuse - Q&A

These questions review how traits provide code reuse, how PHP combines traits with classes, and how to handle conflicts and version-specific behavior. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. What is the main purpose of a PHP trait?

A. Create objects without classes
B. Reuse a focused set of methods across classes
C. Declare a database table
D. Replace every interface

**Answer: B.** Traits package behavior that multiple classes can reuse without requiring those classes to share a parent class.

### 2. Can a trait be instantiated directly?

A. Yes, using <code>new</code>
B. Yes, if it has a constructor
C. No, a class must use it
D. Only if it is final

**Answer: C.** A trait is not a class or object. Include it in a class and instantiate that class.

### 3. Where does a class include a trait?

A. After the class name with <code>extends</code>
B. Inside the class body with <code>use</code>
C. In the constructor with <code>new</code>
D. In an interface declaration

**Answer: B.** The trait's members are made available to the class by writing <code>use TraitName;</code> inside the class body.

### 4. How can a class use two traits?

A. It cannot
B. Use both names in one <code>use</code> statement, separated by commas
C. Extend both traits
D. Implement both traits

**Answer: B.** A class can include multiple traits with a comma-separated <code>use</code> statement.

### 5. Two traits provide the same method name. What must the class do?

A. PHP always chooses the first trait
B. PHP always chooses the last trait
C. Resolve the conflict explicitly
D. Change both traits to abstract

**Answer: C.** Unresolved method conflicts cause an error. Use <code>insteadof</code> to choose the implementation.

### 6. What does <code>insteadof</code> do in trait adaptation?

A. Renames a trait
B. Selects which conflicting method implementation is used for a name
C. Removes the class's constructor
D. Makes a method static

**Answer: B.** It chooses the method that wins the conflict and excludes another trait's method from that name.

### 7. What does the <code>as</code> adaptation operator do?

A. Creates an alias or changes the imported method's visibility
B. Changes the class's parent
C. Copies a trait into a file
D. Implements an interface

**Answer: A.** The alias can provide another name for a method; the syntax can also adjust visibility.

### 8. What is the method precedence when a class, trait, and parent class define the same method?

A. Parent, then trait, then class
B. Trait, then parent, then class
C. Class, then trait, then inherited parent
D. PHP reports an error in all cases

**Answer: C.** A method declared directly in the class wins over a trait method, and a trait method wins over an inherited parent method.

### 9. Can one trait include another trait?

A. No
B. Yes
C. Only if both traits are final
D. Only in PHP 8.4+

**Answer: B.** Traits can be composed from other traits using <code>use</code> inside the trait body.

### 10. What can an abstract method in a trait do?

A. Require the using class to provide a compatible method
B. Prevent a class from using the trait
C. Instantiate the trait
D. Create a default method body

**Answer: A.** An abstract trait method declares a requirement that the class using the trait must satisfy.

### 11. Which PHP version added trait constants?

A. PHP 7.4
B. PHP 8.0
C. PHP 8.2
D. PHP 8.4

**Answer: C.** Traits can define constants starting with PHP 8.2.

### 12. Which PHP version added final adaptation for imported trait methods?

A. PHP 8.0
B. PHP 8.1
C. PHP 8.3
D. PHP 8.5

**Answer: C.** PHP 8.3 added syntax to mark a trait method as final when importing it into a class.

## Short-answer questions

### 13. How is a trait different from a parent class?

**Answer:** A trait provides members that a class can include without becoming a subtype of another class.

**Explanation:** Traits provide horizontal code reuse. Inheritance expresses an "is-a" relationship and allows only one parent class.

### 14. Why can unrelated classes use the same trait?

**Answer:** Trait use does not require the classes to share a parent or type relationship.

**Explanation:** The classes receive the trait's behavior, but the trait itself does not make them subtypes of one another.

### 15. What does a trait dependency on <code>$this->name</code> mean?

**Answer:** The trait expects the using class to provide a compatible <code>name</code> property.

**Explanation:** That assumption can be hard to discover. Make it explicit, or require behavior through an abstract method in the trait.

### 16. What does a trait abstract method require?

**Answer:** It requires the using class to provide a compatible method implementation.

**Explanation:** This lets a trait call class-specific behavior while making the requirement visible to PHP.

### 17. Can an abstract method inside a trait be private?

**Answer:** Yes, on PHP 8.0 and later. Older versions supported only public and protected abstract trait methods.

**Explanation:** The visibility requirement depends on the PHP version. Use public or protected when supporting older runtimes.

### 18. Does an alias created with <code>as</code> necessarily remove the original method name?

**Answer:** No. The alias adds another name and does not by itself rename the original method.

**Explanation:** If <code>insteadof</code> also excludes that trait's conflicting method from the original name, the alias can preserve a way to call its implementation.

### 19. How can a trait method be made protected when imported?

**Answer:** Use the <code>as</code> adaptation with the visibility, such as <code>methodName as protected</code>.

**Explanation:** This changes how the method is exposed by the using class without editing the trait source.

### 20. Can the class using a trait override a method supplied by that trait?

**Answer:** Yes.

**Explanation:** Methods declared directly in the class take precedence over imported trait methods.

### 21. Can a trait override a method inherited from the class's parent?

**Answer:** Yes.

**Explanation:** An imported trait method takes precedence over an inherited parent method. A method directly declared in the class takes precedence over both.

### 22. Why can trait properties cause confusing behavior?

**Answer:** They add state to each class using the trait and can collide with properties declared by the class or other traits.

**Explanation:** Keep trait state limited and consistently named. Avoid relying on version-specific behavior when the trait and parent class declare the same property.

### 23. How should a static method defined in a trait be called?

**Answer:** Call it through a class that uses the trait.

**Explanation:** Directly calling static methods or accessing static properties through the trait name has been deprecated since PHP 8.1.

### 24. When were trait constants introduced?

**Answer:** PHP 8.2.

**Explanation:** On earlier versions, put the constant on the consuming class or use another compatible design.

### 25. What changed about static properties from traits in PHP 8.3?

**Answer:** When a child class uses a trait that its parent also uses, the child's trait static property is treated as distinct from the parent's.

**Explanation:** Code that relied on sharing that static property across the hierarchy may behave differently. Avoid using trait static properties as implicit shared mutable state.

### 26. What changed in trait precedence for properties and constants in PHP 8.5?

**Answer:** Traits are bound to a class before its parent class, changing how same-named trait properties or constants compare with inherited ones.

**Explanation:** Method precedence is unaffected. Avoid same-name property or constant collisions instead of relying on version-dependent composition rules.

### 27. Does using a trait make a class implement an interface?

**Answer:** No, not by itself.

**Explanation:** A trait supplies code, but the class must still declare <code>implements</code> to promise an interface contract. The trait can provide some required methods.

## Code-reading and design questions

### 28. What does this code print?

~~~php
trait CanGreet
{
    public function greet(): string
    {
        return 'Hello';
    }
}

class Member
{
    use CanGreet;
}

echo (new Member())->greet();
~~~

**Answer:** <code>Hello</code>

**Explanation:** The class includes CanGreet, so greet() is available as a Member method.

### 29. What happens when a trait is instantiated directly?

~~~php
trait CanGreet
{
    public function greet(): string
    {
        return 'Hello';
    }
}

$greeting = new CanGreet();
~~~

**Answer:** PHP reports an error.

**Explanation:** A trait is not instantiable. Create a class that uses CanGreet and instantiate that class.

### 30. Which implementation does this class use for <code>greet()</code>?

~~~php
trait Friendly
{
    public function greet(): string
    {
        return 'Hi';
    }
}

trait Formal
{
    public function greet(): string
    {
        return 'Good day';
    }
}

class Greeter
{
    use Friendly, Formal {
        Formal::greet insteadof Friendly;
    }
}
~~~

**Answer:** Formal::greet(), which returns <code>Good day</code>.

**Explanation:** The <code>insteadof</code> rule explicitly selects Formal's conflicting method.

### 31. What does the alias add here?

~~~php
class Greeter
{
    use Friendly, Formal {
        Formal::greet insteadof Friendly;
        Friendly::greet as casualGreet;
    }
}
~~~

**Answer:** It makes Friendly's implementation callable through <code>casualGreet()</code>.

**Explanation:** Formal wins the unqualified greet() conflict. The alias preserves access to Friendly's different implementation under another name.

### 32. Which method is called?

~~~php
class Base
{
    public function label(): string
    {
        return 'Base';
    }
}

trait HasLabel
{
    public function label(): string
    {
        return 'Trait';
    }
}

class Product extends Base
{
    use HasLabel;
}

class SpecialProduct extends Product
{
    public function label(): string
    {
        return 'Class';
    }
}
~~~

**Answer:** Product returns <code>Trait</code>; SpecialProduct returns <code>Class</code>.

**Explanation:** A trait method takes precedence over the inherited parent method, and a method declared directly in a class takes precedence over the trait method.

### 33. What is the requirement of this trait?

~~~php
trait HasTitle
{
    abstract protected function titleText(): string;

    public function title(): string
    {
        return strtoupper($this->titleText());
    }
}
~~~

**Answer:** The using class must provide a compatible protected <code>titleText(): string</code> method.

**Explanation:** The trait's title() method calls that required method to get class-specific text.

### 34. Why can this trait be fragile?

~~~php
trait NameDisplay
{
    public function displayName(): string
    {
        return trim($this->name);
    }
}
~~~

**Answer:** It silently assumes every using class has a <code>name</code> property with a suitable value.

**Explanation:** The trait's dependency is not expressed in its declaration. An abstract method such as <code>name(): string</code> can make the requirement explicit.

### 35. What is the risk in using a trait property that has the same name as one in the class?

**Answer:** The trait and class may have incompatible property definitions, causing composition errors or version-dependent behavior.

**Explanation:** Use distinct names or remove duplicated state. Do not rely on the PHP 8.5 trait-parent precedence change to resolve a design collision.

## Coding questions

### 36. Define a trait that provides a reusable timestamp method and use it in a class.

**Answer:**

~~~php
trait HasTimestamp
{
    public function timestamp(): string
    {
        return date('Y-m-d H:i:s');
    }
}

class Event
{
    use HasTimestamp;
}

$event = new Event();
echo $event->timestamp();
~~~

**Explanation:** The trait method becomes available on Event. The trait itself is not instantiated.

### 37. Use two traits in one class.

**Answer:**

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

class Article
{
    use CanPublish, CanArchive;
}
~~~

**Explanation:** Both focused behaviors are included in Article without requiring a shared parent class.

### 38. Resolve a collision and retain both implementations under separate names.

**Answer:**

~~~php
class GreetingCard
{
    use Friendly, Formal {
        Formal::greet insteadof Friendly;
        Friendly::greet as casualGreet;
    }
}
~~~

**Explanation:** <code>greet()</code> uses Formal. The Friendly implementation is accessible through the alias <code>casualGreet()</code>.

### 39. Add a trait requirement for the using class.

**Answer:**

~~~php
trait HasGreeting
{
    abstract protected function greetingName(): string;

    public function greeting(): string
    {
        return 'Hello, ' . $this->greetingName();
    }
}

class Customer
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
~~~

**Explanation:** The abstract method documents and enforces the behavior the trait needs from Customer.

### 40. On PHP 8.2+, define and access a trait constant through a class.

**Answer:**

~~~php
trait JsonDefaults
{
    public const CONTENT_TYPE = 'application/json';
}

class JsonResponse
{
    use JsonDefaults;
}

echo JsonResponse::CONTENT_TYPE;
~~~

**Explanation:** Trait constants are available starting in PHP 8.2 and are accessed through the class using the trait.

## Key takeaways

- Traits reuse implementation without creating an inheritance relationship.
- Include traits inside a class; traits cannot be instantiated.
- Use <code>insteadof</code> to choose between conflicting methods and <code>as</code> to add an alias or change visibility.
- Class methods override trait methods; trait methods override inherited parent methods.
- Abstract trait methods make requirements on using classes explicit.
- Trait constants require PHP 8.2; final method adaptation requires PHP 8.3.
- Be cautious with static trait properties and member-name collisions across PHP versions.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [070 - Interfaces and contracts](../lessons/070-interfaces-and-contracts.md) | [Lesson](../lessons/071-traits-and-code-reuse.md) | [Example](../examples/071-traits-and-code-reuse.md) | [072 - Class constants and constant expressions](../lessons/072-class-constants-and-constant-expressions.md) |
