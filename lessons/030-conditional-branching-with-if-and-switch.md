# 030 - Conditional Branching with if and switch

Conditional statements let a PHP program choose which statements to run. Use `if`, `elseif`, and `else` for conditions and ranges; use `switch` when one expression is compared with several discrete values.

A condition is evaluated using PHP's truthiness rules. For comparisons where types matter, prefer strict operators such as `===` and `!==`. In a `switch`, case matching is loose, and a missing `break` can cause execution to continue into the next case.

## What you will learn

- How to write `if`, `elseif`, and `else` branches
- How PHP truthiness affects conditions
- How to combine conditions clearly
- When strict comparisons are useful
- How `switch`, `case`, `break`, and `default` work
- How intentional fall-through differs from an accidental missing `break`
- When an `if` chain is clearer than a `switch`

## The `if` statement

An `if` statement runs its body when the condition evaluates to `true`.

~~~php
<?php
$temperature = 28;

if ($temperature > 25) {
    echo 'It is warm.';
}
~~~

Output:

~~~text
It is warm.
~~~

The parentheses contain the condition. Curly braces mark the body. Use braces even when a branch contains just one statement; that keeps later edits predictable and makes nested conditions easier to read.

## Add alternatives with `elseif` and `else`

`elseif` checks another condition only when the preceding conditions were false. PHP executes the first matching branch, then skips the rest of that chain. `else` provides a fallback when none of the conditions matched.

~~~php
<?php
$score = 82;

if ($score >= 90) {
    $grade = 'A';
} elseif ($score >= 80) {
    $grade = 'B';
} elseif ($score >= 70) {
    $grade = 'C';
} else {
    $grade = 'Needs more practice';
}

echo $grade;
~~~

Output:

~~~text
B
~~~

Order conditions from the most specific or highest threshold to the least. If the `70` condition appeared first, a score of `82` would match it and never reach the `80` condition.

An `else` is optional. If every branch is optional and none matches, PHP simply runs none of them.

## Conditions use truthiness

PHP converts the value in an `if` condition to a boolean. These common values are falsey:

- `false`
- `0` and `0.0`
- `''` and `'0'`
- `[]`
- `null`

Other values, including non-empty strings such as `'false'`, are truthy. This can surprise you when a value comes from a form, URL, or decoded input.

~~~php
<?php
$submitted = '0';

if ($submitted) {
    echo 'A value was submitted';
} else {
    echo 'The value is falsey';
}
~~~

Output:

~~~text
The value is falsey
~~~

If the difference matters, check explicitly rather than relying on truthiness. For example, use `$count > 0` when you mean a positive count, or compare a string against the allowed values.

~~~php
<?php
$quantity = 0;

if ($quantity > 0) {
    echo 'In stock';
} else {
    echo 'Out of stock';
}
~~~

## Use comparisons deliberately

Loose comparison `==` may consider values of different types equal. Strict comparison `===` checks both value and type. Use strict comparison for known status strings, enum-like values, booleans, and sentinel values when type conversion is not intended.

~~~php
<?php
$submittedCode = '0';

if ($submittedCode === 0) {
    echo 'Integer zero';
} elseif ($submittedCode === '0') {
    echo 'String zero';
} else {
    echo 'Something else';
}
~~~

Output:

~~~text
String zero
~~~

If a value can arrive in several forms, validate and normalize it at the boundary of the application. Strict comparisons are predictable after the input has a known type.

## Combine conditions

Logical operators let one branch check more than one requirement. Use parentheses when a condition has several parts so the grouping is visible.

~~~php
<?php
$age = 22;
$hasTicket = true;

if ($age >= 18 && $hasTicket) {
    echo 'Entry allowed';
} else {
    echo 'Entry denied';
}
~~~

`&&` short-circuits: when its left side is false, PHP does not evaluate the right side. `||` also short-circuits when its left side is true. Do not put required side effects inside the right side of a short-circuit expression.

~~~php
<?php
if ($isAdministrator || $hasValidInvitation) {
    echo 'Access allowed';
}
~~~

## Nested conditions

An `if` can contain another `if`, but deep nesting can make the decision hard to follow. Prefer combined conditions or early returns when they make the path clearer. Always use braces to show which `else` belongs to which `if`.

~~~php
<?php
if ($accountIsActive) {
    if ($emailIsVerified) {
        echo 'Account ready';
    } else {
        echo 'Verify your email';
    }
} else {
    echo 'Account disabled';
}
~~~

Here, the inner `else` belongs to `$emailIsVerified`. Indentation alone does not determine which `if` an `else` belongs to, so braces prevent a dangling-`else` mistake.

## Use `switch` for discrete choices

A `switch` compares one expression with several `case` values. PHP evaluates the switch expression once, then checks the cases. When a case matches, PHP starts executing statements at that point and continues until it reaches `break` or the end of the switch.

~~~php
<?php
$format = 'json';

switch ($format) {
    case 'html':
        $contentType = 'text/html';
        break;
    case 'json':
        $contentType = 'application/json';
        break;
    case 'text':
        $contentType = 'text/plain';
        break;
    default:
        $contentType = 'application/octet-stream';
}

echo $contentType;
~~~

Output:

~~~text
application/json
~~~

`default` runs when no case matches. By convention, put it last. If there is no matching case and no `default`, the switch body does nothing.

### `switch` uses loose comparison

PHP compares the switch expression to each case using loose comparison. This means a case can match after PHP performs type conversion. If a strict type-sensitive comparison is required, normalize the value and use an `if` chain with `===`, or use the `match` expression covered in the next chapter.

~~~php
<?php
$input = '2';

switch ($input) {
    case 2:
        echo 'Matched the integer case';
        break;
    default:
        echo 'No match';
}
~~~

The string `'2'` can match the integer case because switch matching is loose. Avoid relying on implicit conversion for untrusted or inconsistently typed input.

## `break` prevents accidental fall-through

A `break` exits the switch. Without it, execution continues through the statements of following cases, even when those case labels did not match.

~~~php
<?php
$level = 'basic';

switch ($level) {
    case 'basic':
        echo 'Basic features';
        // Missing break: execution continues into the next case.
    case 'pro':
        echo 'Pro features';
        break;
}
~~~

Output:

~~~text
Basic featuresPro features
~~~

Usually, each case should end with `break`, `return`, or another statement that deliberately leaves the switch. Fall-through is sometimes intentional. To share behavior between several values, group empty cases together:

~~~php
<?php
$day = 'Sunday';

switch ($day) {
    case 'Saturday':
    case 'Sunday':
        echo 'Weekend';
        break;
    default:
        echo 'Weekday';
}
~~~

Both weekend labels lead into the same body. This deliberate grouping is clear because the first case has no statements before the second label.

## `switch` inside a loop

A switch is not a substitute for the loop itself. In a loop, `break` exits the nearest switch or loop. If a `switch` is inside a loop and you need to skip to the next loop iteration, `continue 2` targets the outer loop.

~~~php
<?php
$commands = ['skip', 'print'];

foreach ($commands as $command) {
    switch ($command) {
        case 'skip':
            continue 2;
        case 'print':
            echo 'Printed';
            break;
    }
}
~~~

Output:

~~~text
Printed
~~~

Here `continue 2` skips the rest of the switch and advances the `foreach` loop. A plain `continue` inside a switch behaves like `break` and may produce a warning because it is often a mistake; spell out the intended level.

## `if` chain or `switch`?

| Choose | When it fits |
| --- | --- |
| `if` / `elseif` / `else` | Ranges, unrelated conditions, compound comparisons, or strict `===` checks |
| `switch` | Several discrete choices compared against the same expression, when loose case matching is acceptable |
| `match` | A value-producing expression with strict identity comparisons; see the next chapter |

For example, use `if` for score ranges, because each branch checks a different comparison. Use `switch` for a known format string with a case for each format.

## PHP templates

PHP also supports an alternative syntax that can make control flow easier to read when PHP is embedded in HTML. Use the colon after the condition and close the block with `endif`.

~~~php
<?php if ($isSignedIn): ?>
    <p>Welcome back.</p>
<?php else: ?>
    <p>Please sign in.</p>
<?php endif; ?>
~~~

For ordinary PHP-only files, braces are the usual style. Avoid switching syntaxes repeatedly inside one file.

## Try it yourself

1. Write an `if` / `else` that labels a temperature as `Cold` below 10 and `Warm` otherwise.
2. Use `elseif` to assign a grade for scores at 90, 80, and 70 point thresholds.
3. Test `0`, `'0'`, `''`, `false`, and `null` in an `if` condition.
4. Compare a string input to an integer with both `==` and `===`.
5. Write a condition that requires both an active account and a verified email.
6. Use `switch` to map `html`, `json`, and `text` to content types.
7. Remove one `break` and observe how execution falls through.
8. Group two case labels that share one body.
9. Change the switch value to an unsupported format and handle it with `default`.
10. Rewrite a `switch` that needs strict comparisons as an `if`/`elseif` chain.
11. Use the alternative syntax to display one of two short HTML messages.

## Key takeaways

- An `if` chain runs the first branch whose condition is true; `else` handles the remaining case.
- PHP converts conditions to booleans, so falsey values such as `0`, `'0'`, `''`, `[]`, and `null` need care.
- Prefer `===` when both value and type must match.
- A `switch` checks one expression against discrete case values and compares them loosely.
- End each case with `break`, `return`, or another intentional exit; group empty case labels when sharing behavior.
- Use `if` for ranges and compound or strict conditions; use `switch` for discrete alternatives.
- The next chapter introduces `match`, a strict, value-returning alternative for many discrete choices.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [029 - Ternary, Null Coalescing, and Nullsafe Operators](029-ternary-null-coalescing-and-nullsafe-operators.md) | [Open Q&A](../qnas/030-conditional-branching-with-if-and-switch.md) | [Open example](../examples/030-conditional-branching-with-if-and-switch.md) | [031 - The match expression](031-the-match-expression.md) |
