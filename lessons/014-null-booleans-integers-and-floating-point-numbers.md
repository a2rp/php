# 014 - Null, Booleans, Integers, and Floating-Point Numbers

PHP has several basic value types that appear in almost every program. This lesson introduces the special value **null**, the two boolean values, whole-number integers, and floating-point numbers. You will learn how to recognize each type, inspect it, and avoid common mistakes around numeric limits and decimal precision.

## What you will learn

- What **null** represents and how it differs from **false**, **0**, and an empty string
- How the boolean literals **true** and **false** express a yes-or-no value
- How to write integer literals in decimal, binary, octal, and hexadecimal notation
- Why the integer range depends on the PHP platform
- How floating-point values represent decimal quantities and why their precision is limited
- How to inspect these values with **var_dump()** and type-checking functions
- Why integer division and floating-point equality need care

## Inspecting the four types

A value's type affects the operations and comparisons that make sense for it. These values may have different meanings even when they all represent an empty, unavailable, or numeric-looking state:

~~~php
<?php
$notProvided = null;
$isPublished = false;
$lessonCount = 0;
$discount = 0.0;

var_dump($notProvided);
var_dump($isPublished);
var_dump($lessonCount);
var_dump($discount);
~~~

Output:

~~~text
NULL
bool(false)
int(0)
float(0)
~~~

The values are not interchangeable:

- **null** represents the absence of a value.
- **false** is a boolean answer meaning “no” or “not true.”
- **0** is an integer with numeric value zero.
- **0.0** is a floating-point number with numeric value zero.

Use the value that matches the meaning of the data. Do not use a numeric zero to mean “not supplied” if zero is a valid amount or count.

## The null value

PHP's **null** type has one value: **null**. Assign it when a variable intentionally has no value yet, or when an existing value should be cleared:

~~~php
<?php
$middleName = null;

var_dump($middleName);
var_dump(is_null($middleName));
var_dump(isset($middleName));
~~~

Output:

~~~text
NULL
bool(true)
bool(false)
~~~

**is_null()** checks whether a value is null. **isset()** answers a different question: it returns false when a variable is undefined or when its value is null. The variable above is defined, but **isset()** is still false because its value is null.

Avoid reading a variable that was never assigned. A deliberately assigned **null** value and an undefined variable are different program states, even though **isset()** returns false for both.

An empty string, zero, and false are also not null:

~~~php
<?php
var_dump(is_null(""));
var_dump(is_null(0));
var_dump(is_null(false));
~~~

Output:

~~~text
bool(false)
bool(false)
bool(false)
~~~

Later lessons cover strict comparisons and how null interacts with arrays and forms. For now, remember that **null** is a value with its own type, not a synonym for every kind of empty or missing data.

## Boolean values

A boolean has exactly two values: **true** and **false**. Use them for facts that have two possible answers, such as whether a page is published:

~~~php
<?php
$isPublished = true;
$hasDraft = false;

var_dump($isPublished);
var_dump($hasDraft);
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

The words **true** and **false** are boolean literals. The quoted strings **"true"** and **"false"** are text:

~~~php
<?php
var_dump(true);
var_dump("true");
var_dump(false);
var_dump("false");
~~~

Output:

~~~text
bool(true)
string(4) "true"
bool(false)
string(5) "false"
~~~

Comparison and logical expressions commonly produce booleans. A condition can use a boolean result directly:

~~~php
<?php
$lessonCount = 14;
$hasLessons = $lessonCount > 0;

var_dump($hasLessons);

if ($hasLessons) {
    echo "The course has lessons.";
}
~~~

Output:

~~~text
bool(true)
The course has lessons.
~~~

This lesson uses explicit boolean values. PHP can also interpret other types as true or false in a condition; that behavior is called boolean conversion or truthiness and is covered in the later type-juggling lesson. Do not assume that the string **"false"** is itself the boolean value **false**.

## Integer values

An integer (**int**) is a whole number without a fractional part. Positive and negative values, and zero, can be integers:

~~~php
<?php
$chapterNumber = 14;
$temperatureChange = -3;
$attempts = 0;

var_dump($chapterNumber);
var_dump($temperatureChange);
var_dump($attempts);
~~~

Output:

~~~text
int(14)
int(-3)
int(0)
~~~

### Integer literal notation

Most everyday integer values use decimal (base 10). PHP also accepts binary, octal, and hexadecimal literals:

~~~php
<?php
$decimal = 42;
$binary = 0b101010;
$octal = 0o52;
$hexadecimal = 0x2A;

var_dump($decimal);
var_dump($binary);
var_dump($octal);
var_dump($hexadecimal);
~~~

Output:

~~~text
int(42)
int(42)
int(42)
int(42)
~~~

The prefixes identify the base:

- **0b** or **0B** begins a binary literal.
- **0o** or **0O** begins an octal literal in PHP 8.1 and newer. Older PHP versions use a leading **0** for octal notation.
- **0x** or **0X** begins a hexadecimal literal.
- A number with no base prefix is decimal.

PHP 7.4 and newer also allow underscores between digits to make long numeric literals easier to read:

~~~php
<?php
$oneMillion = 1_000_000;

var_dump($oneMillion);
~~~

Output:

~~~text
int(1000000)
~~~

Underscores are separators for people reading the source; they do not change the numeric value. Do not put them at the start or end of a number, or next to the base prefix.

### Integer limits

PHP integers are signed and have a finite range. The exact range depends on the platform running PHP. Use the built-in constants to inspect it instead of assuming every machine has the same limits:

~~~php
<?php
echo "Integer bytes: " . PHP_INT_SIZE . PHP_EOL;
echo "Minimum integer: " . PHP_INT_MIN . PHP_EOL;
echo "Maximum integer: " . PHP_INT_MAX . PHP_EOL;
~~~

A 64-bit PHP build commonly supports a much larger integer range than a 32-bit build. The displayed values come from the current runtime, so they can differ across systems.

If an integer literal or arithmetic result exceeds the integer range, PHP represents it as a float. That can lose integer precision for very large values. If an application needs integers larger than the platform range, do not assume a normal **int** can store them exactly; use an appropriate arbitrary-precision approach.

## Floating-point numbers

A floating-point number (**float**) can represent a fractional value or a number written with an exponent:

~~~php
<?php
$taxRate = 0.18;
$distance = 12.75;
$largeValue = 1.2e3;
$smallValue = 7e-4;

var_dump($taxRate);
var_dump($distance);
var_dump($largeValue);
var_dump($smallValue);
~~~

The exponent notation means multiplication by a power of ten: **1.2e3** is **1.2 × 10³**, or **1200**. A decimal point or exponent makes these float literals.

A float is an approximation with finite precision. Many familiar decimal fractions, such as **0.1**, cannot be represented exactly in the binary format commonly used by computers. For that reason, arithmetic can produce a result that is extremely close to the expected decimal value without being exactly equal to it.

~~~php
<?php
$result = 0.1 + 0.2;

var_dump($result === 0.3);
~~~

Output:

~~~text
bool(false)
~~~

Do not compare calculated floats directly for exact equality. For calculations that need a tolerance, compare the size of the difference to a deliberately chosen tolerance. For money, a common beginner-safe design is to store amounts as integer minor units, such as cents, and format them for display. Financial applications with more complex decimal requirements may need a decimal arithmetic library.

Do not use a float just because a number is written with a decimal point in a user interface. Decide how the value must behave and how precise it must be.

## Integer division and floating-point division

The **/** operator performs division and returns a float, even when the result looks like a whole number. Use **intdiv()** when the intended result is an integer quotient:

~~~php
<?php
var_dump(17 / 5);
var_dump(intdiv(17, 5));
~~~

Output:

~~~text
float(3.4)
int(3)
~~~

**intdiv()** discards the fractional part by rounding the quotient toward zero. It does not round to the nearest integer. The separate rounding functions and other arithmetic operators are covered in the operators lessons.

## Check a value's type

Use **var_dump()** while learning or debugging to see the type and value together. You can also use specific checks when a program needs to branch based on a type:

~~~php
<?php
$value = 12.5;

var_dump(is_null($value));
var_dump(is_bool($value));
var_dump(is_int($value));
var_dump(is_float($value));
~~~

Output:

~~~text
bool(false)
bool(false)
bool(false)
bool(true)
~~~

These checks describe the current PHP type. They do not prove that a value is valid for the application's purpose. For example, a float can be a valid PHP value while still being an invalid chapter count.

## A practical example

A checkout may distinguish between a price that was not provided, whether an item is taxable, a whole-number quantity, and a fractional measurement:

~~~php
<?php
$shippingFee = null;
$isTaxable = true;
$itemCount = 3;
$weightInKilograms = 1.25;

var_dump($shippingFee);
var_dump($isTaxable);
var_dump($itemCount);
var_dump($weightInKilograms);
~~~

Here **null** means the fee has not been set. **true** records a yes-or-no fact. **3** is a whole-number count. **1.25** is a measured fractional quantity. In a real checkout, money calculations also need a deliberate precision strategy.

## Try it yourself

1. Create one value of each type: null, boolean, integer, and float. Inspect all four with **var_dump()**.
2. Set a variable to **null**, then check it with **is_null()** and **isset()**. Explain why the results differ.
3. Assign **true** and **"true"** to separate variables. Compare their output and types.
4. Write the integer **42** in decimal, binary, octal, and hexadecimal notation. Inspect the values.
5. Print **PHP_INT_MIN**, **PHP_INT_MAX**, and **PHP_INT_SIZE** on your computer.
6. Compare **17 / 5** with **intdiv(17, 5)** and explain why their types and results differ.
7. Evaluate **0.1 + 0.2 === 0.3**. Explain why exact floating-point equality can surprise you.
8. Choose whether each item should be stored as an integer, float, boolean, or null: a number of seats, a measured length, a yes-or-no setting, and a value not yet supplied.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Treating null, false, zero, and an empty string as the same value | They have different types and meanings. | Choose a value that accurately represents the state or data. |
| Expecting isset() to return true for a variable whose value is null | isset() deliberately returns false for null values. | Use is_null() when you need to test specifically for null. |
| Treating the text "false" as a boolean false | A quoted value is a string. | Use the boolean literal false when a boolean is required. |
| Assuming every PHP platform has the same integer range | Integer capacity depends on the platform. | Inspect PHP_INT_MIN, PHP_INT_MAX, and PHP_INT_SIZE. |
| Assuming integer overflow wraps around safely | An out-of-range integer can become a float, changing precision and type. | Check limits when handling large values and choose an appropriate representation. |
| Expecting decimal floats to be exact | Many base-10 fractions have no exact binary floating-point representation. | Use a tolerance for approximate comparisons and a suitable money representation. |
| Using / when the result must be an integer quotient | The division operator returns a float. | Use intdiv() when truncating to an integer quotient is intended. |

## Key takeaways

- **null**, **false**, **0**, and **0.0** have distinct types and meanings.
- A boolean contains exactly **true** or **false**.
- PHP supports decimal, binary, octal, and hexadecimal integer literals.
- Integer limits depend on the runtime platform; inspect the **PHP_INT_*** constants.
- Integers outside their supported range can be represented as floats and lose precision.
- Floats have finite precision, so calculated decimal values should not be compared for exact equality.
- **/** returns a float; **intdiv()** returns an integer quotient rounded toward zero.
- **var_dump()**, **is_null()**, **is_bool()**, **is_int()**, and **is_float()** help inspect values.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [013 - The PHP Type System and Value Model](013-the-php-type-system-and-value-model.md) | [Open Q&A](../qnas/014-null-booleans-integers-and-floating-point-numbers.md) | [Open example](../examples/014-null-booleans-integers-and-floating-point-numbers.md) | [015 - Strings and numeric strings](015-strings-and-numeric-strings.md) |
