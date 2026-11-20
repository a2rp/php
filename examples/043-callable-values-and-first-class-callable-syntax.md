# 043 - Callable Values and First-Class Callable Syntax: Example

## Question

How can one reusable function apply different callable values to a list of messages?

## Goal

Build a small text-processing pipeline that uses:

- A first-class callable for the built-in **trim()** function
- A first-class callable for an instance method
- A first-class callable for a static method
- An invokable object as a callback
- A **callable** parameter type
- **is_callable()** to check both modern and older callback forms

## Project structure

~~~text
callable-values-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**. Run it with PHP 8.1 or later because it uses first-class callable syntax:

~~~php
<?php

class TextDecorator
{
    private string $prefix;

    public function __construct(string $prefix)
    {
        $this->prefix = $prefix;
    }

    public function decorate(string $text): string
    {
        return $this->prefix . strtoupper($text);
    }

    public static function lowercase(string $text): string
    {
        return strtolower($text);
    }
}

class AddExclamation
{
    public function __invoke(string $text): string
    {
        return $text . '!';
    }
}

function transformAll(array $values, callable $callback): array
{
    return array_map($callback, $values);
}

$messages = [
    '  Callables  ',
    ' First-class syntax ',
];

$decorator = new TextDecorator('PHP: ');

$trimText = trim(...);
$decorate = $decorator->decorate(...);
$lowercase = TextDecorator::lowercase(...);
$addExclamation = new AddExclamation();

$cleanMessages = transformAll($messages, $trimText);
$decoratedMessages = transformAll($cleanMessages, $decorate);
$lowercaseMessages = transformAll($decoratedMessages, $lowercase);
$finalMessages = transformAll($lowercaseMessages, $addExclamation);

echo "Final messages:\n";

foreach ($finalMessages as $message) {
    echo $message, "\n";
}

echo 'First-class references are Closure objects: ';
echo ($trimText instanceof Closure && $decorate instanceof Closure
    && $lowercase instanceof Closure) ? "yes\n" : "no\n";

echo 'Invokable object is callable: ';
echo is_callable($addExclamation) ? "yes\n" : "no\n";

$legacyTrim = 'trim';
$legacyDecorate = [$decorator, 'decorate'];

echo 'Older callback forms are callable: ';
echo (is_callable($legacyTrim) && is_callable($legacyDecorate))
    ? "yes\n"
    : "no\n";
~~~

## Expected output

~~~text
Final messages:
PHP: callables!
PHP: first-class syntax!
First-class references are Closure objects: yes
Invokable object is callable: yes
Older callback forms are callable: yes
~~~

## Explanation

### Pass a callback to one reusable function

~~~php
function transformAll(array $values, callable $callback): array
{
    return array_map($callback, $values);
}
~~~

The **callable** parameter accepts any value that PHP can invoke. **transformAll()** does not need to know whether the callback is a function, a method, a closure, or an invokable object. It passes the callback to **array_map()**, which calls it once for each item.

### Create a callable for a built-in function

~~~php
$trimText = trim(...);
~~~

This first-class callable expression creates a **Closure** for the existing **trim()** function. **transformAll()** invokes it later for each message. The function is not run when **trim(...)** is evaluated.

An older form is also shown near the end:

~~~php
$legacyTrim = 'trim';
~~~

This string names the function and is callable when **trim()** is available.

### Create a callable for an instance method

~~~php
$decorate = $decorator->decorate(...);
~~~

The closure refers to the **decorate()** method on this **TextDecorator** object. The method keeps access to the object and its **$prefix** property. It adds the prefix and uppercases the message.

The older object-method representation is:

~~~php
$legacyDecorate = [$decorator, 'decorate'];
~~~

Both forms refer to the same method, but first-class callable syntax creates a **Closure** and makes the reference explicit.

### Create a callable for a static method

~~~php
$lowercase = TextDecorator::lowercase(...);
~~~

The scope-resolution operator identifies the static method. This also creates a closure that can be passed as a callback.

### Pass an invokable object

~~~php
$addExclamation = new AddExclamation();
~~~

**AddExclamation** defines **__invoke()**, so its object can be passed as a callable. **array_map()** calls that method for each message.

The expression **is_callable($addExclamation)** checks that PHP recognizes the object as callable.

### Read the pipeline from top to bottom

Each call to **transformAll()** applies one behavior to every message:

1. **trim(...)** removes surrounding whitespace.
2. **$decorator->decorate(...)** adds **PHP: ** and uppercases the text.
3. **TextDecorator::lowercase(...)** converts the decorated text to lowercase.
4. **$addExclamation** appends an exclamation mark.

The resulting strings are printed after all four transformations.

The checks also show that first-class callable expressions create **Closure** objects, that the **__invoke()** object is callable, and that older function-string and method-array forms remain callable.

## Make a change

1. Add another message to **$messages** and follow it through the pipeline.
2. Change the prefix passed to **TextDecorator**.
3. Remove the lowercase transformation and compare the output.
4. Replace **trim(...)** with **strtolower(...)** and observe why operation order matters.
5. Change **AddExclamation** to append a period instead.
6. Pass **strtoupper(...)** directly to **transformAll()** and compare it with the instance-method callback.

## Common mistakes

- Writing **trim()** when the goal is to create a callable. **trim()** runs immediately; **trim(...)** creates a closure.
- Omitting the required ellipsis from first-class callable syntax.
- Passing an invalid callback to **transformAll()**, whose **callable** parameter requires an invokable value.
- Assuming a function-name string or method array is already a **Closure** object.
- Calling **$decorator->decorate()** without a message when you only meant to acquire a reference.
- Forgetting that the object must define **__invoke()** to be directly callable.
- Running the file on PHP earlier than 8.1.
- Expecting **is_callable()** to validate all arguments that may later be passed to the callback.

## Key takeaways

- A **callable** parameter lets a function accept different kinds of callback values.
- **trim(...)**, **$object->method(...)**, and **ClassName::method(...)** create first-class callable closures on PHP 8.1+.
- The callable is executed later when code invokes it.
- An object with **__invoke()** can be used directly as a callback.
- Function-name strings and method arrays are older callable forms and remain useful when compatibility requires them.
- **is_callable()** checks whether a value is callable in the current context.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [042 - Arrow Functions](../lessons/042-arrow-functions.md) | [Read lesson](../lessons/043-callable-values-and-first-class-callable-syntax.md) | [Open Q&A](../qnas/043-callable-values-and-first-class-callable-syntax.md) | [044 - Partial Function Application](../lessons/044-partial-function-application.md) |
