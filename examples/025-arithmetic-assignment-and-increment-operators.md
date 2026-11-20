# 025 - Arithmetic, Assignment, and Increment Operators: Complete Example

This command-line project practices PHP arithmetic with an invoice, integer-cent money values, whole-item division, remainders, compound assignments, and prefix/postfix updates.

## Question

How can a PHP program use arithmetic operators to calculate an invoice, split items into complete groups, and update counters without hiding how values change?

## Goal

Create this project:

~~~text
arithmetic-practice/
|-- README.md
|-- arithmetic-demo.php
~~~

The script demonstrates:

- Addition, multiplication, division, modulo, integer division, and exponentiation
- Compound arithmetic assignments
- Prices stored as integer cents
- A clear example of prefix and postfix increment and decrement
- Expected output that can be checked by reading the code

## Complete code: arithmetic-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

function formatCents(int $cents): string
{
    $wholeUnits = intdiv($cents, 100);
    $remainingCents = $cents % 100;

    return '$'
        . $wholeUnits
        . '.'
        . str_pad((string) $remainingCents, 2, '0', STR_PAD_LEFT);
}

echo "Basic arithmetic" . PHP_EOL;
echo "17 divided by 5: " . (17 / 5) . PHP_EOL;
echo "17 divided by 5 as an integer: " . intdiv(17, 5) . PHP_EOL;
echo "17 remainder 5: " . (17 % 5) . PHP_EOL;
echo "2 to the power of 4: " . (2 ** 4) . PHP_EOL;
echo PHP_EOL;

$unitPriceCents = 1250;
$quantity = 3;

$subtotalCents = $unitPriceCents * $quantity;
$discountRatePercent = 10;
$discountCents = intdiv(
    $subtotalCents * $discountRatePercent,
    100
);

$discountedSubtotalCents = $subtotalCents;
$discountedSubtotalCents -= $discountCents;

$taxRatePercent = 8;
$taxCents = intdiv(
    $discountedSubtotalCents * $taxRatePercent,
    100
);

$totalCents = $discountedSubtotalCents;
$totalCents += $taxCents;

echo "Invoice" . PHP_EOL;
echo "Unit price: " . formatCents($unitPriceCents) . PHP_EOL;
echo "Quantity: " . $quantity . PHP_EOL;
echo "Subtotal: " . formatCents($subtotalCents) . PHP_EOL;
echo "Discount ("
    . $discountRatePercent
    . "%): "
    . formatCents($discountCents)
    . PHP_EOL;
echo "Subtotal after discount: "
    . formatCents($discountedSubtotalCents)
    . PHP_EOL;
echo "Tax ("
    . $taxRatePercent
    . "%): "
    . formatCents($taxCents)
    . PHP_EOL;
echo "Final total: " . formatCents($totalCents) . PHP_EOL;
echo PHP_EOL;

$itemCount = 10;
$itemsPerBox = 3;
$completeBoxes = intdiv($itemCount, $itemsPerBox);
$remainingItems = $itemCount % $itemsPerBox;

echo "Packing" . PHP_EOL;
echo "Items: " . $itemCount . PHP_EOL;
echo "Items per box: " . $itemsPerBox . PHP_EOL;
echo "Complete boxes: " . $completeBoxes . PHP_EOL;
echo "Remaining items: " . $remainingItems . PHP_EOL;
echo PHP_EOL;

$counter = 4;
$postfixIncrementValue = $counter++;
$prefixIncrementValue = ++$counter;

echo "Postfix increment returns: "
    . $postfixIncrementValue
    . PHP_EOL;
echo "Prefix increment returns: "
    . $prefixIncrementValue
    . PHP_EOL;
echo "Counter after increments: " . $counter . PHP_EOL;

$decrementCounter = 6;
$postfixDecrementValue = $decrementCounter--;
$prefixDecrementValue = --$decrementCounter;

echo "Postfix decrement returns: "
    . $postfixDecrementValue
    . PHP_EOL;
echo "Prefix decrement returns: "
    . $prefixDecrementValue
    . PHP_EOL;
echo "Counter after decrements: " . $decrementCounter . PHP_EOL;
~~~

## Complete code: README.md

~~~~markdown
# Arithmetic Practice

A small PHP command-line project for practicing arithmetic, assignment, modulo, integer division, and increment/decrement operators.

## Requirements

- PHP 7.4 or later
- A terminal

## Run

~~~sh
php arithmetic-demo.php
~~~

## Expected output

~~~text
Basic arithmetic
17 divided by 5: 3.4
17 divided by 5 as an integer: 3
17 remainder 5: 2
2 to the power of 4: 16

Invoice
Unit price: $12.50
Quantity: 3
Subtotal: $37.50
Discount (10%): $3.75
Subtotal after discount: $33.75
Tax (8%): $2.70
Final total: $36.45

Packing
Items: 10
Items per box: 3
Complete boxes: 3
Remaining items: 1

Postfix increment returns: 4
Prefix increment returns: 6
Counter after increments: 6
Postfix decrement returns: 6
Prefix decrement returns: 4
Counter after decrements: 4
~~~
~~~~

## Understand the code

### Integer cents for the invoice

The unit price is stored as **1250 cents** instead of a float. Multiplying it by the quantity gives a subtotal of **3750 cents**.

The discount is ten percent of that subtotal. The example uses intdiv() so the percentage result is an integer number of cents. It then uses **-=** to subtract the discount from the subtotal.

The tax is calculated from the discounted subtotal. The example adds the tax to a separate total with **+=**. For these particular amounts, both percentage calculations produce whole cents. A real application should define what rounding policy it uses whenever a percentage produces a fraction of a cent.

### Formatting an amount

The formatCents() function uses intdiv() to get whole currency units and **%** to get the remaining cents. str_pad() adds a leading zero for amounts with fewer than ten remaining cents.

For example, **1250** cents becomes **$12.50**.

This simple formatter is designed for the non-negative sample prices in this project. A production formatter should account for negative amounts, currency choices, locale, and the application's rounding rules.

### Quotient and remainder

The packing section divides ten items into groups of three. intdiv() returns three complete boxes, and **%** returns one remaining item.

The earlier arithmetic lines show that regular division can produce a fractional result while integer division discards the fractional part.

### Prefix and postfix updates

The first postfix increment returns the old counter value, **4**, then changes the counter to **5**. The prefix increment then changes it to **6** and returns **6**.

The decrement section starts at **6**. Postfix decrement returns **6** before changing the counter to **5**. Prefix decrement changes it to **4** before returning **4**.

Each update is assigned to a separate variable so the old value, new value, and final counter remain easy to inspect.

## Make a change

1. Change the unit price or quantity and calculate the new subtotal.
2. Change the discount to **15** percent and predict the discounted subtotal.
3. Change the tax rate and observe how the final total is calculated.
4. Change the item count to **14** and predict the complete boxes and remaining items.
5. Change **$counter** to another starting value and trace both increment results.
6. Replace one prefix update with a postfix update and compare the returned value.
7. Change the exponent from **4** to **5**.
8. Try an item count divisible by the box capacity and confirm that the remainder is zero.

## Common mistakes

- Using a float for a value that must be represented as an exact count of cents
- Forgetting that intdiv() truncates instead of rounding
- Expecting **%** to return a decimal quotient
- Forgetting that the result of **%** is the remainder
- Using a zero box capacity as the divisor
- Confusing the value returned by a postfix update with the updated counter
- Assuming a percentage calculation always produces a whole number of cents
- Treating the sample money formatter as suitable for every currency or negative value

## Key takeaways

- Arithmetic operators produce values that can be stored, displayed, or used in another calculation.
- intdiv() and **%** work together to calculate whole groups and leftovers.
- Compound assignments make simple running updates concise.
- Prefix and postfix updates both change a variable, but return different values when used in an expression.
- Integer cents are a useful basic representation for exact simple prices.
- Financial calculations still need an explicit rounding policy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [024 - Operator Precedence and Associativity](../lessons/024-operator-precedence-and-associativity.md) | [Read this lesson](../lessons/025-arithmetic-assignment-and-increment-operators.md) | [Open Q&A](../qnas/025-arithmetic-assignment-and-increment-operators.md) | [026 - Comparison, Identity, and Ordering Operators](../lessons/026-comparison-identity-and-ordering-operators.md) |