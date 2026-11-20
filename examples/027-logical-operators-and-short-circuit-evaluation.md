# 027 - Logical Operators and Short-Circuit Evaluation: Complete Example

This command-line project practices logical AND, OR, XOR, NOT, truthiness, assignment precedence, and short-circuit guards.

## Question

How can a PHP program combine conditions safely, and how can you see when short-circuit evaluation skips work?

## Goal

Create this project:

~~~text
logical-operators-practice/
|-- README.md
|-- logical-demo.php
~~~

The script demonstrates:

- AND, OR, and XOR conditions
- Short-circuit behavior with visible function-call messages
- The different precedence of **&&** and **and**, and **||** and **or**
- A non-zero divisor check and a nullable-string guard
- Common truthy and falsey values

## Complete code: logical-demo.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

function logCheck(string $label, bool $result): bool
{
    echo $label . PHP_EOL;

    return $result;
}

echo "Access rules" . PHP_EOL;

$hasAccount = true;
$emailIsVerified = false;
$mayOpenDashboard = $hasAccount && $emailIsVerified;

echo "Dashboard access: ";
var_dump($mayOpenDashboard);

$isMember = false;
$hasInvitation = true;
$mayAttendEvent = $isMember || $hasInvitation;

echo "Event access: ";
var_dump($mayAttendEvent);

$firstChoice = true;
$secondChoice = false;
$exactlyOneChoice = ($firstChoice xor $secondChoice);

echo "Exactly one choice: ";
var_dump($exactlyOneChoice);
echo PHP_EOL;

echo "Short-circuit AND" . PHP_EOL;
$andResult = logCheck("AND left evaluated: false", false)
    && logCheck("AND right evaluated", true);

echo "AND result: ";
var_dump($andResult);
echo PHP_EOL;

echo "Short-circuit OR" . PHP_EOL;
$orResult = logCheck("OR left evaluated: true", true)
    || logCheck("OR right evaluated", false);

echo "OR result: ";
var_dump($orResult);
echo PHP_EOL;

echo "Assignment precedence" . PHP_EOL;

$symbolicAnd = true && false;
$wordAnd = true and false;
$symbolicOr = false || true;
$wordOr = false or true;

echo "Symbolic AND assignment: ";
var_dump($symbolicAnd);
echo "Word AND assignment: ";
var_dump($wordAnd);
echo "Symbolic OR assignment: ";
var_dump($symbolicOr);
echo "Word OR assignment: ";
var_dump($wordOr);
echo PHP_EOL;

echo "Safe guards" . PHP_EOL;

$denominator = 0;
$isSafeQuotient = $denominator !== 0
    && 20 / $denominator > 2;

echo "Division condition: ";
var_dump($isSafeQuotient);

$username = null;
$isValidUsername = $username !== null
    && strlen($username) >= 3;

echo "Username condition: ";
var_dump($isValidUsername);
echo PHP_EOL;

echo "Boolean conversion" . PHP_EOL;
echo "String zero: ";
var_dump((bool) "0");
echo "Word false: ";
var_dump((bool) "false");
echo "Empty array: ";
var_dump((bool) []);
echo "Non-empty array: ";
var_dump((bool) [0]);
~~~

## Complete code: README.md

~~~~markdown
# Logical Operators Practice

A PHP command-line project for practicing boolean conditions, short-circuit evaluation, truthiness, and logical operator precedence.

## Requirements

- PHP 7.4 or later
- A terminal

## Run

~~~sh
php logical-demo.php
~~~

## Expected output

~~~text
Access rules
Dashboard access: bool(false)
Event access: bool(true)
Exactly one choice: bool(true)

Short-circuit AND
AND left evaluated: false
AND result: bool(false)

Short-circuit OR
OR left evaluated: true
OR result: bool(true)

Assignment precedence
Symbolic AND assignment: bool(false)
Word AND assignment: bool(true)
Symbolic OR assignment: bool(true)
Word OR assignment: bool(false)

Safe guards
Division condition: bool(false)
Username condition: bool(false)

Boolean conversion
String zero: bool(false)
Word false: bool(true)
Empty array: bool(false)
Non-empty array: bool(true)
~~~
~~~~

## Understand the code

### Access conditions

The dashboard requires both an account and a verified email, so the example uses **&&**. The event accepts either membership or an invitation, so it uses **||**. The XOR condition is true because exactly one choice is true.

### Short-circuit messages

The helper function prints a message whenever PHP evaluates it.

For AND, the left call returns false. PHP already knows the whole condition must be false, so it does not call the right side.

For OR, the left call returns true. PHP already knows the whole condition must be true, so it skips the right side.

The missing right-side messages are the observable evidence that short-circuiting happened.

### Assignment precedence

The expressions using **&&** and **||** assign their complete boolean results. The word operators **and** and **or** bind less tightly than assignment, so their variables receive the left-side values before the remaining logical operation.

For normal conditions, prefer the symbolic operators. Parentheses can make intentional use of a word operator explicit.

### Safe guards

The division condition checks that the divisor is not zero first. Since that comparison is false, short-circuit AND skips the division.

The username condition checks for null before calling strlen(). A null username makes the first part false, so the function call is skipped.

### Boolean conversion

The string **"0"** and an empty array are falsey. The string **"false"** and a non-empty array are truthy. PHP checks the values themselves, not the English meaning of a string.

## Make a change

1. Verify the email and observe the dashboard result change.
2. Remove the invitation and make the user a member.
3. Set both XOR choices to true, then set both to false.
4. Change the left result in the short-circuit AND call to true and observe the right-side message.
5. Change the left result in the short-circuit OR call to false and observe the right-side message.
6. Change the divisor to a non-zero integer and observe the guarded calculation.
7. Set the username to a string with fewer than three characters, then a longer string.
8. Try **""**, **"0"**, **"false"**, and **[]** in boolean conversions.

## Common mistakes

- Assuming **and** and **&&** have the same precedence
- Assuming **or** and **||** have the same precedence
- Forgetting parentheses when assigning an **xor** expression
- Expecting the right side of a short-circuited expression to run
- Treating **"false"** as false or **"0"** as true
- Assuming logical OR returns the first truthy operand
- Using truthiness when the program needs to distinguish zero, empty text, and null
- Putting important state changes inside logical expressions

## Key takeaways

- AND requires both conditions to be true; OR requires at least one.
- XOR requires exactly one true condition.
- Short-circuit evaluation can safely guard a later operation.
- **&&** and **||** have higher precedence than assignment; **and** and **or** have lower precedence.
- PHP logical operators return booleans.
- Truthiness depends on PHP's rules, not on the words stored in a string.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [026 - Comparison, Identity, and Ordering Operators](../lessons/026-comparison-identity-and-ordering-operators.md) | [Read this lesson](../lessons/027-logical-operators-and-short-circuit-evaluation.md) | [Open Q&A](../qnas/027-logical-operators-and-short-circuit-evaluation.md) | [028 - String, Array, Bitwise, and Other Operators](../lessons/028-string-array-bitwise-and-other-operators.md) |