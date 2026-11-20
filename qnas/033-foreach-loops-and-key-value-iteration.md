# 033 - foreach Loops and Key-Value Iteration: Questions and Answers

Review [033 - foreach Loops and Key-Value Iteration](../lessons/033-foreach-loops-and-key-value-iteration.md). Each answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. What does `foreach ($items as $item)` assign on each iteration?

A. The current value<br>
B. The current array length<br>
C. The next integer index only<br>
D. The array variable name

**Answer: A.** The value-only form assigns the current element's value to `$item`.

### 2. Which syntax assigns both an element's key and value?

A. `foreach ($items as $value => $key)`<br>
B. `foreach ($items as $key => $value)`<br>
C. `foreach ($items key $value)`<br>
D. `foreach ($items as [$key, $value])`

**Answer: B.** The key is on the left of `=>`, and the value is on the right.

### 3. What kinds of values can `foreach` traverse?

A. Only zero-based arrays<br>
B. Arrays and iterable objects<br>
C. Every scalar value<br>
D. Only strings

**Answer: B.** Arrays and iterable objects such as objects implementing `Traversable` can be traversed with `foreach`.

### 4. What happens to array keys during a normal `foreach` traversal?

A. PHP always renumbers them from zero.<br>
B. Existing keys are assigned as they are.<br>
C. String keys are discarded.<br>
D. Keys are sorted numerically first.

**Answer: B.** The loop visits existing array elements in iteration order and does not renumber their keys.

### 5. Does changing a normal loop variable replace a scalar value in the original array?

A. Yes, always<br>
B. No, not by itself<br>
C. Only for strings<br>
D. Only when the key is numeric

**Answer: B.** The regular by-value variable receives the current value; reassigning it does not replace the array element.

### 6. What does `&` before a `foreach` value variable do?

A. Makes the value variable a reference to each original element<br>
B. Sorts the array<br>
C. Converts values to strings<br>
D. Copies the array's keys

**Answer: A.** Assignments through that variable update the corresponding original array elements.

### 7. What should you do with the value variable after a by-reference `foreach`?

A. Call `unset()` on it<br>
B. Increment it<br>
C. Assign it to the array key<br>
D. Pass it to `break`

**Answer: A.** The variable remains a reference to the last element after the loop; `unset()` breaks that reference.

### 8. What does the `Traversable` interface indicate in this context?

A. The object can be iterated with `foreach`.<br>
B. The object can only be used as a string.<br>
C. The object is an array.<br>
D. The object has integer-only properties.

**Answer: A.** Objects implementing the iteration interfaces can provide values to a `foreach` loop.

### 9. In what order does `foreach` visit array elements?

A. Sorted by value<br>
B. In the array's iteration order<br>
C. Always reverse insertion order<br>
D. Only by ascending numeric key

**Answer: B.** It follows the array's element order and preserves the keys associated with each value.

### 10. For `$items = [2 => 'a', 5 => 'b']`, what keys does `foreach` produce?

A. `0`, `1`<br>
B. `1`, `2`<br>
C. `2`, `5`<br>
D. `'a'`, `'b'`

**Answer: C.** The existing keys remain `2` and `5`; `foreach` does not turn them into positions.

### 11. What happens if `foreach` is given a scalar such as an integer?

A. It automatically wraps it in an array.<br>
B. It raises an error because the value is not iterable.<br>
C. It runs zero times without any issue.<br>
D. It converts the integer to a key.

**Answer: B.** `foreach` expects an array or iterable object; validate uncertain values before traversing them.

### 12. From which PHP version is `is_iterable()` available?

A. PHP 5.6<br>
B. PHP 7.0<br>
C. PHP 7.1<br>
D. PHP 8.2

**Answer: C.** The `iterable` type and `is_iterable()` are available from PHP 7.1.

### 13. What is a clear way to transform every array value while preserving its key?

A. Reassign the ordinary `foreach` value variable only<br>
B. Build a result array and assign each transformed value to its key<br>
C. Change the array's internal pointer<br>
D. Use `break` after the first element

**Answer: B.** A separate result array makes the transformation explicit and avoids reference behavior.

### 14. Which PHP version first supports destructuring a row in a `foreach` declaration as `[$name, $age]`?

A. PHP 5.4<br>
B. PHP 7.0<br>
C. PHP 7.1<br>
D. PHP 8.0

**Answer: C.** Array destructuring with the short `[]` syntax in this context is supported from PHP 7.1.

### 15. Why use different variable names at different levels of nested `foreach` loops?

A. PHP requires every variable to start with a different letter.<br>
B. To keep each level's key and value clear and avoid overwriting outer variables.<br>
C. To make nested arrays iterable.<br>
D. To sort the inner array.

**Answer: B.** Distinct names preserve the meaning of the outer and inner values while reading the code.

## Short-answer questions

### 16. Write the basic value-only form of `foreach`.

**Answer:**

~~~php
<?php
foreach ($items as $item) {
    // Process the current value.
}
~~~

**Explanation:** `$item` receives each value in turn.

### 17. Write the key-and-value form of `foreach`.

**Answer:**

~~~php
<?php
foreach ($items as $key => $value) {
    // Process the current key and value.
}
~~~

**Explanation:** PHP assigns the current key and value to the corresponding variables for each element.

### 18. Does `foreach` renumber an array with keys `2` and `5`?

**Answer:** No.

**Explanation:** It visits the keys that already exist; keys are not loop positions unless the array is indexed that way.

### 19. Does modifying a regular scalar `$value` variable in a by-value loop update the original array element?

**Answer:** No.

**Explanation:** Reassigning the loop variable alone does not replace the value stored in the source array.

### 20. How can you keep transformed results without changing the source array?

**Answer:** Initialize a new array and write each transformed value to it inside the loop.

**Explanation:** Use the current key when the result should preserve the source key.

### 21. How do you update the original array elements from a `foreach` loop?

**Answer:** Use a reference value variable, such as `foreach ($items as &$item)`, and assign through it.

**Explanation:** The reference connects `$item` to the current array element.

### 22. Why should you call `unset($item)` after a by-reference loop?

**Answer:** Because `$item` remains a reference to the last array element after the loop.

**Explanation:** Reusing the variable without breaking the reference can unexpectedly overwrite that last element.

### 23. Does `unset($item)` after a reference loop remove the last array element?

**Answer:** No.

**Explanation:** It removes the variable's reference; the array element remains in the array.

### 24. What does `foreach ($profile as $property => $value)` do for a plain object?

**Answer:** It iterates over the object's properties visible from the current scope, assigning each property name and value.

**Explanation:** Classes can customize iteration with interfaces such as `Iterator` or `IteratorAggregate`.

### 25. What does the `iterable` type describe in PHP?

**Answer:** Arrays and objects implementing `Traversable`.

**Explanation:** It can be used to express values that are valid sources for iteration.

### 26. Why might `is_iterable()` be useful before a loop?

**Answer:** It checks whether an uncertain value can be traversed with `foreach`.

**Explanation:** This can prevent an error when data may be a scalar or otherwise non-iterable.

### 27. How can you process a nested array with `foreach`?

**Answer:** Place an inner `foreach` in the outer loop and give each level its own key and value variables.

**Explanation:** The outer loop reaches each group; the inner loop visits that group's entries.

### 28. What must be true for `foreach ($rows as [$name, $age])` to work as intended?

**Answer:** Each row needs the expected list shape and order, and the syntax requires PHP 7.1 or later.

**Explanation:** If fields are irregular or associative, key-value iteration can make the mapping clearer.

### 29. Why should you avoid casually adding or removing elements from the array currently being traversed?

**Answer:** Structural changes can make the traversal harder to predict and maintain.

**Explanation:** Build a separate result array or use an explicit strategy when filtering or changing collection structure.

## True or false

### 30. `foreach ($items as $value)` assigns the current element's value on each iteration.

**Answer: True.** This is the value-only form of the loop.

### 31. `foreach` automatically changes an associative array's keys to sequential integers.

**Answer: False.** Existing keys are preserved.

### 32. Reassigning the value variable in a normal `foreach` replaces a scalar in the source array.

**Answer: False.** Use a separate result array or a reference when you intentionally need to change the original element.

### 33. A by-reference `foreach` can update the original array elements.

**Answer: True.** The value variable refers to each current element.

### 34. After a by-reference loop, the loop variable no longer refers to anything automatically.

**Answer: False.** It remains a reference to the last element until the reference is broken, typically with `unset()`.

### 35. `unset($value)` after a by-reference loop deletes the last item from the array.

**Answer: False.** It removes the variable's reference, not the array element.

### 36. `foreach` can traverse objects that provide iteration behavior.

**Answer: True.** Objects implementing iterable behavior can be used as loop sources.

### 37. Destructuring a row in a `foreach` declaration with `[]` is available in PHP 7.1 and later.

**Answer: True.** The short array destructuring syntax was introduced in PHP 7.1.

## Predict the output

### 38. What does this print?

~~~php
<?php
$colors = ['red', 'green'];

foreach ($colors as $color) {
    echo $color, "\n";
}
~~~

**Answer:**

~~~text
red
green
~~~

**Explanation:** The value-only form visits both array values in order.

### 39. What does this print?

~~~php
<?php
$items = [2 => 'first', 5 => 'second'];

foreach ($items as $key => $value) {
    echo "{$key}: {$value}\n";
}
~~~

**Answer:**

~~~text
2: first
5: second
~~~

**Explanation:** The original keys are preserved during iteration.

### 40. What does this print?

~~~php
<?php
$numbers = [2, 4];

foreach ($numbers as $number) {
    $number *= 10;
}

echo $numbers[0], ' ', $numbers[1];
~~~

**Answer:**

~~~text
2 4
~~~

**Explanation:** Reassigning the ordinary loop variable does not change the source array values.

### 41. What does this print?

~~~php
<?php
$numbers = [1, 2, 3];

foreach ($numbers as &$number) {
    $number *= 2;
}
unset($number);

echo implode(', ', $numbers);
~~~

**Answer:**

~~~text
2, 4, 6
~~~

**Explanation:** The reference variable updates each original element. `unset()` then breaks the reference to the last element.

### 42. What does this print?

~~~php
<?php
$groups = [
    'A' => ['one', 'two'],
    'B' => ['three'],
];

foreach ($groups as $group => $values) {
    foreach ($values as $value) {
        echo "{$group}: {$value}\n";
    }
}
~~~

**Answer:**

~~~text
A: one
A: two
B: three
~~~

**Explanation:** The inner loop visits each value in the current outer group.

## Write the code

### 43. Print every key and value from an associative `$settings` array.

**Answer:**

~~~php
<?php
foreach ($settings as $key => $value) {
    echo "{$key}: {$value}\n";
}
~~~

**Explanation:** The key-value form makes both the setting name and its current value available.

### 44. Create a new `$doubled` array with the same keys as `$numbers` and twice each value. Do not modify `$numbers`.

**Answer:**

~~~php
<?php
$doubled = [];

foreach ($numbers as $key => $number) {
    $doubled[$key] = $number * 2;
}
~~~

**Explanation:** The new array receives each transformed value at its original key, leaving the source array unchanged.

### 45. Increase every value in `$scores` by five in place, then safely end the reference.

**Answer:**

~~~php
<?php
foreach ($scores as &$score) {
    $score += 5;
}
unset($score);
~~~

**Explanation:** The reference updates the source array. Unsetting the loop variable prevents it from remaining attached to the last element.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [032 - Loops with while, do-while, and for](../lessons/032-loops-with-while-do-while-and-for.md) | [Read this lesson](../lessons/033-foreach-loops-and-key-value-iteration.md) | [Open example](../examples/033-foreach-loops-and-key-value-iteration.md) | [034 - break, continue, goto, and declare](../lessons/034-break-continue-goto-and-declare.md) |
