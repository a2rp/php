# 016 - Indexed and Associative Arrays: Questions and Answers

Use this Q&A to review [016 - Indexed and Associative Arrays](../lessons/016-indexed-and-associative-arrays.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What is a PHP array?

A. A value that can only store strings<br>
B. An ordered map that associates values with integer or string keys<br>
C. A class that must be declared before use<br>
D. A value that cannot contain another array

**Answer: B.** PHP arrays are ordered maps. They can be used as lists, dictionaries, or collections of values.

### 2. Which syntax creates an array containing two values?

A. **("red", "blue")**<br>
B. **["red", "blue"]**<br>
C. **{"red", "blue"}**<br>
D. **<red, blue>**

**Answer: B.** Square brackets are PHP's short array syntax.

### 3. What integer key is assigned to the first item when PHP creates an indexed array without explicit keys?

A. **-1**<br>
B. **0**<br>
C. **1**<br>
D. **null**

**Answer: B.** Automatically assigned indexes in a new list begin at **0**.

### 4. Which syntax appends a value to an array?

A. **$items + "new"**<br>
B. **$items[] = "new";**<br>
C. **$items->add("new");**<br>
D. **append($items, "new");**

**Answer: B.** Assigning through empty brackets appends a value using PHP's next integer key.

### 5. What does the **=>** operator do in an array literal?

A. Compares two values<br>
B. Associates a key with a value<br>
C. Appends a value without a key<br>
D. Removes a key

**Answer: B.** In an array literal, **=>** separates a key from its associated value.

### 6. Which is a valid associative-array key type?

A. Array or object<br>
B. Integer or string<br>
C. Float only<br>
D. Boolean only

**Answer: B.** Array keys are integers or strings. PHP converts some other types to integer or string keys; arrays and objects cannot be keys.

### 7. What happens when the same key is specified twice?

A. Both values stay under that key<br>
B. The later value replaces the earlier value<br>
C. The array becomes invalid<br>
D. The earlier value is moved to key zero

**Answer: B.** A key identifies one entry. Assigning the key again replaces its value.

### 8. In a normal PHP array, how is the key **"8"** stored?

A. As the string **"8"** in every case<br>
B. As integer **8**<br>
C. As float **8.0**<br>
D. As boolean **true**

**Answer: B.** A valid decimal integer string such as **"8"** is converted to integer key **8**. A string such as **"08"** remains a string key.

### 9. What does **count($items)** return?

A. The largest key plus one<br>
B. The number of entries in the array<br>
C. The number of string values only<br>
D. The number of nested values at every depth by default

**Answer: B.** **count()** returns the number of entries in the array. It does not calculate the largest key plus one.

### 10. What does **isset($profile["nickname"])** return if that key exists with value **null**?

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. It deletes the entry

**Answer: B.** **isset()** returns false for an array key whose value is **null**.

### 11. Which function checks whether a key exists even if its value is **null**?

A. **count()**<br>
B. **array_key_exists()**<br>
C. **is_array()**<br>
D. **strlen()**

**Answer: B.** **array_key_exists()** checks key presence. It returns true for an existing key even when its value is null.

### 12. How do you read the **city** field from this array?

~~~php
$address = ["city" => "Jaipur"];
~~~

A. **$address.city**<br>
B. **$address["city"]**<br>
C. **$address->city**<br>
D. **city($address)**

**Answer: B.** Use the key in square brackets to read an array value.

### 13. How do you access **18** from this array?

~~~php
$record = ["scores" => ["project" => 18]];
~~~

A. **$record["project"]**<br>
B. **$record["scores"]["project"]**<br>
C. **$record->scores->project**<br>
D. **$record[1][0]**

**Answer: B.** Each bracket pair selects one level: first the **scores** array, then its **project** entry.

### 14. What is the default behavior of **count()** on an array containing another array?

A. It recursively counts every nested element<br>
B. It counts the entries at the top level<br>
C. It returns zero<br>
D. It counts only integer keys

**Answer: B.** By default, **count()** counts entries in the array passed to it. It does not automatically count entries inside nested arrays.

## True or false

### 15. PHP has completely separate types for indexed arrays and associative arrays.

**Answer: False.** Both are PHP arrays. The labels describe common ways of organizing and accessing their keys.

### 16. PHP permits an array to contain both integer keys and string keys.

**Answer: True.** Both key types can appear in the same array, though a consistent key style is usually easier to understand.

### 17. An indexed array must always have consecutive keys starting at zero.

**Answer: False.** That is common for list-like arrays, but keys can be explicit and can have gaps.

### 18. Assigning a new value to an existing key adds a second entry with the same key.

**Answer: False.** The key still identifies one entry; its value is replaced.

### 19. Appending with **$items[] = $value** lets PHP choose an integer key.

**Answer: True.** PHP uses the next integer key according to the array's existing integer keys.

### 20. The string key **"08"** is always converted to integer key **8**.

**Answer: False.** A string with a leading zero such as **"08"** is not converted to an integer key by this rule.

### 21. Removing an indexed entry with **unset()** automatically renumbers all remaining keys.

**Answer: False.** The other keys are preserved, so the array may have gaps.

### 22. **array_key_exists()** returns true for a key that is present with a null value.

**Answer: True.** It checks key presence rather than whether the value is non-null.

## Short-answer questions

### 23. When should you prefer an indexed array?

**Answer:** Use an indexed array when values form a sequence and integer keys are suitable, such as a list of chapter titles.

**Explanation:** With automatic keys, values start at zero and later values receive the next integer key.

### 24. When should you prefer an associative array?

**Answer:** Use an associative array when descriptive names make each value's purpose clear, such as **"title"**, **"author"**, or **"pageCount"**.

**Explanation:** Named keys make it easier to understand what a value represents when reading and maintaining the code.

### 25. What does an empty array look like, and how can you add two values to it?

**Answer:**

~~~php
$topics = [];
$topics[] = "syntax";
$topics[] = "values";
~~~

The array starts with no entries. PHP assigns integer keys as each value is appended.

### 26. Why can **count($items)** differ from the largest integer key plus one?

**Answer:** **count()** counts entries. An array can have gaps or keys that do not start at zero.

**Explanation:** An array with keys **0** and **2** has two entries, so **count()** returns **2**, while the largest key plus one is **3**.

### 27. How can you tell the difference between a missing key and a present key whose value is null?

**Answer:** Use **array_key_exists()** to check whether the key is present. **isset()** returns false both for a missing key and for a key set to null.

### 28. What happens to numeric-looking string keys?

**Answer:** Some strings that look like valid decimal integers, such as **"8"**, are stored as integer keys. A leading-zero string such as **"08"** remains a string key.

**Explanation:** Key conversion can affect lookups and can make differently written keys collide. Prefer descriptive keys when the key is meant to be text.

### 29. What does **unset($items[1])** do to the other entries?

**Answer:** It removes the entry at key **1**. Other entries keep their existing keys; PHP does not renumber them automatically.

### 30. Why might mixing named and numbered keys make an array harder to use?

**Answer:** The array has multiple key conventions, so each lookup and later operation requires more care.

**Explanation:** Although PHP allows both types, use a list-like integer-keyed structure or a descriptive string-keyed structure when that fits the data.

## Read the code

### 31. What does this code print?

~~~php
$colors = ["blue", "green", "orange"];

echo $colors[0] . PHP_EOL;
echo $colors[2] . PHP_EOL;
~~~

**Answer:**

~~~text
blue
orange
~~~

**Explanation:** The first item is at key **0**, and the third item is at key **2**.

### 32. What does this code print?

~~~php
$chapters = ["Introduction"];
$chapters[] = "Variables";

var_dump(count($chapters));
var_dump($chapters[1]);
~~~

**Answer:**

~~~text
int(2)
string(9) "Variables"
~~~

**Explanation:** Appending creates the next integer key, which here is **1**.

### 33. What is the final value of the **theme** key?

~~~php
$settings = [
    "theme" => "light",
    "theme" => "dark",
];

var_dump($settings);
~~~

**Answer:**

~~~text
array(1) {
  ["theme"]=>
  string(4) "dark"
}
~~~

**Explanation:** The second value replaces the value for the duplicate key.

### 34. What do the two checks print?

~~~php
$profile = ["nickname" => null];

var_dump(isset($profile["nickname"]));
var_dump(array_key_exists("nickname", $profile));
~~~

**Answer:**

~~~text
bool(false)
bool(true)
~~~

**Explanation:** The key exists, but its value is null. **isset()** treats a null value as not set; **array_key_exists()** reports the key's presence.

### 35. What are the output and count after this code runs?

~~~php
$scores = [90, 85, 100];
unset($scores[1]);

var_dump($scores);
var_dump(count($scores));
~~~

**Answer:**

~~~text
array(2) {
  [0]=>
  int(90)
  [2]=>
  int(100)
}
int(2)
~~~

**Explanation:** The entry at key **1** is removed. The remaining keys stay **0** and **2**, and there are two entries.

### 36. Which keys are stored by this array literal?

~~~php
$codes = [
    "8" => "eight",
    "08" => "leading zero",
];
~~~

**Answer:** The first key is integer **8**. The second key remains string **"08"**.

**Explanation:** PHP converts a valid integer-like decimal string key without a leading zero. A leading-zero string is not converted by this rule.

## Write code

### 37. Create an indexed array of three subjects, then print the second subject.

**Answer:**

~~~php
$subjects = ["PHP", "HTML", "CSS"];

echo $subjects[1] . PHP_EOL;
~~~

**Output:**

~~~text
HTML
~~~

The first item is at key **0**, so the second item is at key **1**.

### 38. Create an associative array for a book and read its title and page count.

**Answer:**

~~~php
$book = [
    "title" => "Learning PHP",
    "pageCount" => 240,
];

echo $book["title"] . PHP_EOL;
echo $book["pageCount"] . PHP_EOL;
~~~

**Explanation:** The string keys name the fields, which makes each value's role clear.

### 39. Check whether an optional **nickname** key is present when null is a meaningful value.

**Answer:**

~~~php
$profile = [
    "nickname" => null,
];

if (array_key_exists("nickname", $profile)) {
    echo "The nickname key exists." . PHP_EOL;
}
~~~

**Explanation:** **array_key_exists()** is appropriate because the key can exist even if its value is null.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [015 - Strings and numeric strings](../lessons/015-strings-and-numeric-strings.md) | [Read this lesson](../lessons/016-indexed-and-associative-arrays.md) | [Open example](../examples/016-indexed-and-associative-arrays.md) | [017 - Objects, resources, callables, and iterables](../lessons/017-objects-resources-callables-and-iterables.md) |
