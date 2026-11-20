# 018 - Type Juggling, Casting, and Conversion: Complete Example

This command-line example shows how PHP interprets numeric strings in arithmetic, converts values in string and boolean contexts, performs explicit casts, compares values loosely or strictly, changes a variable with **settype()**, and checks numeric input.

## Question

How can a PHP script make automatic conversion visible, perform explicit conversions deliberately, and check numeric input before using it?

## Goal

Create this project:

~~~text
type-conversion-practice/
|-- README.md
|-- conversions.php
~~~

The **conversions.php** script demonstrates:

- A numeric string used in arithmetic without changing the original variable
- A floating-point result from arithmetic with a decimal numeric string
- String concatenation with an integer
- Boolean conversion of false-like and true-like values
- Explicit integer, float, string, and boolean casts
- The difference between **==** and **===**
- Changing a variable in place with **settype()**
- Checking values with **is_numeric()** before conversion

## Complete code: conversions.php

Save this file at the project root:

~~~php
<?php

$quantity = "4";
$total = $quantity * 3;

echo "Original quantity:" . PHP_EOL;
var_dump($quantity);

echo "Calculated total:" . PHP_EOL;
var_dump($total);

$price = "12.5";
$subtotal = $price * 2;

echo "Subtotal:" . PHP_EOL;
var_dump($subtotal);

$message = "Items: " . 4;

echo "Message:" . PHP_EOL;
var_dump($message);

echo "Boolean conversions:" . PHP_EOL;
var_dump((bool) "0");
var_dump((bool) "false");
var_dump((bool) []);
var_dump((bool) [0]);

$input = "12.9";
$whole = (int) $input;
$decimal = (float) "12.5";
$textId = (string) 42;

echo "Original cast input:" . PHP_EOL;
var_dump($input);

echo "Integer cast:" . PHP_EOL;
var_dump($whole);

echo "Float cast:" . PHP_EOL;
var_dump($decimal);

echo "String cast:" . PHP_EOL;
var_dump($textId);

echo "False converted to string:" . PHP_EOL;
var_dump((string) false);

echo "Loose comparison:" . PHP_EOL;
var_dump(0 == "0");

echo "Strict comparison:" . PHP_EOL;
var_dump(0 === "0");

$mutable = "25";
$changedSuccessfully = settype($mutable, "integer");

echo "settype() result:" . PHP_EOL;
var_dump($changedSuccessfully);

echo "Value changed in place:" . PHP_EOL;
var_dump($mutable);

$validInput = "24";
$invalidInput = "24 items";

echo "Numeric input check:" . PHP_EOL;
var_dump(is_numeric($validInput));
var_dump(is_numeric($invalidInput));

if (is_numeric($validInput)) {
    $count = (int) $validInput;
    echo "Validated count: " . $count . PHP_EOL;
}
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Type Conversion Practice

A small PHP CLI script for observing type juggling, casts, and numeric input checks.

## Requirements

- PHP CLI installed and available as **php** in the terminal
- PHP 8 or later for the arithmetic behavior discussed in this chapter

## Check and run

~~~sh
php -l conversions.php
php conversions.php
~~~

The first command checks PHP syntax. The second runs the script.

## What the script demonstrates

- PHP can interpret a numeric string as a number for arithmetic while the original variable remains a string.
- Concatenation converts an integer to text.
- Boolean conversion treats **"0"** and an empty array as false-like, while **"false"** and a non-empty array are true-like.
- Casts create converted values. The original input remains unchanged when the result is stored in another variable.
- **(int)** discards the fractional part toward zero.
- Loose equality can convert operands; strict equality checks both type and value.
- **settype()** changes the variable it receives.
- **is_numeric()** can check for numeric input, but an application must still enforce its own rules.

## Expected output

~~~text
Original quantity:
string(1) "4"
Calculated total:
int(12)
Subtotal:
float(25)
Message:
string(8) "Items: 4"
Boolean conversions:
bool(false)
bool(true)
bool(false)
bool(true)
Original cast input:
string(4) "12.9"
Integer cast:
int(12)
Float cast:
float(12.5)
String cast:
string(2) "42"
False converted to string:
string(0) ""
Loose comparison:
bool(true)
Strict comparison:
bool(false)
settype() result:
bool(true)
Value changed in place:
int(25)
Numeric input check:
bool(true)
bool(false)
Validated count: 24
~~~

The first numeric check is true for **"24"**; the second is false for **"24 items"**. The example only converts the first input after its numeric check.

## Important distinction

A value being numeric does not necessarily mean it is valid for an application's purpose. A quantity may need to be a whole number within a specific range. Add those checks before using converted input in real application logic.
~~~~

## Understand the output

### Arithmetic uses a numeric interpretation

**$quantity** contains the string **"4"**. Multiplication interprets it numerically and produces integer **12**, but the original variable remains a string because the script does not assign a new value to it.

The decimal numeric string **"12.5"** produces a floating-point subtotal. Floats can have precision limitations, so use a deliberate representation for exact currency amounts.

### Concatenation creates text

The integer **4** is converted to its text form when joined to **"Items: "** with the dot operator. The resulting string has eight characters.

### Boolean conversion follows PHP's false-like values

The string **"0"** and empty array are false-like. The non-empty string **"false"** and the array containing **0** are true-like. PHP does not read the word **"false"** as a boolean value.

### Casts create converted results

The original **$input** stays the string **"12.9"**. The integer cast result is stored separately as **$whole**, where the fractional part has been discarded toward zero. The float and string casts similarly create values of their requested types.

Casting boolean false to string produces an empty string. For a user-facing label, choose a specific word rather than relying on that conversion.

### Comparison operators differ

**0 == "0"** is true because loose comparison can convert the operands. **0 === "0"** is false because the types differ.

### settype() modifies the variable

**settype()** converts **$mutable** itself and returns true when successful. The variable is an integer after the call.

### Numeric checks precede conversion

**is_numeric()** accepts **"24"** and rejects **"24 items"**. The script only casts the accepted input. An application should still validate domain requirements such as minimum, maximum, and whether decimals are allowed.

## Make a change

1. Change **$quantity** to **"6"** and predict the total before running the script.
2. Change **$price** to **"12"** and observe the resulting numeric type.
3. Add checks for an empty string, **"0.0"**, and a non-empty array to the boolean section.
4. Cast **"-12.9"** to an integer and inspect how the fractional part is handled.
5. Compare a boolean, integer, and string with **==** and **===**.
6. Change **$validInput** to **"0"**. Note that numeric validity does not mean the value is a useful quantity.
7. Add a rule that a count must be greater than zero before accepting it.
8. Use **settype()** on a different variable and inspect it before and after.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming arithmetic changes the original variable's type | PHP can interpret a numeric string without reassigning the variable. | Inspect the source and result separately. |
| Using non-numeric text in arithmetic | PHP 8 and later throw a **TypeError** for a non-numeric string. | Validate values before arithmetic. |
| Assuming the string **"false"** converts to false | Non-empty strings other than **"0"** are true-like. | Parse accepted text values explicitly. |
| Assuming an integer cast rounds | It discards the fractional part toward zero. | Use a deliberate rounding function when rounding is intended. |
| Treating a successful **is_numeric()** check as complete validation | The value may still violate range or business rules. | Apply the checks required by the data's meaning. |
| Using **==** when types must also match | Loose equality may convert values. | Prefer **===** when both type and value must match. |
| Casting false to string for a visible label | The result is an empty string. | Choose a label such as **"Disabled"** explicitly. |
| Calling **settype()** when the original value is needed | It changes the variable in place. | Cast into a separate variable when both forms are useful. |

## Key takeaways

- Type juggling interprets values according to the operation's context.
- A numeric string can be used in arithmetic while its variable remains a string.
- Casts request a conversion; **settype()** changes the variable in place.
- Boolean conversion treats **"0"** as false-like but **"false"** as true-like.
- Loose comparison may convert operands; strict comparison checks types too.
- **is_numeric()** checks numeric form, not application-specific validity.
- Validate input according to its purpose before converting and using it.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [017 - Objects, resources, callables, and iterables](../lessons/017-objects-resources-callables-and-iterables.md) | [Read this lesson](../lessons/018-type-juggling-casting-and-conversion.md) | [Open Q&A](../qnas/018-type-juggling-casting-and-conversion.md) | [019 - Type declarations and strict typing](../lessons/019-type-declarations-and-strict-typing.md) |
