# 039 - Default, Named, Variadic, and Unpacked Arguments

PHP gives functions several ways to make arguments more convenient. A parameter can have a default value, a caller can name an argument, a variadic parameter can collect extra values, and the **...** unpacking syntax can spread an array into a call.

Use these forms when they make a function easier to call and understand. The basic parameter and positional argument rules from [chapter 038](038-function-parameters-and-arguments.md) still apply.

## What you will learn

- Define and use default parameter values
- Tell an omitted argument apart from an explicit **null**
- Place required and optional parameters in a valid order
- Pass named arguments in PHP 8 and later
- Combine positional and named arguments safely
- Collect a variable number of arguments in a variadic parameter
- Unpack an array or iterable at a call site
- Distinguish **...** in a declaration from **...** in a call

## Default parameter values

A parameter can specify a default value after **=**. PHP uses that default when the caller omits the argument:

~~~php
<?php
function greet(string $name, string $greeting = 'Hello'): void
{
    echo "{$greeting}, {$name}!\n";
}

greet('Asha');
greet('Mina', 'Welcome');
~~~

Output:

~~~text
Hello, Asha!
Welcome, Mina!
~~~

The first call omits **$greeting**, so PHP uses **'Hello'**. The second call supplies a value, so it overrides the default.

A parameter with a default is optional at the call site. A parameter without one is required. Put required parameters before optional ones:

~~~php
function describeItem(string $name, int $quantity = 1): void
{
    echo "{$quantity} x {$name}\n";
}
~~~

Here, the caller must provide **$name**, but can omit **$quantity**.

Declaring an optional parameter before a required one is deprecated in modern PHP and does not provide the intended omission behavior. Place every required parameter first, then the parameters with defaults.

### A default is used only when the argument is omitted

Passing **null** is not the same as omitting an argument. If the parameter accepts null, the explicitly supplied null value is used instead of the default:

~~~php
<?php
function showOption(?string $option = 'standard'): void
{
    var_dump($option);
}

showOption();
showOption(null);
~~~

Output:

~~~text
string(8) "standard"
NULL
~~~

The **?string** declaration permits either a string or null. The first call omits the argument; the second explicitly passes null.

### Default values must be constant expressions

Defaults can use values such as scalar literals, arrays, and **null**. The default must be a constant expression: it cannot depend on a local variable or a function call.

~~~php
function configure(array $options = []): void
{
    // ...
}
~~~

Since PHP 8.1, a default value can also create an object with **new ClassName()**. That version-specific form is useful for some APIs, but beginners can start with scalar and array defaults.

## Named arguments

Named arguments let a caller identify an argument by the parameter name instead of relying only on its position. They are available starting with PHP 8.0:

~~~php
<?php
function printNotice(
    string $recipient,
    string $message,
    string $channel = 'email',
    bool $urgent = false,
): void {
    $priority = $urgent ? 'urgent' : 'normal';
    echo "{$channel} ({$priority}) to {$recipient}: {$message}\n";
}

printNotice(
    recipient: 'Asha',
    message: 'The build finished',
    channel: 'dashboard',
    urgent: true,
);
~~~

Output:

~~~text
dashboard (urgent) to Asha: The build finished
~~~

A named argument uses the declared parameter name followed by a colon and its value. Named arguments can be written in a different order from the declaration:

~~~php
printNotice(
    message: 'Backup complete',
    recipient: 'Operations',
);
~~~

Output:

~~~text
email (normal) to Operations: Backup complete
~~~

Because **channel** and **urgent** were omitted, their defaults are used.

Named arguments can make a call clearer when a function has several similar options. They also let a caller skip a defaulted parameter without supplying every earlier optional value:

~~~php
printNotice('Mina', 'Review requested', urgent: true);
~~~

The first two values are positional and map to **$recipient** and **$message**. The named argument sets **$urgent**, leaving **$channel** at its default.

When combining the two styles, write positional arguments first and named arguments afterward. Do not supply the same parameter twice. The named parameter must exist, and its name must match the function's declaration. Since callers may rely on parameter names, renaming a public parameter can break callers that use named arguments.

## Variadic parameters

A variadic parameter collects zero or more trailing arguments into an array. Put **...** before the parameter name:

~~~php
<?php
function printChecklist(string $title, string ...$items): void
{
    echo "{$title}\n";

    foreach ($items as $item) {
        echo "- {$item}\n";
    }
}

printChecklist(
    'Release checklist',
    'Run the checks',
    'Review the changes',
    'Write the release notes',
);
~~~

Output:

~~~text
Release checklist
- Run the checks
- Review the changes
- Write the release notes
~~~

The ordinary **$title** parameter receives the first argument. The remaining arguments are collected in **$items**, an array. The variadic parameter may receive no values:

~~~php
printChecklist('Empty checklist');
~~~

There can be only one variadic parameter, and it must be last in the parameter list. Ordinary parameters can appear before it. A type declaration before the variadic parameter applies to each collected argument:

~~~php
function countScores(int ...$scores): void
{
    echo count($scores) . " scores\n";
}
~~~

A variadic parameter is useful when a function naturally accepts a flexible number of values. If the inputs form one meaningful collection, taking a single array parameter may be clearer.

## Unpack arguments at the call site

The same **...** syntax can expand an array or iterable into individual arguments when calling a function:

~~~php
<?php
function printDimensions(int $width, int $height, int $depth): void
{
    echo "{$width} x {$height} x {$depth}\n";
}

$dimensions = [120, 80, 35];
printDimensions(...$dimensions);
~~~

Output:

~~~text
120 x 80 x 35
~~~

The three values in the array are unpacked and supplied positionally. Each value must match the corresponding parameter. An array or iterable with the wrong number of values or incompatible values can cause an argument-count or type error.

Do not confuse the two uses:

- In **function collect(string ...$items)**, the declaration collects extra arguments into an array.
- In **collect(...$items)**, the call expands an array or iterable into arguments.

Numeric-keyed arrays are a straightforward way to unpack positional arguments. PHP 8 also supports string-keyed unpacked arguments as named arguments; for beginner code, use a list when you mean positional values and explicit named arguments when you mean parameter names.

## Trailing commas in multi-line calls

A trailing comma can be left after the final argument in a multi-line call. This is useful when arguments are listed vertically and a later edit adds another value:

~~~php
printNotice(
    recipient: 'Asha',
    message: 'The build finished',
    channel: 'dashboard',
);
~~~

Trailing commas in function calls are supported from PHP 7.3. A trailing comma in a function parameter declaration is supported from PHP 8.0. Match these syntax choices to the minimum PHP version your project supports.

## Common mistakes

- Expecting a default to be used when the caller explicitly passes **null**.
- Placing a required parameter after a defaulted parameter.
- Using a named argument with a misspelled or nonexistent parameter name.
- Putting a positional argument after a named argument.
- Supplying one parameter both positionally and by name.
- Assuming named arguments work on PHP versions earlier than 8.0.
- Putting a variadic parameter before another parameter or declaring more than one variadic parameter.
- Confusing variadic collection in a declaration with unpacking at a call.
- Unpacking an array whose values do not match the function's parameter list.
- Using a variadic parameter when a single array parameter would express the input more clearly.

## Try it yourself

1. Add a default **$punctuation** parameter to a greeting function and call it with and without that argument.
2. Pass **null** to a nullable parameter that has a non-null default. Observe which value the body receives.
3. Call **printNotice()** with named arguments in a different order.
4. Skip **$channel** but pass **urgent: true** by name.
5. Write a variadic **printNames()** function and call it with zero, one, and several names.
6. Store three dimensions in an array and unpack the array into a function call.
7. Compare an unpacked list with a string-keyed array passed as named arguments. Use PHP 8 or later for named arguments.

## Key takeaways

- A default is used when an argument is omitted; explicit **null** remains an explicit value.
- Required parameters belong before defaulted parameters.
- Named arguments use parameter names and are available in PHP 8.0 and later.
- Variadic parameters collect trailing arguments into an array and must be last.
- Argument unpacking expands an array or iterable into a call.
- The **...** syntax collects in a declaration and expands at a call site.
- Use the simplest argument style that makes the function call clear.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [038 - Function Parameters and Arguments](038-function-parameters-and-arguments.md) | [Open Q&A](../qnas/039-default-named-variadic-and-unpacked-arguments.md) | [Open example](../examples/039-default-named-variadic-and-unpacked-arguments.md) | [040 - Function Return Types](040-function-return-types.md) |
