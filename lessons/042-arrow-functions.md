# 042 - Arrow Functions

Arrow functions are a compact way to write short anonymous functions in PHP. They are useful when a callback has one clear expression, such as transforming values with **array_map()** or keeping items with **array_filter()**.

PHP introduced arrow functions in PHP 7.4. An arrow function is still an anonymous function value: you can assign it to a variable, pass it to another function, or return it from a function.

## What you will learn

- Write an arrow function with **fn** and **=>**
- Use parameters and return types
- Assign an arrow function to a variable and call it
- Pass an arrow function as a callback
- Understand automatic capture of outer variables by value
- Choose between an arrow function and a regular closure

## Basic syntax

The general form is:

~~~php
fn (parameters): returnType => expression
~~~

For example:

~~~php
$double = fn (int $number): int => $number * 2;

echo $double(6), "\n";
~~~

Output:

~~~text
12
~~~

The **fn** keyword begins the function, the parentheses contain its parameters, and **=>** separates the signature from the expression. PHP evaluates the expression and returns its result automatically.

The arrow function itself is an expression. When assigning it to a variable, finish the assignment with a semicolon.

## Parameters and return types

Arrow functions use the familiar parameter rules: parameter types, default values, and variadic parameters can be declared as needed. A return type can follow the parameter list:

~~~php
$greet = fn (string $name): string => "Hello, {$name}!";

echo $greet('Asha'), "\n";
~~~

Output:

~~~text
Hello, Asha!
~~~

An arrow function may also have no parameters. Keep the empty parentheses:

~~~php
$getMessage = fn (): string => 'Ready';

echo $getMessage(), "\n";
~~~

Every arrow function has one expression. Its value is returned automatically, so do not write a **return** statement or a block body.

## Call an arrow function

Once assigned, an arrow function is called like any other function value:

~~~php
$add = fn (int $left, int $right): int => $left + $right;

echo $add(3, 4), "\n";
~~~

Output:

~~~text
7
~~~

The parentheses in **$add(3, 4)** provide arguments to the arrow function's parameters. The arrow function returns the result of **$left + $right**.

## Use an arrow function as a callback

A callback is a function value passed to other code. Arrow functions work well when the callback is short and its result comes from one expression.

Use **array_map()** to transform each value:

~~~php
$numbers = [1, 2, 3, 4];

$doubled = array_map(
    fn (int $number): int => $number * 2,
    $numbers,
);

echo implode(', ', $doubled), "\n";
~~~

Output:

~~~text
2, 4, 6, 8
~~~

Use **array_filter()** to keep values that satisfy a condition:

~~~php
$numbers = [3, 8, 12, 17, 21];

$largeNumbers = array_filter(
    $numbers,
    fn (int $number): bool => $number >= 12,
);

echo implode(', ', $largeNumbers), "\n";
~~~

Output:

~~~text
12, 17, 21
~~~

The callback's parameter receives one value at a time. The returned value tells the array function what to do with that value.

## An expression can contain a condition

A single expression can include a ternary operator:

~~~php
$labelFor = fn (int $score): string =>
    $score >= 50 ? 'Pass' : 'Try again';

echo $labelFor(72), "\n";
echo $labelFor(35), "\n";
~~~

Output:

~~~text
Pass
Try again
~~~

It can also call another function:

~~~php
$normalize = fn (string $text): string => strtolower(trim($text));

echo $normalize('  PHP  '), "\n";
~~~

Output:

~~~text
php
~~~

The rule is about the function body: it is one expression, not necessarily a very short line. If an expression becomes difficult to read, extract a named function or use a regular closure with a body.

## Outer variables are captured automatically

Arrow functions automatically capture variables from the surrounding scope by value. There is no **use** clause:

~~~php
$taxRate = 0.08;

$withTax = fn (float $price): float => $price * (1 + $taxRate);

echo number_format($withTax(100), 2), "\n";
~~~

Output:

~~~text
108.00
~~~

The arrow function can read **$taxRate** even though it is not one of its parameters. Its value is captured when the arrow function is created.

By-value capture means a later reassignment of the outer variable does not change the value already captured:

~~~php
$taxRate = 0.08;
$withTax = fn (float $price): float => $price * (1 + $taxRate);

$taxRate = 0.10;

echo number_format($withTax(100), 2), "\n";
echo number_format($taxRate * 100, 0), "%\n";
~~~

Output:

~~~text
108.00
10%
~~~

The arrow function still uses the captured **0.08** value, while the outer variable now contains **0.10**.

## Compare with a regular closure

A regular anonymous function must explicitly list an outer variable with **use**:

~~~php
$taxRate = 0.08;

$withTax = function (float $price) use ($taxRate): float {
    return $price * (1 + $taxRate);
};
~~~

The equivalent arrow function captures the same variable automatically:

~~~php
$taxRate = 0.08;

$withTax = fn (float $price): float => $price * (1 + $taxRate);
~~~

Choose an arrow function when its single expression is clear. Choose a regular closure when you need a multi-statement body, an explicit capture list, or a variable captured by reference:

~~~php
$count = 0;

$nextCount = function () use (&$count): int {
    $count++;
    return $count;
};
~~~

Arrow functions capture outer variables by value; they do not provide a **use (&$variable)** form. Use a regular closure when the function needs to change a surrounding variable.

## Nested arrow functions

An arrow function can return another arrow function. Each one can capture values from the scope where it is created:

~~~php
$makeMultiplier = fn (int $factor): Closure =>
    fn (int $number): int => $number * $factor;

$triple = $makeMultiplier(3);

echo $triple(7), "\n";
~~~

Output:

~~~text
21
~~~

Here **$factor** is available to the inner arrow function through automatic by-value capture. In everyday code, use nesting only when it makes the relationship easier to understand.

## Arrow functions and other PHP syntax

- **fn** starts an arrow function.
- **=>** separates the function signature from its expression. It also appears in array key/value syntax, but the surrounding syntax tells PHP what it means.
- An arrow function is a function value, so it can be stored, passed, or returned.
- Arrow functions are also closures; automatic variable capture does not make them named functions.
- Arrow function syntax requires PHP 7.4 or later.
- Arrow functions are different from first-class callable syntax. A later chapter covers turning an existing function into a callable value with syntax such as **strlen(...)**.

## Common mistakes

- Forgetting the parentheses for a zero-parameter function: write **fn () => expression**.
- Writing **fn ($value) { ... }**. Arrow functions use **=> expression**, not a brace-delimited body.
- Writing **return** inside an arrow function. The expression's value is returned automatically.
- Expecting to put multiple statements in an arrow function.
- Adding a **use** clause. Arrow functions capture outer variables automatically.
- Expecting a later outer reassignment to change a captured value.
- Expecting an arrow function to update an outer scalar variable. Use a regular closure with **use (&$value)** for that.
- Choosing a very long expression when a regular closure or named function would be clearer.
- Using arrow-function syntax on a PHP version older than 7.4.

## Try it yourself

1. Assign an arrow function to **$square** that accepts an integer and returns its square.
2. Pass an arrow function to **array_map()** to add five to every number in an array.
3. Use **array_filter()** and an arrow function to keep only strings longer than three characters.
4. Create **$message** before an arrow function, then reassign **$message** afterward. Predict which value the arrow function returns.
5. Rewrite a short regular closure that uses **use ($rate)** as an arrow function.
6. Make a regular closure that increments an outer counter by reference. Explain why the arrow-function form is not the right tool for that job.

## Key takeaways

- The basic syntax is **fn (parameters) => expression**.
- An arrow function returns its single expression automatically.
- Parameters and return types work in the familiar way.
- Arrow functions can be stored in variables or passed as callbacks.
- Variables from the surrounding scope are captured automatically by value.
- A regular closure is a better fit for multiple statements, explicit captures, or by-reference capture.
- Arrow functions require PHP 7.4 or later.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [041 - Anonymous Functions and Closures](041-anonymous-functions-and-closures.md) | [Open Q&A](../qnas/042-arrow-functions.md) | [Open example](../examples/042-arrow-functions.md) | [043 - Callable Values and First-Class Callable Syntax](043-callable-values-and-first-class-callable-syntax.md) |
