# 066 - Static Properties and Methods: Example

## Question

How can static state be shared by objects, how can static methods be called without an object, and how does late static binding let an inherited factory create the called child class?

## Complete example

Save this PHP 8.0+ code as `static-properties-and-methods.php` and run `php static-properties-and-methods.php` from a terminal.

~~~php
<?php

declare(strict_types=1);

class VisitCounter
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

class Temperature
{
    public static function celsiusToFahrenheit(float $celsius): float
    {
        return ($celsius * 9 / 5) + 32;
    }
}

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

class BaseLogger
{
    public static function prefix(): string
    {
        return 'LOG';
    }
}

class SecurityLogger extends BaseLogger
{
    public static function prefix(): string
    {
        return parent::prefix() . ':SECURITY';
    }
}

new VisitCounter();
new VisitCounter();

echo 'Objects created: ' . VisitCounter::createdCount() . PHP_EOL;
echo '20 C in Fahrenheit: '
    . number_format(Temperature::celsiusToFahrenheit(20), 1)
    . PHP_EOL;

echo 'Defined in: ' . User::definedIn() . PHP_EOL;
echo 'Called as: ' . User::calledAs() . PHP_EOL;

$alert = Alert::fromText('Check the server');
echo 'Factory result: ' . $alert::class . ' - ' . $alert->text . PHP_EOL;

echo 'Logger prefix: ' . SecurityLogger::prefix() . PHP_EOL;

BaseCounter::increment();
ApiCounter::increment();
ApiCounter::increment();

echo 'Base count: ' . BaseCounter::count() . PHP_EOL;
echo 'API count: ' . ApiCounter::count() . PHP_EOL;
~~~

## Expected output

~~~text
Objects created: 2
20 C in Fahrenheit: 68.0
Defined in: BaseModel
Called as: User
Factory result: Alert - Check the server
Logger prefix: LOG:SECURITY
Base count: 1
API count: 2
~~~

## Explanation

### A static property is shared class-level state

VisitCounter keeps its created count in a private static property. Each new object increments that one property. The count is read through a public static method, so callers do not write the private state directly.

### A static method does not need an object

Temperature::celsiusToFahrenheit() performs a calculation that needs no particular Temperature object. It can be called directly on the class.

If a method needs an object's data, it should be an instance method. Static methods have no current object and cannot use $this.

### self:: and static:: resolve differently

User inherits methods from BaseModel. definedIn() uses self::class and returns BaseModel because that is where the method is declared. calledAs() uses static::class, so it returns User when invoked as User::calledAs().

### The factory uses late static binding

Message::fromText() returns static and constructs new static(). When Alert calls the inherited factory, the new object is an Alert. Using new self() would create a Message instead.

### parent:: can reuse parent behavior

SecurityLogger::prefix() calls the parent implementation with parent::prefix(), then adds its own label. The child extends the behavior without copying the parent's string.

### Redeclared static properties have separate storage

BaseCounter and ApiCounter each declare a count property. The inherited methods use static::$count, so each call updates and reads the property for the class used at runtime. The parent count ends at one; the child count ends at two.

## Try these changes

1. Create a third VisitCounter object. What number does createdCount() return?
2. Change the temperature input from 20 to 0. What value is printed?
3. Change calledAs() to use self::class. What does the User call print then?
4. Change the factory to use new self($text). Which class does Alert::fromText() create?
5. Remove the count property declaration from ApiCounter. How does the inherited static state behave?
6. Try to use $this inside Temperature::celsiusToFahrenheit(). Why does the method have no current object?

## Key takeaways

- Static properties belong to the class; instance properties belong to objects.
- Static methods are called on a class and have no $this.
- self:: uses the class where the method is defined; static:: uses the runtime called class.
- new static() supports inherited factories that create child instances.
- A child that redeclares a static property has separate storage.
- The complete example uses features available in PHP 8.0 and later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [065 - Visibility and encapsulation](../lessons/065-visibility-and-encapsulation.md) | [Lesson](../lessons/066-static-properties-and-methods.md) | [Q&A](../qnas/066-static-properties-and-methods.md) | [067 - Inheritance and method overriding](../lessons/067-inheritance-and-method-overriding.md) |
