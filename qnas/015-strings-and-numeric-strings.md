# 015 - Strings and Numeric Strings: Questions and Answers

Use this Q&A to review [015 - Strings and Numeric Strings](../lessons/015-strings-and-numeric-strings.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What is a PHP string?

A. Only a whole number<br>
B. A sequence of bytes used to store text or other byte data<br>
C. A variable that must contain a URL<br>
D. A boolean condition

**Answer: B.** PHP strings are byte sequences. When storing text, an encoding such as UTF-8 gives those bytes a text representation.

### 2. Which statement about single-quoted strings is correct?

A. They always interpolate variables.<br>
B. They interpret every backslash escape sequence.<br>
C. They keep variables and most escape sequences literal.<br>
D. They cannot contain a single quote.

**Answer: C.** In a single-quoted string, PHP only treats an escaped single quote and escaped backslash specially. Variables are not expanded.

### 3. Which string prints the value of **$name**?

A. **'$name'**<br>
B. **"$name"**<br>
C. **'{$name}'**<br>
D. **'$name'**

**Answer: B.** Double-quoted strings interpolate variables. A single-quoted string preserves **$name** as text.

### 4. In a double-quoted PHP string, what does **\n** represent?

A. The two literal characters backslash and n in all cases<br>
B. A newline<br>
C. A tab<br>
D. A null value

**Answer: B.** In a double-quoted string, **\n** creates a line-feed character. In a single-quoted string, it remains literal text.

### 5. Which operator joins strings in PHP?

A. **+**<br>
B. **.**<br>
C. **&**<br>
D. **::**

**Answer: B.** The dot operator concatenates its left and right operands. The **+** operator is for arithmetic.

### 6. What does **.=** do?

A. Compares two strings<br>
B. Appends the right-hand string to the left-hand variable<br>
C. Converts a string to an integer<br>
D. Starts a heredoc

**Answer: B.** The concatenating assignment operator appends text to the existing string variable.

### 7. Which syntax creates a multiline string that can interpolate variables?

A. Nowdoc<br>
B. Heredoc<br>
C. A comment<br>
D. A single-quoted string only

**Answer: B.** Heredoc is a multiline string syntax that behaves like a double-quoted string for interpolation.

### 8. Which syntax keeps variable names literal inside a multiline string?

A. Heredoc<br>
B. Nowdoc<br>
C. Double quotes<br>
D. Concatenation

**Answer: B.** Nowdoc uses a quoted marker, such as **<<<'TEMPLATE'**, and does not interpolate variables.

### 9. What does **strlen()** return?

A. Always the number of user-perceived characters<br>
B. The number of bytes in the string<br>
C. The number of words in the string<br>
D. The number of Unicode grapheme clusters

**Answer: B.** **strlen()** counts bytes. A UTF-8 character can use multiple bytes.

### 10. What does **is_numeric("15")** return?

A. **true**<br>
B. **false**<br>
C. **15**<br>
D. **"15"**

**Answer: A.** The string contains a numeric value, so **is_numeric()** returns the boolean **true**. It does not convert the input or change its type.

### 11. What is the type of the value **"15"** before it is used in an arithmetic expression?

A. **int**<br>
B. **float**<br>
C. **string**<br>
D. **bool**

**Answer: C.** Quoted digits are a string. PHP may interpret a numeric string numerically in a numeric context, but the original variable remains a string unless reassigned or explicitly converted.

### 12. In PHP 8.0 and later, what does **is_numeric("42 ")** return?

A. **true**<br>
B. **false**<br>
C. **42**<br>
D. **null**

**Answer: A.** Since PHP 8.0, a numeric string may have trailing whitespace and still be recognized by **is_numeric()**. Earlier versions treated trailing whitespace differently.

### 13. What does **is_numeric("12 apples")** return?

A. **true**, because the string begins with digits<br>
B. **false**, because the whole string is not a numeric string<br>
C. The integer **12**<br>
D. A warning and the integer **12**

**Answer: B.** **is_numeric()** checks whether the value as a whole is numeric. A string with extra words is not a complete numeric string.

### 14. What is the result of this code on PHP 8?

~~~php
$value = "15";
$result = $value + 1;
var_dump($result);
~~~

A. **string(3) "151"**<br>
B. **int(16)**<br>
C. **float(15.1)**<br>
D. **bool(true)**

**Answer: B.** Addition is a numeric context. PHP interprets the numeric string **"15"** as an integer for this expression, so the result is **int(16)**.

### 15. Does adding a numeric string to a number automatically change the original string variable's type?

A. Yes, it becomes an integer immediately.<br>
B. Yes, it becomes a float in every case.<br>
C. No, the arithmetic expression interprets the value; the original variable remains a string.<br>
D. No, PHP cannot perform arithmetic with numeric strings.

**Answer: C.** A value can be interpreted numerically for one expression without changing the variable's stored value or type.

### 16. Which statement about **is_numeric()** is accurate?

A. It verifies every business rule for a form field.<br>
B. It determines whether a value is a number or numeric string.<br>
C. It always returns an integer.<br>
D. It sanitizes text for SQL.

**Answer: B.** **is_numeric()** checks whether a value is numeric. It does not enforce allowed ranges, integer-only requirements, or security rules.

## True or false

### 17. Variables expand inside ordinary single-quoted strings.

**Answer: False.** Single-quoted strings keep variable names literal. Use double quotes or concatenate the variable.

### 18. In a single-quoted string, **\n** normally creates a newline.

**Answer: False.** The sequence remains literal in a single-quoted string. Double-quoted strings interpret **\n** as a line feed.

### 19. The quotes used to delimit a string literal become part of the stored string.

**Answer: False.** The delimiter marks where the source literal begins and ends; it is not included in the value.

### 20. **.** and **.=** are PHP string operators.

**Answer: True.** **.** concatenates two values into a string result, while **.=** appends to a string variable.

### 21. Heredoc and nowdoc are both multiline string syntaxes.

**Answer: True.** Both use an opening and closing identifier. Heredoc interpolates variables; nowdoc preserves the body literally.

### 22. **strlen("café")** can return **4** because the visible word has four characters.

**Answer: False.** In UTF-8, the accented **é** commonly takes two bytes, so this text uses five bytes. **strlen()** counts those bytes.

### 23. **mb_strlen()** is available in every PHP installation without an extension.

**Answer: False.** **mb_strlen()** is provided by the mbstring extension, which must be available and enabled.

### 24. A value for which **is_numeric()** returns true is automatically valid as a positive whole-number quantity.

**Answer: False.** It may be a negative number, a decimal, or exponent notation. Validate the exact format and range the application needs.

### 25. Concatenating untrusted text makes it safe to insert into HTML or SQL.

**Answer: False.** Concatenation only joins values. HTML output needs context-appropriate escaping, while database queries should use parameterized APIs.

### 26. **"15"** is a numeric string and still has the PHP type **string** when assigned.

**Answer: True.** Its contents are numeric, but the quotes make the assigned value a string. A numeric operation may interpret it as a number for that operation.

## Short-answer questions

### 27. When should you choose single quotes rather than double quotes?

**Answer:** Choose single quotes when you want the text to remain literal and do not need variable interpolation or common escape sequences.

**Explanation:** For example, **'$name'** contains the characters dollar sign and name. The double-quoted string **"$name"** expands the variable.

### 28. How can you include a single quote inside a single-quoted string?

**Answer:** Escape it with a backslash.

~~~php
$sentence = 'It\'s a PHP lesson.';
~~~

The backslash tells PHP that the quote is part of the string rather than its closing delimiter.

### 29. How do you concatenate a first name and a last name with a space between them?

**Answer:**

~~~php
$fullName = $firstName . " " . $lastName;
~~~

**Explanation:** The dot joins the first name, a one-space string, and the last name.

### 30. What is the difference between heredoc and nowdoc?

**Answer:** Heredoc is a multiline string that can interpolate variables. Nowdoc uses a quoted closing identifier and keeps variables and escape sequences literal.

**Explanation:** Use heredoc for a multiline message with values inserted into it; use nowdoc for a literal template or code sample.

### 31. Why can **strlen()** be a poor character counter for UTF-8 text?

**Answer:** It counts bytes, and a single UTF-8 character can take multiple bytes.

**Explanation:** For encoding-aware character counts, use an appropriate multibyte function such as **mb_strlen()** when the mbstring extension is enabled. Grapheme counting can require additional Unicode-aware handling.

### 32. What does **is_numeric()** tell you, and what does it not tell you?

**Answer:** It tells you whether PHP recognizes a value as a number or numeric string. It does not tell you whether that value meets application rules.

**Explanation:** A numeric string could be a decimal or negative value when the field requires an integer between **1** and **100**.

### 33. Why can **"15" + 1** produce a number while the variable holding **"15"** remains a string?

**Answer:** PHP interprets the numeric string in the arithmetic expression's numeric context. That interpretation does not itself reassign or change the original variable.

**Explanation:** Inspect both the variable and the expression result with **var_dump()** to see their different types.

### 34. Why should concatenation not be treated as output escaping?

**Answer:** Concatenation combines values but does not encode or sanitize them for HTML, SQL, a shell command, or another output context.

**Explanation:** Each context has different safety rules. Use its appropriate escaping function or parameterized API.

## Read the code

### 35. What is printed?

~~~php
$name = "Asha";
echo 'Hello, $name.';
~~~

**Answer:**

~~~text
Hello, $name.
~~~

**Explanation:** The string uses single quotes, so PHP does not expand **$name**.

### 36. What is printed?

~~~php
$name = "Asha";
echo "Hello, {$name}!";
~~~

**Answer:**

~~~text
Hello, Asha!
~~~

**Explanation:** Double quotes allow variable interpolation. The braces mark the variable name clearly.

### 37. What does the following output show?

~~~php
$numberText = "15";
var_dump($numberText);
var_dump(is_numeric($numberText));
~~~

**Answer:**

~~~text
string(2) "15"
bool(true)
~~~

**Explanation:** The value is a string, and its contents are numeric. **is_numeric()** checks numeric interpretation; it does not change the type.

### 38. What is printed, and what is the type of each result?

~~~php
$left = "Hello";
$right = " PHP";
$joined = $left . $right;
$left .= "!";
var_dump($joined);
var_dump($left);
~~~

**Answer:**

~~~text
string(9) "Hello PHP"
string(6) "Hello!"
~~~

**Explanation:** **$joined** is formed by concatenation. **.=** appends an exclamation mark to **$left**.

### 39. If a UTF-8 file contains this code, what does it print?

~~~php
echo strlen("café");
~~~

**Answer:**

~~~text
5
~~~

**Explanation:** The visible text has four characters, but the accented character commonly occupies two UTF-8 bytes. **strlen()** counts bytes.

### 40. What is printed on PHP 8?

~~~php
$value = "15";
$result = $value + 1;

var_dump($value);
var_dump($result);
~~~

**Answer:**

~~~text
string(2) "15"
int(16)
~~~

**Explanation:** The arithmetic expression interprets the numeric string as an integer for the operation. The original assigned variable remains a string.

## Write code

### 41. Write code that prints a name with double-quoted interpolation and then with concatenation.

**Answer:**

~~~php
$name = "Asha";

echo "Hello, {$name}!" . PHP_EOL;
echo "Hello, " . $name . "!" . PHP_EOL;
~~~

**Output:**

~~~text
Hello, Asha!
Hello, Asha!
~~~

Both approaches can be clear for a simple message. Choose the form that makes the string easiest to read.

### 42. Write a heredoc greeting and a nowdoc template.

**Answer:**

~~~php
$name = "Asha";

$greeting = <<<GREETING
Hello, {$name}!
GREETING;

$template = <<<'TEMPLATE'
Hello, $name!
TEMPLATE;

echo $greeting;
echo $template;
~~~

**Output:**

~~~text
Hello, Asha!
Hello, $name!
~~~

The heredoc expands **$name**. The nowdoc keeps its variable-looking text literal.

### 43. Write a check that only reports whether a supplied value is numeric, without claiming it is fully valid as a quantity.

**Answer:**

~~~php
$submittedValue = "12";

if (is_numeric($submittedValue)) {
    echo "PHP recognizes a numeric value." . PHP_EOL;
} else {
    echo "The value is not numeric." . PHP_EOL;
}
~~~

**Explanation:** This checks numeric interpretation only. A real quantity field still needs checks for integer form, allowed range, and other application rules.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [014 - Null, booleans, integers, and floating-point numbers](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) | [Read this lesson](../lessons/015-strings-and-numeric-strings.md) | [Open example](../examples/015-strings-and-numeric-strings.md) | [016 - Indexed and associative arrays](../lessons/016-indexed-and-associative-arrays.md) |
