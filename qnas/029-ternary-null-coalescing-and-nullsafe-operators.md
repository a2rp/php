# 029 - Ternary, Null Coalescing, and Nullsafe Operators: Questions and Answers

Review [029 - Ternary, Null Coalescing, and Nullsafe Operators](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md). Each answer appears immediately after its question, with an explanation where it helps.

## Multiple-choice questions

### 1. What does the full ternary operator choose between?

A. A class and an object<br>
B. Two values based on a condition<br>
C. Two array keys<br>
D. A variable and a constant

**Answer: B.** The condition selects the true expression or the false expression.

### 2. What does this expression produce when `$name` is an empty string?

~~~php
$name ?: 'Guest'
~~~

A. An empty string<br>
B. `null`<br>
C. `'Guest'`<br>
D. A syntax error

**Answer: C.** The shorthand ternary uses the fallback when its left expression is falsey. An empty string is falsey in PHP.

### 3. Which operator preserves the integer `0` and uses the fallback only for a missing or `null` value?

A. `?:`<br>
B. `??`<br>
C. `!`<br>
D. `&&`

**Answer: B.** Null coalescing preserves `0`; shorthand ternary treats `0` as falsey.

### 4. What does `??` return if its left value is `false`?

A. The fallback value<br>
B. `null`<br>
C. `false`<br>
D. A warning

**Answer: C.** `false` is set and not `null`, so it is returned unchanged.

### 5. Which statement about an absent array key on the left of `??` is correct?

A. It always throws an exception.<br>
B. It produces a warning before the fallback is used.<br>
C. The fallback can be used without the usual undefined-key warning.<br>
D. It silently creates the array key.

**Answer: C.** Null coalescing checks the left side with behavior similar to `isset()` and uses the fallback when the key is missing.

### 6. When does `??=` assign its right-hand value?

A. Whenever the left value is falsey<br>
B. Only when the left value is `null` or missing<br>
C. Whenever the left value is a string<br>
D. On every evaluation

**Answer: B.** Null coalescing assignment uses missing-or-null behavior. It preserves values such as `false`, `0`, and `''`.

### 7. Which PHP version introduced `??=`?

A. PHP 5.6<br>
B. PHP 7.0<br>
C. PHP 7.4<br>
D. PHP 8.2

**Answer: C.** Null coalescing assignment was added in PHP 7.4.

### 8. Which PHP version introduced the nullsafe operator `?->`?

A. PHP 7.0<br>
B. PHP 7.4<br>
C. PHP 8.0<br>
D. PHP 8.3

**Answer: C.** Nullsafe property and method access arrived in PHP 8.0.

### 9. What does `$user?->address?->city` yield when `$user` is `null`?

A. It throws an exception because `address` is missing.<br>
B. `null`<br>
C. `false`<br>
D. An empty array

**Answer: B.** The nullsafe chain short-circuits at the first null value and produces `null`.

### 10. Which expression makes the grouping explicit when combining string concatenation and a null fallback?

A. `'Page: ' . $page ?? '1'`<br>
B. `'Page: ' . ($page ?? '1')`<br>
C. `('Page: ' . $page) ?? '1'`<br>
D. `'Page: ' && ($page ?? '1')`

**Answer: B.** Parentheses ensure the fallback is chosen for `$page` before the result is concatenated with the label.

### 11. What is the best choice if a compact expression has several branches with multiple statements in each branch?

A. A deeply nested ternary<br>
B. A regular `if`/`elseif`/`else` structure<br>
C. `??=`<br>
D. A nullsafe chain

**Answer: B.** Regular conditionals are clearer when branches perform several actions.

### 12. What does `$value ?: 'default'` test about `$value`?

A. Whether it is an integer<br>
B. Whether it is truthy<br>
C. Whether it is an object<br>
D. Whether it is defined as a constant

**Answer: B.** Shorthand ternary evaluates the truthiness of its left expression.

### 13. What does `??` do when the left value is the string `'0'`?

A. Uses the fallback because `'0'` is falsey<br>
B. Returns `'0'` because it is not `null`<br>
C. Converts it to integer `0`<br>
D. Throws a type error

**Answer: B.** Null coalescing does not test general truthiness; it preserves any non-null value.

### 14. Which is valid use of a nullsafe chain?

A. `$user?->name = 'Asha';`<br>
B. `unset($user?->name);`<br>
C. `$name = $user?->getName();`<br>
D. `&$name = $user?->name;`

**Answer: C.** Nullsafe access is for reading properties and calling methods. It cannot be used as a write target, reference, or `unset()` target.

### 15. In PHP 8 and later, what must you do with a nested full ternary?

A. Convert it to a constant<br>
B. Parenthesize the nested ternary or use a clearer conditional structure<br>
C. Add `??=` between the branches<br>
D. Put it inside an array

**Answer: B.** The full ternary is non-associative in PHP 8 and later, so nested ternaries need explicit grouping.

## Short-answer questions

### 16. Write the general form of a full ternary expression.

**Answer:** `condition ? valueWhenTrue : valueWhenFalse`.

**Explanation:** PHP evaluates the condition, then returns the expression after `?` when true or after `:` when false.

### 17. What is the difference between `?:` and `??`?

**Answer:** `?:` uses truthiness; `??` uses missing-or-null behavior.

**Explanation:** For example, `0 ?: 5` returns `5`, while `0 ?? 5` returns `0`.

### 18. Name four falsey values that can trigger the fallback in `?:`.

**Answer:** Examples include `false`, `0`, `'0'`, `''`, `[]`, and `null`.

**Explanation:** PHP's falsey set includes all of these values, so do not use `?:` if one of them is a meaningful value to preserve.

### 19. Which value types are preserved by `??` even though they may be falsey?

**Answer:** Values such as `false`, `0`, `'0'`, `''`, and `[]` are preserved.

**Explanation:** The fallback is chosen only when the left value is missing or `null`.

### 20. What common behavior does `??` share with `isset()`?

**Answer:** A value is considered available only when it exists and is not `null`.

**Explanation:** This lets `??` handle an absent variable or array key without the usual undefined-value diagnostic.

### 21. What does `??=` do to an existing array element whose value is `false`?

**Answer:** It leaves the element as `false`.

**Explanation:** The element exists and is not `null`; `??=` does not replace it based on truthiness.

### 22. What happens to the remaining part of a `?->` chain after the current object value is `null`?

**Answer:** PHP skips the remaining chain and the expression evaluates to `null`.

**Explanation:** This avoids trying to access a property or call a method on a null value.

### 23. Does `?->` protect against calling a misspelled method on a non-null object?

**Answer:** No.

**Explanation:** It only short-circuits when the value immediately to its left is `null`. Invalid members on an existing object still cause an error.

### 24. Can you assign a new value through `$user?->address?->city`?

**Answer:** No.

**Explanation:** Nullsafe access is read-only. Check that each needed object exists, then update it with a regular assignment or method call.

### 25. Which PHP version first supports `??`?

**Answer:** PHP 7.0.

**Explanation:** The null coalescing operator was introduced in PHP 7.0.

### 26. Which PHP version first supports `??=`?

**Answer:** PHP 7.4.

**Explanation:** Code using null coalescing assignment cannot be parsed by earlier PHP versions.

### 27. Which PHP version first supports `?->`?

**Answer:** PHP 8.0.

**Explanation:** Applications that support PHP 7 must avoid the nullsafe operator or use explicit null checks.

### 28. Why should you parenthesize a nested full ternary?

**Answer:** It makes the intended grouping explicit and is required for nested full ternaries in PHP 8 and later.

**Explanation:** Before PHP 8, unparenthesized nested ternaries had left-to-right behavior and that behavior was deprecated in PHP 7.4.

### 29. When is a regular `if` statement preferable to a ternary expression?

**Answer:** When the decision is complex, branches perform multiple statements, or the compact expression is hard to read.

**Explanation:** Ternaries are expressions for small value choices, not a replacement for every conditional block.

## True or false

### 30. The shorthand ternary operator uses its fallback when the left value is the integer `0`.

**Answer: True.** `0` is falsey in PHP.

### 31. The null coalescing operator uses its fallback when the left value is `false`.

**Answer: False.** `false` is set and not `null`, so `??` returns it.

### 32. The null coalescing operator can provide a fallback for a missing array key without the usual undefined-key warning.

**Answer: True.** The operator checks whether the left value exists and is not `null` before deciding to use it.

### 33. `??=` overwrites an existing value whenever that value is falsey.

**Answer: False.** It assigns only for a missing or `null` left side. A value such as `false`, `0`, or `''` is retained.

### 34. A nullsafe chain short-circuits when an intermediate object value is `null`.

**Answer: True.** The rest of that chain is skipped and the expression returns `null`.

### 35. The nullsafe operator hides all errors produced by a method call.

**Answer: False.** It only checks for `null` at a link in the chain; it does not hide undefined methods or errors raised by a method that runs.

### 36. In PHP 8 and later, nested full ternaries must use explicit parentheses.

**Answer: True.** The full ternary operator is non-associative in these versions.

### 37. Nullsafe property access can be used on the left side of an assignment.

**Answer: False.** Nullsafe access is read-only and cannot be an assignment target.

## Predict the output

### 38. What does this print?

~~~php
<?php
$value = 0;
echo $value ?: 'fallback';
~~~

**Answer:**

~~~text
fallback
~~~

**Explanation:** Zero is falsey, so shorthand ternary selects the fallback.

### 39. What does this print?

~~~php
<?php
$value = 0;
echo $value ?? 'fallback';
~~~

**Answer:**

~~~text
0
~~~

**Explanation:** Zero exists and is not `null`, so null coalescing preserves it.

### 40. What does this print?

~~~php
<?php
$options = ['enabled' => false];
$options['enabled'] ??= true;
var_export($options['enabled']);
~~~

**Answer:**

~~~text
false
~~~

**Explanation:** `??=` does not replace a non-null `false` value.

### 41. What does this print?

~~~php
<?php
$user = null;
$name = $user?->name ?? 'Guest';
echo $name;
~~~

**Answer:**

~~~text
Guest
~~~

**Explanation:** Nullsafe access yields `null`, then `??` chooses `'Guest'`.

### 42. What does this print?

~~~php
<?php
$settings = ['theme' => '', 'locale' => null];
echo $settings['theme'] ?? 'light';
echo "\n";
echo $settings['locale'] ?? 'en';
~~~

**Answer:**

~~~text

en
~~~

**Explanation:** The first line prints an empty string because it is non-null. The second line uses `en` because the locale is `null`.

## Write the code

### 43. Use a full ternary to set `$label` to `Open` when `$isOpen` is true and `Closed` otherwise.

**Answer:**

~~~php
<?php
$label = $isOpen ? 'Open' : 'Closed';
~~~

**Explanation:** The condition selects one string value. Both outcomes have the same role: a status label.

### 44. Set `$page` to the existing `page` query value, or to `'1'` when it is missing or `null`. Preserve the string `'0'`.

**Answer:**

~~~php
<?php
$page = $query['page'] ?? '1';
~~~

**Explanation:** Null coalescing preserves `'0'`; a shorthand ternary would treat it as falsey and replace it.

### 45. Read the city from an optional `$user` object and use `'Unknown'` when the user, address, or city is `null`.

**Answer:**

~~~php
<?php
$city = $user?->address?->city ?? 'Unknown';
~~~

**Explanation:** Nullsafe access handles missing objects in the chain, and null coalescing supplies the final display value. The `?->` syntax requires PHP 8.0 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [028 - String, Array, Bitwise, and Other Operators](../lessons/028-string-array-bitwise-and-other-operators.md) | [Read this lesson](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) | [Open example](../examples/029-ternary-null-coalescing-and-nullsafe-operators.md) | [030 - Conditional Branching with if and switch](../lessons/030-conditional-branching-with-if-and-switch.md) |
