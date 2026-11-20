# 044 - Partial Function Application

Partial function application creates a new callable by supplying some arguments now and leaving placeholders for arguments that will arrive later. PHP 8.6 introduced this syntax for functions, methods, and other callable expressions.

Use **?** for one argument that the returned closure will require. Use **...** for the remaining arguments that should be passed through when that closure is called. The original function is not executed until the returned closure is invoked.

## What you will learn

- Create a closure by supplying some function arguments and leaving placeholders
- Understand the difference between **?** and **...**
- Use partial application to adapt a function for **array_map()** and other callbacks
- Use named placeholders to reorder a closure's parameters
- Know when provided arguments are evaluated
- Avoid syntax combinations that PHP does not support

## Basic idea

Suppose a function expects two strings:

~~~php
function greet(string $greeting, string $name): string
{
    return "{$greeting}, {$name}!";
}
~~~

You can supply the greeting now and leave the name for later:

~~~php
$greetWithHello = greet('Hello', ?);

echo $greetWithHello('Asha'), "\n";
echo $greetWithHello('Mina'), "\n";
~~~

Output:

~~~text
Hello, Asha!
Hello, Mina!
~~~

The expression **greet('Hello', ?)** does not call **greet()** immediately. It creates a **Closure** that stores **'Hello'** and has one required parameter for the **?** placeholder.

Calling **$greetWithHello('Asha')** fills that placeholder and then calls **greet()**.

## The single-argument placeholder: ?

A question mark stands for exactly one argument. Each **?** becomes one parameter of the returned closure:

~~~php
function volume(int $length, int $width, int $height): int
{
    return $length * $width * $height;
}

$volumeForLengthTwo = volume(2, ?, ?);

echo $volumeForLengthTwo(3, 4), "\n";
~~~

Output:

~~~text
24
~~~

The first closure parameter fills **$width**; the second fills **$height**. PHP calls **volume(2, 3, 4)** when the closure is invoked.

A placeholder makes that closure parameter required even if the corresponding parameter in the original function has a default value. The placeholder means one argument must be supplied to the new closure.

## The remaining-arguments placeholder: ...

The **...** placeholder stands for all parameters not already supplied or represented by earlier placeholders. It can appear at most once and must be the final argument:

~~~php
$volumeForLengthTwo = volume(2, ...);

echo $volumeForLengthTwo(3, 4), "\n";
~~~

Output:

~~~text
24
~~~

The closure accepts the remaining width and height, then forwards them to **volume()**.

For example, if the original function has optional parameters, **...** preserves their normal defaults when the closure is called without those arguments.

## Delay a call until later

If every original argument is supplied and **...** appears last, PHP creates a closure that takes no additional arguments. The function runs only when the closure is called:

~~~php
$calculate = volume(2, 3, 4, ...);

echo "The closure has been created.\n";
echo $calculate(), "\n";
~~~

Output:

~~~text
The closure has been created.
24
~~~

The target function is not called when **$calculate** is created. It runs when **$calculate()** is invoked.

This is useful when a callback or another part of a program expects a callable, but the operation should happen later.

## Use a partially applied function as a callback

Many callbacks expect a function that accepts one value. Partial application can adapt a function that normally needs more arguments:

~~~php
$words = ['red apple', 'green pear', 'blue berry'];

$removeSpaces = str_replace(' ', '', ?);

$compactWords = array_map($removeSpaces, $words);

echo implode(', ', $compactWords), "\n";
~~~

Output:

~~~text
redapple, greenpear, blueberry
~~~

The first two arguments to **str_replace()** are supplied when the closure is created. The **?** leaves the subject string for **array_map()** to provide for each item.

The same idea works with a callback that needs a fixed configuration value:

~~~php
function applyTax(float $rate, float $price): float
{
    return $price * (1 + $rate);
}

$withEightPercentTax = applyTax(0.08, ?);

$prices = [10, 25, 5];
$totals = array_map($withEightPercentTax, $prices);
~~~

Here **array_map()** supplies the price while the closure keeps the tax rate.

## Use named placeholders to choose parameter order

Named arguments can provide fixed values, and a placeholder can name an argument that should be supplied later:

~~~php
$remove = str_replace(
    subject: ?,
    search: ?,
    replace: '',
);

echo $remove('a-b-c', '-'), "\n";
~~~

Output:

~~~text
abc
~~~

The resulting closure's parameters follow the order in which the named placeholders appear: first **$subject**, then **$search**. This lets you shape the callback's parameter order while still naming the original function parameters.

As with ordinary named arguments, positional arguments must come before named arguments.

## Additional arguments and ...

When the closure is created without a final **...** placeholder, arguments passed beyond its own parameters are ignored. With **...**, extra arguments are forwarded to the original callable.

~~~php
function greetWithDefault(string $name, string $greeting = 'Hello'): string
{
    return "{$greeting}, {$name}!";
}

$onePlaceholder = greetWithDefault(?);

echo $onePlaceholder('Asha', 'Goodbye'), "\n";

$forwardRemaining = greetWithDefault(?, ...);

echo $forwardRemaining('Asha', 'Goodbye'), "\n";
~~~

Output:

~~~text
Hello, Asha!
Goodbye, Asha!
~~~

The first closure has one parameter. The extra **'Goodbye'** argument is ignored, so the original function uses its default greeting. The second closure forwards remaining arguments, so **'Goodbye'** becomes the greeting.

This behavior matters when a callback provider may pass more values than the closure explicitly requires.

## Provided expressions are evaluated once

Arguments supplied when the closure is created are evaluated at that time, not on every later call:

~~~php
function currentGreeting(): string
{
    echo "Greeting selected.\n";
    return 'Welcome';
}

function greetAtTime(string $greeting, string $name): string
{
    return "{$greeting}, {$name}!";
}

$welcome = greetAtTime(currentGreeting(), ?);

echo "Closure created.\n";
echo $welcome('Asha'), "\n";
echo $welcome('Mina'), "\n";
~~~

Output:

~~~text
Greeting selected.
Closure created.
Welcome, Asha!
Welcome, Mina!
~~~

**currentGreeting()** runs once while **$welcome** is created. Its returned string is stored in the closure. By comparison, an arrow function such as **fn ($name) => greetAtTime(currentGreeting(), $name)** would call **currentGreeting()** each time the arrow function runs.

## The returned closure keeps the callable

Partial application determines the target function or method when the closure is created:

~~~php
$length = strlen(?);

echo $length('PHP'), "\n";
~~~

The closure already refers to **strlen()**. You supply the string later.

This builds on first-class callable syntax. With no provided arguments, **strlen(...)** creates a callable reference with all of the original function's parameters left for later. With partial application, you can also provide some arguments and place placeholders for the rest.

## Closures preserve parameter information

The returned closure's placeholder parameters use the corresponding parameter names, types, and by-reference behavior from the original callable. Its return type also follows the target callable.

This helps the resulting closure behave like a callback tailored to the remaining arguments.

## Restrictions

- Partial function application requires PHP 8.6 or later.
- **?** stands for exactly one argument.
- **...** may appear at most once and must be last.
- Do not combine partial-application placeholders with argument unpacking such as **...$values**.
- Do not combine placeholders with the nullsafe operator.
- Constructor expressions such as **new Example(?, ...)** are not supported.
- Functions that inspect the calling scope, including **func_get_args()**, **compact()**, **extract()**, and **get_defined_vars()**, cannot be used with partial application.
- There must be a valid number of supplied arguments and placeholders for the target callable.
- Positional arguments must precede named arguments.

## Common mistakes

- Writing a normal function call with **?** and expecting it to run immediately.
- Forgetting that **?** creates a required parameter in the returned closure, even when the original parameter had a default.
- Using **...** before another argument or using it more than once.
- Confusing the partial-application **...** placeholder with argument unpacking **...$array**.
- Expecting provided expressions to run each time the returned closure is called.
- Forgetting **...** when additional arguments need to reach optional or variadic parameters of the original function.
- Using partial function application on a PHP version older than 8.6.
- Trying to partially apply a constructor with **new**.

## Try it yourself

1. Partially apply **greet()** with a fixed greeting and a placeholder for the name.
2. Use **volume(2, ?, ?)** to create a closure that calculates volumes for a fixed length.
3. Use **array_map()** with **str_replace(' ', '', ?)** to remove spaces from several strings.
4. Compare **greetWithDefault(?)** with **greetWithDefault(?, ...)** when the returned closure receives two arguments.
5. Use named placeholders with **str_replace()** to choose a useful closure parameter order.
6. Supply an expression that prints a message as a fixed argument and observe when it is evaluated.
7. Add a variadic parameter to a target function and pass remaining values through **...**.

## Key takeaways

- Partial function application creates a closure from an existing callable and some supplied arguments.
- **?** leaves exactly one required argument for the returned closure.
- **...** forwards all remaining arguments and must be last.
- The target function runs only when the returned closure is invoked.
- Fixed argument expressions are evaluated when the closure is created.
- Named placeholders can control the order of the resulting closure's parameters.
- The feature requires PHP 8.6 or later.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [043 - Callable Values and First-Class Callable Syntax](043-callable-values-and-first-class-callable-syntax.md) | [Open Q&A](../qnas/044-partial-function-application.md) | [Open example](../examples/044-partial-function-application.md) | [045 - Passing and Returning Values by Reference](045-passing-and-returning-values-by-reference.md) |
