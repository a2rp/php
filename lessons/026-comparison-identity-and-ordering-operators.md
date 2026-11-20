# 026 - Comparison, Identity, and Ordering Operators

Comparison operators compare two values. Most of them return a boolean, either **true** or **false**. The spaceship operator returns an integer that describes the ordering of two comparable values.

## What you will learn

- How equality differs from identity
- How to compare values in an order
- What the spaceship operator returns
- How PHP compares numeric strings and other mixed types
- Why strict comparisons are usually easier to reason about
- Why comparison expressions cannot be chained like mathematical notation
- How to compare floating-point values with a tolerance

## Comparison operators at a glance

| Operator | Name | Result |
| --- | --- | --- |
| == | Equal | True when values are equal after type juggling |
| === | Identical | True when values are equal and have the same type |
| != | Not equal | True when values are not equal after type juggling |
| <> | Not equal | Another spelling of loose inequality |
| !== | Not identical | True when values differ or have different types |
| < | Less than | True when the left value is less than the right value |
| > | Greater than | True when the left value is greater than the right value |
| <= | Less than or equal to | True when the left value is less than or equal to the right value |
| >= | Greater than or equal to | True when the left value is greater than or equal to the right value |
| <=> | Spaceship | Returns a value below, equal to, or above zero |

A comparison expression can be saved in a variable or used in a conditional:

~~~php
<?php
$minimumAge = 18;
$age = 20;

$isOldEnough = $age >= $minimumAge;

var_dump($isOldEnough);
~~~

Output:

~~~text
bool(true)
~~~

The expression **$age >= $minimumAge** is evaluated first. It produces **true**, which is stored in **$isOldEnough**.

## Equality and identity

The loose equality operator **==** compares values after PHP's comparison type juggling. That can make values with different types compare equal.

~~~php
<?php
$storedId = 42;
$submittedId = "42";

var_dump($storedId == $submittedId);
var_dump($storedId === $submittedId);
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

The values compare equal with **==**, but they are not identical: one is an integer and one is a string. The strict identity operator **===** requires both the same value and the same type.

Use **===** for predictable checks when you expect a specific type and value. Use **!==** when you need to check that the value or type differs.

~~~php
<?php
$role = "editor";

var_dump($role === "editor");
var_dump($role !== "admin");
~~~

Output:

~~~text
bool(true)
bool(true)
~~~

The loose not-equal operators **!=** and **<>** both compare values after type juggling. The strict not-identical operator **!==** also considers the types:

~~~php
<?php
var_dump(42 != "42");
var_dump(42 !== "42");
~~~

Output:

~~~text
bool(false)
bool(true)
~~~

The first comparison sees equal values after conversion. The second sees different types.

## Comparing numeric values

When comparing numbers, ordering operators answer whether one value is less than, greater than, or equal to another:

~~~php
<?php
$score = 84;

var_dump($score < 70);
var_dump($score >= 70);
var_dump($score <= 100);
~~~

Output:

~~~text
bool(false)
bool(true)
bool(true)
~~~

A common range check has two separate comparisons: the value must be at least the minimum and no greater than the maximum. PHP does not allow the comparisons to be chained as if they were mathematical notation. The next chapter explains how logical operators combine boolean results.

Keep each comparison explicit. For example, check **$minimum <= $value** and **$value <= $maximum** as two comparisons, then combine their results.

## Comparing strings and different types

If both operands are numeric strings, PHP compares them numerically. A numeric string is a string whose contents can be interpreted as a number.

~~~php
<?php
var_dump("10" < "2");
var_dump("10" == 10);
var_dump("10" === 10);
~~~

Output:

~~~text
bool(false)
bool(true)
bool(false)
~~~

The first comparison is numeric: ten is not less than two. The second comparison treats the numeric string and integer as equal after type juggling. The strict comparison is false because a string and integer have different types.

When both values are ordinary strings, PHP compares their string values in lexicographic order:

~~~php
<?php
var_dump("apple" < "banana");
var_dump("pear" < "peach");
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

String ordering is not the same as a language-aware, case-insensitive, or locale-aware human sort. For user-facing text, choose a comparison method that matches the language and sorting rules your application needs.

### PHP 8 changed number-to-string comparisons

In PHP 8 and later, when a number is compared to a non-numeric string with a loose comparison, PHP compares their string representations rather than converting the non-numeric string to zero.

~~~php
<?php
var_dump(0 == "apple");
~~~

On PHP 8 or later, the result is:

~~~text
bool(false)
~~~

Before PHP 8.0, this comparison returned **true** because the non-numeric string was converted to zero. Code that relies on loose comparisons across different types can therefore behave differently across PHP versions. Prefer validating and converting input explicitly, then using strict comparisons.

## Booleans and loose comparisons

Loose comparisons involving booleans can also surprise you because PHP interprets both values as booleans:

~~~php
<?php
var_dump(0 == false);
var_dump(0 === false);
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

The values are loosely equal after boolean conversion, but they are not identical because one is an integer and the other is a boolean. When the distinction matters, use **===** or **!==**.

## The spaceship operator

The spaceship operator **<=>** compares two values and returns:

- A value less than zero if the left operand is less than the right operand
- Zero if the operands compare equal
- A value greater than zero if the left operand is greater than the right operand

For common numeric values, the results are **-1**, **0**, or **1**:

~~~php
<?php
var_dump(3 <=> 8);
var_dump(8 <=> 8);
var_dump(12 <=> 8);
~~~

Output:

~~~text
int(-1)
int(0)
int(1)
~~~

It is useful when a callback needs to return an ordering result, such as a sorting function:

~~~php
<?php
$firstPrice = 1250;
$secondPrice = 1800;

$order = $firstPrice <=> $secondPrice;

var_dump($order);
~~~

Output:

~~~text
int(-1)
~~~

The spaceship operator is also called the three-way comparison operator. It does not return a boolean; use **<**, **>**, or **===** when the code needs a boolean condition.

## Do not chain comparisons

Relational comparison operators such as **<**, **<=**, **>**, and **>=** are non-associative. Do not put them next to one another in a chain:

~~~text
$minimum <= $value <= $maximum
~~~

This is not valid PHP syntax. Write each comparison separately and combine the resulting booleans in a clear expression. Logical operators for combining those results are covered next.

## Comparing floating-point values

Floating-point values can contain small representation and rounding differences. Avoid using **==** or **===** to decide whether two calculations with floats are mathematically equal.

For an approximate comparison, define an acceptable tolerance:

~~~php
<?php
$actual = 0.1 + 0.2;
$expected = 0.3;
$tolerance = 0.00001;

$isCloseEnough = abs($actual - $expected) < $tolerance;

var_dump($isCloseEnough);
~~~

Output:

~~~text
bool(true)
~~~

Choose a tolerance appropriate for the scale and requirements of the calculation. A tolerance suitable for one domain may be far too loose or too strict for another.

## Compare values that have a meaningful ordering

Equality and identity checks answer whether values match. Ordering operators answer which value comes before or after another. Use ordering operators only with values whose ordering makes sense and is defined.

For example, comparing two numbers or two ordinary strings is meaningful. Do not assume PHP defines a useful application-specific order for unrelated or incomparable values. Convert or validate values to a deliberate type before comparing them.

## Common mistakes

- Using **==** when the type must match as well as the value
- Assuming **==** never converts values
- Treating **<>** as different from **!=**; both are loose inequality
- Treating **!==** as another spelling of **!=**; **!==** also checks type
- Forgetting that numeric strings can be compared numerically
- Assuming PHP 7 and PHP 8 compare a number with a non-numeric string the same way
- Comparing integers to booleans loosely when the type distinction matters
- Chaining comparisons as if PHP used mathematical range notation
- Using equality operators to compare calculated floats exactly
- Treating the spaceship operator as if it returned a boolean
- Assuming ordinary string ordering matches a user's language or locale

## Try it yourself

1. Compare the integer **25** with the string **"25"** using both **==** and **===**.
2. Compare **25** and **30** using **<**, **>**, **<=**, and **>=**.
3. Compare two numeric strings and predict whether PHP compares them numerically.
4. Compare two ordinary words and observe their lexicographic ordering.
5. Compare **0** and **false** with both loose and strict equality.
6. Use **<=>** to compare two numbers in both orders.
7. Replace a chained range comparison with two separate comparisons.
8. Compare a calculated float using a tolerance instead of direct equality.
9. Explain why loose comparison of user-provided strings can be risky.
10. On PHP 8 or later, inspect the result of comparing **0** with a non-numeric string using **==**.

## Key takeaways

- Comparison operators produce booleans, except **<=>**, which returns an ordering value.
- **==** and **!=** allow type juggling; **===** and **!==** compare both type and value.
- Prefer strict comparisons when the type is part of the condition.
- PHP compares numeric strings numerically, but loose comparison rules can depend on the types involved.
- PHP 8 changed how numbers compare loosely with non-numeric strings.
- Relational comparisons cannot be chained; write the comparisons separately.
- Use a suitable tolerance when approximate floating-point values must be compared.
- Use ordering operators only when the compared values have a meaningful defined order.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [025 - Arithmetic, Assignment, and Increment Operators](025-arithmetic-assignment-and-increment-operators.md) | [Open Q&A](../qnas/026-comparison-identity-and-ordering-operators.md) | [Open example](../examples/026-comparison-identity-and-ordering-operators.md) | [027 - Logical Operators and Short-Circuit Evaluation](027-logical-operators-and-short-circuit-evaluation.md) |