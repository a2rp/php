# 063 - Properties, Methods, and Object Creation

A class groups data and behavior. In PHP, **properties** hold an object's state and **methods** define operations that can read or change that state. Creating an object gives you an instance to work with.

This chapter focuses on declaring and using these members. Constructors and destructors are covered in the next chapter.

## Learning goals

By the end of this chapter, you should be able to:

- Declare typed properties with defaults.
- Understand when a typed property must be initialized before it is read.
- Create objects with `new` and access instance members with `->`.
- Declare methods with parameters and return types.
- Use `$this` to work with the current instance.
- Choose between `public`, `protected`, and `private` visibility.
- Avoid accidental dynamic properties.

## Properties hold object state

A property is a variable declared inside a class. Properties can have a type and a default value:

~~~php
class Book
{
    public string $title = 'Untitled';
    public int $pageCount = 0;
}
~~~

The visibility keyword says which code can access the property. The type says what kind of value it can contain. The default gives each new object an initial value for that property.

Typed properties were introduced in PHP 7.4. The examples in this chapter use typed properties and work with PHP 7.4 or later.

### Uninitialized typed properties

A typed property without a default starts uninitialized. It must be assigned before it is read:

~~~php
class Profile
{
    public string $displayName;
}

$profile = new Profile();
$profile->displayName = 'Mina';

echo $profile->displayName;
~~~

If code reads `$profile->displayName` before assigning it, PHP throws an `Error`. Choose a default when an initial value is meaningful. For an optional string, a nullable property with a null default is one option:

~~~php
class Profile
{
    public ?string $displayName = null;
}
~~~

Nullable does not mean untyped: it allows either a string or `null`.

## Create an object and access its properties

Use `new` to create an object. Use `->` followed by the property name to read or write an instance property:

~~~php
$book = new Book();

$book->title = 'PHP for Beginners';
$book->pageCount = 240;

echo $book->title . PHP_EOL;
echo $book->pageCount . PHP_EOL;
~~~

Each object has its own instance properties:

~~~php
$firstBook = new Book();
$secondBook = new Book();

$firstBook->title = 'PHP for Beginners';

echo $firstBook->title . PHP_EOL;  // PHP for Beginners
echo $secondBook->title . PHP_EOL; // Untitled
~~~

The two calls to `new Book()` created separate objects with independent state.

A property declaration does not make a property value global to every instance. Static properties are a separate class-level feature and are not covered in this chapter.

## Methods define object behavior

A method is a function declared in a class. It can accept parameters, return a value, and use the object's properties:

~~~php
class Book
{
    public string $title = 'Untitled';
    private int $pageCount = 0;

    public function setPageCount(int $pages): void
    {
        if ($pages < 0) {
            throw new InvalidArgumentException(
                'Page count cannot be negative.'
            );
        }

        $this->pageCount = $pages;
    }

    public function getPageCount(): int
    {
        return $this->pageCount;
    }

    public function summary(): string
    {
        return $this->title . ' (' . $this->pageCount . ' pages)';
    }
}

$book = new Book();
$book->title = 'PHP for Beginners';
$book->setPageCount(240);

echo $book->summary();
~~~

Call an instance method with `$object->methodName()`. Supply arguments inside the parentheses. The declared return type documents and constrains the value the method returns. Use `void` when the method is not intended to return a value.

### The $this variable

Inside a non-static instance method, `$this` refers to the object receiving the call. In the example, `$this->pageCount` means the `pageCount` property on that specific book.

When this call runs:

~~~php
$book->setPageCount(240);
~~~

`$this` inside `setPageCount()` refers to `$book`.

## Visibility controls access

PHP provides three common visibility levels for properties and methods:

| Visibility | Accessible from |
| --- | --- |
| `public` | Any code that can access the object |
| `protected` | The class and its child classes |
| `private` | Only the class that declares the member |

Use `public` for operations other code is meant to call. Use `private` for implementation details that should only be accessed inside the class. Use `protected` when a class and its subclasses need a shared member.

A public property is easy to use, but outside code can assign any value allowed by its type, bypassing business rules. A private property with a public method can validate changes:

~~~php
class InventoryItem
{
    private int $quantity = 0;

    public function setQuantity(int $quantity): void
    {
        if ($quantity < 0) {
            throw new InvalidArgumentException(
                'Quantity cannot be negative.'
            );
        }

        $this->quantity = $quantity;
    }

    public function quantity(): int
    {
        return $this->quantity;
    }
}
~~~

The caller uses the public methods while the class keeps control of its private state.

Visibility is not a complete security boundary for an application, but it is an important way to express and enforce a class's intended API.

## Instance creation without a constructor

A class without a constructor can be instantiated without arguments:

~~~php
$book = new Book();
~~~

The object is created with its property defaults. You can then use its public API to set or inspect state.

A class may define a constructor to initialize each instance from arguments, but that topic is covered next. Until then, avoid reading an uninitialized typed property and prefer sensible defaults or methods that establish valid state.

## Dynamic properties

A dynamic property is a property assigned to an object even though it was never declared in the class:

~~~php
$book->publisherName = 'Example Press';
~~~

If `publisherName` was not declared on `Book`, this creates a dynamic property on that instance. In PHP 8.2 and later, creating dynamic properties on ordinary classes is deprecated unless the class explicitly allows them or implements property overloading.

Declare the property instead:

~~~php
class Book
{
    public string $publisherName = '';
}
~~~

Declaring properties makes the class's expected state easier to discover and helps catch misspelled property names. For objects intentionally designed to accept arbitrary properties, PHP has specific mechanisms; do not use accidental dynamic properties as a substitute for a defined design.

## Common mistakes

- Reading a typed property before assigning a value.
- Using a value that does not match a property's declared type.
- Forgetting parentheses when calling a method.
- Using `->` for an instance member without an object.
- Trying to access a private or protected member from outside its allowed scope.
- Making all properties public and allowing callers to bypass important rules.
- Assigning undeclared properties and relying on dynamic-property behavior.
- Expecting instance properties to be shared across all objects of the class.
- Confusing an instance property with a static property.
- Putting object initialization logic in arbitrary caller code when the next chapter's constructor may be a better fit.

## Key takeaways

- Properties hold object state; methods implement object behavior.
- Typed properties improve clarity, but an uninitialized typed property cannot be read.
- Use `new` to create an object and `->` to access its instance members.
- Inside an instance method, `$this` refers to the object receiving the call.
- Visibility controls where members can be accessed.
- Encapsulate state when changes need validation or other rules.
- Declare properties explicitly; dynamic property creation is deprecated on ordinary classes as of PHP 8.2.
- Constructors, which initialize objects from creation arguments, are covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [062 - Classes, objects, and object-oriented PHP](062-classes-objects-and-object-oriented-php.md) | [Open Q&A](../qnas/063-properties-methods-and-object-creation.md) | [Open example](../examples/063-properties-methods-and-object-creation.md) | [064 - Constructors and destructors](064-constructors-and-destructors.md) |
