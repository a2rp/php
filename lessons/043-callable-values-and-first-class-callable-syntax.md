# 043 - Callable Values and First-Class Callable Syntax

A callback is behavior passed to another part of a program. PHP can represent that behavior in several ways: a function name, a method reference, an object with an **__invoke()** method, or a **Closure** object.

First-class callable syntax, added in PHP 8.1, provides a clear way to turn an existing function or method into a **Closure** value. For example, **strlen(...)** creates a callable value that can be stored, passed, or called later.

## What you will learn

- Understand what PHP means by a callable
- Recognize common callable representations
- Accept a callback with the **callable** type declaration
- Create a **Closure** from an existing function or method with first-class callable syntax
- Pass and invoke callable values
- Distinguish a callable value from calling the function immediately
- Know when to use **Closure::fromCallable()** and **is_callable()**

## What is a callable?

A callable is a value that PHP can invoke as a function. A callback is a callable supplied to another function or method.

For example, **trim** is a built-in function. It can be named as a callback and then invoked:

~~~php
<?php
$callback = 'trim';

echo $callback('  PHP  '), "\n";
~~~

Output:

~~~text
PHP
~~~

The variable **$callback** contains the function's name as a string. Calling **$callback(...)** invokes that function.

Other values can also be callable. The exact forms depend on whether the target is a function, an object method, a static method, or an invokable object.

## Common callable forms

### A function name

A string can name a globally available function:

~~~php
$callback = 'trim';

echo $callback('  hello  '), "\n";
~~~

Output:

~~~text
hello
~~~

Use a function name string when it is useful to select a function by name. For a modern first-class reference to the function, PHP 8.1+ also supports **trim(...)**, shown later.

### An object method

A two-item array can refer to an object and one of its methods:

~~~php
<?php
class TextFormatter
{
    public function uppercase(string $text): string
    {
        return strtoupper($text);
    }
}

$formatter = new TextFormatter();
$callback = [$formatter, 'uppercase'];

echo $callback('hello'), "\n";
~~~

Output:

~~~text
HELLO
~~~

The first array item is the object. The second is the method name.

### A static method

A two-item array can also refer to a class and a static method:

~~~php
<?php
class Label
{
    public static function make(string $text): string
    {
        return '[' . $text . ']';
    }
}

$callback = [Label::class, 'make'];

echo $callback('ready'), "\n";
~~~

Output:

~~~text
[ready]
~~~

**Label::class** evaluates to the class name as a string. The referenced method must be callable in the scope where the callback is used.

### A Closure object

Anonymous functions and arrow functions create **Closure** objects:

~~~php
$callback = fn (string $text): string => strtoupper($text);

echo $callback('php'), "\n";
~~~

Output:

~~~text
PHP
~~~

The callable value is the closure itself.

### An invokable object

An object can be callable when its class defines the **__invoke()** method:

~~~php
<?php
class Prefixer
{
    public function __construct(
        private string $prefix,
    ) {
    }

    public function __invoke(string $text): string
    {
        return $this->prefix . $text;
    }
}

$callback = new Prefixer('Note: ');

echo $callback('read the chapter'), "\n";
~~~

Output:

~~~text
Note: read the chapter
~~~

Calling **$callback(...)** invokes the object's **__invoke()** method.

## Accept a callback with the callable type

A function or method can declare a parameter as **callable** when it expects behavior from its caller:

~~~php
<?php
function runCallback(callable $callback, string $value): string
{
    return $callback($value);
}

$result = runCallback('trim', '  ready  ');

echo $result, "\n";
~~~

Output:

~~~text
ready
~~~

The type declaration checks that the supplied value is callable. It does not specify the callback's complete parameter or return-type signature. Document those expectations and, where useful, validate behavior in the function that invokes the callback.

The **callable** type is a parameter or return type, not a general-purpose property type. If a class property must specifically hold a closure object, declare it as **Closure**:

~~~php
private Closure $formatter;
~~~

Add **use Closure;** if the class is in a namespace and refers to the global **Closure** class by its short name.

## Create a first-class callable

PHP 8.1 introduced first-class callable syntax. Put **(...)** after an existing callable expression to create a **Closure** value:

~~~php
<?php
$length = strlen(...);

echo $length('elephant'), "\n";
~~~

Output:

~~~text
8
~~~

The ellipsis is part of this syntax. It is not a placeholder for omitted code or a set of arguments to pass now. The expression creates a closure that refers to **strlen**. The function runs later, when **$length** is called.

First-class callable syntax works with ordinary functions, instance methods, static methods, and invokable objects:

~~~php
<?php
class Formatter
{
    public function upper(string $text): string
    {
        return strtoupper($text);
    }

    public static function surround(string $text): string
    {
        return '[' . $text . ']';
    }
}

$formatter = new Formatter();

$uppercase = $formatter->upper(...);
$surround = Formatter::surround(...);

echo $uppercase('php'), "\n";
echo $surround('ready'), "\n";
~~~

Output:

~~~text
PHP
[ready]
~~~

Each expression creates a **Closure**. The closure can be assigned to a variable, passed to a callback parameter, or returned to another part of the program.

## Pass a first-class callable to a callback function

First-class callable syntax is useful when an existing function already performs the operation you need:

~~~php
<?php
$words = ['  alpha ', ' beta  ', ' gamma '];

$cleanWords = array_map(trim(...), $words);

echo implode(', ', $cleanWords), "\n";
~~~

Output:

~~~text
alpha, beta, gamma
~~~

**trim(...)** creates a closure for the existing **trim()** function. **array_map()** calls that closure for every array item.

Compare it with an arrow function:

~~~php
$cleanWords = array_map(
    fn (string $word): string => trim($word),
    $words,
);
~~~

Use **trim(...)** when you want to pass the existing function unchanged. Use an arrow function when you need to add, transform, or combine behavior.

## First-class callable syntax is not a function call

Parentheses with actual arguments call the function now:

~~~php
$length = strlen('elephant');
~~~

Parentheses containing only the callable ellipsis create a closure for later:

~~~php
$lengthOf = strlen(...);
$result = $lengthOf('elephant');
~~~

The first expression stores the integer result. The second stores a callable value, then invokes it with a string.

This syntax does not pre-fill arguments. Binding initial arguments for later use is a separate feature called partial function application.

## First-class callable syntax and Closure::fromCallable()

Before PHP 8.1, **Closure::fromCallable()** could turn a callable into a **Closure**:

~~~php
$lengthOf = Closure::fromCallable('strlen');
~~~

With PHP 8.1+, the equivalent first-class callable form is shorter:

~~~php
$lengthOf = strlen(...);
~~~

Both create a closure for the existing function. Use **Closure::fromCallable()** when supporting an older PHP version that has the method but not first-class callable syntax.

## Scope and method visibility

A first-class callable uses the scope in which the callable is created. This lets an object method create a closure for a private method and return that closure to code outside the class:

~~~php
<?php
class SecretFormatter
{
    public function formatter(): Closure
    {
        return $this->formatSecret(...);
    }

    private function formatSecret(string $text): string
    {
        return 'Secret: ' . $text;
    }
}

$format = (new SecretFormatter())->formatter();

echo $format('hello'), "\n";
~~~

Output:

~~~text
Secret: hello
~~~

The method is private, but the first-class callable is acquired from inside the class where it is accessible. The returned closure keeps the scope needed to call that method.

Do not expose private methods this way unless the class is intentionally granting that behavior.

## Check whether a value is callable

Use **is_callable()** when a value is dynamic and you need a runtime check:

~~~php
$callback = 'trim';

if (is_callable($callback)) {
    echo $callback('  checked  '), "\n";
}
~~~

Output:

~~~text
checked
~~~

A successful check means PHP recognizes the value as callable in the current context. It does not prove that calling it with a particular set of arguments will succeed. The target can still reject the arguments or raise an error.

For a callback parameter that is required, a **callable** type declaration is usually clearer than checking manually at every call site.

## Common mistakes

- Writing **strlen()** when you mean to create a callable reference. Use **strlen(...)**.
- Treating the **...** in **strlen(...)** as optional omitted text. It is required syntax.
- Expecting first-class callable syntax to run the function immediately.
- Expecting it to store arguments to be supplied later. It stores a reference to the callable, not pre-filled arguments.
- Confusing a **callable** value with a **Closure** object. Some callable forms are strings, arrays, or invokable objects.
- Using a method callback that is not accessible from the scope where it is created or called.
- Assuming **is_callable()** guarantees that every future invocation will accept any arguments.
- Trying to use **callable** as a class property type.
- Using first-class callable syntax on PHP earlier than 8.1.
- Assuming language constructs such as **echo** are ordinary functions that can be referenced this way.

## Try it yourself

1. Create a first-class callable for **strtolower()** and pass it to **array_map()**.
2. Add a **callable** parameter to a function that applies a callback to a string.
3. Create a first-class callable for an instance method and call the resulting closure.
4. Create a first-class callable for a public static method.
5. Compare **strtoupper(...)** with **fn (string $text): string => strtoupper($text)**.
6. Use **is_callable()** on a function-name string and on a non-callable integer.
7. Create a private method and return its first-class callable from a public method. Check which scope creates the callable.
8. Explain the difference between **strlen('PHP')** and **strlen(...)**.

## Key takeaways

- A callable is any value PHP can invoke; a **Closure** is one particular callable object.
- Function names, method arrays, closures, and invokable objects can represent callbacks.
- Use **callable** to type-hint a callback parameter.
- PHP 8.1+ uses **existingFunction(...)** or **$object->method(...)** to create a closure for an existing callable.
- The ellipsis is required and the target function is called only when the resulting closure is invoked.
- First-class callable syntax creates a **Closure** and uses the scope where it is acquired.
- Use **is_callable()** for dynamic runtime checks and **Closure::fromCallable()** when that conversion is more suitable.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [042 - Arrow Functions](042-arrow-functions.md) | [Open Q&A](../qnas/043-callable-values-and-first-class-callable-syntax.md) | [Open example](../examples/043-callable-values-and-first-class-callable-syntax.md) | [044 - Partial Function Application](044-partial-function-application.md) |
