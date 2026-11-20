# 016 - Indexed and Associative Arrays: Complete Example

This command-line example stores chapter names in an indexed array and student details in an associative array. It also demonstrates nested values, key checks, numeric-looking keys, replacement, counting, and removal without automatic reindexing.

## Question

How can a PHP program organize a list and named fields, then read and update those values without assuming every array key is a position?

## Goal

Create this project:

~~~text
array-practice/
|-- README.md
|-- arrays.php
~~~

The **arrays.php** script demonstrates:

- Creating an indexed array and appending an item
- Creating and updating an associative array
- Reading a nested value
- Comparing **isset()** and **array_key_exists()** for a null value
- Seeing how numeric-looking string keys are stored
- Replacing an existing value
- Removing an entry while preserving other keys
- Counting array entries

## Complete code: arrays.php

Save this file at the project root:

~~~php
<?php

$chapters = [
    "Introduction",
    "Variables",
];

$chapters[] = "Arrays";

echo "Chapter 0: " . $chapters[0] . PHP_EOL;
echo "Chapter 2: " . $chapters[2] . PHP_EOL;
echo "Chapter count before removal: " . count($chapters) . PHP_EOL;

$student = [
    "name" => "Asha",
    "city" => "Jaipur",
    "nickname" => null,
    "scores" => [
        "quiz" => 9,
        "project" => 18,
    ],
];

echo "Student: " . $student["name"] . PHP_EOL;
echo "City: " . $student["city"] . PHP_EOL;

var_dump(isset($student["nickname"]));
var_dump(array_key_exists("nickname", $student));

$student["scores"]["project"] = 20;
echo "Project score: " . $student["scores"]["project"] . PHP_EOL;

$codes = [
    "8" => "integer-like key",
    "08" => "text key with leading zero",
];

var_dump(array_keys($codes));

$settings = [
    "theme" => "light",
];

$settings["theme"] = "dark";
var_dump($settings);

unset($chapters[1]);

var_dump($chapters);
echo "Chapter count after removal: " . count($chapters) . PHP_EOL;
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Array Practice

A small PHP CLI script for practicing indexed and associative arrays.

## Check and run

~~~sh
php -l arrays.php
php arrays.php
~~~

## What it demonstrates

- Appending to an indexed array
- Reading and updating associative values
- Accessing a nested array
- Checking a null-valued key
- Numeric-looking array keys
- Replacing an existing value
- Removing an entry without renumbering keys
- Counting array entries
~~~~

This Markdown block is the README file to create in the project. Keep the PHP source in **arrays.php**.

## Check and run the program

Open a terminal in **array-practice/** and check the syntax:

~~~sh
php -l arrays.php
~~~

For a valid file, PHP reports that no syntax errors were detected. Run it with:

~~~sh
php arrays.php
~~~

Expected output:

~~~text
Chapter 0: Introduction
Chapter 2: Arrays
Chapter count before removal: 3
Student: Asha
City: Jaipur
bool(false)
bool(true)
Project score: 20
array(2) {
  [0]=>
  int(8)
  [1]=>
  string(2) "08"
}
array(1) {
  ["theme"]=>
  string(4) "dark"
}
array(2) {
  [0]=>
  string(12) "Introduction"
  [2]=>
  string(6) "Arrays"
}
Chapter count after removal: 2
~~~

## Understand the output

### Appending uses an integer key

**$chapters[] = "Arrays"** appends the value after the first two entries. The automatically assigned keys are **0**, **1**, and **2**. The key is how the value is addressed; it is not guaranteed to stay equal to its position after changes.

### Named keys make fields clear

The **$student** array uses strings such as **"name"**, **"city"**, and **"scores"** to name each value. The nested **scores** array uses its own keys, so the project score is accessed as **$student["scores"]["project"]**.

### A null-valued key exists even though isset() is false

The **nickname** key is present and its value is **null**. Thus **isset()** returns false, while **array_key_exists()** returns true. These functions answer different questions.

### Numeric-looking keys can be converted

The key **"8"** is stored as integer **8**. The key **"08"** remains a string because of its leading zero. **array_keys()** shows the actual key types.

### Updating a key replaces its previous value

The **theme** key starts as **"light"** and is then assigned **"dark"**. The array contains one **theme** entry with the new value.

### Removing a value does not renumber other keys

**unset($chapters[1])** removes **"Variables"**. The remaining keys are **0** and **2**, and **count()** returns **2** because there are two entries.

## Make a change

1. Add another chapter with **$chapters[] = "..."** and inspect the assigned key.
2. Change **$student["city"]** and print the updated value.
3. Add a **"reading"** score under **$student["scores"]** and read it by its key.
4. Change **"nickname"** from **null** to a string. Compare the results from **isset()** and **array_key_exists()** again.
5. Add **"8"** and **"08"** as keys to a new array. Inspect **array_keys()**.
6. Remove a different chapter key with **unset()**. Check the remaining keys and the count.
7. Assign a different value to **$settings["theme"]** and confirm the previous value is replaced.
8. Add another array inside **$student** and read one of its values using one bracket pair per nesting level.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming the first indexed value is at key 1 | Automatic indexes start at 0. | Use the actual key when reading an array. |
| Assuming a key always equals an item's current position | Keys can have gaps after an item is removed. | Treat keys as identifiers; reindex deliberately if a sequential list is required. |
| Expecting unset() to renumber the array | Other keys remain unchanged. | Inspect the actual keys and count entries separately. |
| Assuming isset() is true for a null-valued key | isset() returns false when a key's value is null. | Use array_key_exists() when you need to know whether the key is present. |
| Repeating an associative key to store multiple values | The later assignment replaces the earlier value. | Use a nested array when one key should contain multiple values. |
| Assuming the string key "8" stays a string | PHP converts some integer-like string keys to integers. | Use descriptive text keys when the key is meant to be a name. |
| Reading a key that might be missing | It can produce a warning and hide a data issue. | Check the key before accessing it. |
| Assuming count() returns the largest index plus one | Arrays may have key gaps or string keys. | Use count() for the number of entries. |

## Key takeaways

- PHP arrays associate values with integer or string keys.
- Use integer keys for list-like data and descriptive string keys for named fields.
- Append with **[]**, read or update using the key in brackets, and remove with **unset()**.
- Nested arrays use one key at each level.
- **isset()** returns false for null-valued keys; **array_key_exists()** checks key presence.
- Duplicate keys replace earlier values, and some numeric-looking keys are converted.
- Removing an entry does not automatically renumber remaining integer keys.
- **count()** returns the number of entries, not the largest index plus one.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [015 - Strings and numeric strings](../lessons/015-strings-and-numeric-strings.md) | [Read this lesson](../lessons/016-indexed-and-associative-arrays.md) | [Open Q&A](../qnas/016-indexed-and-associative-arrays.md) | [017 - Objects, resources, callables, and iterables](../lessons/017-objects-resources-callables-and-iterables.md) |
