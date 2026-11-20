# 066 - Static Properties and Methods

A static property or method belongs to a class rather than to one particular object. Static members are useful for class-level state and operations that do not need an object's instance data.

## Learning goals

By the end of this chapter, you should be able to:

- Explain the difference between instance and static members.
- Declare, read, and update static properties with the scope-resolution operator.
- Declare and call static methods without creating an object.
- Understand why a static method has no `$this`.
- Choose between `self::`, `parent::`, and `static::`.
- Explain late static binding in an inheritance hierarchy.
- Avoid using static state when instance state or dependency injection is a better fit.

## Instance members and class members

An instance property belongs to one object. Each object has its own value. A static property belongs to a class, so it is shared by uses of that class rather than stored separately on every object.

An instance method runs in an object context and can use `$this`. A static method is called on a class and has no current object, so it cannot use `$this`.

| Member kind | Belongs to | Access syntax |
| --- | --- | --- |
| Instance property | One object | `$object->property` |
| Instance method | One object | `$object->method()` |
| Static property | A class | `ClassName::$property` |
| Static method | A class | `ClassName::method()` |

A static member is not simply a public instance member with different syntax. Its state and behavior have class-level semantics.

## Static properties

Declare a static property with the `static` keyword. Access it using `ClassName::$property` or, from within class code, a scope keyword such as `self::$property`.

~~~php
class VisitCounter
{
    public static int $visits = 0;

    public function __construct()
    {
        self::$visits++;
    }
}

$first = new VisitCounter();
$second = new VisitCounter();

echo VisitCounter::$visits; // 2
~~~

Both objects update the same class-level property. The value is not copied onto each instance.

Use `::`, the scope-resolution operator, for static properties. The object operator `->` accesses instance properties and is not the correct way to access a static property.

~~~php
class Settings
{
    public static string $environment = 'development';
}

echo Settings::$environment;

// Incorrect: a static property is not an instance property.
// $settings = new Settings();
// echo $settings->environment;
~~~

Static properties can have visibility just like other class properties. A private static property can keep class-level state controlled by the declaring class.

~~~php
class RequestTracker
{
    private static int $count = 0;

    public static function record(): void
    {
        self::$count++;
    }

    public static function count(): int
    {
        return self::$count;
    }
}

RequestTracker::record();
RequestTracker::record();

echo RequestTracker::count(); // 2
~~~

Static properties must be declared in the class. They are not a substitute for undeclared dynamic properties.

## Static methods

A static method is called on a class without first creating an object.

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

A static method has no object context. The pseudo-variable `$this` is unavailable inside it.

~~~php
class Formatter
{
    public static function uppercase(string $value): string
    {
        return strtoupper($value);
    }
}

echo Formatter::uppercase('php'); // PHP
~~~

If an operation needs a particular object's properties, make it an instance method:

~~~php
class Product
{
    public function __construct(
        private string $name,
        private int $priceInCents,
    ) {
    }

    public function label(): string
    {
        return $this->name . ' - ' . $this->priceInCents . ' cents';
    }
}

$product = new Product('Notebook', 1299);
echo $product->label();
~~~

The label depends on the specific product instance, so it needs `$this`. Making it static would remove access to that object context.

Calling a non-static method statically is an error in PHP 8 and later:

~~~php
class Cart
{
    public function total(): int
    {
        return 0;
    }
}

// Cart::total(); // Error: total() is not a static method.
~~~

Call instance methods through an object, and call static methods through the class.

## Choosing static or instance methods

A static method is a good fit when the operation:

- Does not depend on state belonging to one object.
- Is meaningfully associated with the class.
- Can be called without constructing an instance.

A static factory method is one possible use:

~~~php
class Color
{
    private function __construct(
        public int $red,
        public int $green,
        public int $blue,
    ) {
    }

    public static function fromHex(string $hex): self
    {
        $hex = ltrim($hex, '#');

        if (!preg_match('/^[0-9a-fA-F]{6}$/', $hex)) {
            throw new InvalidArgumentException(
                'Color must contain exactly six hexadecimal digits.'
            );
        }

        return new self(
            hexdec(substr($hex, 0, 2)),
            hexdec(substr($hex, 2, 2)),
            hexdec(substr($hex, 4, 2)),
        );
    }
}

$color = Color::fromHex('#336699');
echo $color->red; // 51
~~~

The factory is static because it creates a new object without needing an existing Color instance. The returned Color still has instance properties.

Do not make a method static only to avoid passing an object. If the operation depends on replaceable services or object state, passing dependencies through an instance is often clearer and easier to test.

## Scope-resolution keywords

Inside class code, these keywords help identify which class's member is being used:

- `self::` refers to the class where the current method is defined.
- `parent::` refers to the parent class.
- `static::` refers to the class that was called at runtime. This is late static binding.

The distinction matters when a child class inherits a method:

~~~php
class BaseModel
{
    public static function definedIn(): string
    {
        return self::class;
    }

    public static function calledAs(): string
    {
        return static::class;
    }
}

class User extends BaseModel
{
}

echo User::definedIn() . PHP_EOL; // BaseModel
echo User::calledAs() . PHP_EOL;  // User
~~~

The method is defined in BaseModel, so `self::class` resolves to BaseModel. The late-bound `static::class` resolves to the class used for the call, User.

### Late static binding in a factory

Late static binding is useful when an inherited method should create the called child class:

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

The return type `static` means the called class or a compatible subclass. Using `new static` creates an instance of that called class. With `new self`, the method would create Message instead.

Use `static::` when polymorphic, late-bound behavior is intended. Use `self::` when the implementation should always refer to the class where it is written.

## Inherited static properties

Static properties are shared at class level, but inheritance and redeclaration affect which storage is referenced.

If a static property is declared only on a parent class, parent and child references to that inherited property use the same storage. If a child redeclares a static property with the same name, the parent and child have distinct properties.

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

class ApiCounter extends BaseCounter
{
    protected static int $count = 0;
}

BaseCounter::increment();
ApiCounter::increment();
ApiCounter::increment();

echo BaseCounter::count() . PHP_EOL; // 1
echo ApiCounter::count() . PHP_EOL;  // 2
~~~

The child redeclares `$count`, so it has its own value. The use of `static::$count` lets the called class select the corresponding property. This distinction matters for counters, registries, and other state that might otherwise be unintentionally shared.

## Static state and its limits

Static state can be convenient for a simple class-level value, but it is shared state. Any code that can change it affects later code using the same class in the same PHP execution context.

Use static state with care:

- It can make behavior depend on which code ran earlier.
- Tests may affect one another if they share mutable static state.
- Long-running processes can retain the value between handled requests or jobs.
- It can hide dependencies that would be clearer as constructor arguments.
- A static counter is not automatically persistent across separate PHP process runs.

Prefer an instance property when the value belongs to one object. Prefer a constant for a fixed class-level value. Use a static property when shared mutable class-level state is truly part of the design.

## Common mistakes

- **Using `->` for a static property:** use the scope-resolution operator, such as `Config::$mode`.
- **Using `$this` in a static method:** a static method has no current object.
- **Calling an instance method statically:** PHP 8 and later throw an Error.
- **Making every helper static:** static methods can make dependencies and test setup less explicit.
- **Using `self::` when a child class should be selected:** consider `static::` for late static binding.
- **Assuming parent and child static properties always share one value:** redeclaring the property in the child creates distinct storage.
- **Treating static state as permanent storage:** it is not a database or file and normally does not survive separate process executions.

## Key takeaways

- Instance state belongs to objects; static state belongs to classes.
- Static properties use `ClassName::$property` or `self::$property` syntax.
- Static methods do not have `$this` and should not require a particular object instance.
- `self::` is resolved where code is defined; `static::` uses late static binding.
- Inherited static properties share storage unless a child redeclares the property.
- Prefer constants for fixed values and instance members for per-object behavior.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [065 - Visibility and encapsulation](065-visibility-and-encapsulation.md) | [Q&A](../qnas/066-static-properties-and-methods.md) | [Example](../examples/066-static-properties-and-methods.md) | [067 - Inheritance and method overriding](067-inheritance-and-method-overriding.md) |
