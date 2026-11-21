# Class constants and constant expressions - Q&A

These questions review class constant syntax, visibility, constant expressions, inheritance, and version-specific features. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. How do you access a public class constant?

A. <code>$ClassName->VALUE</code>
B. <code>ClassName::VALUE</code>
C. <code>ClassName.VALUE</code>
D. <code>ClassName::$VALUE</code>

**Answer: B.** The scope resolution operator <code>::</code> is used to access a class constant.

### 2. Which syntax declares a class constant?

A. <code>public $const NAME = 'x';</code>
B. <code>public const NAME = 'x';</code>
C. <code>constant public NAME = 'x';</code>
D. <code>public static NAME = 'x';</code>

**Answer: B.** The <code>const</code> keyword declares a class constant. Class constants do not have a dollar sign.

### 3. What does this declaration mean?

~~~php
class Status
{
    public const OK = 200;
}
~~~

A. Every Status object has a mutable OK property
B. OK is a fixed class-level value
C. OK is a method
D. Status cannot be instantiated

**Answer: B.** A class constant belongs to the class and is not stored separately on each object.

### 4. What is the default visibility of a class constant?

A. Private
B. Protected
C. Public
D. Static

**Answer: C.** Class constants are public by default. Visibility can also be explicitly declared.

### 5. Which version introduced visibility modifiers for class constants?

A. PHP 5.6
B. PHP 7.1
C. PHP 8.0
D. PHP 8.3

**Answer: B.** Public, protected, and private class constant visibility became available in PHP 7.1.

### 6. What does a final class constant prevent?

A. Reading the constant
B. Reassigning it at runtime
C. A child class from redeclaring it
D. The class from being instantiated

**Answer: C.** A final class constant cannot be redefined in a child class.

### 7. Which version added final class constants?

A. PHP 7.4
B. PHP 8.0
C. PHP 8.1
D. PHP 8.4

**Answer: C.** Class constants can be marked final starting with PHP 8.1.

### 8. Which version added typed class constants?

A. PHP 7.4
B. PHP 8.1
C. PHP 8.2
D. PHP 8.3

**Answer: D.** PHP 8.3 introduced type declarations for class constants.

### 9. What is a constant expression?

A. An expression that reads a mutable object property
B. A compile-time expression composed of supported constant values and operations
C. Any function call
D. A statement that changes a constant

**Answer: B.** Constant expressions use values and operations PHP permits when defining a constant; runtime work such as calling <code>time()</code> is not generally valid there.

### 10. Which keyword resolves a constant to the class where the method was defined?

A. <code>parent::</code>
B. <code>static::</code>
C. <code>self::</code>
D. <code>this::</code>

**Answer: C.** <code>self::</code> refers to the class where the method was defined.

### 11. Which syntax follows the class that was called at runtime?

A. <code>self::</code>
B. <code>static::</code>
C. <code>parent::</code>
D. <code>const::</code>

**Answer: B.** <code>static::</code> uses late static binding and can resolve to a child class.

### 12. What does <code>SomeClass::class</code> produce?

A. An instance of SomeClass
B. A string containing the class name
C. The class's constructor
D. A class constant named class

**Answer: B.** The special <code>::class</code> syntax evaluates to the class name as a string.

## Short-answer questions

### 13. Why use a class constant instead of a property?

**Answer:** Use a class constant for a fixed value shared by the class; use a property for data that varies or can change.

**Explanation:** A constant cannot be reassigned during execution. A property can represent object or mutable class state.

### 14. Does each object instance store its own copy of a class constant?

**Answer:** No.

**Explanation:** The value belongs to the class and is allocated once per class, not once per instance.

### 15. Can a public class constant be accessed from outside the class?

**Answer:** Yes, using <code>ClassName::CONSTANT</code>.

**Explanation:** Public visibility makes it part of the externally accessible API.

### 16. What does protected visibility allow for a class constant?

**Answer:** The declaring class and its descendants can access it.

**Explanation:** Unrelated external code cannot access a protected constant directly.

### 17. What does private visibility allow for a class constant?

**Answer:** Only the class that declares it can access it.

**Explanation:** A private constant is an implementation detail and is not exposed to child classes or outside callers.

### 18. Can a child redeclare a non-final class constant?

**Answer:** Yes, subject to compatibility rules.

**Explanation:** A child may provide a specialized value. A final constant removes that extension point.

### 19. Why should a constant not be used for a value from <code>time()</code>?

**Answer:** <code>time()</code> computes a value at runtime, while a constant expression must use values PHP can resolve in a constant declaration.

**Explanation:** Use a method or property when a value depends on the current runtime.

### 20. Can an array be stored in a class constant?

**Answer:** Yes, PHP supports constant arrays.

**Explanation:** Use a constant array for fixed lists or lookup maps. Do not use it for data that must be modified.

### 21. What is the difference between <code>self::VALUE</code> and <code>static::VALUE</code> in a parent method?

**Answer:** <code>self::VALUE</code> resolves to the defining class; <code>static::VALUE</code> follows the runtime called class.

**Explanation:** Use late static binding only when subclass overrides should affect the inherited method.

### 22. What is a typed class constant?

**Answer:** A class constant declared with an explicit type, supported since PHP 8.3.

**Explanation:** The declared value must match the type, making the expected value clearer and helping PHP catch invalid declarations.

### 23. What is the purpose of a final class constant?

**Answer:** It prevents child classes from redefining a value that must remain fixed in the hierarchy.

**Explanation:** It is useful for invariants, but should not be used if subclass customization is intentional.

### 24. Can a class constant be reassigned like a property?

**Answer:** No.

**Explanation:** Constants cannot be changed after declaration. Use a property if the value needs to vary during runtime.

### 25. What is the special syntax <code>ClassName::class</code> useful for?

**Answer:** It obtains the class name string without manually typing that name.

**Explanation:** It is useful for class-name references, including namespaced classes, without constructing an object.

### 26. What changed for class constant typing in PHP 8.3?

**Answer:** Class constants can declare supported scalar or array types, and PHP checks that the assigned value conforms.

**Explanation:** Before 8.3, omit the type from the class constant declaration.

## Code-reading and design questions

### 27. What does this program print?

~~~php
class HttpStatus
{
    public const NOT_FOUND = 404;
}

echo HttpStatus::NOT_FOUND;
~~~

**Answer:** <code>404</code>

**Explanation:** The class constant is accessed using the class name and scope resolution operator.

### 28. What does this program print?

~~~php
class Units
{
    public const MINUTES = 60;
    public const HOURS_PER_DAY = 24;
    public const MINUTES_PER_DAY = self::MINUTES * self::HOURS_PER_DAY;
}

echo Units::MINUTES_PER_DAY;
~~~

**Answer:** <code>1440</code>

**Explanation:** The constant expression multiplies two class constants to define another fixed value.

### 29. What is the visibility problem here?

~~~php
class SecretCodes
{
    private const INTERNAL = 'x1';
}

echo SecretCodes::INTERNAL;
~~~

**Answer:** The external access is invalid because INTERNAL is private.

**Explanation:** Private constants can be accessed only from within the declaring class.

### 30. What does this program print?

~~~php
class BaseConfig
{
    public const LABEL = 'Base';

    public static function fixedLabel(): string
    {
        return self::LABEL;
    }

    public static function calledLabel(): string
    {
        return static::LABEL;
    }
}

class ChildConfig extends BaseConfig
{
    public const LABEL = 'Child';
}

echo ChildConfig::fixedLabel() . PHP_EOL;
echo ChildConfig::calledLabel();
~~~

**Answer:**

~~~text
Base
Child
~~~

**Explanation:** <code>self::</code> is resolved where the method was defined. <code>static::</code> follows the called class through late static binding.

### 31. Why does this fail on PHP versions before 8.3?

~~~php
class Limits
{
    public const int MAX_USERS = 100;
}
~~~

**Answer:** Typed class constants were introduced in PHP 8.3.

**Explanation:** On earlier versions, declare it as <code>public const MAX_USERS = 100;</code> without the type.

### 32. What does final prevent in this example?

~~~php
class Protocol
{
    final public const VERSION = '1';
}

class CustomProtocol extends Protocol
{
    public const VERSION = '2';
}
~~~

**Answer:** PHP rejects the child declaration of VERSION.

**Explanation:** Since PHP 8.1, a final class constant cannot be redefined by a child class.

### 33. Why is this declaration invalid?

~~~php
class RuntimeDefaults
{
    public const STARTED_AT = time();
}
~~~

**Answer:** <code>time()</code> is a runtime function call, not a supported constant expression for this declaration.

**Explanation:** If the current time is needed, compute it in a method or store it in a property.

### 34. Should changing application state be modeled as a class constant?

**Answer:** No.

**Explanation:** Constants are fixed. Mutable state belongs in an appropriate property or external state store.

### 35. Is it safe to narrow a public interface constant to protected in an implementation?

**Answer:** No.

**Explanation:** An implementation must preserve the interface's public accessibility contract. PHP 8.3 checks class-constant visibility compatibility more strictly.

## Coding questions

### 36. Define public constants for common HTTP responses.

**Answer:**

~~~php
class HttpStatus
{
    public const OK = 200;
    public const CREATED = 201;
    public const NOT_FOUND = 404;
}
~~~

**Explanation:** These values are stable, meaningful class-level names that callers can access with <code>HttpStatus::OK</code> and similar expressions.

### 37. Define a constant expression using another class constant.

**Answer:**

~~~php
class TimeUnits
{
    public const SECONDS_PER_MINUTE = 60;
    public const MINUTES_PER_HOUR = 60;
    public const SECONDS_PER_HOUR =
        self::SECONDS_PER_MINUTE * self::MINUTES_PER_HOUR;
}
~~~

**Explanation:** The derived value is computed from other constant values and remains fixed.

### 38. Restrict a constant to the class hierarchy.

**Answer:**

~~~php
class RetryPolicy
{
    protected const MAX_ATTEMPTS = 3;
}
~~~

**Explanation:** The constant is available inside RetryPolicy and child classes, but not as a public API for unrelated callers.

### 39. Use late static binding so child classes can provide a label.

**Answer:**

~~~php
class Formatter
{
    public const LABEL = 'Base';

    public static function label(): string
    {
        return static::LABEL;
    }
}

class JsonFormatter extends Formatter
{
    public const LABEL = 'JSON';
}

echo JsonFormatter::label();
~~~

**Explanation:** The result is <code>JSON</code> because <code>static::</code> uses the called subclass.

### 40. Declare a typed constant for PHP 8.3 or later.

**Answer:**

~~~php
class RetryPolicy
{
    public const int MAX_ATTEMPTS = 3;
    public const array DELAYS = [1, 2, 4];
}
~~~

**Explanation:** The types make the expected value shapes explicit. Remove the type declarations when supporting PHP versions before 8.3.

## Key takeaways

- Class constants are fixed class-level values accessed with <code>::</code>.
- Constant expressions combine supported constant values and operations.
- Visibility determines which code may access a constant.
- Child classes may redeclare non-final constants; final constants prohibit it.
- <code>self::</code> resolves to the defining class, while <code>static::</code> follows the called class.
- Typed class constants and dynamic constant fetch were added in PHP 8.3.
- Use a property or method for values that change at runtime.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [071 - Traits and code reuse](../lessons/071-traits-and-code-reuse.md) | [Lesson](../lessons/072-class-constants-and-constant-expressions.md) | [Example](../examples/072-class-constants-and-constant-expressions.md) | [073 - Readonly properties and classes](../lessons/073-readonly-properties-and-classes.md) |
