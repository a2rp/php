# 038 - Function Parameters and Arguments

A parameter is a named variable in a function declaration. An argument is the value or expression supplied by a caller. Parameters let one function work with different input each time it is called.

This chapter covers required, positional arguments and typed parameters. Defaults, named arguments, variadic arguments, and argument unpacking are covered in the next chapter. Passing and returning by reference is covered later.

## What you will learn

- Tell parameters and arguments apart
- Declare one or more parameters in a function signature
- Pass values and expressions when calling a function
- Match positional arguments to parameters in order
- Use a parameter as a local variable inside the function
- Understand the default pass-by-value behavior for scalar values
- Recognize missing-argument and type errors

## Parameters and arguments

The function declaration contains **parameters**. The function call supplies **arguments**.

~~~php
<?php
function greet(string $name): void
{
    echo "Hello, {$name}!\n";
}

greet('Asha');
~~~

Output:

~~~text
Hello, Asha!
~~~

In the declaration, **$name** is a parameter: it names a value the function expects. In the call, **'Asha'** is the argument: it is the value supplied for that parameter.

A useful way to remember the terms:

- **Parameter:** the receiving variable written in the function declaration.
- **Argument:** the value or expression written in the function call.

## Declare a parameter

A parameter is written inside the parentheses in the function signature. A type may be placed before its variable name:

~~~php
function greet(string $name): void
{
    echo "Hello, {$name}!\n";
}
~~~

Here, **string** is the parameter's type declaration and **$name** is its name. The function body can use **$name** like a local variable. This example uses a type declaration; PHP's scalar coercion behavior depends on strict typing. Review [chapter 019 on type declarations and strict typing](019-type-declarations-and-strict-typing.md) for those rules.

A function may declare multiple parameters, separated by commas:

~~~php
function printItem(string $name, int $quantity, float $unitPrice): void
{
    $lineTotal = $quantity * $unitPrice;
    echo "{$name}: {$quantity} items; line total {$lineTotal}\n";
}
~~~

Each parameter gets its own name and receives one argument in the call.

## Pass positional arguments

With positional arguments, PHP matches each argument to a parameter by its position:

~~~php
<?php
function describeBook(string $title, string $author, int $year): void
{
    echo "{$title} by {$author} ({$year})\n";
}

describeBook('The Long Road', 'Mina Shah', 2024);
~~~

Output:

~~~text
The Long Road by Mina Shah (2024)
~~~

The first argument, **'The Long Road'**, is assigned to **$title**. The second is assigned to **$author**, and the third to **$year**. Keep the call's order aligned with the declaration.

A call can supply variables, literals, or other expressions as arguments:

~~~php
<?php
function showNextScore(int $score): void
{
    echo "Next score: " . ($score + 1) . "\n";
}

$currentScore = 9;
showNextScore($currentScore);
showNextScore(4 + 7);
~~~

Output:

~~~text
Next score: 10
Next score: 12
~~~

PHP evaluates the argument expressions from left to right before entering the function body. The resulting values are then assigned to the corresponding parameters.

## Required parameters

A parameter with no default value is required: its caller must supply an argument for it.

~~~php
<?php
function printCoordinates(int $x, int $y): void
{
    echo "x={$x}, y={$y}\n";
}

printCoordinates(12, 5);
~~~

Output:

~~~text
x=12, y=5
~~~

If a call omits a required argument, PHP raises an **ArgumentCountError**. Supply the values the function requires. Optional parameters with defaults are the subject of [chapter 039](039-default-named-variadic-and-unpacked-arguments.md).

## Parameters are local to the function

A parameter is a local variable belonging to the function call. It is available inside the function body, not automatically in the caller's scope:

~~~php
<?php
function showLabel(string $label): void
{
    echo "Inside: {$label}\n";
}

$label = 'Outside';
showLabel('Inside value');
echo "Caller: {$label}\n";
~~~

Output:

~~~text
Inside: Inside value
Caller: Outside
~~~

The function's **$label** parameter and the caller's **$label** variable have the same spelling, but they belong to different scopes. The argument supplies a value to the parameter; it does not make the caller's variable the function's local variable.

Prefer passing a function the values it needs through parameters instead of relying on hidden outside state. Passing a value into a parameter does not by itself send a result back; return values are covered separately.

## PHP passes arguments by value by default

By default, a function receives a value. Assigning a new scalar value to the parameter does not reassign the caller's variable:

~~~php
<?php
function useTemporaryScore(int $score): void
{
    $score = 100;
    echo "Inside: {$score}\n";
}

$score = 7;
useTemporaryScore($score);
echo "Caller: {$score}\n";
~~~

Output:

~~~text
Inside: 100
Caller: 7
~~~

The parameter starts with the argument's value. Reassigning the local parameter changes that local variable, not **$score** in the caller. PHP may optimize how values are stored internally; the behavior to rely on is that ordinary by-value assignment does not reassign the caller's scalar variable.

Reference parameters explicitly allow a function to work with the caller's variable itself. They change the call's behavior and are covered in [chapter 045](045-passing-and-returning-values-by-reference.md). Use them only when that shared modification is intentional.

## Type declarations on parameters

A parameter can declare the kind of value it expects:

~~~php
<?php
function repeatLabel(string $label, int $times): void
{
    for ($count = 0; $count < $times; $count++) {
        echo $label . "\n";
    }
}

repeatLabel('Practice', 2);
~~~

Output:

~~~text
Practice
Practice
~~~

A type declaration helps document intent and lets PHP check arguments. If an argument is not accepted for the declared type, PHP can raise a **TypeError**. For scalar types such as **string**, **int**, and **float**, whether PHP coerces a compatible value or rejects it depends on strict typing in the calling file. See [chapter 019](019-type-declarations-and-strict-typing.md) for the full rules.

## Common mistakes

- Calling a parameter an argument or an argument a parameter when describing the code.
- Declaring **$name** but forgetting to supply its required argument.
- Reversing positional arguments so values reach the wrong parameters.
- Expecting a parameter to be visible as a local variable in the caller.
- Expecting reassignment of a scalar parameter to change the caller's variable.
- Assuming type declarations always reject every value of a different scalar type; strict typing affects scalar coercion.
- Confusing a function's printed output with a returned value.
- Adding defaults, named arguments, or **...** syntax before understanding the basic positional mapping; those are covered in chapter 039.

## Try it yourself

1. Define **showGreeting(string $name)** and call it with two different names.
2. Define **printRectangle(int $width, int $height)** and print both values in a message.
3. Swap the arguments in a two-parameter call. Notice why order matters.
4. Pass a variable as an argument, then reassign the same variable inside the function's parameter. Print the caller's variable afterward.
5. Remove one required argument and observe PHP's error.
6. Change an argument to a value incompatible with its declared parameter type and observe the type error under your file's strict-typing mode.

## Key takeaways

- Parameters are variables declared in a function signature; arguments are values or expressions supplied by a caller.
- Positional arguments are assigned to parameters from left to right.
- A parameter is local to its function call.
- Required parameters must receive arguments.
- PHP passes arguments by value by default; reassigning a scalar parameter does not reassign the caller's variable.
- Type declarations document and check parameter values; strict typing affects scalar coercion.
- Defaults, named arguments, variadic arguments, and unpacking are covered next.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [037 - Defining and Calling Functions](037-defining-and-calling-functions.md) | [Open Q&A](../qnas/038-function-parameters-and-arguments.md) | [Open example](../examples/038-function-parameters-and-arguments.md) | [039 - Default, Named, Variadic, and Unpacked Arguments](039-default-named-variadic-and-unpacked-arguments.md) |
