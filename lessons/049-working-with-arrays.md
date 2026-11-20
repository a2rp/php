# 049 - Working with Arrays

A PHP **array** is an ordered map that associates keys with values. It can act like a numbered list, a dictionary, or a nested collection. Arrays are one of the main ways PHP programs group related data.

## What you will learn

- Create indexed, associative, and nested arrays.
- Read, add, update, and remove array elements.
- Loop through keys and values with **foreach**.
- Check keys and values safely, including values that are **null**.
- Filter, reindex, sort, and combine arrays.
- Understand the difference between **array_merge()** and the array union operator.

## Create arrays

Use square brackets to create an array. A trailing comma after the last item is allowed and is convenient in multiline arrays.

~~~php
<?php

$colors = ['red', 'green', 'blue'];

$empty = [];

$numbers = [
    10,
    20,
    30,
];
~~~

When you omit keys, PHP assigns integer keys starting at **0**:

~~~php
<?php

$colors = ['red', 'green', 'blue'];

echo $colors[0];
echo $colors[2];
~~~

An **associative array** uses descriptive keys:

~~~php
<?php

$user = [
    'name' => 'Asha',
    'email' => 'asha@example.test',
    'active' => true,
];

echo $user['name'];
~~~

PHP arrays can contain integer and string keys together, but prefer one clear shape for a particular data structure. Array keys are integers or strings; other key types are converted. For example, **true** becomes integer **1**, **false** becomes **0**, **null** becomes an empty string, and a valid decimal integer string such as **'8'** becomes integer **8**. A key written more than once replaces the earlier value for that key.

## Read and change elements

Read a value with its key in square brackets. Add or update an item by assigning to that key.

~~~php
<?php

$colors = ['red', 'green'];

$firstColor = $colors[0];
$colors[1] = 'blue';
$colors[] = 'purple';

print_r($colors);
~~~

The empty bracket syntax **$array[] = $value** appends a new item. For ordinary lists, PHP assigns the next integer key. For associative arrays, choose an explicit descriptive key:

~~~php
<?php

$user = ['name' => 'Asha'];
$user['role'] = 'editor';

echo $user['role'];
~~~

Reading a key that does not exist raises a warning in current PHP and yields **null**. Do not assume that an absent key is already present. The null coalescing operator **??** gives a fallback when a key is missing or its value is **null**:

~~~php
<?php

$user = ['name' => 'Asha'];

$role = $user['role'] ?? 'reader';
echo $role;
~~~

Use **isset()** to check that a variable or key exists and is not **null**. Use **array_key_exists()** when you need to know whether the key itself exists, even if its value is **null**:

~~~php
<?php

$settings = [
    'theme' => null,
];

var_dump(isset($settings['theme'])); // false
var_dump(array_key_exists('theme', $settings)); // true
~~~

**isset()** also avoids the missing-key warning when checking an array key.

Remove one entry with **unset()**:

~~~php
<?php

$colors = ['red', 'green', 'blue'];
unset($colors[1]);

print_r($colors);
~~~

Removing an item does not automatically renumber the remaining integer keys. This is useful when keys have meaning. To turn the remaining values into a fresh zero-based list, assign the result of **array_values()**:

~~~php
<?php

$colors = ['red', 'green', 'blue'];
unset($colors[1]);

$colors = array_values($colors);
print_r($colors);
~~~

## Loop through arrays

Use **foreach** to visit each value:

~~~php
<?php

$colors = ['red', 'green', 'blue'];

foreach ($colors as $color) {
    echo $color . "\n";
}
~~~

For an associative array, capture both the key and value:

~~~php
<?php

$user = [
    'name' => 'Asha',
    'role' => 'editor',
];

foreach ($user as $key => $value) {
    echo $key . ': ' . $value . "\n";
}
~~~

By default, **foreach** gives the loop a value to work with without changing the array element. Use a reference only when you deliberately want to modify each stored value in place; then call **unset()** on the reference variable after the loop so it cannot accidentally keep pointing at the last element.

~~~php
<?php

$scores = [2, 4, 6];

foreach ($scores as &$score) {
    $score *= 10;
}
unset($score);
~~~

For most beginner code, a non-reference **foreach** is simpler and safer.

## Count and search

**count()** returns the number of top-level entries in an array:

~~~php
<?php

$profile = [
    'name' => 'Asha',
    'skills' => ['HTML', 'CSS', 'PHP'],
];

echo count($profile); // 2 top-level entries
echo count($profile['skills']); // 3 skills
~~~

Use **in_array()** to search values. Pass **true** as the third argument when the value's type must match as well:

~~~php
<?php

$allowedIds = [7, 12, 20];

var_dump(in_array(12, $allowedIds, true));
var_dump(in_array('12', $allowedIds, true));
~~~

The first check is true and the second is false because integer **12** and string **'12'** are different types. Strict comparisons help avoid surprising matches when the array can contain mixed types.

Use **array_key_exists()** or **isset()** to search keys; **in_array()** searches values instead.

## Filter and transform

**array_map()** applies a callback to each value and returns the transformed values. **array_filter()** keeps values for which its callback returns **true**.

~~~php
<?php

$prices = [12, 5, 20, 8];

$withTax = array_map(
    fn (int $price): int => $price + 2,
    $prices
);

$affordable = array_filter(
    $prices,
    fn (int $price): bool => $price <= 10
);

print_r($withTax);
print_r($affordable);
~~~

**array_filter()** preserves the original keys. A filtered indexed array can therefore have gaps. Use **array_values()** if you specifically need a newly indexed list:

~~~php
<?php

$prices = [12, 5, 20, 8];

$affordable = array_filter(
    $prices,
    fn (int $price): bool => $price <= 10
);

$affordableList = array_values($affordable);
print_r($affordableList);
~~~

Avoid calling **array_filter()** without a callback when zero, **false**, **null**, an empty string, or the string **'0'** might be valid data. Without a callback, it removes values PHP considers empty.

## Sort arrays

**sort()** sorts values in ascending order and assigns new integer keys. Use it for a list when the old keys do not need to stay attached to the values.

~~~php
<?php

$colors = ['green', 'blue', 'red'];
sort($colors);

print_r($colors);
~~~

For an associative array, **asort()** sorts by value while preserving each value's key. **ksort()** sorts by key while preserving key-value associations:

~~~php
<?php

$scores = [
    'Mina' => 82,
    'Asha' => 95,
    'Leo' => 88,
];

asort($scores);
print_r($scores);

ksort($scores);
print_r($scores);
~~~

Sorting functions modify the array they receive. Pick a sorting function based on whether to sort by keys or values and whether keys must remain associated. Use explicit flags such as **SORT_NUMERIC** or **SORT_STRING** when mixed value types could make ordinary comparison unclear.

## Combine arrays

**array_merge()** combines arrays. Numeric keys are appended and renumbered; when string keys repeat, the value from the later array replaces the earlier one:

~~~php
<?php

$defaults = [
    'theme' => 'light',
    'items' => 3,
];

$settings = [
    'theme' => 'dark',
];

$combined = array_merge($defaults, $settings);
print_r($combined);
~~~

The array union operator **+** has different behavior: when a key exists on the left, its value wins and the right-side value for that key is ignored. New right-side keys are added.

~~~php
<?php

$defaults = ['theme' => 'light', 'items' => 3];
$settings = ['theme' => 'dark', 'language' => 'en'];

$union = $defaults + $settings;
print_r($union);
~~~

Here **theme** remains **light**. Choose **array_merge()** or **+** based on the desired duplicate-key behavior; they are not interchangeable.

## Nested arrays

An array can contain other arrays. Read nested values by chaining keys:

~~~php
<?php

$course = [
    'title' => 'PHP Basics',
    'lessons' => [
        ['title' => 'Variables', 'minutes' => 12],
        ['title' => 'Arrays', 'minutes' => 18],
    ],
];

echo $course['lessons'][1]['title'];
~~~

Nested structures are useful for related records, but check each expected level before accessing values that may be missing. The null coalescing operator can provide a fallback for a known path:

~~~php
<?php

$course = [];

$title = $course['lessons'][0]['title'] ?? 'Untitled';
echo $title;
~~~

For dynamic or more complex data, validate its shape at the boundary where it enters your program instead of assuming every nested key exists.

## List-specific checks

A PHP array can have integer keys without being a sequential list. In PHP 8.1 and later, **array_is_list()** checks whether an array's keys are exactly **0, 1, 2, ...** in that order:

~~~php
<?php

var_dump(array_is_list(['a', 'b']));
var_dump(array_is_list([1 => 'b', 0 => 'a']));
~~~

The first is a list; the second is not because its keys are out of order. If your code supports a PHP version before 8.1, do not call **array_is_list()** unless you provide a compatible alternative.

## Common mistakes

- Treating every array as a zero-based list; associative keys can be strings or nonsequential integers.
- Accessing an absent key without checking or providing a fallback.
- Using **isset()** when an existing **null** value must count as present.
- Assuming **unset()** reindexes the remaining numeric keys.
- Forgetting that **array_filter()** preserves keys.
- Sorting an associative array with **sort()** and losing its key associations.
- Expecting **array_merge()** and **+** to resolve duplicate keys the same way.
- Searching values with a key-checking function or searching keys with **in_array()**.

## Try it yourself

1. Make a list of three favorite languages and print each with **foreach**.
2. Create an associative profile with a name, role, and active status.
3. Add a key, update it, and remove another key with **unset()**.
4. Compare **isset()** and **array_key_exists()** for a key whose value is **null**.
5. Filter a list of numbers, inspect the preserved keys, and then reindex with **array_values()**.
6. Sort an associative score array by value while preserving keys, then sort it by key.
7. Combine two settings arrays with **array_merge()** and with **+**; explain the duplicate-key difference.
8. Use **array_is_list()** on a sequential array and an array with a missing or reordered key.

## Key takeaways

- PHP arrays are ordered maps with integer or string keys.
- Use **foreach** for readable iteration; use key-value iteration when keys matter.
- Check for missing keys and decide whether **null** means absent or is a real value.
- **array_filter()** preserves keys; **array_values()** creates a fresh zero-based list.
- Sorting functions differ in sort direction, key preservation, and whether they sort by key or value.
- **array_merge()** and array **+** have different duplicate-key rules.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [048 - Working with Strings](048-working-with-strings.md) | [Open Q&A](../qnas/049-working-with-arrays.md) | [Open example](../examples/049-working-with-arrays.md) | [050 - Regular Expressions with PCRE](050-regular-expressions-with-pcre.md) |
