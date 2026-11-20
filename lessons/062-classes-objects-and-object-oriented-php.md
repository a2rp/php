# 062 - Classes, Objects, and Object-Oriented PHP

Object-oriented programming (OOP) organizes related state and behavior around objects. In PHP, a class describes a kind of object, and an object is one instance created from that class.

OOP is one way to organize a program, not a requirement for every PHP script. Small scripts and simple transformations can remain functions. Classes become useful when data and behavior belong together or when an application needs clear, reusable components.

## Learning goals

By the end of this chapter, you should be able to:

- Explain the difference between a class and an object instance.
- Describe how objects group state and behavior.
- Create instances with `new` and recognize instance member access.
- Explain the role of `$this` inside an instance method.
- Distinguish two separate objects from two variables referring to one object.
- Recognize the main OOP ideas and when a class is useful.

## Class and object

A **class** is a definition: it describes what members instances of that class can have.

An **object** is a particular instance created from that definition. Two objects can come from the same class while keeping independent state.

A useful analogy is a form template and the submitted forms created from it. The template describes the fields; each completed form stores its own values. The analogy is only a starting point: PHP objects can also contain behavior, rules, and relationships with other objects.

## State and behavior

- **State** is information held by an object, usually through its properties.
- **Behavior** is work the object can perform, usually through its methods.

Here is a small class with one method:

~~~php
<?php

declare(strict_types=1);

class Lamp
{
    public function turnOn(): void
    {
        echo "Lamp is on." . PHP_EOL;
    }
}

$deskLamp = new Lamp();
$deskLamp->turnOn();
~~~

The `Lamp` class defines the type of object. The `new Lamp()` expression creates an instance. The `->` operator accesses a member on that instance, and `turnOn()` calls its method.

## Create separate instances

Calling `new` again creates another object with its own identity:

~~~php
$deskLamp = new Lamp();
$readingLamp = new Lamp();

var_dump($deskLamp === $readingLamp); // bool(false)
~~~

Both variables refer to objects made from `Lamp`, but the objects are not the same instance.

A class can also define state. This short example previews properties and methods, which are covered in detail in the next chapter:

~~~php
class Counter
{
    public int $value = 0;

    public function increment(): void
    {
        $this->value++;
    }
}

$firstCounter = new Counter();
$secondCounter = new Counter();

$firstCounter->increment();

echo $firstCounter->value . PHP_EOL;  // 1
echo $secondCounter->value . PHP_EOL; // 0
~~~

Each instance has its own `value`. The change to `$firstCounter` does not change `$secondCounter`.

## What $this means

Inside an instance method, `$this` refers to the object on which that method was called.

~~~php
class Greeting
{
    public string $name = 'Alex';

    public function message(): string
    {
        return 'Hello, ' . $this->name . '!';
    }
}

$greeting = new Greeting();
echo $greeting->message();
~~~

When `$greeting->message()` runs, `$this` refers to `$greeting`. It lets the method read or change the current object's members.

A non-static method needs an object context. Call it on an instance with `->`. Class-level features that use `static` and `::` have different rules and are covered later.

## Variables can refer to the same object

Assigning one object variable to another does not create a cloned object. Both variables can refer to the same instance:

~~~php
$original = new Counter();
$alias = $original;

$alias->increment();

echo $original->value . PHP_EOL; // 1
echo $alias->value . PHP_EOL;    // 1
var_dump($original === $alias);  // bool(true)
~~~

Changing the object through `$alias` is visible through `$original`, because they refer to that same instance.

Reassigning one variable is different from changing the object:

~~~php
$original = new Counter();
$alias = $original;

$alias = new Counter();
$alias->increment();

echo $original->value . PHP_EOL; // 0
echo $alias->value . PHP_EOL;    // 1
var_dump($original === $alias);  // bool(false)
~~~

Now `$alias` refers to a new object. Reassignment did not replace the object referenced by `$original`.

Use the `clone` keyword when a shallow object copy is required. Cloning has its own behavior and is covered separately; it does not automatically deep-copy every object contained within the original.

## Object equality and identity

PHP has two comparisons that are useful to understand:

- `==` compares objects by class and their properties.
- `===` checks whether both variables refer to the very same instance of the same class.

~~~php
class Badge
{
    public string $label = 'Ready';
}

$first = new Badge();
$second = new Badge();
$anotherNameForFirst = $first;

var_dump($first == $second);                 // bool(true)
var_dump($first === $second);                // bool(false)
var_dump($first === $anotherNameForFirst);   // bool(true)
~~~

The first two objects have matching class and property values, so they compare equal with `==`. They are still separate instances, so `===` is false. The last variable points to the same instance as `$first`.

Do not use `==` when your question is whether two variables identify the same object. Use `===` for that.

## Core OOP ideas

| Idea | Meaning |
| --- | --- |
| Encapsulation | Keep related data and behavior together, and control how other code interacts with them |
| Abstraction | Expose the operations callers need while hiding unnecessary implementation detail |
| Inheritance | Define a specialized class based on another class |
| Polymorphism | Let different object types respond to a shared operation in their own way |

These ideas are introduced here; the following chapters build them up with properties, methods, visibility, constructors, interfaces, and inheritance. Prefer composition and simple designs when they communicate the relationships more clearly than a class hierarchy.

## When a class helps

A class can be useful when:

- Several values form one meaningful concept, such as a date range or a shopping cart.
- State has rules that should be enforced consistently.
- Several operations work on the same state.
- You want to substitute one implementation for another behind a shared contract.
- A large application needs clearly defined boundaries between responsibilities.

A class is not automatically better than a function. A one-line formatting helper with no state may be clearer as a function. Avoid creating classes that only wrap one unrelated function or that combine many unrelated responsibilities.

## Naming and organization

By convention, PHP class names use PascalCase, such as `ShoppingCart` or `InvoiceLine`. PHP class declarations use the `class` keyword followed by the class name and a body in braces.

~~~php
class ShoppingCart
{
    // Properties and methods are declared here.
}
~~~

Class naming style is a convention; follow the project's existing conventions. Later chapters cover namespaces and file organization for larger projects.

## Common mistakes

- Treating a class definition as if it were already an object.
- Forgetting that `new` creates an instance.
- Calling an instance method without an object.
- Assuming two calls to `new` return the same object.
- Assuming assigning an object variable creates a new copy.
- Confusing changing an object with reassigning a variable.
- Using `==` when the question is whether variables refer to the same instance.
- Putting unrelated responsibilities into one class.
- Creating a class for every small function without a clear benefit.

## Key takeaways

- A class describes a type; an object is one instance of that type.
- Objects combine state and behavior.
- Use `new` to create an instance and `->` to access its instance members.
- Inside an instance method, `$this` refers to the calling object.
- Assigning an object variable does not clone the object; both variables can identify the same instance.
- Use `===` to check object identity and `==` to compare object values.
- OOP is useful when it clarifies data, behavior, and responsibilities; it is not mandatory for every task.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [061 - Debugging, diagnostics, and application logging](061-debugging-diagnostics-and-application-logging.md) | [Open Q&A](../qnas/062-classes-objects-and-object-oriented-php.md) | [Open example](../examples/062-classes-objects-and-object-oriented-php.md) | [063 - Properties, methods, and object creation](063-properties-methods-and-object-creation.md) |
