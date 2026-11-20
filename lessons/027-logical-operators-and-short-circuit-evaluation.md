# 027 - Logical Operators and Short-Circuit Evaluation

Logical operators combine or invert truth values. They are commonly used in conditions to decide whether an action should happen. Short-circuit evaluation lets PHP skip an operand when the result is already known.

## What you will learn

- How **&&**, **and**, **||**, **or**, **xor**, and **!** work
- How logical operators convert values to booleans
- Which common values PHP considers false
- How short-circuit evaluation can guard an operation
- Why **and** and **or** can behave unexpectedly with assignment
- Why logical operators return booleans instead of one of their operands

## Logical operators at a glance

| Operator | Meaning | Result |
| --- | --- | --- |
| **&&** | Logical AND | True when both operands are truthy |
| **and** | Logical AND | Same truth rule as **&&**, but lower precedence |
| **||** | Logical OR | True when at least one operand is truthy |
| **or** | Logical OR | Same truth rule as **||**, but lower precedence |
| **xor** | Exclusive OR | True when exactly one operand is truthy |
| **!** | Logical NOT | Reverses the truth value |

A logical operator evaluates its operands in a logical context. The result of a logical expression is a boolean:

~~~php
<?php
$hasAccount = true;
$emailIsVerified = false;

$canOpenDashboard = $hasAccount && $emailIsVerified;

var_dump($canOpenDashboard);
~~~

Output:

~~~text
bool(false)
~~~

Both sides of **&&** must be truthy for the result to be true.

## AND: both conditions must be true

Use **&&** or **and** when an action requires two conditions:

~~~php
<?php
$age = 20;
$hasTicket = true;

$mayEnter = $age >= 18 && $hasTicket;

var_dump($mayEnter);
~~~

Output:

~~~text
bool(true)
~~~

The conditions are evaluated as booleans. **&&** returns true only when both are true.

## OR: either condition can be true

Use **||** or **or** when either condition is enough:

~~~php
<?php
$isMember = false;
$hasInvitation = true;

$mayAttend = $isMember || $hasInvitation;

var_dump($mayAttend);
~~~

Output:

~~~text
bool(true)
~~~

The result is true if at least one operand is truthy.

## XOR: exactly one condition is true

The **xor** operator returns true when exactly one operand is truthy. If both operands have the same truth value, it returns false.

~~~php
<?php
var_dump(true xor false);
var_dump(true xor true);
var_dump(false xor false);
~~~

Output:

~~~text
bool(true)
bool(false)
bool(false)
~~~

For assigning an **xor** result, use parentheses because **xor** has lower precedence than assignment:

~~~php
<?php
$firstChoice = true;
$secondChoice = false;

$exactlyOneChoice = ($firstChoice xor $secondChoice);

var_dump($exactlyOneChoice);
~~~

Output:

~~~text
bool(true)
~~~

## NOT: reverse a truth value

The **!** operator negates a value in a logical context:

~~~php
<?php
$isPaused = false;
$mayContinue = !$isPaused;

var_dump($mayContinue);
~~~

Output:

~~~text
bool(true)
~~~

Use **!** when the condition is naturally expressed as a negative, such as “not paused.” Avoid double negatives that make a condition hard to read.

## Truthy and falsey values

When PHP needs a boolean in a condition, it converts the value to **true** or **false**. These values are false:

- The boolean **false**
- Integer **0**
- Float **0.0** and **-0.0**
- The empty string **""**
- The string **"0"**
- An empty array **[]**
- **null**

Most other values are true. In particular, the string **"false"** is true because it is a non-empty string other than **"0"**.

~~~php
<?php
var_dump((bool) "0");
var_dump((bool) "false");
var_dump((bool) []);
var_dump((bool) [0]);
~~~

Output:

~~~text
bool(false)
bool(true)
bool(false)
bool(true)
~~~

An explicit **(bool)** cast is useful when you need to show or store the conversion. In a condition, PHP converts the value automatically:

~~~php
<?php
$items = [];

if ($items) {
    echo "There are items." . PHP_EOL;
} else {
    echo "The list is empty." . PHP_EOL;
}
~~~

Output:

~~~text
The list is empty.
~~~

Do not use a truthiness check when you need to distinguish a particular value from other falsey values. For example, **0**, **""**, and **null** are different data even though each is false in a logical context. Use a specific comparison or a type check when that distinction matters.

## Short-circuit evaluation

The symbolic operators **&&** and **||** stop once the result is known:

- With **&&**, if the left operand is false, the right operand is skipped.
- With **||**, if the left operand is true, the right operand is skipped.

The word operators **and** and **or** also short-circuit. Their difference from **&&** and **||** is precedence, not the truth rule.

~~~php
<?php
$denominator = 0;

$isValidDivision = $denominator !== 0
    && 20 / $denominator > 2;

var_dump($isValidDivision);
~~~

Output:

~~~text
bool(false)
~~~

Because **$denominator !== 0** is false, PHP skips the division. This pattern can protect an operation, but keep the guard clear and check the exact type and range the operation requires.

A nullable value can also be checked before a function that requires a string:

~~~php
<?php
$username = null;

$isUsableUsername = $username !== null
    && strlen($username) >= 3;

var_dump($isUsableUsername);
~~~

Output:

~~~text
bool(false)
~~~

When the username is **null**, the right operand is skipped, so **strlen()** is not called with null.

## Precedence: && and and are not interchangeable with assignment

The operators **&&** and **||** have higher precedence than assignment. The word operators **and** and **or** have lower precedence than assignment.

~~~php
<?php
$symbolicAnd = true && false;
$wordAnd = true and false;

$symbolicOr = false || true;
$wordOr = false or true;

var_dump($symbolicAnd);
var_dump($wordAnd);
var_dump($symbolicOr);
var_dump($wordOr);
~~~

Output:

~~~text
bool(false)
bool(true)
bool(true)
bool(false)
~~~

The symbolic forms assign the complete logical result. The word forms group after assignment:

~~~text
$wordAnd = (true and false)
~~~

is not how PHP groups the expression. PHP groups it like this:

~~~text
($wordAnd = true) and false
~~~

Therefore, **$wordAnd** receives **true**. The same issue explains why **$wordOr** receives **false** in the example.

Prefer **&&** and **||** for ordinary conditions. If you intentionally use **and** or **or** with assignment, add parentheses to show the intended grouping.

The same precedence warning applies to **xor** when it appears in an assignment. Parenthesize the **xor** expression.

## Logical operators return booleans

In PHP, **&&**, **||**, **and**, **or**, **xor**, and **!** produce boolean results. They do not return one of the original operands.

~~~php
<?php
$result = 0 || "ready";

var_dump($result);
~~~

Output:

~~~text
bool(true)
~~~

The string **"ready"** is truthy, so the logical OR expression returns **true**, not the string. Use a conditional expression or the null coalescing operator when you need to select and return one of several values; those operators have their own behavior and are covered separately.

## Keep side effects out of logical operands

Short-circuiting determines whether the right operand is evaluated. Keep important changes and output in explicit statements instead of hiding them inside a condition.

~~~php
<?php
$attempts = 0;

$attempts++;
$isAllowed = $attempts <= 3;

var_dump($isAllowed);
~~~

Now the counter update is visible and happens before the comparison. This is easier to review than combining a state change with several conditions.

## Common mistakes

- Assuming **and** has the same precedence as **&&**
- Assuming **or** has the same precedence as **||**
- Forgetting that **and** and **or** bind less tightly than assignment
- Assigning an **xor** expression without parentheses
- Treating **"false"** as false; it is a truthy non-empty string
- Treating **"0"** as truthy; PHP considers that string false
- Using a truthiness check when the code must distinguish zero, empty text, and null
- Expecting **||** to return the first truthy operand; it returns a boolean
- Placing a function call or operation on the right side without understanding when short-circuiting skips it
- Hiding state changes inside complicated logical expressions

## Try it yourself

1. Make an AND condition that is true only when a user is signed in and has verified an email.
2. Make an OR condition that allows entry for either a member or an invited guest.
3. Use **xor** to check whether exactly one of two boolean flags is true.
4. Use **!** to invert a boolean variable.
5. Predict the boolean conversion of **0**, **"0"**, **"false"**, **[]**, and **[0]**.
6. Use **&&** to check that a divisor is non-zero before dividing.
7. Compare the assigned values from **true && false** and **true and false**.
8. Compare the assigned values from **false || true** and **false or true**.
9. Parenthesize an **xor** expression assigned to a variable.
10. Rewrite a complicated condition so each check and state change is easy to read.

## Key takeaways

- Logical AND requires both operands to be truthy; logical OR requires at least one.
- **xor** is true only when exactly one operand is truthy.
- **!** reverses a truth value.
- PHP converts values to booleans in logical contexts; **"0"** and an empty array are false, while **"false"** is true.
- **&&**, **||**, **and**, and **or** short-circuit.
- **&&** and **||** bind more tightly than assignment; **and**, **or**, and **xor** bind less tightly.
- PHP logical operators return booleans, not one of their operands.
- Put state changes in explicit statements and use short-circuit guards deliberately.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [026 - Comparison, Identity, and Ordering Operators](026-comparison-identity-and-ordering-operators.md) | [Open Q&A](../qnas/027-logical-operators-and-short-circuit-evaluation.md) | [Open example](../examples/027-logical-operators-and-short-circuit-evaluation.md) | [028 - String, Array, Bitwise, and Other Operators](028-string-array-bitwise-and-other-operators.md) |