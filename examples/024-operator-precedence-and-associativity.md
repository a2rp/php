# 024 - Operator Precedence and Associativity: Complete Example

This PHP CLI example compares implicit operator grouping with explicit parentheses. It also shows associativity, the precedence difference between **&&** and **and**, PHP 8 concatenation grouping, nested ternaries, null coalescing, and a safe way to express a range check.

## Question

How can parentheses and a few clear intermediate variables make expressions behave as intended and remain readable?

## Goal

Create this project:

~~~text
precedence-practice/
|-- README.md
|-- precedence-demo.php
~~~

The **precedence-demo.php** script demonstrates:

- Multiplication grouping before addition
- Parentheses changing the result
- Left-associative subtraction and right-associative exponentiation
- The difference between **&&** and **and** when assigning
- Parenthesized arithmetic before string concatenation
- A nested ternary with explicit grouping
- A range check written as two comparisons
- A default value with **??**
- Separate statements for a variable read and increment

## Complete code: precedence-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

$implicitGrouping = 1 + 5 * 3;
$explicitGrouping = (1 + 5) * 3;

$leftAssociative = 10 - 3 - 2;
$rightAssociative = 2 ** 3 ** 2;

$symbolicAnd = true && false;
$wordAnd = true and false;

$totalLabel = "Total: " . (10 + 5);

$score = 45;
$result = $score >= 50
    ? "pass"
    : ($score >= 40 ? "review" : "retry");

$low = 10;
$value = 15;
$high = 20;
$insideRange = $low <= $value && $value <= $high;

$name = null;
$displayName = $name ?? "Guest";

$counter = 1;
$previousCounter = $counter;
$counter++;
$counterSum = $previousCounter + $counter;

echo "Multiplication first: " . $implicitGrouping . PHP_EOL;
echo "Parentheses first: " . $explicitGrouping . PHP_EOL;
echo "Left associative subtraction: " . $leftAssociative . PHP_EOL;
echo "Right associative exponentiation: " . $rightAssociative . PHP_EOL;
echo "Assignment with &&: ";
var_dump($symbolicAnd);
echo "Assignment with and: ";
var_dump($wordAnd);
echo $totalLabel . PHP_EOL;
echo "Nested ternary: " . $result . PHP_EOL;
echo "Inside range: ";
var_dump($insideRange);
echo "Default name: " . $displayName . PHP_EOL;
echo "Counter sum: " . $counterSum . PHP_EOL;
~~~

## Complete code: README.md

~~~~markdown
# Operator Precedence Practice

A small command-line exercise for learning how PHP groups expressions with precedence, associativity, and parentheses.

## Requirements

- PHP 7.4 or later
- A terminal

The main script uses syntax supported by PHP 7.4 or later. The chapter explains the PHP 8.0 ternary grouping changes, and the code uses parentheses to state its grouping clearly.

## Run

~~~sh
php precedence-demo.php
~~~

## Expected output

~~~text
Multiplication first: 16
Parentheses first: 18
Left associative subtraction: 5
Right associative exponentiation: 512
Assignment with &&: bool(false)
Assignment with and: bool(true)
Total: 15
Nested ternary: review
Inside range: bool(true)
Default name: Guest
Counter sum: 3
~~~
~~~~

## Expected output

~~~text
Multiplication first: 16
Parentheses first: 18
Left associative subtraction: 5
Right associative exponentiation: 512
Assignment with &&: bool(false)
Assignment with and: bool(true)
Total: 15
Nested ternary: review
Inside range: bool(true)
Default name: Guest
Counter sum: 3
~~~

## Understand the code

### Precedence and parentheses

In **1 + 5 * 3**, multiplication groups before addition, producing **16**. In **(1 + 5) * 3**, the parentheses force addition first, producing **18**.

### Associativity

Subtraction groups from the left, so **10 - 3 - 2** means **(10 - 3) - 2** and produces **5**.

Exponentiation groups from the right, so **2 ** 3 ** 2** means **2 ** (3 ** 2)** and produces **512**.

### Logical words and symbols

**&&** has higher precedence than assignment, so **$symbolicAnd** receives the result **false**.

**and** has lower precedence than assignment, so **$wordAnd** receives **true** before the remaining logical operation is evaluated. This difference is subtle; prefer **&&** and **||** for normal conditions.

### Concatenation and arithmetic

The calculation **10 + 5** is grouped inside parentheses before the result is concatenated with **"Total: "**. Parentheses make the intended output explicit and avoid relying on PHP-version differences.

### Ternary and comparisons

The nested ternary is parenthesized so the second condition belongs to the first expression's false branch. The range check expresses both comparisons separately and joins them with **&&**.

### Separate state changes

The example stores the counter's current value, increments it on its own line, and then adds the two values. No result depends on an unspecified order of evaluation.

## Optional PHP 8.5+ pipe example

PHP 8.5 added the pipe operator. Save and run this separately on PHP 8.5 or later:

~~~php
<?php
$text = "  php notes  ";

$cleanText = $text
    |> trim(...)
    |> strtoupper(...);

echo $cleanText . PHP_EOL;
~~~

Output:

~~~text
PHP NOTES
~~~

The main project runs on PHP 8.0 or later. Only this optional pipe example requires PHP 8.5.

## Make a change

1. Change the parentheses in **$explicitGrouping** and predict the new value.
2. Change **10 - 3 - 2** to **10 - (3 - 2)** and compare the output.
3. Put parentheses around **true and false** and observe what value is assigned.
4. Change the score to **55**, **42**, and **20** to see each ternary result.
5. Move **$value** outside the range and inspect the boolean result.
6. Replace the arithmetic in **$totalLabel** with an intermediate variable.
7. On PHP 8.5 or later, add another callable step to the pipe chain.

## Common mistakes

- Assuming multiplication and addition group from left to right
- Forgetting exponentiation is right-associative
- Treating **and** as if it had the same precedence as **&&**
- Omitting parentheses when mixing arithmetic and concatenation
- Writing a range check as a chained comparison
- Nesting ternaries without explicit parentheses
- Assuming precedence also guarantees operand evaluation order
- Running the optional **|>** example on PHP before 8.5

## Key takeaways

- Precedence decides which operator groups more tightly.
- Associativity decides grouping when operators share a precedence level.
- Parentheses show the intended grouping directly.
- **&&** and **and** have different precedence relative to assignment.
- PHP 8 changed concatenation precedence and made ternaries non-associative.
- PHP 8.5 added the **|>** pipe operator.
- Separate statements are clearer when an expression changes and reads the same state.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [023 - Expressions and value evaluation](../lessons/023-expressions-and-value-evaluation.md) | [Read this lesson](../lessons/024-operator-precedence-and-associativity.md) | [Open Q&A](../qnas/024-operator-precedence-and-associativity.md) | [025 - Arithmetic, assignment, and increment operators](../lessons/025-arithmetic-assignment-and-increment-operators.md) |