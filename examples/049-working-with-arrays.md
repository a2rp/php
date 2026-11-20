# 049 - Working with Arrays: Complete Example

This example builds a small course dashboard using PHP arrays. It demonstrates list and associative arrays, reading and changing values, appending and removing entries, loops, null-aware key checks, strict value searches, filtering, sorting, and combining arrays.

## Goal

Use arrays to represent colors, a user profile, a score list, and configuration settings. Then read and transform those values with PHP's array syntax and functions.

## Requirements

- PHP 8.0 or later for the arrow functions and strict typing used in the example.
- The example uses built-in array functions and does not need an extra extension.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

// A list uses integer keys that PHP assigns automatically.
$colors = ['red', 'green', 'blue'];
$colors[1] = 'teal';
$colors[] = 'purple';

echo 'Colors: ' . implode(', ', $colors) . "\n";

foreach ($colors as $index => $color) {
    echo $index . ': ' . $color . "\n";
}

// An associative array uses descriptive string keys.
$user = [
    'name' => 'Asha',
    'role' => 'editor',
    'active' => true,
];

$user['lastLogin'] = 'today';

echo 'Name: ' . $user['name'] . "\n";
echo 'Role: ' . ($user['role'] ?? 'reader') . "\n";
echo 'Email: ' . ($user['email'] ?? 'not provided') . "\n";

// isset() checks that a key exists and is not null.
// array_key_exists() checks whether the key exists, even when null.
$settings = [
    'theme' => null,
];

echo 'Theme has a non-null value: ';
echo isset($settings['theme']) ? "yes\n" : "no\n";

echo 'Theme key exists: ';
echo array_key_exists('theme', $settings) ? "yes\n" : "no\n";

// Count a top-level array and one of its nested arrays separately.
$profile = [
    'name' => 'Asha',
    'skills' => ['HTML', 'CSS', 'PHP'],
];

echo 'Profile entries: ' . count($profile) . "\n";
echo 'Skills: ' . count($profile['skills']) . "\n";

// Strict membership distinguishes an integer from a numeric string.
$allowedIds = [7, 12, 20];

echo 'Has integer 12: ';
echo in_array(12, $allowedIds, true) ? "yes\n" : "no\n";

echo 'Has string 12 (strict): ';
echo in_array('12', $allowedIds, true) ? "yes\n" : "no\n";

// unset() removes an entry but keeps the other original keys.
$queue = ['first', 'second', 'third'];
unset($queue[1]);

echo 'Keys after removal: ' . implode(', ', array_keys($queue)) . "\n";

$queueAsList = array_values($queue);
echo 'Reindexed queue: ' . implode(', ', $queueAsList) . "\n";

// array_filter() preserves keys; array_values() makes a compact list.
$prices = [12, 5, 20, 8];

$affordable = array_filter(
    $prices,
    fn (int $price): bool => $price <= 10
);

echo "Filtered prices with original keys:\n";
print_r($affordable);

$affordableList = array_values($affordable);

echo "Filtered prices reindexed:\n";
print_r($affordableList);

// asort() sorts values and keeps the association with each name.
$scores = [
    'Mina' => 82,
    'Asha' => 95,
    'Leo' => 88,
];

asort($scores);

echo 'Scores ascending: ';
$scoreLabels = [];

foreach ($scores as $name => $score) {
    $scoreLabels[] = $name . ': ' . $score;
}

echo implode(', ', $scoreLabels) . "\n";

// array_merge() and + resolve duplicate keys differently.
$defaults = [
    'theme' => 'light',
    'itemsPerPage' => 3,
];

$custom = [
    'theme' => 'dark',
    'language' => 'en',
];

$mergedSettings = array_merge($defaults, $custom);
$unionSettings = $defaults + $custom;

echo 'Merged theme: ' . $mergedSettings['theme'] . "\n";
echo 'Union theme: ' . $unionSettings['theme'] . "\n";
echo 'Merged item count: ' . $mergedSettings['itemsPerPage'] . "\n";
echo 'Merged language: ' . $mergedSettings['language'] . "\n";

// Nested arrays group related records.
$course = [
    'title' => 'PHP Basics',
    'lessons' => [
        ['title' => 'Variables', 'minutes' => 12],
        ['title' => 'Arrays', 'minutes' => 18],
    ],
];

$secondLesson = $course['lessons'][1]['title'] ?? 'Untitled';
echo 'Second lesson: ' . $secondLesson . "\n";
~~~

## Expected output

~~~text
Colors: red, teal, blue, purple
0: red
1: teal
2: blue
3: purple
Name: Asha
Role: editor
Email: not provided
Theme has a non-null value: no
Theme key exists: yes
Profile entries: 2
Skills: 3
Has integer 12: yes
Has string 12 (strict): no
Keys after removal: 0, 2
Reindexed queue: first, third
Filtered prices with original keys:
Array
(
    [1] => 5
    [3] => 8
)
Filtered prices reindexed:
Array
(
    [0] => 5
    [1] => 8
)
Scores ascending: Mina: 82, Leo: 88, Asha: 95
Merged theme: dark
Union theme: light
Merged item count: 3
Merged language: en
Second lesson: Arrays
~~~

## How it works

### Lists and associative arrays

**$colors** starts as a list. PHP assigns keys **0**, **1**, and **2**. The assignment to **$colors[1]** changes the second value, and **$colors[]** appends a fourth. The **foreach** loop reads both each key and its value.

**$user** is associative: string keys such as **'name'** and **'role'** label their values. The **??** operator supplies a fallback for the missing **email** key without requiring a separate existence check.

### Null-aware checks and counts

The **theme** key is present but its value is **null**. **isset()** reports no non-null value, while **array_key_exists()** reports that the key exists.

**count($profile)** counts the two top-level keys. The skills are a separate nested array, so **count($profile['skills'])** returns three.

### Remove, filter, and reindex

**unset($queue[1])** removes the second item without shifting key **2**. **array_keys()** shows the remaining keys. **array_values()** copies the values into a new zero-based list.

The callback passed to **array_filter()** keeps prices at or below ten. The filtered result retains keys **1** and **3**; **array_values()** removes those gaps for the compact list.

### Sort and combine

**asort()** orders the score values while preserving each name as its key. Collecting formatted labels in a separate list makes the output easy to read.

**array_merge()** lets the later **theme** value override the earlier one. The **+** union operator keeps the left-hand **theme** value instead. Both results add **language** because that key only appears on the right.

### Nested arrays

The **course** array contains a list of lesson records. Chaining keys selects the second lesson title. The **??** fallback keeps the read safe if that path is absent or null.

## Try it yourself

1. Add a color and confirm that the appended key follows the existing integer keys.
2. Remove the first color and print the remaining keys before and after **array_values()**.
3. Add a **null** setting and compare **isset()** with **array_key_exists()**.
4. Change the filter condition to keep prices greater than ten.
5. Replace **asort()** with **ksort()** and compare the order.
6. Change the duplicate **theme** key and compare the results from **array_merge()** and **+**.
7. Add a third lesson and safely read its title with a fallback.

## Common mistakes

- Assuming integer keys always remain consecutive after an element is removed.
- Assuming **array_filter()** automatically creates a compact list.
- Using **isset()** when a null-valued key should count as present.
- Using **in_array()** without strict mode when array values can have different types.
- Sorting an associative array with **sort()** and losing the original key associations.
- Assuming **array_merge()** and **+** use the same duplicate-key rule.

## Key takeaways

- Use indexed arrays for ordered lists and descriptive string keys for records.
- Use **foreach** to read values or key-value pairs.
- Choose **isset()** or **array_key_exists()** based on how null values should be treated.
- Filtering can leave key gaps; **array_values()** produces a fresh list.
- Choose a sorting function based on whether keys must remain associated with values.
- **array_merge()** and array union **+** have different duplicate-key behavior.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [048 - Working with Strings](../lessons/048-working-with-strings.md) | [Read lesson](../lessons/049-working-with-arrays.md) | [Open Q&A](../qnas/049-working-with-arrays.md) | [050 - Regular Expressions with PCRE](../lessons/050-regular-expressions-with-pcre.md) |
