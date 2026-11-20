# 023 - Expressions and Value Evaluation: Complete Example

This PHP CLI project demonstrates how expressions produce values during a calculation, how a function return becomes an expression value, how prefix and postfix increment differ, how truthiness works, and how short-circuit logic can skip a function call.

## Question

How can a PHP program combine values into readable calculations while making each result and side effect easy to understand?

## Goal

Create this project:

~~~text
expression-practice/
|-- README.md
|-- invoice-calculator.php
~~~

The **invoice-calculator.php** script demonstrates:

- Arithmetic, comparison, and ternary expressions
- A function call whose returned string is used in output
- Prefix and postfix increment results
- The difference between the strings **"0"** and **"false"** in a boolean context
- Short-circuit evaluation skipping a function call
- Keeping calculations and state changes in separate statements

## Complete code: invoice-calculator.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

function formatMoney(float $amount): string
{
    return '$' . number_format($amount, 2);
}

function confirmAddress(): bool
{
    echo "Address check executed." . PHP_EOL;
    return true;
}

$unitPrice = 18.50;
$quantity = 3;
$subtotal = $unitPrice * $quantity;

$discountRate = 0.10;
$discount = $subtotal >= 50
    ? $subtotal * $discountRate
    : 0.0;

$total = $subtotal - $discount;
$qualifiesForFreeShipping = $total >= 50;

echo "Subtotal: " . formatMoney($subtotal) . PHP_EOL;
echo "Discount: " . formatMoney($discount) . PHP_EOL;
echo "Total: " . formatMoney($total) . PHP_EOL;
echo "Free shipping: "
    . ($qualifiesForFreeShipping ? "yes" : "no")
    . PHP_EOL;

$counter = 4;
$postfixValue = $counter++;
$prefixValue = ++$counter;

echo "Postfix result: " . $postfixValue . PHP_EOL;
echo "Prefix result: " . $prefixValue . PHP_EOL;
echo "Counter: " . $counter . PHP_EOL;

$zeroString = "0";
$wordFalse = "false";

echo "String 0 is truthy: "
    . ((bool) $zeroString ? "yes" : "no")
    . PHP_EOL;
echo "String false is truthy: "
    . ((bool) $wordFalse ? "yes" : "no")
    . PHP_EOL;

$itemsInStock = false;
$canShip = $itemsInStock && confirmAddress();

echo "Can ship: " . ($canShip ? "yes" : "no") . PHP_EOL;
echo "Total type: " . get_debug_type($total) . PHP_EOL;
~~~

## Complete code: README.md

~~~~markdown
# Expression Practice

A small command-line exercise for understanding PHP expressions, returned values, side effects, truthiness, and short-circuit logic.

## Requirements

- PHP 8.0 or later
- A terminal

## Run

~~~sh
php invoice-calculator.php
~~~

## Expected output

~~~text
Subtotal: $55.50
Discount: $5.55
Total: $49.95
Free shipping: no
Postfix result: 4
Prefix result: 6
Counter: 6
String 0 is truthy: no
String false is truthy: yes
Can ship: no
Total type: float
~~~
~~~~

## Expected output

~~~text
Subtotal: $55.50
Discount: $5.55
Total: $49.95
Free shipping: no
Postfix result: 4
Prefix result: 6
Counter: 6
String 0 is truthy: no
String false is truthy: yes
Can ship: no
Total type: float
~~~

The output does not contain **Address check executed.** because short-circuit evaluation skips the call.

## Understand the code

### Calculation expressions

**$unitPrice * $quantity** produces the subtotal. The comparison **$subtotal >= 50** produces a boolean, which the ternary expression uses to choose the discount calculation or **0.0**.

The later assignments store intermediate results in **$subtotal**, **$discount**, and **$total**. Naming the steps makes the calculation easier to read than putting the entire invoice formula in one long expression.

### Function call values

**formatMoney()** returns a string. The call **formatMoney($subtotal)** is an expression whose value is placed into the output string.

The function **confirmAddress()** also returns a boolean, but it has an additional side effect: it prints a message. Its return value and printed output are separate effects.

### Prefix and postfix increment

The counter starts at **4**:

- **$counter++** produces **4**, then changes the counter to **5**.
- **++$counter** changes it to **6**, then produces **6**.

The final counter is **6**. Keeping the prefix and postfix operations on separate lines makes their order clear.

### Truthiness

The string **"0"** converts to **false** in a boolean context. The string **"false"** is non-empty and converts to **true**. The spelling of a string does not make it a boolean value.

For validation, compare with the exact value your application expects instead of relying on broad truthiness.

### Short-circuit evaluation

**$itemsInStock** is false, so **&&** already has a false result. PHP skips the call to **confirmAddress()**. That is why the message from inside the function does not appear in the output.

Short-circuiting is useful for guarding a later check. Avoid placing essential actions in the right side of a condition, because that action may be skipped.

## Make a change

1. Change **$quantity** to **2** and recompute the printed subtotal and discount.
2. Change the free-shipping threshold from **50** to another amount.
3. Change **$counter++** to **++$counter** and record the changed expression result.
4. Cast **"0.0"** to boolean and compare it with **"0"**.
5. Set **$itemsInStock** to **true** and observe when **confirmAddress()** runs.
6. Replace the multi-step calculation with one parenthesized expression, then compare which version is easier to review.
7. Add a comparison that checks whether **$total** is exactly a chosen amount.

## Common mistakes

- Expecting the function call's printed output to be the same as its returned value
- Forgetting that assignment and increment expressions change program state
- Assuming postfix increment produces the updated value
- Assuming the word **"false"** makes a string false
- Treating **"0"** as truthy because it is non-empty
- Assuming the right side of **&&** always runs
- Putting essential side effects inside a short-circuited expression
- Combining too many operations into one hard-to-read expression

## Key takeaways

- Each arithmetic, comparison, assignment, or function-call expression produces a value.
- Intermediate variables make calculations easier to inspect.
- Prefix and postfix increment produce different values.
- PHP truthiness has special cases, including **"0"**.
- Short-circuit logic can skip the right-hand expression.
- Keep important side effects and state changes explicit.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [022 - Constants, predefined constants, and magic constants](../lessons/022-constants-predefined-constants-and-magic-constants.md) | [Read this lesson](../lessons/023-expressions-and-value-evaluation.md) | [Open Q&A](../qnas/023-expressions-and-value-evaluation.md) | [024 - Operator precedence and associativity](../lessons/024-operator-precedence-and-associativity.md) |