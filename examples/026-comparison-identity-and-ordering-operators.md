# 026 - Comparison, Identity, and Ordering Operators: Complete Example

This command-line project compares IDs and scores, sorts products by price, checks PHP's number-to-string comparison behavior, and compares approximate floating-point values with a tolerance.

## Question

How can a PHP program compare values deliberately, distinguish matching values from matching types, and sort records by a numeric field?

## Goal

Create this project:

~~~text
comparison-practice/
|-- README.md
|-- comparison-demo.php
~~~

The script demonstrates:

- Loose equality and strict identity
- Less-than, greater-than, and inclusive comparisons
- The PHP 8 behavior for a number compared with a non-numeric string
- The spaceship operator in a sorting callback
- A tolerance-based check for an approximate float calculation

## Complete code: comparison-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

$storedUserId = 42;
$submittedUserId = "42";

echo "User ID checks" . PHP_EOL;
echo "Loose equality: ";
var_dump($storedUserId == $submittedUserId);
echo "Strict identity: ";
var_dump($storedUserId === $submittedUserId);
echo PHP_EOL;

$score = 84;
$passingScore = 70;
$maximumScore = 100;

$atLeastPassingScore = $score >= $passingScore;
$notAboveMaximum = $score <= $maximumScore;

echo "Score checks" . PHP_EOL;
echo "At least the passing score: ";
var_dump($atLeastPassingScore);
echo "Not above the maximum: ";
var_dump($notAboveMaximum);
echo PHP_EOL;

echo "String and type comparisons" . PHP_EOL;
echo "Numeric strings compared numerically: ";
var_dump("10" < "2");
echo "Number compared with non-numeric string (PHP 8+): ";
var_dump(0 == "apple");
echo "Zero loosely equals false: ";
var_dump(0 == false);
echo "Zero is identical to false: ";
var_dump(0 === false);
echo "Apple comes before banana: ";
var_dump("apple" < "banana");
echo PHP_EOL;

echo "Spaceship comparisons" . PHP_EOL;
var_dump(3 <=> 8);
var_dump(8 <=> 8);
var_dump(12 <=> 8);
echo PHP_EOL;

$actual = 0.1 + 0.2;
$expected = 0.3;
$tolerance = 0.00001;
$isCloseEnough = abs($actual - $expected) < $tolerance;

echo "Float result is within tolerance: ";
var_dump($isCloseEnough);
echo PHP_EOL;

$products = [
    ["name" => "Notebook", "price" => 450],
    ["name" => "Pen", "price" => 125],
    ["name" => "Bag", "price" => 1800],
];

usort(
    $products,
    function (array $first, array $second): int {
        return $first["price"] <=> $second["price"];
    }
);

echo "Products ordered by price" . PHP_EOL;
foreach ($products as $product) {
    echo $product["name"]
        . ": "
        . $product["price"]
        . " cents"
        . PHP_EOL;
}
~~~

## Complete code: README.md

~~~~markdown
# Comparison Practice

A PHP command-line project for practicing equality, identity, ordering, the spaceship operator, and approximate float comparisons.

## Requirements

- PHP 8.0 or later
- A terminal

PHP 8.0 or later is required for the demonstrated comparison between the integer 0 and the non-numeric string "apple".

## Run

~~~sh
php comparison-demo.php
~~~

## Expected output

~~~text
User ID checks
Loose equality: bool(true)
Strict identity: bool(false)

Score checks
At least the passing score: bool(true)
Not above the maximum: bool(true)

String and type comparisons
Numeric strings compared numerically: bool(false)
Number compared with non-numeric string (PHP 8+): bool(false)
Zero loosely equals false: bool(true)
Zero is identical to false: bool(false)
Apple comes before banana: bool(true)

Spaceship comparisons
int(-1)
int(0)
int(1)

Float result is within tolerance: bool(true)

Products ordered by price
Pen: 125 cents
Notebook: 450 cents
Bag: 1800 cents
~~~
~~~~

## Understand the code

### Loose equality and strict identity

The stored ID is an integer, while the submitted ID is a string. The loose comparison **==** considers the numeric values equal after type juggling. The identity comparison **===** returns false because the types differ.

For IDs and other values where the type matters, validate and convert input deliberately, then compare with **===**.

### Ordering checks

The score is compared separately with the passing score and the maximum. Each comparison returns a boolean. The following chapter covers logical operators that can combine such results into a single condition.

### String and type behavior

The two numeric strings **"10"** and **"2"** are compared numerically, so the first is not less than the second.

The comparison **0 == "apple"** is false in PHP 8 and later. Before PHP 8, this loose comparison returned true because the non-numeric string was converted to zero. The example's PHP version requirement makes the expected output unambiguous.

Zero loosely compares equal to false after boolean conversion, but they are not identical. The final string comparison demonstrates ordinary lexicographic ordering; this is not a locale-aware display sort.

### The spaceship operator

The three numeric spaceship expressions return a negative value, zero, and a positive value. These are the ordering signals that the sorting callback needs.

### Approximate float comparison

The sum of **0.1 + 0.2** may not be represented as exactly **0.3**. The example measures the difference and checks whether it is smaller than a chosen tolerance.

The tolerance must match the accuracy needs of the actual application.

### Sorting with a comparison callback

usort() calls the callback with two products. The callback compares their integer prices with **<=>**:

- A negative result puts the first product before the second.
- Zero indicates equal prices.
- A positive result puts the first product after the second.

The products are then displayed in ascending price order.

## Make a change

1. Change the stored ID or submitted ID and compare the loose and strict results.
2. Set the score below the passing score, then above the maximum.
3. Change the numeric strings and predict their numeric ordering.
4. Try the number-to-string comparison on PHP 7 and PHP 8 to see the version difference.
5. Reverse the spaceship operands in the sorting callback and inspect the product order.
6. Change a product price so two products have equal prices.
7. Reduce the float tolerance and see how it affects the approximate comparison.
8. Compare text with different capitalization and observe ordinary string ordering.

## Common mistakes

- Assuming **==** also checks the operand types
- Using **!=** when the requirement is strict not-identity
- Forgetting that HTML form values arrive as strings
- Relying on the pre-PHP-8 comparison of a number and a non-numeric string
- Assuming the spaceship operator returns a boolean
- Writing a chained range comparison
- Comparing floats for exact equality
- Treating ordinary string order as locale-aware
- Sorting with the spaceship operator on values that are not known to be comparable

## Key takeaways

- **==** allows type juggling; **===** checks both value and type.
- Use strict comparisons after validating and converting input when exact types matter.
- Ordering comparisons return booleans, while **<=>** returns an ordering value.
- PHP 8 changed loose comparisons between numbers and non-numeric strings.
- Float comparisons often need a domain-appropriate tolerance.
- A spaceship comparison is useful in a sorting callback.
- Ordinary string comparison does not replace locale-aware text sorting.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [025 - Arithmetic, Assignment, and Increment Operators](../lessons/025-arithmetic-assignment-and-increment-operators.md) | [Read this lesson](../lessons/026-comparison-identity-and-ordering-operators.md) | [Open Q&A](../qnas/026-comparison-identity-and-ordering-operators.md) | [027 - Logical Operators and Short-Circuit Evaluation](../lessons/027-logical-operators-and-short-circuit-evaluation.md) |