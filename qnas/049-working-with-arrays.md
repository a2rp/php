# 049 - Working with Arrays: Q&A

Review PHP arrays as ordered maps, their keys and values, safe access, iteration, common transformations, sorting, and combining arrays. Try each question before reading its answer. See the [chapter lesson](../lessons/049-working-with-arrays.md).

## Multiple-choice questions

### 1. What is a PHP array?

A. An ordered map that associates keys with values
B. A list that must always start at key zero
C. A collection that can only contain strings
D. A fixed-size sequence

**Answer:** A. An ordered map that associates keys with values.

**Explanation:** PHP arrays can behave as lists, dictionaries, or nested collections. Their keys may be integers or strings.

### 2. What keys are assigned to **['red', 'green']**?

A. **'red'** and **'green'**
B. **0** and **1**
C. **1** and **2**
D. The array has no keys

**Answer:** B. **0** and **1**.

**Explanation:** When keys are omitted, PHP assigns integer keys to the values, starting at zero for a new list.

### 3. How do you read the value stored under the string key **'email'**?

A. **$user.email**
B. **$user('email')**
C. **$user['email']**
D. **$user->email**

**Answer:** C. **$user['email']**

**Explanation:** Array elements are accessed with square brackets. The arrow operator is used for object properties.

### 4. What happens when the same key is assigned twice in one array?

A. PHP keeps both values under the same key.
B. The later assignment replaces the earlier value.
C. The array becomes a list.
D. PHP always throws an exception.

**Answer:** B. The later assignment replaces the earlier value.

**Explanation:** Each key identifies one value. A later value for the same key overwrites the earlier one.

### 5. What does **$items[] = 'new'** do?

A. Removes the final item
B. Appends a value using an automatically chosen integer key
C. Sorts the array
D. Adds a string key named **'new'**

**Answer:** B. Appends a value using an automatically chosen integer key.

**Explanation:** Empty brackets append to an array. Use an explicit key to add an associative entry.

### 6. What value does this expression use if **$user['role']** is missing or null?

~~~php
$user['role'] ?? 'reader'
~~~

A. **false**
B. **'reader'**
C. An empty array
D. It always raises an undefined-key warning

**Answer:** B. **'reader'**

**Explanation:** The null coalescing operator uses its right-hand value when the left-hand value is missing or **null**.

### 7. What does **isset($settings['theme'])** return if that key exists with a value of **null**?

A. **true**
B. **false**
C. **null**
D. It returns the key

**Answer:** B. **false**

**Explanation:** **isset()** returns true only when the variable or key exists and is not **null**.

### 8. Which check returns true when a key exists even if its value is **null**?

A. **in_array()**
B. **count()**
C. **array_key_exists()**
D. **array_values()**

**Answer:** C. **array_key_exists()**

**Explanation:** It checks for the key itself. Unlike **isset()**, it considers a key with a null value present.

### 9. What happens to the other integer keys after **unset($items[1])**?

A. They are automatically renumbered.
B. They remain as they were; the removed key leaves a gap.
C. The whole array is deleted.
D. They become string keys.

**Answer:** B. They remain as they were; the removed key leaves a gap.

**Explanation:** Use **array_values()** if the remaining values must be reindexed as a zero-based list.

### 10. Which **foreach** form gives you both each key and its value?

A. **foreach ($items as $key, $value)**
B. **foreach ($items as $key => $value)**
C. **foreach ($key => $value in $items)**
D. **foreach ($items[$key] as $value)**

**Answer:** B. **foreach ($items as $key => $value)**

**Explanation:** This syntax binds the current key to **$key** and the current value to **$value**.

### 11. What does **count($profile)** count when **$profile** contains a nested array?

A. Only the top-level entries by default
B. Every nested value recursively by default
C. Only integer keys
D. The number of bytes in every string

**Answer:** A. Only the top-level entries by default.

**Explanation:** A normal **count()** counts entries at that level. You can count a nested array separately or use **COUNT_RECURSIVE** when recursive counting is actually needed.

### 12. What does **array_filter()** do with keys by default?

A. It always reindexes integer keys.
B. It preserves the keys of the values it keeps.
C. It converts all keys to strings.
D. It deletes every key.

**Answer:** B. It preserves the keys of the values it keeps.

**Explanation:** A filtered list can have gaps in its numeric keys. Call **array_values()** when a fresh indexed list is required.

### 13. What does **array_values($array)** return?

A. All keys with their original types
B. The values indexed numerically from zero
C. Only values that are not null
D. A sorted copy

**Answer:** B. The values indexed numerically from zero.

**Explanation:** **array_values()** discards the original keys and creates a new numerically indexed array.

### 14. Which function sorts values in ascending order and assigns new integer keys?

A. **ksort()**
B. **asort()**
C. **sort()**
D. **array_keys()**

**Answer:** C. **sort()**

**Explanation:** **sort()** sorts by value and reindexes the array. For associative arrays where key-value associations must stay together, consider **asort()**.

### 15. How does **array_merge()** handle a repeated string key?

A. The value from the later array replaces the earlier value.
B. The earlier value always wins.
C. Both values are discarded.
D. It creates a duplicate key.

**Answer:** A. The value from the later array replaces the earlier value.

**Explanation:** Numeric keys are appended and renumbered; repeated string keys take their value from the later array.

## Short-answer questions

### 16. What is the difference between an indexed array and an associative array?

**Answer:** An indexed array commonly uses integer keys for list-like values. An associative array uses descriptive string keys to look up values.

**Explanation:** They are both PHP arrays and can technically mix integer and string keys. Keeping a clear, consistent shape makes data easier to use.

### 17. What key types can PHP arrays use?

**Answer:** Integer and string keys.

**Explanation:** Other scalar key types may be converted to an integer or string. For example, **true** becomes **1**, **false** becomes **0**, and **null** becomes an empty string. Prefer explicit integer or string keys.

### 18. Why is **in_array($value, $items, true)** often a safer membership check?

**Answer:** The strict third argument requires the value and its type to match.

**Explanation:** Without strict comparison, values of different types can compare equal. Strict mode distinguishes, for example, integer **12** from string **'12'**.

### 19. Explain **isset()** versus **array_key_exists()** for a null-valued key.

**Answer:** **isset()** returns false when the key is missing or its value is null. **array_key_exists()** returns true if the key exists, even when the value is null.

**Explanation:** Choose based on whether a stored **null** should count as present.

### 20. What is a safe way to read an optional array key with a default?

**Answer:** Use the null coalescing operator, such as **$role = $user['role'] ?? 'reader';**.

**Explanation:** It supplies the default when the key is missing or its value is null.

### 21. What should you remember if you use a reference in a **foreach** loop?

**Answer:** Use a reference only when intentionally changing the original array elements, then call **unset()** on the loop reference variable afterward.

**Explanation:** Without clearing it, the loop variable can remain bound to the last array element and a later assignment can change that element.

### 22. How can you count entries inside one nested array?

**Answer:** Pass that nested array to **count()**, for example **count($profile['skills'])**.

**Explanation:** By default **count($profile)** counts only the top-level entries. **COUNT_RECURSIVE** counts nested arrays too, but it includes the nested arrays themselves in its total.

### 23. Why might a filtered numeric array still have nonsequential keys?

**Answer:** **array_filter()** preserves the original keys of the values it keeps.

**Explanation:** Assign **array_values($filtered)** when the next operation expects a compact zero-based list.

### 24. What is a risk of calling **array_filter($values)** without a callback?

**Answer:** PHP removes values it considers empty, such as **0**, **false**, **null**, **''**, and **'0'**.

**Explanation:** If any of those are valid data, provide a callback that states exactly which values should remain.

### 25. What does **array_is_list()** check, and which PHP version introduced it?

**Answer:** It checks whether an array has consecutive integer keys from **0** in order. It is available starting in PHP 8.1.

**Explanation:** An array with keys **0** and **2**, or with the same keys in a different order, is not a list.

### 26. How do **sort()** and **asort()** differ?

**Answer:** Both sort values in ascending order. **sort()** assigns new integer keys; **asort()** preserves the association between each value and its key.

**Explanation:** Use **asort()** when sorting values in an associative array without losing the labels attached to them.

### 27. What does **ksort()** sort?

**Answer:** It sorts an array by its keys in ascending order while keeping each key associated with its value.

**Explanation:** Use it when key order matters. Use **asort()** to sort by values while preserving keys.

### 28. How does the array union operator **+** resolve duplicate keys?

**Answer:** The left-hand array's value wins for duplicate keys; values from the right-hand array are added only for keys not already present on the left.

**Explanation:** This differs from **array_merge()**, where a later duplicate string key replaces the earlier value.

### 29. What should code do before reading a deeply nested array value from uncertain input?

**Answer:** Validate that the expected keys and nested structure exist, or use a suitable fallback for optional paths.

**Explanation:** A nested array shape should not be assumed when data comes from an external or dynamic source. Validate input at its boundary.

## True or false

### 30. PHP arrays preserve an order for their entries.

**Answer:** True.

**Explanation:** A PHP array is an ordered map. Iteration follows the array's current element order.

### 31. A PHP array can only have integer keys.

**Answer:** False.

**Explanation:** Keys can be integers or strings, and a single array can contain both kinds.

### 32. Appending with **$items[] = $value** chooses an integer key automatically.

**Answer:** True.

**Explanation:** PHP assigns an integer key for the appended entry. Use an explicit string key for an associative entry.

### 33. Calling **unset($items[1])** automatically shifts later numeric keys down.

**Answer:** False.

**Explanation:** The remaining keys stay unchanged. Use **array_values()** if reindexing is required.

### 34. **array_filter()** returns a compact zero-based list by default.

**Answer:** False.

**Explanation:** It preserves the keys of retained elements, so numeric gaps may remain.

### 35. **array_values()** preserves the original keys.

**Answer:** False.

**Explanation:** It returns the values with fresh integer keys starting from zero.

### 36. **sort()** modifies its array argument and assigns new integer keys.

**Answer:** True.

**Explanation:** It sorts in place by values and reindexes the array.

### 37. With **$left + $right**, a duplicate key from the right side replaces the left-side value.

**Answer:** False.

**Explanation:** For duplicate keys, the left-hand value is kept. **array_merge()** has different behavior for string keys.

## Predict the output

### 38. What does this print?

~~~php
<?php

$colors = ['red', 'green', 'blue'];
echo $colors[1];
~~~

**Answer:**

~~~text
green
~~~

**Explanation:** The omitted keys are **0**, **1**, and **2**. The value at key **1** is **green**.

### 39. What does this print?

~~~php
<?php

$items = ['pen'];
$items[] = 'book';
$items['favorite'] = 'map';

echo $items[1] . ' / ' . $items['favorite'];
~~~

**Answer:**

~~~text
book / map
~~~

**Explanation:** The appended value receives the next integer key, **1**. The explicit string key stores **map**.

### 40. What does this print?

~~~php
<?php

$letters = ['a', 'b', 'c'];
unset($letters[1]);
print_r($letters);
~~~

**Answer:**

~~~text
Array
(
    [0] => a
    [2] => c
)
~~~

**Explanation:** **unset()** removes key **1** but does not renumber key **2**.

### 41. What does this print?

~~~php
<?php

$data = ['note' => null];

var_dump(isset($data['note']));
var_dump(array_key_exists('note', $data));
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** **isset()** requires a non-null value. **array_key_exists()** checks only whether the key exists.

### 42. What does this print?

~~~php
<?php

$defaults = ['theme' => 'light', 'items' => 2];
$custom = ['theme' => 'dark', 'language' => 'en'];

$result = array_merge($defaults, $custom);
print_r($result);
~~~

**Answer:**

~~~text
Array
(
    [theme] => dark
    [items] => 2
    [language] => en
)
~~~

**Explanation:** The later string key **theme** replaces the earlier value. The other keys remain in the merged result.

## Coding questions

### 43. Write code that removes scores below 70, then makes the result a zero-based list.

**Answer:**

~~~php
<?php

$scores = [85, 62, 91, 70];

$passingScores = array_filter(
    $scores,
    fn (int $score): bool => $score >= 70
);

$passingScores = array_values($passingScores);

print_r($passingScores);
~~~

**Expected output:**

~~~text
Array
(
    [0] => 85
    [1] => 91
    [2] => 70
)
~~~

**Explanation:** The filter keeps scores of at least **70** and initially preserves their old keys. **array_values()** creates the compact list.

### 44. Write a function that prints every key-value pair in an associative profile.

**Answer:**

~~~php
<?php

function printProfile(array $profile): void
{
    foreach ($profile as $key => $value) {
        echo $key . ': ' . $value . "\n";
    }
}

printProfile([
    'name' => 'Asha',
    'role' => 'editor',
]);
~~~

**Expected output:**

~~~text
name: Asha
role: editor
~~~

**Explanation:** **foreach ($profile as $key => $value)** makes both pieces of each entry available.

### 45. Combine defaults and user settings so user string keys override defaults, then explain the result.

**Answer:**

~~~php
<?php

$defaults = [
    'theme' => 'light',
    'itemsPerPage' => 10,
];

$userSettings = [
    'theme' => 'dark',
];

$settings = array_merge($defaults, $userSettings);

print_r($settings);
~~~

**Expected output:**

~~~text
Array
(
    [theme] => dark
    [itemsPerPage] => 10
)
~~~

**Explanation:** **array_merge()** uses the later value for a duplicate string key. If the left-hand value should win instead, use the array union operator **+**.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [048 - Working with Strings](../lessons/048-working-with-strings.md) | [Read lesson](../lessons/049-working-with-arrays.md) | [Open example](../examples/049-working-with-arrays.md) | [050 - Regular Expressions with PCRE](../lessons/050-regular-expressions-with-pcre.md) |
