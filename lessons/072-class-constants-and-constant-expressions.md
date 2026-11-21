# Class constants and constant expressions

A class constant names a value that belongs to a class rather than to one object instance. Constants are useful for stable values such as limits, format names, and application-wide codes.

A **constant expression** is a compile-time expression used to define a constant value. PHP allows supported scalar operations and references to other constants in these expressions.

## What you will learn

- declare and access a class constant
- control constant visibility
- build values from constant expressions
- understand constant inheritance and final constants
- distinguish <code>self::</code> from <code>static::</code>
- use typed constants and other newer features when the PHP version supports them

## Declare and access a class constant

Use the <code>const</code> keyword inside a class. Class constants do not use a dollar sign:

~~~php
class ApiResponse
{
    public const SUCCESS = 200;
    public const NOT_FOUND = 404;
}

echo ApiResponse::SUCCESS;
~~~

The scope resolution operator <code>::</code> accesses a class constant. Inside the declaring class, <code>self::</code> refers to that class:

~~~php
class ApiResponse
{
    public const SUCCESS = 200;

    public function successCode(): int
    {
        return self::SUCCESS;
    }
}

$response = new ApiResponse();
echo $response->successCode();
~~~

A class constant is shared by the class; it is not a separate value stored on every object. Once declared, it cannot be reassigned like a property.

## Choose constant visibility

Class constants are public by default. Since PHP 7.1, they can be explicitly public, protected, or private:

~~~php
class ConnectionPolicy
{
    public const DEFAULT_TIMEOUT = 30;
    protected const MAX_RETRIES = 4;
    private const DRIVER_NAME = 'pdo';
}
~~~

- **public**: accessible from anywhere
- **protected**: accessible in the declaring class and its descendants
- **private**: accessible only in the declaring class

Prefer public constants only when the value is part of the class's supported API. Keep implementation details protected or private.

## Constant expressions

A class constant can be initialized with a supported constant expression:

~~~php
class TimeUnits
{
    public const SECONDS_PER_MINUTE = 60;
    public const MINUTES_PER_HOUR = 60;
    public const SECONDS_PER_HOUR =
        self::SECONDS_PER_MINUTE * self::MINUTES_PER_HOUR;
    public const LABEL = 'One hour contains '
        . self::SECONDS_PER_HOUR . ' seconds.';
}

echo TimeUnits::LABEL;
~~~

Expected output:

~~~text
One hour contains 3600 seconds.
~~~

Expressions can combine literal values, scalar operators, arrays of constant values, and references to other constants. They are evaluated as part of defining the constant, not by calling an instance method each time.

A function call such as <code>time()</code> is a runtime operation, not a stable class constant expression:

~~~php
class InvalidDefaults
{
    // This is not a valid constant expression:
    // public const CREATED_AT = time();
}
~~~

If a value must be computed at runtime or vary per object, use a method or property instead of a constant.

## Arrays in class constants

Constant arrays are useful for fixed maps and lists:

~~~php
class HttpStatus
{
    public const LABELS = [
        200 => 'OK',
        404 => 'Not Found',
        500 => 'Server Error',
    ];
}

echo HttpStatus::LABELS[404];
~~~

Expected output:

~~~text
Not Found
~~~

The constant array itself cannot be modified. Use a separate variable when you need mutable data.

## Constant inheritance

A child class can inherit a class constant. If the inherited constant is not final, the child may redeclare it:

~~~php
class Document
{
    public const KIND = 'generic';
}

class Invoice extends Document
{
    public const KIND = 'invoice';
}

echo Invoice::KIND;
~~~

Expected output:

~~~text
invoice
~~~

Use this only when the specialized class truly needs a different class-level value and the parent contract permits it.

## Final class constants

Since PHP 8.1, a class constant can be final. A child class then cannot redeclare it:

~~~php
class Protocol
{
    final public const VERSION = '1.0';
}

class CustomProtocol extends Protocol
{
    // Invalid: VERSION is final and cannot be redeclared.
}
~~~

Use final when the value is part of an invariant that every subclass must preserve. Do not mark constants final without a design reason; it removes a customization point.

## Typed class constants in PHP 8.3+

PHP 8.3 added types for class constants. The declared value must match the type:

~~~php
class RetryPolicy
{
    public const int MAX_ATTEMPTS = 3;
    public const string STRATEGY = 'fixed';
    public const array DELAYS = [1, 2, 4];
}
~~~

Before PHP 8.3, omit the type:

~~~php
class RetryPolicy
{
    public const MAX_ATTEMPTS = 3;
    public const STRATEGY = 'fixed';
}
~~~

Typed constants make the expected value type explicit. When overriding a typed constant, preserve type and visibility compatibility with the parent declaration.

## Compare <code>self::</code> and <code>static::</code>

Inside a parent class, <code>self::CONSTANT</code> refers to the class where the method was defined. <code>static::CONSTANT</code> uses the called class at runtime through late static binding:

~~~php
class BaseFormatter
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

class JsonFormatter extends BaseFormatter
{
    public const LABEL = 'JSON';
}

echo JsonFormatter::fixedLabel() . PHP_EOL;
echo JsonFormatter::calledLabel();
~~~

Expected output:

~~~text
Base
JSON
~~~

Use <code>self::</code> when the parent implementation must always use its own constant. Use <code>static::</code> when a child class is expected to supply the value.

This behavior is related to late static binding. Choose deliberately because <code>static::</code> makes subclass overrides affect inherited methods.

## Dynamic constant lookup in PHP 8.3+

When the constant name is stored in a variable, PHP 8.3+ supports this syntax:

~~~php
class FeatureFlags
{
    public const SEARCH = true;
    public const EXPORT = false;
}

$name = 'SEARCH';
var_dump(FeatureFlags::{$name});
~~~

For fixed names, prefer ordinary syntax such as <code>FeatureFlags::SEARCH</code>. Dynamic lookup is useful only when the name itself is genuinely selected at runtime.

## The <code>::class</code> name constant

The special <code>::class</code> syntax returns a class name as a string:

~~~php
class Report
{
}

echo Report::class;
~~~

It is useful for passing or storing class names without manually writing a string. It does not create an instance or load a class by itself.

## Class constants versus properties

| Class constant | Property |
| --- | --- |
| Class-level value shared by the class | Object state, or static class state |
| Access with <code>ClassName::NAME</code> or <code>self::NAME</code> | Access object state with <code>$this->name</code> |
| Cannot be reassigned | May be mutable unless declared readonly |
| Suitable for fixed configuration and semantic values | Suitable for values that vary at runtime or by object |

Do not use a constant for data that changes during execution. A static property is mutable shared state; a class constant is fixed.

## Version notes

| PHP version | Class constant feature |
| --- | --- |
| PHP 7.1 | Public, protected, and private visibility |
| PHP 8.1 | Final class constants |
| PHP 8.3 | Typed constants and dynamic class constant fetch |
| PHP 8.5 | Expanded constant expressions, including casts, closures, and first-class callables |

Target the minimum PHP version your project supports. The core examples in this lesson use syntax available from PHP 7.1, except where a section explicitly states a newer version.

## Common mistakes

- **Adding a dollar sign:** class constants are named without <code>$</code>.
- **Using a constant for changing data:** use a property or method for runtime values.
- **Calling functions in constant expressions:** function calls such as <code>time()</code> need runtime evaluation.
- **Making every constant public:** expose only values that callers should rely on.
- **Confusing <code>self::</code> and <code>static::</code>:** the former is fixed to the defining class; the latter follows the called class.
- **Overriding a final constant:** subclasses cannot redeclare it.
- **Using typed constants before PHP 8.3:** remove the type or raise the supported PHP version.
- **Using newer constant-expression syntax on older versions:** verify the project's minimum runtime.
- **Treating <code>::class</code> as object creation:** it returns a class name string.

## Key takeaways

- Class constants hold stable class-level values and are accessed with <code>::</code>.
- Constant expressions can combine supported scalar values and other constants.
- Constant visibility controls who can access a value.
- Non-final constants can be redeclared in child classes; final constants cannot.
- Final constants require PHP 8.1, typed constants and dynamic lookup require PHP 8.3.
- <code>self::</code> refers to the defining class; <code>static::</code> follows the called class.
- Use properties or methods when values need to vary at runtime.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [071 - Traits and code reuse](071-traits-and-code-reuse.md) | [Q&A](../qnas/072-class-constants-and-constant-expressions.md) | [Example](../examples/072-class-constants-and-constant-expressions.md) | [073 - Readonly properties and classes](073-readonly-properties-and-classes.md) |
