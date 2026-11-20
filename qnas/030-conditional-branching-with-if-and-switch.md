# 030 - Conditional Branching with if and switch: Questions and Answers

Review [030 - Conditional Branching with if and switch](../lessons/030-conditional-branching-with-if-and-switch.md). Every answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. When does the body of an `if` statement run?

A. Whenever the condition evaluates to `true`<br>
B. Only when the condition is a string<br>
C. Only when the condition is `null`<br>
D. Whenever the condition evaluates to `false`

**Answer: A.** PHP converts the condition to a boolean and runs the body when the result is true.

### 2. If two conditions in an `if`/`elseif` chain are true, which branch runs?

A. Both branches<br>
B. Only the first true branch<br>
C. Only the last true branch<br>
D. Neither branch

**Answer: B.** PHP runs the first matching branch, then skips the remaining branches in that chain.

### 3. Which value is falsey in PHP?

A. `'false'`<br>
B. `'0'`<br>
C. `'0.0'`<br>
D. `'no'`

**Answer: B.** The string `'0'` is one of PHP's falsey values. Other non-empty strings such as `'false'` are truthy.

### 4. Which operator checks that a value and its type both match?

A. `=`<br>
B. `==`<br>
C. `===`<br>
D. `<>`

**Answer: C.** Strict identity checks both value and type.

### 5. How does PHP compare a switch expression with its `case` values?

A. With strict identity only<br>
B. With loose comparison<br>
C. By comparing variable names<br>
D. By sorting the values

**Answer: B.** `switch` uses loose comparison, so type conversions may affect which case matches.

### 6. What does `break` do at the end of a normal `case` body?

A. Restarts the switch<br>
B. Exits the switch<br>
C. Skips only the next statement<br>
D. Repeats the matching case

**Answer: B.** `break` stops execution of the switch body at that point.

### 7. What happens when a matching case has no `break` and no other exit?

A. PHP raises a syntax error<br>
B. Execution continues into the following case body<br>
C. The switch expression is evaluated again<br>
D. The default case runs immediately

**Answer: B.** This is called fall-through; it can be intentional but is often an accidental bug.

### 8. When does the `default` branch run?

A. Before the first case<br>
B. When every case matches<br>
C. When no case matches<br>
D. After every case

**Answer: C.** `default` provides the fallback for an unmatched switch value.

### 9. What happens if a switch has no matching case and no `default`?

A. PHP executes the first case<br>
B. No switch body statements run<br>
C. PHP automatically throws an exception<br>
D. PHP repeats the switch

**Answer: B.** With no match and no default, the switch body does nothing.

### 10. How can two case labels share one body?

A. Put two case labels together before the body<br>
B. Write two `default` labels<br>
C. Use `continue` between their labels<br>
D. Nest one switch inside the other

**Answer: A.** Empty cases fall through to the next label, so both labels reach the same statements.

### 11. How many times is the switch expression evaluated?

A. Once for every case<br>
B. Once when entering the switch<br>
C. Twice<br>
D. Only if there is a default

**Answer: B.** PHP evaluates the switch expression once and compares that result with the case values.

### 12. In a `switch` inside a loop, what does `continue 2` target?

A. The next statement in the switch<br>
B. The next case label<br>
C. The next iteration of the outer loop<br>
D. The next function call

**Answer: C.** The level `2` skips the switch and continues the enclosing loop.

### 13. Which structure is usually clearest for checking score ranges?

A. A `switch` on the score with many case expressions<br>
B. An `if`/`elseif` chain<br>
C. A nullsafe operator<br>
D. An `else` by itself

**Answer: B.** Each branch can express a range comparison directly, such as `$score >= 80`.

### 14. Why should nested `if` statements use braces?

A. Braces make conditions truthy<br>
B. Braces clarify which `if` an `else` belongs to<br>
C. Braces change loose comparisons to strict comparisons<br>
D. Braces are required only for `switch`

**Answer: B.** Indentation does not determine how PHP pairs an `else`; braces make the structure explicit.

### 15. Which feature is introduced in the next chapter and compares values strictly while returning a value?

A. `goto`<br>
B. `match`<br>
C. `default`<br>
D. `elseif`

**Answer: B.** The next chapter covers the `match` expression, which uses strict identity and returns a value.

## Short-answer questions

### 16. Write the basic syntax of an `if` statement with braces.

**Answer:**

~~~php
<?php
if ($condition) {
    // Statements for the true case.
}
~~~

**Explanation:** The statements inside the braces run only when the condition evaluates to true.

### 17. What is the role of `elseif` and `else`?

**Answer:** `elseif` checks another condition when earlier conditions were false; `else` runs when none of the preceding conditions matched.

**Explanation:** Only the first true branch in an `if` chain runs.

### 18. Name five common falsey values in PHP.

**Answer:** `false`, `0`, `'0'`, `''`, `[]`, and `null` are examples.

**Explanation:** These values can all make an `if` condition fail. When a value is meaningful, compare it explicitly.

### 19. Is the non-empty string `'false'` truthy or falsey in PHP?

**Answer:** Truthy.

**Explanation:** PHP treats non-empty strings as true except for the special string `'0'`.

### 20. What is the difference between `==` and `===`?

**Answer:** `==` compares values after loose type conversion; `===` requires both the value and type to match.

**Explanation:** Use strict identity when implicit conversion is not part of the intended logic.

### 21. Does `switch` use strict comparison for case matching?

**Answer:** No. It uses loose comparison.

**Explanation:** A value such as the string `'2'` can match an integer case `2`.

### 22. What does `break` do in a switch case?

**Answer:** It exits the switch.

**Explanation:** Without an exit statement, execution can proceed into the statements under later case labels.

### 23. What is fall-through in a switch?

**Answer:** Execution continues from a matching case into the following case body because no `break` or other exit was reached.

**Explanation:** Fall-through is sometimes intentional but should be made clear; otherwise add an exit at the end of the case.

### 24. How do you intentionally share one switch body between multiple values?

**Answer:** Put consecutive case labels above the shared body, with no statements between those labels.

**Explanation:** For example, `case 'Saturday':` followed by `case 'Sunday':` can share a weekend body.

### 25. Is `default` required in a switch?

**Answer:** No.

**Explanation:** If no case matches and there is no default, PHP runs none of the switch body statements.

### 26. How many times does PHP evaluate the expression in `switch ($value)`?

**Answer:** Once when entering the switch.

**Explanation:** PHP stores that result for matching against the case values.

### 27. How do you skip to the next outer-loop iteration from a switch inside a loop?

**Answer:** Use `continue 2`.

**Explanation:** `continue 2` targets the loop outside the switch. A plain `continue` in a switch behaves like `break` and may warn because it is often unintentional.

### 28. When is an `if`/`elseif` chain better than a switch?

**Answer:** When conditions check ranges, combine several requirements, or need strict comparisons.

**Explanation:** An `if` chain can express each distinct condition directly without relying on loose switch matching.

### 29. What is the main distinction between a switch and the upcoming `match` expression?

**Answer:** `switch` is a statement with loose case matching and possible fall-through; `match` is an expression that uses strict identity and does not fall through.

**Explanation:** Chapter 031 covers `match` in detail.

## True or false

### 30. In an `if`/`elseif` chain, PHP runs every branch whose condition is true.

**Answer: False.** PHP runs only the first matching branch and skips the rest of that chain.

### 31. The string `'0'` is falsey, while the non-empty string `'false'` is truthy.

**Answer: True.** PHP treats `'0'` specially; other non-empty strings are truthy.

### 32. Strict comparison `===` checks both value and type.

**Answer: True.** It does not treat a numeric string as identical to the corresponding integer.

### 33. PHP `switch` case matching is strict by default.

**Answer: False.** Switch uses loose comparison.

### 34. A matching case automatically stops at the next case label.

**Answer: False.** Execution continues through following case bodies until `break`, another exit, or the end of the switch.

### 35. A `default` branch runs only when no case matched.

**Answer: True.** It is the switch's unmatched-value fallback.

### 36. Two empty case labels can intentionally share the statements below them.

**Answer: True.** The first label has no body statements, so execution continues to the shared body under the next label.

### 37. Curly braces help avoid ambiguity about which nested `if` an `else` belongs to.

**Answer: True.** Braces explicitly mark the branch boundaries.

## Predict the output

### 38. What does this print?

~~~php
<?php
$value = '0';

if ($value) {
    echo 'truthy';
} else {
    echo 'falsey';
}
~~~

**Answer:**

~~~text
falsey
~~~

**Explanation:** The string `'0'` is falsey in PHP.

### 39. What does this print?

~~~php
<?php
$value = 'false';

echo $value ? 'truthy' : 'falsey';
~~~

**Answer:**

~~~text
truthy
~~~

**Explanation:** `'false'` is a non-empty string and is therefore truthy.

### 40. What does this print?

~~~php
<?php
$score = 82;

if ($score >= 90) {
    echo 'A';
} elseif ($score >= 80) {
    echo 'B';
} else {
    echo 'C';
}
~~~

**Answer:**

~~~text
B
~~~

**Explanation:** The first condition is false, and the second is true, so the `B` branch runs.

### 41. What does this print?

~~~php
<?php
$input = '2';

switch ($input) {
    case 2:
        echo 'integer case';
        break;
    default:
        echo 'default';
}
~~~

**Answer:**

~~~text
integer case
~~~

**Explanation:** Switch compares loosely, so the numeric string `'2'` matches the integer case `2`.

### 42. What does this print?

~~~php
<?php
$kind = 'basic';

switch ($kind) {
    case 'basic':
        echo 'Basic ';
    case 'pro':
        echo 'Pro';
        break;
}
~~~

**Answer:**

~~~text
Basic Pro
~~~

**Explanation:** There is no `break` after the matching `basic` case, so execution falls through to the `pro` body.

## Write the code

### 43. Write an `if`/`elseif`/`else` chain that sets `$label` to `Adult` for ages 18 and above, `Teen` for ages 13 through 17, and `Child` otherwise.

**Answer:**

~~~php
<?php
if ($age >= 18) {
    $label = 'Adult';
} elseif ($age >= 13) {
    $label = 'Teen';
} else {
    $label = 'Child';
}
~~~

**Explanation:** The conditions are ordered from the higher range downward. An age of 18 or more matches before PHP checks the teen condition.

### 44. Write a switch that maps `html` to `text/html`, `json` to `application/json`, and every other format to `application/octet-stream`.

**Answer:**

~~~php
<?php
switch ($format) {
    case 'html':
        $contentType = 'text/html';
        break;
    case 'json':
        $contentType = 'application/json';
        break;
    default:
        $contentType = 'application/octet-stream';
}
~~~

**Explanation:** Each matching case exits with `break`; `default` handles any other format.

### 45. Write a strict `if`/`elseif` chain that distinguishes the integer `0` from the string `'0'`.

**Answer:**

~~~php
<?php
if ($value === 0) {
    $kind = 'integer zero';
} elseif ($value === '0') {
    $kind = 'string zero';
} else {
    $kind = 'other';
}
~~~

**Explanation:** The strict identity operator distinguishes values that loose comparison may treat as equivalent.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [029 - Ternary, Null Coalescing, and Nullsafe Operators](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) | [Read this lesson](../lessons/030-conditional-branching-with-if-and-switch.md) | [Open example](../examples/030-conditional-branching-with-if-and-switch.md) | [031 - The match expression](../lessons/031-the-match-expression.md) |
