# 028 - String, Array, Bitwise, and Other Operators

PHP has operators for joining strings, combining arrays, manipulating integer bits, and checking an object's type. Some symbols have different meanings depending on the operand types, so use them deliberately and add parentheses when the intent might be unclear.

## What you will learn

- How the concatenation operator joins strings
- How PHP's array union operator handles duplicate keys
- How bitwise operators work on integer bits
- How bitwise operators differ from logical operators
- How **instanceof** checks an object's class or interface
- Why the error-control and shell-execution operators should generally be avoided
- Where null-aware and conditional operators are covered next

## String concatenation

The dot operator **.** joins values as strings and returns a string. The plus operator **+** performs numeric addition for numeric operands; it does not concatenate strings.

~~~php
<?php
$firstName = "Sam";
$lastName = "Rivera";

$fullName = $firstName . " " . $lastName;

echo $fullName . PHP_EOL;
~~~

Output:

~~~text
Sam Rivera
~~~

Concatenation can also be used when displaying a calculation. Parentheses make the arithmetic grouping explicit:

~~~php
<?php
$unitPrice = 12;
$quantity = 3;

$message = "Subtotal: " . ($unitPrice * $quantity);

echo $message . PHP_EOL;
~~~

Output:

~~~text
Subtotal: 36
~~~

In PHP 8 and later, **.** has lower precedence than arithmetic addition and subtraction. Parenthesize mixed expressions or calculate a named value first. Chapter [024 - Operator Precedence and Associativity](024-operator-precedence-and-associativity.md) covers that version change and grouping rules.

### Concatenating assignment

The operator **.=** appends a value to a string variable and stores the new string back in that variable:

~~~php
<?php
$message = "PHP";
$message .= " operators";

echo $message . PHP_EOL;
~~~

Output:

~~~text
PHP operators
~~~

Use **.=** for a straightforward string update. Keep calculations and formatting separate when that makes the result easier to read.

## Array union

When both operands are arrays, **+** creates an array union. It keeps the left array's value for a key that appears in both arrays, then adds keys that appear only in the right array.

~~~php
<?php
$defaults = [
    "theme" => "light",
    "itemsPerPage" => 20,
];

$userSettings = [
    "theme" => "dark",
    "language" => "en",
];

$settings = $defaults + $userSettings;

var_dump($settings);
~~~

Output:

~~~text
array(3) {
  ["theme"]=>
  string(5) "light"
  ["itemsPerPage"]=>
  int(20)
  ["language"]=>
  string(2) "en"
}
~~~

The default theme remains **"light"** because the left array wins for duplicate keys. To give the user's values priority, reverse the operands:

~~~php
<?php
$settings = $userSettings + $defaults;
~~~

Array union is based on keys. It does not append every value or automatically renumber numeric keys. For ordinary list concatenation or replacement behavior, choose an array function designed for that job. Array comparison operators are covered with the other comparisons in [026 - Comparison, Identity, and Ordering Operators](026-comparison-identity-and-ordering-operators.md).

### Array union assignment

The operator **+=** applies array union and stores the result in the left variable:

~~~php
<?php
$options = ["theme" => "light"];
$options += ["theme" => "dark", "language" => "en"];

var_dump($options);
~~~

The existing **"theme"** value remains **"light"**; the new **"language"** key is added.

To append one value to the end of a list, use array append syntax:

~~~php
<?php
$colors = ["blue", "green"];
$colors[] = "purple";
~~~

Array append syntax **[]** is different from array union with **+**.

## Bitwise operators

Bitwise operators inspect or change the individual bits in integer values. They are used in areas such as flags, permissions, protocol fields, and low-level data formats.

| Operator | Name | Effect on integer bits |
| --- | --- | --- |
| **&** | Bitwise AND | A bit is set when it is set in both operands |
| **|** | Bitwise OR | A bit is set when it is set in either operand |
| **^** | Bitwise XOR | A bit is set when it is set in exactly one operand |
| **~** | Bitwise NOT | Inverts each bit |
| **<<** | Left shift | Shifts bits to the left |
| **>>** | Right shift | Shifts bits to the right |

For small positive integers, the operations can be pictured with binary digits:

~~~text
6 = 0110
3 = 0011

6 & 3 = 0010 = 2
6 | 3 = 0111 = 7
6 ^ 3 = 0101 = 5
~~~

PHP example:

~~~php
<?php
$first = 6;
$second = 3;

var_dump($first & $second);
var_dump($first | $second);
var_dump($first ^ $second);
var_dump($first << 1);
var_dump($first >> 1);
~~~

Output:

~~~text
int(2)
int(7)
int(5)
int(12)
int(3)
~~~

The bitwise NOT operator **~** flips every bit in the integer representation, including the sign-related bits. Its result is therefore not simply the positive version of the operand. Use it with a clear bitmask purpose.

### Bitmasks

A bitmask uses one distinct bit for each flag. Combine flags with bitwise OR, check a flag with bitwise AND, and remove a flag by clearing its bit.

~~~php
<?php
const CAN_READ = 1;
const CAN_WRITE = 2;
const CAN_DELETE = 4;

$permissions = CAN_READ | CAN_WRITE;

$mayRead = ($permissions & CAN_READ) !== 0;
$mayDelete = ($permissions & CAN_DELETE) !== 0;

$permissions &= ~CAN_WRITE;

var_dump($mayRead);
var_dump($mayDelete);
var_dump($permissions);
~~~

Output:

~~~text
bool(true)
bool(false)
int(1)
~~~

The parentheses around each bitwise AND are important: they make the mask operation happen before the comparison. Bitwise operators are not a substitute for ordinary boolean conditions.

### Bitwise and logical operators are different

Similar-looking operators have different purposes:

| Logical operator | Bitwise operator | Typical purpose |
| --- | --- | --- |
| **&&** | **&** | Logical AND combines truth values; bitwise AND works on bits |
| **||** | **|** | Logical OR combines truth values; bitwise OR works on bits |
| **xor** | **^** | Logical XOR compares truth values; bitwise XOR works on bits |
| **!** | **~** | Logical NOT reverses truthiness; bitwise NOT flips integer bits |

Use logical operators for conditions. Use bitwise operators when the program intentionally stores multiple flags in an integer or manipulates a binary representation.

If both operands of **&**, **|**, or **^** are strings, PHP performs a byte-wise string operation and returns a string. That behavior is not Unicode character logic. Avoid using bitwise operations to manipulate user-visible text.

## The instanceof type operator

The **instanceof** operator checks whether an object is an instance of a class, a subclass, or a class that implements an interface. It returns a boolean.

~~~php
<?php
interface Exportable
{
    public function export(): string;
}

class Invoice implements Exportable
{
    public function export(): string
    {
        return "invoice";
    }
}

$invoice = new Invoice();

var_dump($invoice instanceof Invoice);
var_dump($invoice instanceof Exportable);
var_dump($invoice instanceof stdClass);
~~~

Output:

~~~text
bool(true)
bool(true)
bool(false)
~~~

The check is false when the left value is not an object of the requested class or interface. Object construction and interfaces receive fuller treatment in later object-oriented programming chapters.

## Special operators to recognize

### Error-control operator

The prefix **@** suppresses certain diagnostics from the expression that follows it. It does not fix the underlying problem, and it can hide useful information during development. Prefer checking return values and handling errors explicitly instead of adding **@** to silence them.

### Shell-execution operator

PHP has a special execution operator that uses backticks around a shell command. It runs a command through the system shell and returns command output. This creates security and portability concerns, especially if command text includes untrusted input. Avoid it in beginner application code; use a deliberate process API and safe argument handling when a program truly needs to start another process.

### Operators covered in the next chapter

The ternary operator, null coalescing operator, and nullsafe operator each have distinct behavior and precedence rules. They are covered in [029 - Ternary, Null Coalescing, and Nullsafe Operators](029-ternary-null-coalescing-and-nullsafe-operators.md).

## Common mistakes

- Using **+** for string concatenation instead of **.**
- Mixing concatenation and arithmetic without parentheses
- Assuming array **+** appends or overwrites duplicate keys
- Assuming array union renumbers numeric keys
- Confusing **&**, **|**, **^**, or **~** with logical operators
- Forgetting that bitwise AND should be parenthesized before a comparison
- Treating bitwise string operations as Unicode text operations
- Expecting **~** to return the positive opposite of an integer
- Using **@** to hide errors instead of handling them
- Running shell commands through the execution operator with untrusted input
- Using **instanceof** when the value is expected to be a scalar rather than an object

## Try it yourself

1. Join a first and last name with a space using **.**
2. Append a suffix to a message with **.=**
3. Combine a default associative array and a user-preference array with **+**; identify which duplicate value survives.
4. Reverse the array operands and compare the result.
5. Use **&**, **|**, and **^** with **6** and **3**.
6. Shift **5** one place left and right and predict the integer results.
7. Define three permission constants and check whether a flag is present.
8. Remove one permission flag with a bitwise mask.
9. Explain why **&&** is not interchangeable with **&**.
10. Check an object against both its class and an interface with **instanceof**.

## Key takeaways

- **.** joins strings; **.=** appends to and updates a string variable.
- Array **+** is a left-biased union by key, not a list append operation.
- Bitwise operators work on integer bits; logical operators work on truth values.
- Bitmasks can combine, check, and clear flags.
- **instanceof** checks whether an object belongs to a class hierarchy or implements an interface.
- The **@** and shell-execution operators exist, but ordinary application code should generally avoid them.
- Use parentheses when different operator families appear in the same expression.
- Ternary, null coalescing, and nullsafe operators are covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [027 - Logical Operators and Short-Circuit Evaluation](027-logical-operators-and-short-circuit-evaluation.md) | [Open Q&A](../qnas/028-string-array-bitwise-and-other-operators.md) | [Open example](../examples/028-string-array-bitwise-and-other-operators.md) | [029 - Ternary, Null Coalescing, and Nullsafe Operators](029-ternary-null-coalescing-and-nullsafe-operators.md) |