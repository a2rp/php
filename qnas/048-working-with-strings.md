# 048 - Working with Strings: Q&A

Review PHP string literals, interpolation, concatenation, common string functions, and the difference between bytes and text characters. Try each question before reading its answer. See the [chapter lesson](../lessons/048-working-with-strings.md).

## Multiple-choice questions

### 1. What is a PHP string at its basic level?

A. A sequence of bytes
B. A list of integers only
C. A special HTML element
D. A Unicode grapheme array

**Answer:** A. A sequence of bytes.

**Explanation:** PHP strings store bytes. They often represent text encoded as UTF-8, but PHP string operations do not automatically treat each visible character as one unit.

### 2. Which statement about single-quoted strings is correct?

A. Variables are always interpolated.
B. They support every double-quoted escape sequence.
C. Variables remain literal, and only quote and backslash have special escapes.
D. They cannot contain apostrophes.

**Answer:** C. Variables remain literal, and only quote and backslash have special escapes.

**Explanation:** In single-quoted strings, escape an apostrophe as **\'** and a backslash as **\\**. A variable name such as **$name** is not substituted.

### 3. Which string form interpolates variables?

A. Single-quoted only
B. Double-quoted
C. Both quoted forms identically
D. Neither quoted form

**Answer:** B. Double-quoted.

**Explanation:** Double-quoted strings interpolate supported variable expressions and recognize escape sequences. Single-quoted strings are mostly literal.

### 4. Why write **{$name}** when more text follows a variable?

A. It converts the value to an integer.
B. It clearly marks where the variable name ends.
C. It escapes the entire string for HTML.
D. It prevents the string from being allocated.

**Answer:** B. It clearly marks where the variable name ends.

**Explanation:** Braces avoid ambiguity. In **"{$name}Suffix"**, PHP knows the variable is **$name**, followed by the literal word **Suffix**.

### 5. What does heredoc support?

A. Variable interpolation, like a double-quoted string
B. No variables under any circumstances
C. Only numeric values
D. Automatic HTML escaping

**Answer:** A. Variable interpolation, like a double-quoted string.

**Explanation:** Heredoc is a multiline string syntax that behaves much like a double-quoted string for interpolation.

### 6. What is the key behavior of nowdoc?

A. It automatically joins array items.
B. It keeps its content literal and does not interpolate variables.
C. It can contain only one line.
D. It converts text to UTF-8.

**Answer:** B. It keeps its content literal and does not interpolate variables.

**Explanation:** A nowdoc opening identifier is quoted. The text is treated like literal single-quoted content.

### 7. Which operator joins two strings in PHP?

A. **+**
B. **.**
C. **&**
D. **=>**

**Answer:** B. **.**

**Explanation:** PHP uses the dot operator to concatenate strings. The plus operator is used for arithmetic, not string joining.

### 8. What does **.=** do?

A. Appends a value to the string variable on its left
B. Compares two strings loosely
C. Removes matching characters
D. Splits a string into an array

**Answer:** A. Appends a value to the string variable on its left.

**Explanation:** **$message .= '!'** is shorthand for assigning the old value plus the appended string back to **$message**.

### 9. What does **trim($value)** do by default?

A. Removes all spaces, including spaces between words
B. Removes common whitespace characters from the beginning and end
C. Replaces every tab with a space
D. Removes all punctuation

**Answer:** B. Removes common whitespace characters from the beginning and end.

**Explanation:** **trim()** operates on the edges of a string. It does not remove internal whitespace or perform general Unicode whitespace normalization.

### 10. What does **explode(',', $text)** return?

A. A string with commas removed
B. An array split at commas
C. A boolean indicating whether a comma exists
D. The number of commas

**Answer:** B. An array split at commas.

**Explanation:** **explode()** splits a string at a separator and returns an array of parts.

### 11. Which function joins array items into one string?

A. **implode()**
B. **explode()**
C. **substr()**
D. **trim()**

**Answer:** A. **implode()**

**Explanation:** **implode($separator, $parts)** combines array elements with the chosen separator.

### 12. What does **str_replace('old', 'new', $text)** return?

A. A modified copy with matching text replaced
B. The position of the first match
C. A list of every character
D. The original value, always unchanged in the result

**Answer:** A. A modified copy with matching text replaced.

**Explanation:** Assign the returned value if you want to keep the replaced result. The function does not change the original string variable in place.

### 13. In PHP 8 and later, what does **str_contains($text, $part)** check?

A. Whether **$text** contains **$part**, with a case-sensitive match
B. Whether both strings have the same byte length
C. Whether **$part** is valid HTML
D. Whether **$text** starts with **$part**

**Answer:** A. Whether **$text** contains **$part**, with a case-sensitive match.

**Explanation:** It returns **true** when the substring is found, including when it appears at position zero, and **false** otherwise.

### 14. What is important about **strpos()** when checking for a match?

A. It returns only **true** or **false**.
B. A match at the beginning returns **0**, so compare strictly with **false**.
C. It always returns a character count for Unicode text.
D. It changes the input string.

**Answer:** B. A match at the beginning returns **0**, so compare strictly with **false**.

**Explanation:** Use **strpos($text, $part) !== false**. A loose comparison can confuse the valid position **0** with **false**.

### 15. What does **strlen()** measure?

A. Bytes in the string
B. User-perceived grapheme clusters in every encoding
C. Words separated by spaces
D. Number of variables interpolated

**Answer:** A. Bytes in the string.

**Explanation:** In UTF-8, a visible character may use multiple bytes. Use **mb_strlen()** for encoded characters or grapheme functions for user-perceived grapheme clusters when those are the required units.

## Short-answer questions

### 16. Which escape sequences are special in a single-quoted PHP string?

**Answer:** The escaped apostrophe **\'** and escaped backslash **\\**.

**Explanation:** Other backslash sequences in a single-quoted string are generally kept as backslash plus the following character. Double-quoted strings support a broader set of escapes.

### 17. How do single-quoted and double-quoted strings differ for variables?

**Answer:** A variable such as **$name** remains literal in a single-quoted string. In a double-quoted string, a supported variable expression can be interpolated.

**Explanation:** Choose the form that makes the intent readable. Use concatenation or a formatting function if you need an arbitrary expression.

### 18. Give one reason to use braces around an interpolated variable.

**Answer:** Braces show exactly where the variable name ends, especially when literal text immediately follows it.

**Explanation:** For example, **"{$language}Guide"** refers to **$language** followed by **Guide**.

### 19. How do heredoc and nowdoc differ?

**Answer:** Heredoc interpolates variables like a double-quoted string; nowdoc leaves its contents literal like a single-quoted string.

**Explanation:** Heredoc uses an unquoted identifier. Nowdoc uses a quoted identifier in the opening syntax.

### 20. What is the difference between **.** and **.=**?

**Answer:** **.** creates a concatenated result. **.=** appends to and reassigns the variable on its left.

**Explanation:** **$full = $first . ' ' . $last** creates a combined value; **$message .= '!'** extends an existing value.

### 21. How does **substr()** choose a portion of a string?

**Answer:** It takes a starting byte offset and an optional byte length. A negative offset counts from the end.

**Explanation:** **substr()** is byte-oriented, so it can split a multibyte UTF-8 character. Use **mb_substr()** when the operation should follow encoded character boundaries.

### 22. How should a **strpos()** result be checked safely?

**Answer:** Compare it strictly with **false**, for example **strpos($text, $part) !== false**.

**Explanation:** A match at the first position returns integer **0**, which is a valid result and must not be mistaken for no match.

### 23. Are **str_contains()**, **str_starts_with()**, and **str_ends_with()** case-sensitive?

**Answer:** Yes.

**Explanation:** For example, checking whether **'PHP'** contains **'php'** returns **false**. Normalize text first if a case-insensitive comparison is required, using an appropriate function for its encoding.

### 24. Why does **trim()** not clean spaces between words?

**Answer:** By default, **trim()** removes matching characters only from the beginning and end of a string.

**Explanation:** To replace or normalize internal whitespace, use a different transformation appropriate to the text and requirements.

### 25. Name the three different units that may be relevant when measuring text.

**Answer:** Bytes, encoded characters (such as Unicode code points), and user-perceived grapheme clusters.

**Explanation:** They are not interchangeable. UTF-8 characters can use multiple bytes, and a visible grapheme can contain multiple code points.

### 26. Which PHP functions can count and slice multibyte text?

**Answer:** **mb_strlen()** counts encoded characters and **mb_substr()** extracts text by encoded character positions, when the mbstring extension is available.

**Explanation:** Use the intended encoding, commonly UTF-8. These functions do not necessarily count user-perceived grapheme clusters as one.

### 27. When might **grapheme_strlen()** be more appropriate than **mb_strlen()**?

**Answer:** When the requirement is to count user-perceived grapheme clusters, such as a visible character made from multiple code points.

**Explanation:** The **intl** extension provides grapheme-aware functions. Whether that distinction matters depends on the feature being built.

### 28. What does **htmlspecialchars()** do when used for HTML text output?

**Answer:** It converts special HTML characters so the value is displayed as text rather than parsed as markup.

**Explanation:** It is for an HTML output context, not a universal sanitizer. JavaScript, CSS, URLs, SQL, and other contexts need their own context-appropriate handling.

### 29. Why is **explode(',', $line)** not always a correct CSV parser?

**Answer:** A valid CSV field can contain a comma inside quotes, so splitting at every comma can break a field apart.

**Explanation:** Use a CSV-aware parser such as **str_getcsv()** for CSV data rather than assuming the delimiter never occurs inside a quoted field.

## True or false

### 30. A single-quoted string interpolates the value of **$name**.

**Answer:** False.

**Explanation:** The variable name remains literal in single-quoted strings.

### 31. PHP uses **+** to concatenate two strings.

**Answer:** False.

**Explanation:** PHP uses **.** to concatenate strings. **+** is an arithmetic operator.

### 32. By default, **trim()** removes whitespace between words.

**Answer:** False.

**Explanation:** It removes matching characters from the string's beginning and end.

### 33. For UTF-8 text, **strlen()** returns the number of visible characters.

**Answer:** False.

**Explanation:** It returns the number of bytes. A UTF-8 character can occupy more than one byte.

### 34. **substr()** and string offsets use byte positions.

**Answer:** True.

**Explanation:** They can split a multibyte UTF-8 character. Use a multibyte-aware function when character boundaries matter.

### 35. **str_contains()** treats uppercase and lowercase letters as equivalent.

**Answer:** False.

**Explanation:** The check is case-sensitive.

### 36. Variables inside nowdoc content are interpolated.

**Answer:** False.

**Explanation:** Nowdoc content stays literal.

### 37. HTML escaping with **htmlspecialchars()** is the correct escaping method for every output context.

**Answer:** False.

**Explanation:** Escaping must match the destination context. HTML escaping is not JavaScript, CSS, URL, or SQL escaping.

## Predict the output

### 38. What does this print?

~~~php
<?php

$name = 'Maya';
echo 'Hello, $name';
~~~

**Answer:**

~~~text
Hello, $name
~~~

**Explanation:** Single-quoted strings do not interpolate variables.

### 39. What does this print?

~~~php
<?php

$name = 'Maya';
echo "Hello, {$name}!";
~~~

**Answer:**

~~~text
Hello, Maya!
~~~

**Explanation:** Double-quoted strings interpolate the braced variable.

### 40. What does this print?

~~~php
<?php

$message = 'Hello';
$message .= ', PHP';
echo $message;
~~~

**Answer:**

~~~text
Hello, PHP
~~~

**Explanation:** **.=** appends the right-hand string to the existing variable.

### 41. What does this print?

~~~php
<?php

$parts = explode('-', 'PHP-048');
echo implode(' ', $parts);
~~~

**Answer:**

~~~text
PHP 048
~~~

**Explanation:** **explode()** splits at the hyphen; **implode()** joins the pieces with a space.

### 42. What does this print?

~~~php
<?php

$text = 'PHP strings';
var_dump(strpos($text, 'PHP') !== false);
~~~

**Answer:**

~~~text
bool(true)
~~~

**Explanation:** **strpos()** finds the text at byte offset zero. The strict comparison correctly treats zero as a found position.

## Coding questions

### 43. Write a function that trims a name and returns it with a greeting prefix.

**Answer:**

~~~php
<?php

function greetingFor(string $name): string
{
    $cleanName = trim($name);

    return 'Hello, ' . $cleanName . '!';
}

echo greetingFor('  Maya  ');
~~~

**Explanation:** **trim()** removes surrounding whitespace, and **.** joins the strings. The function returns a string rather than printing from inside the transformation.

### 44. Show how to count and take the first three encoded characters of UTF-8 text.

**Answer:**

~~~php
<?php

$text = 'café';

echo mb_strlen($text, 'UTF-8');
echo "\n";
echo mb_substr($text, 0, 3, 'UTF-8');
~~~

**Expected output:**

~~~text
4
caf
~~~

**Explanation:** The mbstring extension must be available. These functions count and slice encoded characters rather than raw bytes; grapheme-aware handling may be needed for user-perceived character boundaries.

### 45. Display untrusted text as HTML text instead of interpreting it as markup.

**Answer:**

~~~php
<?php

$userText = '<b>Hello</b>';

echo htmlspecialchars(
    $userText,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
~~~

**Expected output:**

~~~text
&lt;b&gt;Hello&lt;/b&gt;
~~~

**Explanation:** In an HTML text context, the browser displays the tag characters instead of creating a bold element. Use escaping appropriate to the exact output context.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [047 - Iterators, Traversable, and iterable Values](../lessons/047-iterators-traversable-and-iterable-values.md) | [Read lesson](../lessons/048-working-with-strings.md) | [Open example](../examples/048-working-with-strings.md) | [049 - Working with Arrays](../lessons/049-working-with-arrays.md) |
