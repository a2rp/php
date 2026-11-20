# 028 - String, Array, Bitwise, and Other Operators: Complete Example

This command-line project joins strings, compares array union with list merging, manipulates permission flags with bitwise operators, and checks an object with **instanceof**.

## Question

How do PHP's string, array, bitwise, and type operators behave in a small program, and how can the output make their differences visible?

## Goal

Create this project:

~~~text
operator-practice/
|-- README.md
|-- operator-demo.php
~~~

The script demonstrates:

- String concatenation and concatenating assignment
- Parentheses around arithmetic before concatenation
- Left-biased array union and list merging
- Array equality versus array identity
- Integer bitwise operations and permission flags
- Class and interface checks with **instanceof**

## Complete code: operator-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

echo "Strings" . PHP_EOL;

$firstName = "Ada";
$lastName = "Lovelace";
$fullName = $firstName . " " . $lastName;

$welcome = "Welcome, " . $fullName;
$welcome .= "!";

echo $welcome . PHP_EOL;

$subtotal = 18;
$tax = 3;
echo "Calculated total: " . ($subtotal + $tax) . PHP_EOL;
echo PHP_EOL;

echo "Arrays" . PHP_EOL;

$defaults = [
    "theme" => "light",
    "itemsPerPage" => 20,
];

$userSettings = [
    "theme" => "dark",
    "language" => "en",
];

$defaultFirst = $defaults + $userSettings;
$userFirst = $userSettings + $defaults;

echo "Default-first theme: " . $defaultFirst["theme"] . PHP_EOL;
echo "User-first theme: " . $userFirst["theme"] . PHP_EOL;

$leftList = ["red", "blue"];
$rightList = ["green", "yellow"];

$unionList = $leftList + $rightList;
$mergedList = array_merge($leftList, $rightList);

echo "Union list: " . implode(", ", $unionList) . PHP_EOL;
echo "Merged list: " . implode(", ", $mergedList) . PHP_EOL;

$firstArray = ["color" => "blue", "size" => "M"];
$reorderedArray = ["size" => "M", "color" => "blue"];

echo "Same array values: ";
var_dump($firstArray == $reorderedArray);
echo "Same array identity: ";
var_dump($firstArray === $reorderedArray);
echo PHP_EOL;

echo "Bitwise operations" . PHP_EOL;

$firstBits = 6;
$secondBits = 3;

echo "6 & 3: " . ($firstBits & $secondBits) . PHP_EOL;
echo "6 | 3: " . ($firstBits | $secondBits) . PHP_EOL;
echo "6 ^ 3: " . ($firstBits ^ $secondBits) . PHP_EOL;
echo "6 << 1: " . ($firstBits << 1) . PHP_EOL;
echo "8 >> 1: " . (8 >> 1) . PHP_EOL;

const CAN_READ = 1;
const CAN_WRITE = 2;
const CAN_DELETE = 4;

$permissions = CAN_READ | CAN_WRITE;
$mayRead = ($permissions & CAN_READ) !== 0;
$mayDelete = ($permissions & CAN_DELETE) !== 0;

echo "Can read: ";
var_dump($mayRead);
echo "Can delete: ";
var_dump($mayDelete);

$permissions &= ~CAN_WRITE;

echo "Permissions after removing write: "
    . $permissions
    . PHP_EOL;
echo PHP_EOL;

echo "Type checks" . PHP_EOL;

interface Exportable
{
    public function export(): string;
}

class Invoice implements Exportable
{
    public function export(): string
    {
        return "Invoice #1001";
    }
}

$invoice = new Invoice();

echo $invoice->export() . PHP_EOL;

var_dump($invoice instanceof Invoice);
var_dump($invoice instanceof Exportable);
var_dump($invoice instanceof stdClass);
~~~

## Complete code: README.md

~~~~markdown
# String, Array, Bitwise, and Other Operators

A PHP command-line project for practicing concatenation, array union, bitwise operations, bitmask flags, and the instanceof type operator.

## Requirements

- PHP 7.4 or later
- A terminal

## Run

~~~sh
php operator-demo.php
~~~

## Expected output

~~~text
Strings
Welcome, Ada Lovelace!
Calculated total: 21

Arrays
Default-first theme: light
User-first theme: dark
Union list: red, blue
Merged list: red, blue, green, yellow
Same array values: bool(true)
Same array identity: bool(false)

Bitwise operations
6 & 3: 2
6 | 3: 7
6 ^ 3: 5
6 << 1: 12
8 >> 1: 4
Can read: bool(true)
Can delete: bool(false)
Permissions after removing write: 1

Type checks
Invoice #1001
bool(true)
bool(true)
bool(false)
~~~
~~~~

## Understand the code

### String operations

The dot operator joins the first name, a space, and the last name. The **.=** operator appends the exclamation mark to the existing greeting.

The arithmetic total is parenthesized before concatenation. The result is calculated as **18 + 3**, then joined to the label.

### Array union and list merging

The associative arrays have a duplicate **theme** key. The left-hand array wins, so **$defaultFirst** keeps **"light"** and **$userFirst** keeps **"dark"**.

The numeric lists both use keys **0** and **1**. Array union keeps the left values at those keys, so the union list contains only **red** and **blue**. array_merge() appends the list values and renumbers the numeric keys, so the merged list contains all four colors.

The two associative arrays have the same key/value pairs but a different order. Loose equality returns true; strict identity returns false because identity also checks order and types.

### Bitwise operations

The integer **6** is binary **0110**, and **3** is binary **0011**:

- AND keeps shared set bits, producing **0010**, or **2**.
- OR keeps bits set in either value, producing **0111**, or **7**.
- XOR keeps bits set in exactly one value, producing **0101**, or **5**.
- A left shift by one gives **12**; a right shift of **8** by one gives **4**.

### Permission flags

Each permission has its own bit value. Bitwise OR combines read and write. Parenthesized bitwise AND checks each flag before comparing the result with zero. The final compound assignment clears write while leaving read enabled.

Use parentheses around bitwise checks. PHP comparison operators have higher precedence than bitwise AND, so the parentheses state the intended calculation.

### The instanceof checks

The invoice object is an **Invoice** and implements **Exportable**, so both checks return true. It is not a **stdClass**, so that check returns false.

The project does not use the error-control or shell-execution operators. They can hide diagnostics or run commands through the system shell and need careful handling.

## Make a change

1. Change the names and join them with the dot operator.
2. Change the duplicate theme value and reverse the order of the array operands.
3. Add another numeric key to the lists and compare union with array_merge().
4. Reorder the array keys and compare **==** with **===**.
5. Change the bit values and predict each bitwise result.
6. Add **CAN_DELETE** to the permission mask and check it.
7. Remove **CAN_READ** instead of **CAN_WRITE**.
8. Add another class that implements **Exportable** and check it with **instanceof**.

## Common mistakes

- Using **+** to concatenate strings
- Mixing arithmetic and concatenation without parentheses
- Expecting array union to append numeric list values
- Assuming the right array wins for duplicate union keys
- Treating array **==** and **===** as interchangeable
- Confusing bitwise **&** with logical **&&**
- Comparing a bitmask without parentheses
- Treating byte-wise string bitwise operations as Unicode text operations
- Using **@** to suppress errors instead of handling failures
- Passing untrusted text into a shell command

## Key takeaways

- Use **.** and **.=** for string concatenation.
- Array **+** combines by key and preserves the left value for duplicate keys.
- array_merge() has different behavior for numeric list keys.
- Array identity checks order and value types in addition to key/value pairs.
- Bitwise operators manipulate integer bits; bitmasks can represent several flags in one integer.
- **instanceof** checks classes, parent classes, and interfaces.
- Keep special error-control and shell-execution features out of ordinary application code unless their behavior is specifically required and safely handled.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [027 - Logical Operators and Short-Circuit Evaluation](../lessons/027-logical-operators-and-short-circuit-evaluation.md) | [Read this lesson](../lessons/028-string-array-bitwise-and-other-operators.md) | [Open Q&A](../qnas/028-string-array-bitwise-and-other-operators.md) | [029 - Ternary, Null Coalescing, and Nullsafe Operators](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) |