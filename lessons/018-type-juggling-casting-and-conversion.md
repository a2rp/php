# 018 - Type Juggling, Casting, and Conversion

PHP values have types, but PHP can sometimes interpret a value as another type for the operation being performed. This automatic behavior is called **type juggling** or **implicit conversion**. A cast requests an explicit conversion. Knowing the difference helps prevent surprising results when working with user input, numbers, strings, and conditions.

## What you will learn

- How the value assigned to a variable determines its current type
- What PHP means by type juggling
- How arithmetic and string operations can interpret values
- How PHP converts values in a logical condition
- How loose and strict comparisons differ at a basic level
- How to explicitly cast values to **int**, **float**, **string**, and **bool**
- How **settype()** differs from a cast expression
- Why conversion is not the same as validating input
- How PHP 8 handles non-numeric strings in arithmetic

## A variable's type follows its value

PHP variables do not need a type declaration when they are created. Their current type is determined by the value assigned to them:

~~~php
<?php
$value = "12";
var_dump($value);

$value = 12;
var_dump($value);
~~~

Output:

~~~text
string(2) "12"
int(12)
~~~

The variable first contains a string and then contains an integer. Assigning a new value changes what type the variable currently holds.

## Automatic conversion depends on context

In some operations, PHP interprets a value as another type to complete the operation. The value is not necessarily changed in the variable itself.

### Numeric context

Arithmetic operators use numeric values. A numeric string can be interpreted as a number:

~~~php
<?php
$quantity = "4";
$total = $quantity * 3;

var_dump($quantity);
var_dump($total);
var_dump($quantity);
~~~

Output:

~~~text
string(1) "4"
int(12)
string(1) "4"
~~~

The multiplication uses the numeric value **4**, but **$quantity** remains a string. The arithmetic result is an integer here.

A numeric string containing a decimal can produce a floating-point result:

~~~php
<?php
$price = "12.5";
$subtotal = $price * 2;

var_dump($subtotal);
~~~

Output:

~~~text
float(25)
~~~

Starting with PHP 8, arithmetic with a string that cannot be interpreted as a number throws a **TypeError**. Do not rely on an invalid string being silently treated as zero.

~~~php
<?php
$quantity = "four";

// In PHP 8+, this throws a TypeError:
$total = $quantity * 3;
~~~

Check and validate input before doing arithmetic with it.

### String context

The concatenation operator **.** joins strings. Scalar values used in concatenation are converted to strings:

~~~php
<?php
$count = 3;
$message = "Items: " . $count;

echo $message . PHP_EOL;
~~~

Output:

~~~text
Items: 3
~~~

Numbers become their text representation. Booleans have a less obvious string form: **true** becomes **"1"**, while **false** becomes an empty string. Avoid relying on implicit conversion when a different text label is needed.

~~~php
<?php
var_dump((string) true);
var_dump((string) false);
~~~

Output:

~~~text
string(1) "1"
string(0) ""
~~~

Arrays and most objects are not ordinary strings. Convert or format structured data deliberately rather than concatenating it as if it were a scalar.

### Logical context

A condition interprets a value as true or false. The following values are false-like in a boolean context:

- **false**
- Integer **0** and floating-point **0.0**
- The empty string **""**
- The string **"0"**
- An empty array **[]**
- **null**

Other values are true-like. In particular, the non-empty string **"false"** is true-like, and a non-empty array is true-like.

~~~php
<?php
var_dump((bool) "0");
var_dump((bool) "false");
var_dump((bool) []);
var_dump((bool) [0]);
~~~

Output:

~~~text
bool(false)
bool(true)
bool(false)
bool(true)
~~~

The string **"false"** contains text and is not the same value as the boolean **false**. When reading text such as a setting or form field, parse its allowed values explicitly instead of assuming words like **"false"** will automatically become false.

### Comparison context

Loose comparison with **==** can convert values before comparing them. Strict comparison with **===** requires both the value and the type to match:

~~~php
<?php
var_dump(0 == "0");
var_dump(0 === "0");
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

Use strict comparison when the type matters. The comparison lesson later covers comparison operators and their rules in more detail.

## Explicit casting

A cast asks PHP to convert a value to a requested type. The cast applies to the resulting expression; it does not change the type of the original variable unless you assign the result back.

### Cast to integer

Use **(int)** or **(integer)** to request an integer:

~~~php
<?php
$amount = "12.9";
$wholeAmount = (int) $amount;

var_dump($wholeAmount);
var_dump($amount);
~~~

Output:

~~~text
int(12)
string(4) "12.9"
~~~

Casting a floating-point number to an integer discards the fractional part; it does not round to the nearest integer. For example, **(int) 12.9** is **12**, and **(int) -12.9** is **-12**.

### Cast to float

Use **(float)** or **(double)** to request a floating-point number:

~~~php
<?php
$input = "12.5";
$amount = (float) $input;

var_dump($amount);
~~~

Output:

~~~text
float(12.5)
~~~

Floats have limited precision. Do not use them for exact decimal currency arithmetic; use an appropriate decimal representation or integer minor units for monetary values.

### Cast to string

Use **(string)** to request a string representation:

~~~php
<?php
$id = 42;
$textId = (string) $id;

var_dump($textId);
~~~

Output:

~~~text
string(2) "42"
~~~

A boolean converted to a string becomes **"1"** for true or **""** for false. If a human-readable label is needed, choose that label yourself:

~~~php
<?php
$isPublished = false;
$label = $isPublished ? "Published" : "Draft";

echo $label . PHP_EOL;
~~~

Output:

~~~text
Draft
~~~

### Cast to boolean

Use **(bool)** or **(boolean)** to ask PHP for the boolean interpretation of a value:

~~~php
<?php
var_dump((bool) 0);
var_dump((bool) 5);
var_dump((bool) "");
var_dump((bool) "hello");
~~~

Output:

~~~text
bool(false)
bool(true)
bool(false)
bool(true)
~~~

Boolean casting follows the false-like values described above. It does not parse arbitrary words: **(bool) "false"** is true because the string is non-empty and is not **"0"**.

## Cast expression versus settype()

A cast converts a value for one expression. To keep the original value and its type, assign the cast result to another variable:

~~~php
<?php
$input = "25";
$number = (int) $input;

var_dump($input);
var_dump($number);
~~~

Output:

~~~text
string(2) "25"
int(25)
~~~

The **settype()** function changes the type of the variable it receives:

~~~php
<?php
$value = "25";
settype($value, "integer");

var_dump($value);
~~~

Output:

~~~text
int(25)
~~~

Use a cast when you need a converted expression. Use **settype()** only when you specifically want to change the variable's value and type in place. In ordinary application code, a clearly named new variable often makes the conversion easier to follow.

## Conversion is not validation

A cast does not prove that input is valid for your program. For example, converting a submitted value to an integer does not establish that it is a valid quantity, is in range, or is safe to use for a particular operation.

**is_numeric()** can check whether a value is a number or numeric string:

~~~php
<?php
$input = "24";

if (is_numeric($input)) {
    $quantity = (int) $input;
    echo "Quantity: " . $quantity . PHP_EOL;
} else {
    echo "Enter a numeric quantity." . PHP_EOL;
}
~~~

Output:

~~~text
Quantity: 24
~~~

Applications often need additional checks, such as requiring a whole number, setting a minimum and maximum, or validating a date format. Choose validation rules based on what the input means, then convert it to the representation your code needs.

Type declarations and how they interact with coercion are covered in the next chapter.

## Try it yourself

1. Assign a numeric string to a variable, multiply it by an integer, and inspect both the original variable and the result with **var_dump()**.
2. Concatenate an integer to a message. Then inspect how **true** and **false** behave when cast to strings.
3. Cast **"12.9"** to **int** and explain why the result is not rounded.
4. Test the boolean conversion of **"0"**, **"false"**, an empty string, and an empty array.
5. Compare **0 == "0"** with **0 === "0"**.
6. Use **settype()** on a string variable and inspect the variable before and after.
7. Check a numeric input with **is_numeric()** before converting it. Add a range rule that makes sense for your example.
8. In PHP 8+, try arithmetic with a non-numeric string and observe the **TypeError**. Remove the invalid operation before continuing.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming a numeric operation changes the variable's type | PHP can interpret a value numerically for one operation while the variable remains a string. | Inspect the original variable and result separately. |
| Assuming every string is valid for arithmetic | PHP 8 throws a **TypeError** for a non-numeric string in arithmetic. | Validate input before using it as a number. |
| Assuming **(int)** rounds a float | Integer casting discards the fractional part toward zero. | Use a deliberate rounding function when rounding is required. |
| Treating the string **"false"** as boolean false | It is a non-empty string, so its boolean conversion is true. | Parse known text values explicitly. |
| Assuming **(bool)** verifies an input's meaning | It only applies PHP's boolean conversion rules. | Validate allowed values and meaning separately. |
| Using **==** when types must match | Loose comparison may convert operands before comparing. | Use **===** when both type and value must match. |
| Expecting a cast to validate a value | Conversion can produce a value that is still outside the application's allowed range. | Validate first, then convert. |
| Using **settype()** when the original value should be kept | **settype()** changes the variable in place. | Assign a cast result to a new, clearly named variable. |
| Expecting false to become the text **"false"** | Casting boolean false to string produces an empty string. | Choose a display label explicitly. |

## Key takeaways

- A variable's current type comes from the value it stores.
- Type juggling is PHP's context-dependent automatic interpretation of values.
- Numeric strings may be used in arithmetic without changing the original variable.
- PHP 8 throws a **TypeError** for arithmetic with a non-numeric string.
- Boolean conversion has specific false-like values; notably, **"false"** is true-like while **"0"** is false-like.
- **==** may compare after conversion; **===** also requires matching types.
- Casts request conversions; **settype()** changes a variable's type in place.
- Conversion is not validation. Check that input meets the application's rules before using it.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [017 - Objects, resources, callables, and iterables](017-objects-resources-callables-and-iterables.md) | [Open Q&A](../qnas/018-type-juggling-casting-and-conversion.md) | [Open example](../examples/018-type-juggling-casting-and-conversion.md) | [019 - Type declarations and strict typing](019-type-declarations-and-strict-typing.md) |
