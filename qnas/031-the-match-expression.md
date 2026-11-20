# 031 - The match Expression: Questions and Answers

Review [031 - The match Expression](../lessons/031-the-match-expression.md). Each answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. Which PHP version introduced the `match` expression?

A. PHP 7.0<br>
B. PHP 7.4<br>
C. PHP 8.0<br>
D. PHP 8.3

**Answer: C.** `match` is available starting with PHP 8.0.

### 2. What does a `match` expression produce?

A. Always a boolean<br>
B. The result expression for the matching arm<br>
C. A new array of all matching arms<br>
D. The name of the matched variable

**Answer: B.** A match expression returns the result associated with the first matching arm.

### 3. How does `match` compare its subject with an arm condition?

A. With `==` only<br>
B. With `===`<br>
C. With `!=`<br>
D. It converts every value to a string first

**Answer: B.** Match uses strict identity, so both type and value must match.

### 4. What does `match (2) { '2' => 'string', default => 'other' }` return?

A. `'string'`<br>
B. `'other'`<br>
C. Integer `2`<br>
D. It always throws

**Answer: B.** The integer `2` is not identical to the string `'2'`, so the `default` arm matches.

### 5. What is the purpose of `default` in a match expression?

A. It runs before the other arms.<br>
B. It provides a result when no earlier arm matches.<br>
C. It repeats the previous arm.<br>
D. It converts the subject to a boolean.

**Answer: B.** `default` handles subjects not matched by another arm.

### 6. What happens if no arm matches and there is no `default`?

A. The expression returns `null`.<br>
B. The first arm runs.<br>
C. PHP throws `UnhandledMatchError`.<br>
D. PHP silently skips the expression.

**Answer: C.** A match expression must be exhaustive at runtime; an unmatched value throws this error.

### 7. What does a comma between match conditions mean?

A. Evaluate both and combine their results.<br>
B. Either condition may match the same result expression.<br>
C. Run the first result and then the second.<br>
D. It is valid only before `default`.

**Answer: B.** Comma-separated conditions are alternatives that share one arm result.

### 8. What happens after a match arm is selected?

A. PHP continues into the next arm.<br>
B. The matching result is produced and the match ends.<br>
C. PHP requires a `break` statement.<br>
D. PHP evaluates every result expression.

**Answer: B.** Match arms do not fall through, and no `break` is needed.

### 9. What kind of code belongs after `=>` in a match arm?

A. An expression that produces a result<br>
B. Any number of statement blocks<br>
C. A label followed by a colon<br>
D. Only a boolean literal

**Answer: A.** Each arm provides an expression. Move multi-step work into a function and call it from the arm.

### 10. Which subject is commonly used to make match arms test boolean conditions?

A. `match (null)`<br>
B. `match (true)`<br>
C. `match ([])`<br>
D. `match (default)`

**Answer: B.** With `true` as the subject, boolean conditions can be compared to it, allowing ordered condition matching.

### 11. If two `match (true)` arm conditions evaluate to true, which one wins?

A. Both arms run.<br>
B. The last true arm wins.<br>
C. The first matching arm wins.<br>
D. PHP throws an error.

**Answer: C.** Match checks arms in order and stops at the first match.

### 12. Which statement is true about arm result expressions?

A. Every result expression is evaluated before matching.<br>
B. Only the selected arm's result expression is evaluated.<br>
C. A result expression must be a string.<br>
D. The result expression is always discarded.

**Answer: B.** PHP evaluates the result expression for the selected arm only.

### 13. Which is a valid use of a match expression in a function?

A. Return it as the function result<br>
B. Use it only after an `echo`<br>
C. Use it only in a `switch`<br>
D. It cannot be used in functions

**Answer: A.** Since match is an expression, it can be returned directly.

### 14. What is a key difference between `switch` and `match`?

A. `switch` returns values and `match` does not.<br>
B. `switch` compares loosely and can fall through; `match` compares strictly and does not fall through.<br>
C. `switch` requires PHP 8.0 and `match` works in PHP 5.<br>
D. They have identical comparison and flow behavior.

**Answer: B.** Match uses `===`, returns a value, and ends after a selected arm.

### 15. Why add a `default` arm when matching user-provided values?

A. To avoid handling unknown input<br>
B. To give unmatched values a defined result or an explicit error<br>
C. To make earlier arms run faster<br>
D. To enable fall-through

**Answer: B.** A default makes the unmatched case explicit. It can return a fallback or throw a meaningful exception.

## Short-answer questions

### 16. Write the basic shape of a `match` expression.

**Answer:**

~~~php
<?php
$result = match ($subject) {
    $firstCondition => $firstResult,
    $secondCondition => $secondResult,
    default => $fallbackResult,
};
~~~

**Explanation:** The subject is compared with the conditions, and the selected arm's expression becomes the value of the whole match.

### 17. What is the main difference between match comparison and switch comparison?

**Answer:** Match compares strictly with `===`; switch compares loosely.

**Explanation:** A match between integer `2` and string `'2'` does not succeed, while switch can treat them as equivalent.

### 18. How can several values share one match result?

**Answer:** Separate those values with commas before the arrow.

**Explanation:** For example, `'Saturday', 'Sunday' => 'Weekend'` maps either value to the same result.

### 19. What is the purpose of the `default` arm?

**Answer:** It returns a result when no other arm matched.

**Explanation:** Including it makes a match expression exhaustive for unexpected values.

### 20. What happens at runtime when no match arm and no default handle the subject?

**Answer:** PHP throws an `UnhandledMatchError`.

**Explanation:** The expression cannot produce an arm result for an unhandled subject.

### 21. Does a match expression need a `break` after an arm?

**Answer:** No.

**Explanation:** Match does not fall through. Once an arm is selected, its result is produced and the match ends.

### 22. What does `match (true)` let you do?

**Answer:** Use boolean conditions as arms and return the result for the first condition that evaluates to true.

**Explanation:** It can express ordered ranges, although an `if`/`elseif` chain may be clearer for complex logic.

### 23. Why does the order of range conditions matter in `match (true)`?

**Answer:** Match selects the first true condition, so broader or overlapping conditions can prevent later ones from being reached.

**Explanation:** Put the most specific range first, such as checking senior ages before a general adult condition.

### 24. Can a match arm contain a multi-statement block?

**Answer:** No. An arm provides an expression.

**Explanation:** Put multi-step work in a function and call the function as the arm's result expression.

### 25. Is the subject expression evaluated again for every arm?

**Answer:** No. It is evaluated once.

**Explanation:** PHP uses that subject value while checking the arms in order.

### 26. Are all match result expressions evaluated before PHP chooses an arm?

**Answer:** No. Only the result expression belonging to the selected arm is evaluated.

**Explanation:** Results in unmatched arms are not evaluated.

### 27. Can a match expression be used as a standalone statement?

**Answer:** Yes, when its result is not needed; terminate it with a semicolon.

**Explanation:** The selected arm expression still runs, so a function call can perform the action.

### 28. Which PHP version is required for `match`?

**Answer:** PHP 8.0 or later.

**Explanation:** Older runtimes cannot parse match syntax.

### 29. When is an `if` chain preferable to `match (true)`?

**Answer:** When conditions are complex, contain several steps, or are easier to understand as ordinary branches.

**Explanation:** `match (true)` is an option for concise ordered conditions, not a requirement to replace every `if` chain.

## True or false

### 30. A match expression is available in PHP 8.0 and later.

**Answer: True.** PHP introduced the match expression in PHP 8.0.

### 31. Match uses loose comparison like `switch`.

**Answer: False.** Match uses strict identity (`===`).

### 32. A match expression returns the result of its selected arm.

**Answer: True.** That is why it can be assigned, returned, or used in another expression.

### 33. Match arms fall through unless they contain `break`.

**Answer: False.** Match never falls through and does not use `break`.

### 34. Comma-separated conditions can share a single match result.

**Answer: True.** Any one of the listed alternatives can select that arm.

### 35. If no arm matches and there is no `default`, match returns `null`.

**Answer: False.** PHP throws `UnhandledMatchError`.

### 36. Only the selected arm's result expression is evaluated.

**Answer: True.** Unselected result expressions are not evaluated.

### 37. In `match (true)`, a later true arm overrides an earlier true arm.

**Answer: False.** PHP uses the first matching arm.

## Predict the output

### 38. What does this print?

~~~php
<?php
$value = match (2) {
    2 => 'integer',
    '2' => 'string',
    default => 'other',
};

echo $value;
~~~

**Answer:**

~~~text
integer
~~~

**Explanation:** The subject is the integer `2`, so it strictly matches the integer arm.

### 39. What does this print?

~~~php
<?php
$value = match ('2') {
    2 => 'integer',
    '2' => 'string',
    default => 'other',
};

echo $value;
~~~

**Answer:**

~~~text
string
~~~

**Explanation:** The subject is a string, so it matches the string arm, not the integer arm.

### 40. What does this print?

~~~php
<?php
$size = match (3) {
    1, 2, 3 => 'small',
    4, 5 => 'large',
    default => 'unknown',
};

echo $size;
~~~

**Answer:**

~~~text
small
~~~

**Explanation:** The integer `3` is one of the comma-separated values in the first arm.

### 41. What does this print?

~~~php
<?php
try {
    $label = match ('missing') {
        'ready' => 'Ready',
    };
} catch (UnhandledMatchError $error) {
    echo 'unhandled';
}
~~~

**Answer:**

~~~text
unhandled
~~~

**Explanation:** No arm matches and there is no default, so PHP throws `UnhandledMatchError` and the catch block handles it.

### 42. What does this print?

~~~php
<?php
$age = 17;

$group = match (true) {
    $age >= 18 => 'Adult',
    $age >= 13 => 'Teenager',
    default => 'Child',
};

echo $group;
~~~

**Answer:**

~~~text
Teenager
~~~

**Explanation:** The first condition is false and the second is true, so the match returns `Teenager`.

## Write the code

### 43. Use `match` to return `Success` for HTTP statuses 200, 201, and 204; return `Not found` for 404; and return `Other` for any other value.

**Answer:**

~~~php
<?php
$label = match ($status) {
    200, 201, 204 => 'Success',
    404 => 'Not found',
    default => 'Other',
};
~~~

**Explanation:** Related success codes share one arm. The default handles statuses not listed.

### 44. Use `match (true)` to label a temperature `Hot` at 30 or above, `Warm` at 20 or above, and `Cool` otherwise.

**Answer:**

~~~php
<?php
$label = match (true) {
    $temperature >= 30 => 'Hot',
    $temperature >= 20 => 'Warm',
    default => 'Cool',
};
~~~

**Explanation:** Check the higher threshold first so a hot temperature does not match the broader warm condition first.

### 45. Write a function that maps `'on'` to `Enabled`, `'off'` to `Disabled`, and every other string to `Unknown` using `match`.

**Answer:**

~~~php
<?php
function settingLabel(string $setting): string
{
    return match ($setting) {
        'on' => 'Enabled',
        'off' => 'Disabled',
        default => 'Unknown',
    };
}
~~~

**Explanation:** Match is an expression, so it can be returned directly from a function. Its string comparisons are strict.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [030 - Conditional Branching with if and switch](../lessons/030-conditional-branching-with-if-and-switch.md) | [Read this lesson](../lessons/031-the-match-expression.md) | [Open example](../examples/031-the-match-expression.md) | [032 - Loops with while, do-while, and for](../lessons/032-loops-with-while-do-while-and-for.md) |
