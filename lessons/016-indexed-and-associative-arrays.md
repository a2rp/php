# 016 - Indexed and Associative Arrays

An array groups multiple values under keys. Use an indexed array when the values form a sequence, and an associative array when descriptive keys make the data easier to understand. PHP arrays are ordered maps, so one array can use both integer and string keys.

## What you will learn

- How to create an array with PHP's short array syntax
- How indexed arrays use integer keys
- How associative arrays use descriptive string keys
- How to read, add, update, and remove array entries
- How PHP assigns keys when you append values
- How duplicate and numeric-looking keys behave
- How to count entries and inspect whether a key exists
- How nested arrays organize related data

## Create an array

Use square brackets to create an array. Separate values with commas. A trailing comma after the last item is allowed and makes multi-line arrays easier to edit:

~~~php
<?php
$colors = [
    "blue",
    "green",
    "orange",
];

var_dump($colors);
~~~

Output:

~~~text
array(3) {
  [0]=>
  string(4) "blue"
  [1]=>
  string(5) "green"
  [2]=>
  string(6) "orange"
}
~~~

When no key is supplied, PHP starts at integer key **0** and assigns the next integer key to each item. The older **array()** syntax is also valid, but short brackets are common in modern PHP:

~~~php
$colors = array("blue", "green", "orange");
~~~

An empty array can be written as **[]**:

~~~php
$nothingYet = [];
~~~

## Indexed arrays

An indexed array uses integer keys to identify values. You can provide keys explicitly, or let PHP assign them:

~~~php
<?php
$chapters = [
    0 => "Introduction",
    1 => "Variables",
    2 => "Arrays",
];

echo $chapters[0] . PHP_EOL;
echo $chapters[2] . PHP_EOL;
~~~

Output:

~~~text
Introduction
Arrays
~~~

The integer key inside the square brackets selects the value. The first item is commonly at key **0**, but an indexed array does not have to have consecutive keys.

### Append an item

Use empty square brackets to append an item. PHP assigns the next integer key:

~~~php
<?php
$chapters = ["Introduction", "Variables"];
$chapters[] = "Arrays";

var_dump($chapters);
~~~

Output:

~~~text
array(3) {
  [0]=>
  string(12) "Introduction"
  [1]=>
  string(9) "Variables"
  [2]=>
  string(6) "Arrays"
}
~~~

You can also append to an initially empty array:

~~~php
$topics = [];
$topics[] = "syntax";
$topics[] = "values";
~~~

## Associative arrays

An associative array uses descriptive string keys. The **=>** operator associates each key with its value:

~~~php
<?php
$student = [
    "name" => "Asha",
    "chapter" => 16,
    "isActive" => true,
];

echo $student["name"] . PHP_EOL;
echo $student["chapter"] . PHP_EOL;
~~~

Output:

~~~text
Asha
16
~~~

Descriptive keys make it clear what each value means. The key is written as a string in the array definition and used inside the brackets when you access that value.

Add or replace an entry by assigning to its key:

~~~php
<?php
$student = [
    "name" => "Asha",
    "chapter" => 16,
];

$student["city"] = "Jaipur";
$student["chapter"] = 17;

var_dump($student);
~~~

The **city** key is new. The existing **chapter** key receives a replacement value.

## PHP arrays are ordered maps

An array associates each value with an integer or string key and retains the entries in insertion order. “Indexed” and “associative” describe common ways of using arrays; PHP uses the same array type for both.

An array can contain integer keys and string keys together, and its values can have different types. For clarity, most application arrays should use a consistent shape and key style:

~~~php
<?php
$record = [
    "name" => "Asha",
    "chapter" => 16,
    0 => "active",
];

var_dump($record);
~~~

Mixing key styles is allowed, but it can make the data harder to reason about. Choose integer keys for list-like values and descriptive string keys for named fields.

## Array keys and numeric-looking strings

Array keys are integers or strings. PHP converts some other key types and some numeric-looking string keys:

- A string containing a valid decimal integer, such as **"8"**, is stored as integer key **8**.
- A string such as **"08"** remains a string key because of its leading zero.
- Boolean keys become integer **0** or **1**.
- A float key is converted to an integer, dropping its fractional part.
- A **null** key becomes an empty string key.
- Arrays and objects cannot be used as array keys.

These conversions can make two written keys refer to the same stored key. For beginner code, use explicit integer keys for indexed data and descriptive non-numeric strings for associative keys.

~~~php
<?php
$codes = [
    "8" => "integer-like key",
    "08" => "text key with leading zero",
];

var_dump($codes);
~~~

The first key is stored as integer **8**; the second remains the string **"08"**.

## Duplicate keys replace earlier values

Each key can identify only one value in an array. If the same key is assigned more than once, the later value replaces the earlier value:

~~~php
<?php
$settings = [
    "theme" => "light",
    "theme" => "dark",
];

var_dump($settings);
~~~

Output:

~~~text
array(1) {
  ["theme"]=>
  string(4) "dark"
}
~~~

This is replacement, not a second entry. If multiple values belong under one category, store an array as the value:

~~~php
<?php
$settings = [
    "themes" => ["light", "dark"],
];
~~~

Nested arrays are introduced below.

## Read and update nested arrays

An array can contain another array. Use one bracket pair for each level:

~~~php
<?php
$student = [
    "name" => "Asha",
    "scores" => [
        "quiz" => 9,
        "project" => 18,
    ],
];

echo $student["scores"]["project"] . PHP_EOL;

$student["scores"]["project"] = 20;
echo $student["scores"]["project"] . PHP_EOL;
~~~

Output:

~~~text
18
20
~~~

Keep the data structure understandable. If an array becomes deeply nested or unclear, later lessons about objects and data design can help you choose a better structure.

## Count the entries

Use **count()** to get the number of entries in an array:

~~~php
<?php
$chapters = ["Introduction", "Variables", "Arrays"];

echo count($chapters) . PHP_EOL;
~~~

Output:

~~~text
3
~~~

**count()** counts the entries in that array. For nested arrays, its default behavior counts only the top-level entries; it does not automatically count every value at every depth.

## Check whether a key exists

Use **isset()** when you need to know whether an array entry exists and its value is not **null**:

~~~php
<?php
$profile = [
    "name" => "Asha",
    "nickname" => null,
];

var_dump(isset($profile["name"]));
var_dump(isset($profile["nickname"]));
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

To distinguish a missing key from a key that exists with a **null** value, use **array_key_exists()**:

~~~php
<?php
$profile = [
    "nickname" => null,
];

var_dump(array_key_exists("nickname", $profile));
var_dump(array_key_exists("city", $profile));
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

**isset()** is false for a key whose value is **null**. **array_key_exists()** checks whether the key is present, including when its value is null.

Do not read an optional or missing array key before checking whether it exists. Referencing a missing key can produce a warning and an absent value, which can hide data mistakes.

## Remove an entry

Use **unset()** to remove an array entry:

~~~php
<?php
$scores = [90, 85, 100];
unset($scores[1]);

var_dump($scores);
echo count($scores) . PHP_EOL;
~~~

Output:

~~~text
array(2) {
  [0]=>
  int(90)
  [2]=>
  int(100)
}
2
~~~

Removing an entry does not automatically renumber the remaining integer keys. The array has two entries, but its keys are **0** and **2**. Do not assume the key of an item is the same as its current position.

## Inspect arrays while learning

Use **var_dump()** to see keys, values, and value types:

~~~php
<?php
$course = [
    "title" => "PHP",
    "chapterCount" => 16,
];

var_dump($course);
~~~

For readable debugging output, **print_r()** can be useful, but **var_dump()** also shows the types of values.

## Try it yourself

1. Create an indexed array with three favorite foods and print the first and third values.
2. Create an empty array and append three chapter titles with **[]**.
3. Create an associative array for a book with title, author, and page count. Read each value by its key.
4. Update a value and add a new key. Inspect the full array.
5. Create an array where **"8"** and **"08"** are keys. Inspect how PHP stores them.
6. Assign the same associative key twice. Confirm that the later value replaces the earlier one.
7. Make an associative array that contains a nested array and read a value from the inner array.
8. Create a key with a **null** value. Compare **isset()** and **array_key_exists()** for it.
9. Remove the middle element from an indexed array with **unset()**. Inspect the remaining keys and count.
10. Explain why **count()** is not necessarily the largest integer key plus one.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming every array begins at key 1 | Automatically assigned indexes begin at 0. | Read the key explicitly and use the index intended by the data. |
| Treating an indexed array's key as its current position | Keys can have gaps or start elsewhere. | Access by the actual key, or deliberately reindex when a sequential list is required. |
| Expecting unset() to renumber the remaining keys | Removing an entry preserves other keys. | Check the actual keys; use a dedicated reindexing operation when needed. |
| Reusing an associative key accidentally | The later value replaces the earlier one. | Use unique keys or group multiple values under an array key. |
| Assuming a numeric-looking string key always remains text | PHP converts some integer-like string keys to integer keys. | Use descriptive text keys and inspect the array when key type matters. |
| Using isset() to detect a key whose value may be null | isset() returns false for null values. | Use array_key_exists() when presence itself matters. |
| Reading a key that may not exist | This can produce a warning and hide a missing-data bug. | Check the key before accessing it. |
| Assuming count() recursively counts nested values | By default it counts only the entries at the current level. | Count nested arrays deliberately when the application needs a recursive total. |

## Key takeaways

- PHP arrays are ordered maps that associate values with integer or string keys.
- Indexed arrays commonly use consecutive integer keys beginning at **0**.
- Associative arrays use descriptive string keys for named values.
- Use **[]** to append, bracket keys to read or update, and **unset()** to remove.
- Duplicate keys replace earlier values; integer-like string keys may be converted.
- **count()** counts entries at the current level, not the largest index plus one.
- Use **isset()** for a present non-null value and **array_key_exists()** when a null-valued key still counts as present.
- Nested arrays are accessed with one key at each level.
- Array traversal with **foreach** is covered in a later lesson.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [015 - Strings and numeric strings](015-strings-and-numeric-strings.md) | [Open Q&A](../qnas/016-indexed-and-associative-arrays.md) | [Open example](../examples/016-indexed-and-associative-arrays.md) | [017 - Objects, resources, callables, and iterables](017-objects-resources-callables-and-iterables.md) |
