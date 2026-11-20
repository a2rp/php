# 028 - String, Array, Bitwise, and Other Operators: Questions and Answers

Use this Q&A to review [028 - String, Array, Bitwise, and Other Operators](../lessons/028-string-array-bitwise-and-other-operators.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which operator joins strings in PHP?

A. **+**<br>
B. **.**<br>
C. **&**<br>
D. **|**

**Answer: B.** The dot operator concatenates string values.

### 2. What does **.=** do?

A. Compares two strings strictly<br>
B. Appends a value to a string variable and stores the result<br>
C. Performs numeric division<br>
D. Combines arrays by key

**Answer: B.** Concatenating assignment appends to the current string and updates that variable.

### 3. When both operands are arrays, what does **+** do?

A. Appends all values and renumbers every key<br>
B. Creates an array union by key, with the left value winning on duplicate keys<br>
C. Sorts both arrays<br>
D. Compares the arrays

**Answer: B.** Array union adds keys from the right that are not already present on the left.

### 4. In an array union, which value is kept when both arrays contain the same key?

A. The left array's value<br>
B. The right array's value<br>
C. Both values are kept automatically<br>
D. Neither value is kept

**Answer: A.** PHP preserves the value from the left-hand array for duplicate keys.

### 5. What happens with overlapping numeric keys in array union?

A. PHP always renumbers all keys<br>
B. The left array's values win for matching keys; only new keys are added<br>
C. The right array always wins<br>
D. The operation throws an error

**Answer: B.** Array union uses keys and does not automatically append or renumber list values.

### 6. Which function is commonly used when list values should be appended and numeric keys renumbered?

A. **array_merge()**<br>
B. **array_union()**<br>
C. **array_sort()**<br>
D. **array_equal()**

**Answer: A.** array_merge() has list-merging behavior that differs from the **+** array union operator.

### 7. What is the result of bitwise **6 & 3**?

A. **1**<br>
B. **2**<br>
C. **5**<br>
D. **7**

**Answer: B.** In binary, **6** is **0110** and **3** is **0011**; only the **0010** bit is set in both.

### 8. What is the result of bitwise **6 | 3**?

A. **2**<br>
B. **5**<br>
C. **7**<br>
D. **9**

**Answer: C.** Bitwise OR sets each bit that appears in either operand, producing binary **0111**, which is **7**.

### 9. What is the result of bitwise **6 ^ 3**?

A. **2**<br>
B. **5**<br>
C. **6**<br>
D. **7**

**Answer: B.** Bitwise XOR sets bits that appear in exactly one operand, producing binary **0101**, which is **5**.

### 10. Which pair distinguishes bitwise XOR from logical XOR?

A. **^** is bitwise; **xor** is logical<br>
B. **xor** is bitwise; **^** is logical<br>
C. They are identical spellings<br>
D. Neither operator exists in PHP

**Answer: A.** The caret works on bits; the word **xor** combines truth values.

### 11. Which statement about **~** and **!** is correct?

A. Both are logical NOT operators.<br>
B. **~** inverts bits; **!** reverses a truth value.<br>
C. **!** inverts bits; **~** concatenates strings.<br>
D. Both append values to arrays.

**Answer: B.** Bitwise NOT and logical NOT act on different kinds of values.

### 12. What is **5 << 1** for this small positive integer?

A. **2**<br>
B. **5**<br>
C. **10**<br>
D. **25**

**Answer: C.** A left shift by one moves the bits one place left, giving **10**.

### 13. What does **instanceof** check?

A. Whether a string contains a substring<br>
B. Whether an object belongs to a class hierarchy or implements an interface<br>
C. Whether an integer is even<br>
D. Whether two arrays have the same values

**Answer: B.** The type operator returns a boolean for an object and a class or interface check.

### 14. What is the safest way to compare a bitmask check with zero?

A. **$flags & READ !== 0**<br>
B. **($flags & READ) !== 0**<br>
C. **$flags && READ !== 0**<br>
D. **$flags | READ === 0**

**Answer: B.** Parentheses make the bitwise AND happen before the comparison.

### 15. Why should shell execution through PHP's execution operator be avoided with untrusted input?

A. It always sorts the output.<br>
B. It can expose the application to command injection and portability problems.<br>
C. It converts every value to an array.<br>
D. It disables strict types.

**Answer: B.** Shell commands interpret their input; untrusted command text can become executable shell syntax.

## Short-answer questions

### 16. What does the concatenation operator **.** do?

**Answer:** It joins values as a string and returns the combined string.

**Explanation:** For example, **"PHP" . " notes"** produces **"PHP notes"**.

### 17. What should you do when concatenation and arithmetic appear together?

**Answer:** Add parentheses around the arithmetic or calculate it in a separate variable.

**Explanation:** This makes the intended grouping explicit, especially because PHP 8 changed the precedence of **.** relative to arithmetic.

### 18. What does **$message .= "!"** do?

**Answer:** It appends **"!"** to the current value of **$message** and stores the updated string there.

**Explanation:** It is the concatenation assignment operator.

### 19. Does array union **+** combine array values or array keys?

**Answer:** It combines arrays by key.

**Explanation:** A right-side element is added only if its key is not already present on the left.

### 20. Which duplicate value wins in **$left + $right**?

**Answer:** The value from **$left**.

**Explanation:** Array union is left-biased for keys present in both arrays.

### 21. How do **==** and **===** compare arrays differently?

**Answer:** **==** checks for the same key/value pairs; **===** also requires the same order and value types.

**Explanation:** Two arrays can compare equal while not being identical.

### 22. What is a bitmask?

**Answer:** An integer that uses separate bits to represent separate flags.

**Explanation:** Bitwise OR can combine flags; bitwise AND can test whether a flag is present.

### 23. How can you test whether a flag is present in **$permissions**?

**Answer:** Use **($permissions & FLAG) !== 0**.

**Explanation:** The bitwise AND keeps only bits common to the permissions and the flag.

### 24. How can a flag be removed from a bitmask?

**Answer:** Use **$permissions &= ~FLAG**.

**Explanation:** Bitwise NOT inverts the flag bits, and bitwise AND clears the matching bit in the stored permissions.

### 25. What happens if both operands of bitwise **&**, **|**, or **^** are strings?

**Answer:** PHP applies the operation byte by byte and returns a string.

**Explanation:** This is byte-level behavior, not Unicode character or user-visible text processing.

### 26. What can **instanceof** check besides a concrete class?

**Answer:** It can check whether an object implements an interface.

**Explanation:** It also recognizes a parent class in the object's inheritance hierarchy.

### 27. What does the error-control operator **@** do?

**Answer:** It suppresses certain diagnostics from the expression that follows it.

**Explanation:** It can hide useful information and does not repair the underlying error, so explicit handling is usually clearer.

### 28. What does PHP's shell-execution operator do?

**Answer:** It runs a command through the system shell and returns command output.

**Explanation:** Avoid it with untrusted input because shell interpretation creates security and portability risks.

### 29. How is array append syntax **$items[] = $value** different from array union?

**Answer:** It appends a value to the end of an array; array union adds values by key and keeps the left value for duplicate keys.

**Explanation:** Use each form for its specific behavior.

## True or false

### 30. The plus operator joins strings in PHP.

**Answer: False.** The dot operator concatenates strings; plus performs arithmetic for numeric values.

### 31. In PHP 8 and later, parentheses make mixed concatenation and arithmetic clearer.

**Answer: True.** Parentheses explicitly show which calculation should happen before concatenation.

### 32. Array union always keeps the right-hand value for duplicate keys.

**Answer: False.** The left-hand value wins.

### 33. Array union automatically renumbers all numeric keys.

**Answer: False.** It preserves keys and adds only keys not already present.

### 34. The bitwise operator **&** has the same purpose as logical **&&**.

**Answer: False.** **&** works on bits; **&&** combines truth values.

### 35. Bitwise XOR is written **^**, while logical XOR is written **xor**.

**Answer: True.** They are separate operators with different operand behavior.

### 36. The **instanceof** operator returns a boolean.

**Answer: True.** It reports whether an object matches a class or interface check.

### 37. The error-control operator **@** is a substitute for error handling.

**Answer: False.** It suppresses diagnostics; it does not handle the failure or explain how the program should recover.

## Read the code

### 38. What does this code print?

~~~php
<?php
$first = "Ada";
$last = "Lovelace";

echo $first . " " . $last . PHP_EOL;
~~~

**Answer:**

~~~text
Ada Lovelace
~~~

**Explanation:** The dot operators join the first name, a space, and the last name.

### 39. What does this code print?

~~~php
<?php
$defaults = ["theme" => "light"];
$user = ["theme" => "dark", "language" => "en"];
$settings = $defaults + $user;

echo $settings["theme"] . PHP_EOL;
echo $settings["language"] . PHP_EOL;
~~~

**Answer:**

~~~text
light
en
~~~

**Explanation:** The left array wins for the duplicate **theme** key. The right array contributes the new **language** key.

### 40. What does this code print?

~~~php
<?php
$first = ["color" => "blue", "size" => "M"];
$reordered = ["size" => "M", "color" => "blue"];

var_dump($first == $reordered);
var_dump($first === $reordered);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Loose array equality ignores element order when the key/value pairs match. Strict array identity also requires the same order and value types.

### 41. What does this code print?

~~~php
<?php
$first = 6;
$second = 3;

var_dump($first & $second);
var_dump($first | $second);
var_dump($first ^ $second);
~~~

**Answer:**

~~~text
int(2)
int(7)
int(5)
~~~

**Explanation:** AND keeps shared bits, OR keeps bits set in either operand, and XOR keeps bits set in exactly one operand.

### 42. What does this code print?

~~~php
<?php
$items = new ArrayObject();

var_dump($items instanceof ArrayObject);
var_dump($items instanceof stdClass);
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** The object is an **ArrayObject**, not a **stdClass** instance.

## Write code

### 43. Join **$firstName** and **$lastName** with a space, then append an exclamation mark to the result.

**Answer:**

~~~php
<?php
$greeting = $firstName . " " . $lastName;
$greeting .= "!";
~~~

**Explanation:** The dot joins strings; **.=** updates the existing string.

### 44. Define read and write permission flags, combine both permissions, check for write access, and remove the write permission.

**Answer:**

~~~php
<?php
const CAN_READ = 1;
const CAN_WRITE = 2;

$permissions = CAN_READ | CAN_WRITE;
$mayWrite = ($permissions & CAN_WRITE) !== 0;
$permissions &= ~CAN_WRITE;
~~~

**Explanation:** OR combines flags, AND checks for a flag, and AND with the inverted flag mask clears it.

### 45. Check whether **$document** is an instance of **Exportable**.

**Answer:**

~~~php
<?php
$isExportable = $document instanceof Exportable;
~~~

**Explanation:** **instanceof** returns a boolean and works with interfaces as well as classes.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [027 - Logical Operators and Short-Circuit Evaluation](../lessons/027-logical-operators-and-short-circuit-evaluation.md) | [Read this lesson](../lessons/028-string-array-bitwise-and-other-operators.md) | [Open example](../examples/028-string-array-bitwise-and-other-operators.md) | [029 - Ternary, Null Coalescing, and Nullsafe Operators](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) |