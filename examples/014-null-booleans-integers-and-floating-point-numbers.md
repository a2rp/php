# 014 - Null, Booleans, Integers, and Floating-Point Numbers: Complete Example

This command-line example uses **null**, booleans, integers, and floats in a small inventory and checkout summary. It prints each type, compares integer and floating-point division, and demonstrates why decimal float equality needs care.

## Question

How can a PHP program represent a value that has not been supplied, a yes-or-no setting, a whole-number count, and a measured decimal value without confusing their types?

## Goal

Create this project:

~~~text
primitive-values/
|-- README.md
|-- values.php
~~~

The **values.php** script demonstrates:

- An assigned **null** value and the difference between **is_null()** and **isset()**
- The difference between boolean values and text that looks like a boolean
- Decimal, binary, octal, and hexadecimal integer literals
- A readable integer literal with digit separators
- A float, a floating-point comparison, and a formatted price
- The difference between **/** and **intdiv()**
- Type checks using **is_bool()**, **is_int()**, and **is_float()**

## Complete code: values.php

Save this file at the project root:

~~~php
<?php

$shippingFee = null;
$isTaxable = true;
$itemCount = 3;
$weightInKilograms = 1.25;

var_dump($shippingFee);
var_dump(is_null($shippingFee));
var_dump(isset($shippingFee));

var_dump(is_null(""));
var_dump(is_null(0));
var_dump(is_null(false));

var_dump($isTaxable);
var_dump("true");
var_dump(false);
var_dump("false");

$decimal = 42;
$binary = 0b101010;
$octal = 0o52;
$hexadecimal = 0x2A;
$oneMillion = 1_000_000;

var_dump($decimal);
var_dump($binary);
var_dump($octal);
var_dump($hexadecimal);
var_dump($oneMillion);

var_dump($weightInKilograms);
var_dump(is_bool($isTaxable));
var_dump(is_int($itemCount));
var_dump(is_float($weightInKilograms));

var_dump(17 / 5);
var_dump(intdiv(17, 5));

$calculatedTotal = 0.1 + 0.2;
var_dump($calculatedTotal === 0.3);

$priceInCents = 1299;
echo "Formatted price: " . number_format($priceInCents / 100, 2) . PHP_EOL;
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Primitive Values Practice

A small PHP CLI script for inspecting null, booleans, integers, and floating-point numbers.

## Check and run

~~~sh
php -l values.php
php values.php
~~~

## What it demonstrates

- Distinguishing null from false, zero, and an empty string
- Comparing a boolean with text that looks like a boolean
- Inspecting integer literals in multiple bases
- Checking the types of values
- Comparing ordinary division with integer division
- Seeing why exact float equality can fail
- Formatting an amount stored in integer cents
~~~~

This Markdown block is the README file to create in the project. Keep the PHP source in **values.php**.

## Check and run the program

Open a terminal in **primitive-values/** and check the syntax:

~~~sh
php -l values.php
~~~

For a valid file, PHP reports that no syntax errors were detected. Run it with:

~~~sh
php values.php
~~~

Expected output on PHP 8.1 or newer:

~~~text
NULL
bool(true)
bool(false)
bool(false)
bool(false)
bool(false)
bool(true)
string(4) "true"
bool(false)
string(5) "false"
int(42)
int(42)
int(42)
int(42)
int(1000000)
float(1.25)
bool(true)
bool(true)
bool(true)
float(3.4)
int(3)
bool(false)
Formatted price: 12.99
~~~

The **0o52** octal literal requires PHP 8.1 or newer. If using an older PHP version, replace it with a leading-zero octal literal such as **052**.

## Understand the output

### Null is distinct from other empty-looking values

**$shippingFee** is deliberately assigned **null**, so its first output line is **NULL**. **is_null($shippingFee)** returns true. **isset($shippingFee)** returns false because **isset()** treats a null value like a value that is not set for its check.

The next three checks return false because an empty string, integer zero, and boolean false each have a type other than null.

### Boolean values are not strings

**$isTaxable** contains the boolean **true**. The value **"true"** is a string because it is quoted. Similarly, **false** is a boolean and **"false"** is a string.

### Different integer notation can describe the same value

**$decimal**, **$binary**, **$octal**, and **$hexadecimal** are written in different bases. Each literal represents the integer **42**, so **var_dump()** prints **int(42)** four times.

**$oneMillion** is written with underscores for readability. The underscores do not change the integer value.

### Type checks report the current PHP type

**is_bool($isTaxable)**, **is_int($itemCount)**, and **is_float($weightInKilograms)** each return true. These checks tell you the PHP type, not whether a value is valid for a particular business rule.

### Division operators return different types

**17 / 5** returns the float **3.4**. **intdiv(17, 5)** returns the integer quotient **3**, discarding the fractional part toward zero.

### Float comparison can be surprising

The result of **0.1 + 0.2** is not represented exactly as the same binary floating-point value as **0.3**. Therefore, the strict comparison produces **false**. Use a carefully chosen tolerance for approximate calculations instead of expecting exact float equality.

### Store money with an explicit precision strategy

The example stores **1299** as integer cents and formats the displayed amount as **12.99**. This avoids storing the original price itself as a float. Larger financial applications can need additional rules for taxes, rounding, and decimal arithmetic.

## Make a change

1. Change **$shippingFee** from **null** to **0**. Compare **is_null()** and **isset()** again.
2. Change **$isTaxable** to **false**, then compare it with the string **"false"**.
3. Change the integer literals to another value represented in decimal, binary, octal, and hexadecimal. Confirm that all four output values match.
4. Print **PHP_INT_SIZE**, **PHP_INT_MIN**, and **PHP_INT_MAX** to inspect the integer range on your runtime.
5. Replace **17** and **5** with another pair of integers. Compare the results from **/** and **intdiv()**.
6. Try another decimal sum such as **0.1 + 0.7**. Compare it with the decimal value you expect and explain why exact float equality is risky.
7. Change **$priceInCents** to **2500** and observe the formatted output.
8. Change **$weightInKilograms** to a whole-number-looking decimal such as **2.0**. Check that PHP still stores it as a float.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Replacing a not-yet-provided value with false or zero | Those values can have valid meanings of their own. | Use null when the application state truly means no value. |
| Expecting isset() to be true for a null variable | isset() intentionally returns false for null. | Use is_null() for a specific null check. |
| Assuming "false" is a boolean | Quoted text is a string. | Store the boolean literal false when a boolean is needed. |
| Assuming all integer bases print differently | Base notation is source syntax; the runtime value is still an integer. | Inspect the evaluated value with var_dump(). |
| Comparing decimal arithmetic with strict float equality | The binary approximation may differ by a tiny amount. | Use a suitable tolerance or an appropriate exact-decimal representation. |
| Treating intdiv() as rounding to the nearest integer | It truncates the quotient toward zero. | Choose a rounding function when nearest-value rounding is required. |
| Storing a price as a float without a rounding plan | Repeated decimal arithmetic can accumulate representation error. | Choose a documented money representation and rounding policy. |

## Key takeaways

- A **null** value is different from **false**, **0**, and an empty string.
- **isset()** returns false for a variable assigned **null**; **is_null()** checks specifically for null.
- Boolean literals are not the same as strings containing the words **"true"** or **"false"**.
- Binary, octal, hexadecimal, and decimal literals can represent the same integer.
- Integer range is platform-dependent; inspect the **PHP_INT_*** constants.
- **/** returns a float, while **intdiv()** returns an integer quotient toward zero.
- Floating-point values have limited precision; avoid exact equality checks for calculated decimals.
- Store money using a deliberate precision and rounding strategy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [013 - The PHP Type System and Value Model](../lessons/013-the-php-type-system-and-value-model.md) | [Read this lesson](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) | [Open Q&A](../qnas/014-null-booleans-integers-and-floating-point-numbers.md) | [015 - Strings and numeric strings](../lessons/015-strings-and-numeric-strings.md) |
