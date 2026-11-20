# 066 - Static Properties and Methods: Q&A

Practice class-level state, static calls, and late static binding. Every question has its answer and explanation immediately after it. See the [chapter lesson](../lessons/066-static-properties-and-methods.md) and [complete example](../examples/066-static-properties-and-methods.md).

## Multiple-choice questions

### 1. What does a static property belong to?

A. One object only
B. The class
C. The current function call only
D. The PHP file

**Answer:** B. The class.

**Explanation:** A static property is class-level state. It is not stored as a separate value on every object instance.

### 2. What is normally true of an instance property?

A. All objects automatically share it.
B. It belongs to an object, so separate instances have separate values.
C. It can only store strings.
D. It must be accessed using the scope-resolution operator.

**Answer:** B. It belongs to an object, so separate instances have separate values.

**Explanation:** Instance properties represent per-object state and are accessed through an object, for example with the object operator.

### 3. Which syntax accesses a static property from outside its class?

A. VisitCounter->visits
B. VisitCounter::$visits
C. VisitCounter::visits()
D. $VisitCounter->visits

**Answer:** B. VisitCounter::$visits.

**Explanation:** Static properties use the scope-resolution operator, followed by a dollar sign and the property name.

### 4. How can a public static method be called without creating an object?

A. ClassName::method()
B. ClassName->method()
C. $this::method()
D. new ClassName::method()

**Answer:** A. ClassName::method().

**Explanation:** Static methods belong to the class and do not require an instance.

### 5. Which statement about $this in a static method is correct?

A. It refers to the most recently created object.
B. It is always null.
C. It is unavailable because the method has no current object.
D. It refers to the parent class.

**Answer:** C. It is unavailable because the method has no current object.

**Explanation:** The pseudo-variable $this represents the current object. A static method can be called without an object, so it has no $this.

### 6. What happens when code statically calls a non-static method in PHP 8 or later?

A. PHP creates an object automatically.
B. The method becomes static.
C. PHP throws an Error.
D. The call always works but emits a notice.

**Answer:** C. PHP throws an Error.

**Explanation:** Call instance methods on an object. A non-static method may depend on that object's properties through $this.

### 7. Inside a method declared in BaseModel, what does self::class resolve to?

A. Always the runtime child class
B. The class where the method was defined
C. The object variable's name
D. The parent class of BaseModel

**Answer:** B. The class where the method was defined.

**Explanation:** self:: is resolved in the class where the code is written.

### 8. Inside an inherited method, what does static::class use?

A. The class that was called at runtime
B. Only the class where the method was written
C. The first object created in the process
D. The parent class

**Answer:** A. The class that was called at runtime.

**Explanation:** static:: provides late static binding and can select an overriding member from the runtime called class.

### 9. What does parent:: refer to inside a child class?

A. The current object
B. The parent class
C. The runtime class of the child object
D. A global function

**Answer:** B. The parent class.

**Explanation:** parent:: is used to access a parent's class-level method or property from child-class code.

### 10. Which return type expresses that a static factory returns the called class or a compatible subclass?

A. object
B. self
C. static
D. parent

**Answer:** C. static.

**Explanation:** The static return type uses late static binding. It is useful with new static() in inherited factory methods.

### 11. A static property is declared only in a parent class and is inherited. What generally happens when parent and child access it?

A. Each child automatically receives an independent copy.
B. They refer to the inherited class-level property.
C. It becomes an instance property.
D. It is changed to private.

**Answer:** B. They refer to the inherited class-level property.

**Explanation:** When the child does not redeclare the property, it refers to the inherited static storage.

### 12. What happens if a child class redeclares an inherited static property with the same name?

A. The parent and child have distinct static properties.
B. Both declarations are discarded.
C. It becomes a local variable.
D. PHP automatically converts both to constants.

**Answer:** A. The parent and child have distinct static properties.

**Explanation:** Redeclaration gives the child its own class-level property. This matters when tracking per-class state.

### 13. Which is a better fit for a fixed, unchanging class-level value?

A. A mutable static property
B. An instance property
C. A class constant
D. A global variable

**Answer:** C. A class constant.

**Explanation:** Constants communicate that a value is fixed. A mutable static property should be used only when shared class-level state is intentional.

### 14. Does a static property automatically persist after a PHP process ends?

A. Yes, it is written to disk.
B. Yes, it becomes a database row.
C. No, static state is not persistent storage.
D. Only if its name begins with an underscore.

**Answer:** C. No, static state is not persistent storage.

**Explanation:** Static properties live in the execution context. They do not automatically survive separate PHP process executions.

### 15. What is a risk of mutable static state in tests?

A. It cannot be read in a test.
B. One test can affect another through shared state.
C. It makes all properties private.
D. PHP disables assertions.

**Answer:** B. One test can affect another through shared state.

**Explanation:** A test that changes a static property may leave the class in a different state for later tests unless the value is reset.

## Short-answer questions

### 16. Explain the difference between a static property and an instance property.

**Answer:** A static property belongs to the class and is shared class-level state. An instance property belongs to one object.

**Explanation:** Two objects have separate instance values, but they use the same static property unless inheritance redeclaration creates another class-specific property.

### 17. How do you read a static property inside a class method?

**Answer:** Use a class-scope keyword and the scope-resolution operator, for example self::$count.

**Explanation:** Static properties use the scope-resolution operator; the object operator is for instance properties.

### 18. Why is $this unavailable inside a static method?

**Answer:** A static method can run without any object, so there is no current instance for $this to represent.

**Explanation:** If the method needs per-object state, it should usually be an instance method.

### 19. Give one appropriate use for a static method.

**Answer:** A class-level conversion helper or a factory that can create an object without needing an existing instance.

**Explanation:** The operation should make sense without an object's state. Static methods can still return newly created objects.

### 20. When is an instance method preferable to a static method?

**Answer:** When the operation depends on the current object's properties or injected dependencies.

**Explanation:** Instance methods have access to $this and make per-object behavior explicit.

### 21. Explain the difference between self:: and static::.

**Answer:** self:: refers to the class where the method is defined. static:: refers to the class used at runtime for the call.

**Explanation:** static:: enables late static binding, allowing inherited methods to work with child classes.

### 22. What does new static() do in an inherited factory method?

**Answer:** It creates an object of the runtime called class.

**Explanation:** If a child invokes the inherited factory, new static() creates the child object. new self() instead creates an object of the class where the factory was defined.

### 23. What does parent:: help a child class do?

**Answer:** It lets the child refer to a member declared by its parent.

**Explanation:** It is commonly used when a child extends or wraps parent behavior, including calling a parent implementation.

### 24. Why should static properties use explicit visibility?

**Answer:** Explicit visibility communicates which code is allowed to read or change the class-level state.

**Explanation:** Static properties support public, protected, and private visibility like other properties. A private property can be controlled through class methods.

### 25. Does creating a static counter make the count survive separate web requests or command-line runs?

**Answer:** No, not automatically.

**Explanation:** Static properties persist only for the lifetime of the relevant PHP execution context. Use durable storage when the value must outlive that context.

### 26. Why is a class constant preferable to a static property for a fixed value?

**Answer:** A constant communicates that the value is not meant to change.

**Explanation:** A mutable static property invites shared state changes; a class constant is clearer for stable class-level configuration or identifiers.

### 27. Can a static method access a static property?

**Answer:** Yes, if visibility permits it, using syntax such as self::$count or static::$count.

**Explanation:** Both belong to class-level scope. A static method still cannot use $this to access instance properties.

### 28. What is the difference between a class static property and a static local variable inside a function?

**Answer:** A class static property belongs to a class; a function's static local variable retains a local value between calls to that function in the same execution context.

**Explanation:** They are different PHP features despite using the same keyword. A function-local static is not accessed through ClassName::$property.

### 29. Why can static state make code harder to test?

**Answer:** It can hide dependencies and let one test's changes affect other code or later tests.

**Explanation:** Instance dependencies can be replaced or configured per object, while shared static state needs careful reset and isolation.

### 30. Should every helper method be declared static?

**Answer:** No. Make a helper static only when it does not require instance state and class-level invocation is useful.

**Explanation:** Unnecessary static methods make dependencies less explicit and can make substitution or isolated testing harder.

## Scenarios and code review

### 31. What is wrong with this access?

~~~php
class Config
{
    public static string $mode = 'dev';
}

echo Config::$mode;
~~~

**Answer:** Nothing is wrong; this is the correct syntax for reading the static property.

**Explanation:** Static properties use the class name and scope-resolution operator.

### 32. What is wrong with this access?

~~~php
class Config
{
    public static string $mode = 'dev';
}

$config = new Config();
echo $config->mode;
~~~

**Answer:** The code uses the object operator for a static property. Use Config::$mode.

**Explanation:** The object operator looks for an instance property. A static property is accessed through the scope-resolution operator.

### 33. Why does this method fail when called?

~~~php
class Label
{
    public static function forProduct(): string
    {
        return $this->name;
    }
}
~~~

**Answer:** A static method has no $this, and the class shown does not declare a name instance property.

**Explanation:** If the label depends on a product object's name, make the method an instance method and store the name on each object.

### 34. What happens when this call runs on PHP 8 or later?

~~~php
class Report
{
    public function summary(): string
    {
        return 'Summary';
    }
}

Report::summary();
~~~

**Answer:** PHP throws an Error because summary() is not static.

**Explanation:** Create a Report object and call its summary() method, or intentionally redesign the operation as static if it needs no instance state.

### 35. What do these calls print?

~~~php
class Base
{
    public static function definedHere(): string
    {
        return self::class;
    }

    public static function calledClass(): string
    {
        return static::class;
    }
}

class Child extends Base
{
}

echo Child::definedHere() . PHP_EOL;
echo Child::calledClass() . PHP_EOL;
~~~

**Answer:** First Base, then Child.

**Explanation:** self::class is resolved where the method is defined. static::class uses the runtime called class.

### 36. If a parent class declares a static property and a child does not redeclare it, do they use separate copies?

**Answer:** No. The child uses the inherited static property.

**Explanation:** Redeclaration is what gives the child a distinct property with the same name.

### 37. A child redeclares a parent's static property with the same name. Should the code assume both classes still share one value?

**Answer:** No.

**Explanation:** The parent and child properties are distinct class-level storage. Use the intended class scope and late static binding deliberately.

### 38. A request counter is stored in a static property. Will it automatically count requests after PHP starts a new worker process?

**Answer:** No.

**Explanation:** Static state is in-memory execution state, not durable storage. Use an appropriate shared store if a count must persist across processes.

### 39. In a base-class method, which expression is more likely to select a child class's redeclared static property: self::$count or static::$count?

**Answer:** static::$count.

**Explanation:** self:: stays bound to the class where the code is defined. static:: uses the runtime called class through late static binding.

### 40. A static method needs a database connection that varies between application environments. Should the method hide the connection in mutable static state?

**Answer:** Usually no. Prefer an instance service that receives the connection as a dependency.

**Explanation:** Injected instance dependencies are explicit and easier to replace in tests. Static mutable state hides where the dependency comes from and who can change it.

## Coding and applied questions

### 41. Write a class with a private static counter. Each new object should increment it, and callers should read it through a public static method.

**Answer:**

~~~php
class Session
{
    private static int $created = 0;

    public function __construct()
    {
        self::$created++;
    }

    public static function createdCount(): int
    {
        return self::$created;
    }
}

new Session();
new Session();

echo Session::createdCount(); // 2
~~~

**Explanation:** The count belongs to the class, not to either individual Session object. The private property keeps direct writes outside the class from changing it.

### 42. Write a static conversion method that converts Celsius to Fahrenheit.

**Answer:**

~~~php
class Temperature
{
    public static function celsiusToFahrenheit(float $celsius): float
    {
        return ($celsius * 9 / 5) + 32;
    }
}

echo Temperature::celsiusToFahrenheit(20); // 68
~~~

**Explanation:** The calculation has no need for object-specific state, so it can be called on the class.

### 43. Write an inherited factory that returns an instance of the called class.

**Answer:**

~~~php
class Message
{
    public function __construct(
        public string $text,
    ) {
    }

    public static function fromText(string $text): static
    {
        return new static($text);
    }
}

class Alert extends Message
{
}

$alert = Alert::fromText('Check the server');
echo $alert::class; // Alert
~~~

**Explanation:** static as a return type and new static() use late static binding, so an inherited call creates the child class.

### 44. Demonstrate separate parent and child static counters by redeclaring the property in the child.

**Answer:**

~~~php
class BaseCounter
{
    protected static int $count = 0;

    public static function increment(): void
    {
        static::$count++;
    }

    public static function count(): int
    {
        return static::$count;
    }
}

class ChildCounter extends BaseCounter
{
    protected static int $count = 0;
}

BaseCounter::increment();
ChildCounter::increment();
ChildCounter::increment();

echo BaseCounter::count() . PHP_EOL; // 1
echo ChildCounter::count() . PHP_EOL; // 2
~~~

**Explanation:** The child redeclares the property, giving it separate storage. The use of static::$count selects the property belonging to the called class.

### 45. Refactor this code so the product name comes from an object instead of trying to use $this in a static method.

~~~php
class Product
{
    public function __construct(
        private string $name,
    ) {
    }

    public static function label(): string
    {
        return $this->name;
    }
}
~~~

**Answer:**

~~~php
class Product
{
    public function __construct(
        private string $name,
    ) {
    }

    public function label(): string
    {
        return $this->name;
    }
}

$product = new Product('Notebook');
echo $product->label(); // Notebook
~~~

**Explanation:** The label depends on this Product object's name, so label() should be an instance method with access to $this.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [065 - Visibility and encapsulation](../lessons/065-visibility-and-encapsulation.md) | [Lesson](../lessons/066-static-properties-and-methods.md) | [Example](../examples/066-static-properties-and-methods.md) | [067 - Inheritance and method overriding](../lessons/067-inheritance-and-method-overriding.md) |
