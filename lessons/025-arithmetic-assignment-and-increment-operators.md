# 025 - Arithmetic, Assignment, and Increment Operators

PHP arithmetic operators calculate with numeric values. Assignment operators store values in variables, and increment or decrement operators change a value by one.

## What you will learn

- How to add, subtract, multiply, divide, find a remainder, and raise a number to a power
- How division, modulo, and intdiv() differ
- How arithmetic results can be integers or floats
- How compound assignment updates a variable
- How prefix and postfix increment and decrement return different values
- Why division by zero and floating-point precision need care

## Arithmetic operators

For ordinary numeric values, PHP provides these operators:

| Operator | Meaning | Example | Result |
| --- | --- | --- | --- |
| + | Addition | 8 + 3 | 11 |
| - | Subtraction | 8 - 3 | 5 |
| * | Multiplication | 8 * 3 | 24 |
| / | Division | 8 / 3 | Approximately 2.6667 |
| % | Remainder (modulo) | 8 % 3 | 2 |
| ** | Exponentiation | 2 ** 3 | 8 |

The unary operators + and - can also mark a positive or negative numeric value:

~~~php
<?php
$temperatureChange = -4;
$positiveValue = +12;

echo $temperatureChange . PHP_EOL;
echo $positiveValue . PHP_EOL;
~~~

Output:

~~~text
-4
12
~~~

Operator precedence and associativity determine how combined expressions are grouped. Use parentheses when they make the intended calculation easier to read. See [024 - Operator Precedence and Associativity](024-operator-precedence-and-associativity.md) for those rules.

## Division and integer division

The division operator / calculates a quotient. Its result is usually a float when the quotient has a fractional part. If both operands are integers and divide exactly, PHP can return an integer.

~~~php
<?php
var_dump(8 / 2);
var_dump(7 / 2);
~~~

Output:

~~~text
int(4)
float(3.5)
~~~

Use intdiv() when you need the integer quotient. It discards the fractional part by truncating toward zero; it does not round to the nearest integer.

~~~php
<?php
echo intdiv(7, 2) . PHP_EOL;
echo intdiv(-7, 2) . PHP_EOL;
~~~

Output:

~~~text
3
-3
~~~

Use / for a regular quotient and intdiv() for an integer quotient. Both require a non-zero divisor. A zero divisor raises a DivisionByZeroError in current PHP versions, so check a divisor when its value can come from data or user input.

## Remainders with the modulo operator

The % operator returns the remainder after integer division. PHP converts both operands to integers before calculating it, so use % for integer remainder work rather than decimal arithmetic.

~~~php
<?php
echo 8 % 3 . PHP_EOL;
echo 12 % 4 . PHP_EOL;
echo -8 % 3 . PHP_EOL;
~~~

Output:

~~~text
2
0
-2
~~~

A result of 0 means the dividend divides evenly by the divisor. The remainder has the same sign as the dividend, the value on the left side of %.

A common use is checking whether an integer is even:

~~~php
<?php
$number = 14;

if ($number % 2 === 0) {
    echo "Even" . PHP_EOL;
}
~~~

The comparison operator === is covered in [026 - Comparison, Identity, and Ordering Operators](026-comparison-identity-and-ordering-operators.md).

## Exponentiation

The exponentiation operator, written as two asterisks, raises the value on its left to the power on its right.

~~~php
<?php
echo 2 ** 4 . PHP_EOL;
echo 5 ** 2 . PHP_EOL;
echo 2 ** -2 . PHP_EOL;
~~~

Output:

~~~text
16
25
0.25
~~~

Exponentiation can produce a float, such as when the exponent is negative. Parentheses can clarify a negative base:

~~~php
<?php
echo (-2) ** 2 . PHP_EOL;
~~~

Output:

~~~text
4
~~~

## Arithmetic results and numeric types

An arithmetic expression produces a value. Depending on the operands and operation, that value can be an int or a float.

~~~php
<?php
$wholeNumber = 6 + 4;
$fractionalNumber = 7 / 2;

var_dump($wholeNumber);
var_dump($fractionalNumber);
~~~

Output:

~~~text
int(10)
float(3.5)
~~~

Integer values have platform-dependent limits. If an integer calculation exceeds the available integer range, PHP can produce a float instead. Do not assume every arithmetic result remains an integer.

Floats have limited precision. For example, decimal fractions such as 0.1 cannot be represented exactly in binary floating-point. Avoid relying on the last digits of a float for exact calculations.

For money, a common basic approach is to store whole minor units as integers: for example, store $12.50 as 1250 cents. Apply a deliberate rounding policy when calculating percentages or taxes. For financial systems, use a suitable decimal or arbitrary-precision approach when the requirements demand it.

## Assignment stores a value

The basic assignment operator = stores the value from the right side in the variable on the left:

~~~php
<?php
$score = 10;
$score = $score + 5;

echo $score . PHP_EOL;
~~~

Output:

~~~text
15
~~~

Assignment is not a comparison. It changes a variable. Comparison operators such as === answer a question about two values and are covered in the next chapter.

An assignment expression itself has the value being assigned. Keep simple assignments on their own line so that each change is easy to follow:

~~~php
<?php
$first = 7;
$second = $first;

echo $second . PHP_EOL;
~~~

Both variables now contain 7. Later changing one ordinary scalar variable does not change the other.

## Compound arithmetic assignment

A compound assignment performs an arithmetic operation and stores the result back into the variable:

| Compound operator | Equivalent numeric operation |
| --- | --- |
| += | $value = $value + $amount |
| -= | $value = $value - $amount |
| *= | $value = $value * $factor |
| /= | $value = $value / $divisor |
| %= | $value = $value % $divisor |
| **= | $value = $value ** $power |

Example:

~~~php
<?php
$total = 20;
$total += 5;
$total *= 2;
$total -= 10;

echo $total . PHP_EOL;
~~~

Output:

~~~text
40
~~~

The updates happen in order: 20 + 5 is 25, 25 * 2 is 50, and 50 - 10 is 40.

Use a compound operator when it makes a straightforward update clearer. The same division-by-zero and numeric-type considerations apply to /= and %=.

## Increment and decrement

Increment ++ adds one. Decrement -- subtracts one. Both have prefix and postfix forms.

| Form | What the expression returns | What happens to the variable |
| --- | --- | --- |
| ++$value | The updated value | Increases by one first |
| $value++ | The old value | Increases by one after returning it |
| --$value | The updated value | Decreases by one first |
| $value-- | The old value | Decreases by one after returning it |

The difference matters when the operator's result is used in an assignment or output:

~~~php
<?php
$count = 4;
$postfixValue = $count++;
$prefixValue = ++$count;

echo $postfixValue . PHP_EOL;
echo $prefixValue . PHP_EOL;
echo $count . PHP_EOL;
~~~

Output:

~~~text
4
6
6
~~~

First, the postfix increment gives 4 to $postfixValue and then changes $count to 5. Next, the prefix increment changes $count to 6 before giving 6 to $prefixValue.

If you only need to change the variable, put the operation on its own line:

~~~php
<?php
$count++;
$count--;
~~~

This is easier to read than embedding a state change inside a longer expression. The same rule applies to decrementing: postfix returns the old value; prefix returns the decremented value.

## A note about the plus sign

For numbers, + adds. When both operands are arrays, PHP uses + for array union instead. That array behavior is separate from numeric arithmetic; use this chapter's arithmetic examples with numeric operands.

## Avoid zero divisors

Do not perform division with a divisor that might be zero without handling that case. For user-provided or calculated divisors, validate before using /, %, or intdiv():

~~~php
<?php
$divisor = 0;

if ($divisor === 0) {
    echo "A non-zero divisor is required." . PHP_EOL;
} else {
    echo 20 / $divisor . PHP_EOL;
}
~~~

The guarded branch prevents the invalid operation. Input validation and error handling are developed further in later chapters.

## Common mistakes

- Using / when an integer quotient is required; use intdiv() for that purpose.
- Expecting intdiv() to round; it truncates toward zero.
- Treating % as a decimal remainder operator; its operands are converted to integers.
- Forgetting that % keeps the sign of its left operand.
- Dividing by zero or using zero as the right operand of %, /, or intdiv().
- Assuming a calculation with floats is exact to every decimal digit.
- Using = as if it compared two values; it assigns a value.
- Confusing the old value returned by postfix $value++ with the updated value returned by prefix ++$value.
- Hiding several state changes inside one complicated expression.
- Assuming + always means numeric addition when working with arrays.

## Try it yourself

1. Calculate the sum, difference, product, and quotient of two variables.
2. Find the quotient and remainder when 29 is divided by 4.
3. Compare 29 / 4 with intdiv(29, 4).
4. Use % to decide whether a positive integer is even or odd.
5. Calculate 3 raised to the fourth power.
6. Update a running total with +=, then apply a discount with -=.
7. Predict the values of a variable and an assigned result after postfix increment.
8. Repeat the previous exercise with prefix increment and compare the assigned result.
9. Rewrite an expression containing both a variable update and another read as separate statements.
10. Store a price in cents and calculate a quantity subtotal using integers.

## Key takeaways

- PHP's basic arithmetic operators are +, -, *, /, %, and **.
- / returns a regular quotient; intdiv() returns an integer quotient truncated toward zero.
- % returns an integer remainder, and its operands are converted to integers.
- Arithmetic can produce either integers or floats; integer overflow can also result in a float.
- Floats have limited precision, so they are not a reliable way to represent every exact decimal amount.
- Compound assignments update a variable using an arithmetic operation.
- Prefix increment or decrement changes the value before returning it; postfix returns the old value first.
- Check that a divisor is non-zero before dividing.
- Keep variable changes explicit and easy to read.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [024 - Operator Precedence and Associativity](024-operator-precedence-and-associativity.md) | [Open Q&A](../qnas/025-arithmetic-assignment-and-increment-operators.md) | [Open example](../examples/025-arithmetic-assignment-and-increment-operators.md) | [026 - Comparison, Identity, and Ordering Operators](026-comparison-identity-and-ordering-operators.md) |