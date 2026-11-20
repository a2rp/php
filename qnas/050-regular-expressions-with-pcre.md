# 050 - Regular Expressions with PCRE: Q&A

Review PHP's PCRE patterns, delimiters, modifiers, character classes, anchors, groups, matching, replacement, splitting, and safe use of dynamic text. Try each question before reading its answer. See the [chapter lesson](../lessons/050-regular-expressions-with-pcre.md).

## Multiple-choice questions

### 1. Which PHP functions provide PCRE regular-expression operations?

A. **preg_*** functions
B. **array_*** functions
C. **file_*** functions
D. **str_*** functions only

**Answer:** A. **preg_*** functions.

**Explanation:** PHP's **preg_*** functions use PCRE2 for operations such as matching, replacing, and splitting text.

### 2. What must surround a PCRE pattern string in PHP?

A. Quotes only
B. Delimiters
C. Parentheses only
D. Curly braces only

**Answer:** B. Delimiters.

**Explanation:** A delimiter marks where the pattern begins and ends, such as **/pattern/** or **~pattern~**.

### 3. In **'/\d+/'**, what does **\d+** mean?

A. One or more digits
B. Exactly one digit
C. One or more letters
D. A literal backslash followed by d

**Answer:** A. One or more digits.

**Explanation:** **\d** matches a digit and **+** means one or more repetitions.

### 4. Which pattern position anchor means the absolute start of the subject?

A. **\A**
B. **\z**
C. **\b**
D. **\s**

**Answer:** A. **\A**

**Explanation:** **\A** anchors at the absolute beginning of the subject string. **\z** anchors at its absolute end.

### 5. What does **[A-Z]** match?

A. One uppercase ASCII letter
B. Any number from A to Z
C. A lowercase letter only
D. The literal text **A-Z**

**Answer:** A. One uppercase ASCII letter.

**Explanation:** Square brackets define a character class; this range matches one uppercase ASCII letter.

### 6. Which quantifier means “zero or more” of the previous token?

A. **+**
B. **?**
C. *****
D. **{2}**

**Answer:** C. *****

**Explanation:** **\*** allows zero or more repetitions. **+** means one or more, and **?** means zero or one.

### 7. What is the purpose of **(?:...)**?

A. A non-capturing group
B. A lookbehind that matches any text
C. A delimiter
D. A newline escape

**Answer:** A. A non-capturing group.

**Explanation:** It groups pattern alternatives or tokens without adding a captured group to the match results.

### 8. What does the **i** modifier do?

A. Makes matching case-insensitive
B. Enables multiline anchors
C. Makes the dot match newlines
D. Converts text to UTF-8

**Answer:** A. Makes matching case-insensitive.

**Explanation:** Modifiers follow the closing delimiter, as in **/php/i**.

### 9. What does the **m** modifier change?

A. It makes **^** and **$** work at line boundaries in multiline text.
B. It makes all matching Unicode-aware.
C. It makes **.** match every newline.
D. It disables captures.

**Answer:** A. It makes **^** and **$** work at line boundaries in multiline text.

**Explanation:** Without **m**, these anchors refer to the subject's start and end by default.

### 10. What does the **s** modifier change?

A. It lets **.** match newline characters.
B. It makes matching case-sensitive.
C. It causes **\s** to stop matching spaces.
D. It makes the pattern anchored.

**Answer:** A. It lets **.** match newline characters.

**Explanation:** Normally **.** does not match a newline. The **s** modifier changes that behavior.

### 11. What does the **u** modifier indicate?

A. Treat the pattern and subject as UTF-8 for matching
B. Convert every subject to UTF-8 automatically
C. Match uppercase letters only
D. Use Unicode normalization

**Answer:** A. Treat the pattern and subject as UTF-8 for matching.

**Explanation:** It does not convert or normalize text. The subject must be valid UTF-8 for Unicode-mode matching.

### 12. What does **preg_match()** return when the pattern matches?

A. **1**
B. **0**
C. **false**
D. The matched string directly

**Answer:** A. **1**

**Explanation:** It returns **1** for a match, **0** for no match, and **false** when the matching operation fails.

### 13. Which operator should be used to test a successful **preg_match()** result?

A. **== true**
B. **=== 1**
C. **!= 0**
D. **=== false**

**Answer:** B. **=== 1**

**Explanation:** A strict comparison distinguishes a match from no match and from an error result.

### 14. What does **preg_quote($literal, '~')** do?

A. Escapes regex metacharacters in the literal and the **~** delimiter
B. Converts the string to uppercase
C. Quotes a replacement string for **preg_replace()**
D. Performs a regex match

**Answer:** A. Escapes regex metacharacters in the literal and the **~** delimiter.

**Explanation:** Use it when dynamic text should be treated literally inside a pattern. It is not for escaping replacement strings.

### 15. Which function splits a string at regular-expression matches?

A. **preg_split()**
B. **preg_replace()**
C. **preg_match_all()**
D. **preg_quote()**

**Answer:** A. **preg_split()**

**Explanation:** It returns an array of pieces separated by matches of the pattern.

## Short-answer questions

### 16. Why are single-quoted PHP strings commonly used for regex patterns?

**Answer:** They preserve most backslashes instead of interpreting them as double-quoted PHP escape sequences.

**Explanation:** A regex already uses backslashes for constructs such as **\d** and **\A**, so single quotes usually make the pattern easier to read.

### 17. What do **\A** and **\z** mean?

**Answer:** **\A** matches only the absolute start of the subject, and **\z** matches only the absolute end.

**Explanation:** Use both when a whole input must match a format rather than merely contain a matching substring.

### 18. What is the difference between **^** and **\A** when the **m** modifier is used?

**Answer:** With **m**, **^** can match the beginning of each line; **\A** remains anchored to the absolute start of the subject.

**Explanation:** For whole-subject validation, absolute anchors make the intended boundary explicit.

### 19. What does a caret mean in **[^,]**?

**Answer:** When it is the first character inside a character class, it negates the class; this matches one character other than a comma.

**Explanation:** A caret elsewhere in the pattern or class can have a different meaning.

### 20. What is the difference between greedy and lazy quantifiers?

**Answer:** A greedy quantifier tries to consume as much text as possible while allowing the whole pattern to match. A lazy quantifier, formed by adding **?**, tries to consume as little as possible.

**Explanation:** For example, **.+** is greedy and **.+?** is lazy. Lazy matching does not turn a regex into a parser.

### 21. When should a non-capturing group be used?

**Answer:** Use **(?:...)** when parentheses are needed to group tokens or alternatives but the grouped text does not need to be captured.

**Explanation:** It avoids adding an unnecessary numbered capture to the matches array.

### 22. What is the difference between **preg_match()** and **preg_match_all()**?

**Answer:** **preg_match()** finds the first match; **preg_match_all()** finds every match and returns the number found.

**Explanation:** Both can populate a matches array. Their match counts and result structures differ according to how many occurrences exist.

### 23. What do the three possible **preg_match()** return outcomes mean?

**Answer:** **1** means matched, **0** means no match, and **false** means matching failed.

**Explanation:** Test with **=== 1** for a match and handle **0** and **false** separately when an invalid pattern or runtime error matters.

### 24. What is captured by **$matches[0]** after a successful **preg_match()**?

**Answer:** The complete text matched by the full pattern.

**Explanation:** Capturing groups are also placed at numeric indexes, and named groups can be accessed by their names.

### 25. What does the **x** modifier allow?

**Answer:** It allows whitespace and comments in a pattern outside character classes, making complex patterns easier to read.

**Explanation:** An unescaped **#** outside a character class begins a comment to the end of that pattern line.

### 26. How does **preg_replace()** differ from **str_replace()**?

**Answer:** **preg_replace()** replaces text matching a regular-expression pattern; **str_replace()** replaces literal strings.

**Explanation:** Prefer **str_replace()** for exact text. Use the regex function when pattern-based matching is actually needed.

### 27. When is **preg_replace_callback()** useful?

**Answer:** Use it when each match must be transformed by code, such as a calculation or conditional replacement.

**Explanation:** The callback receives match information and returns replacement text for that occurrence.

### 28. Why should a dynamic literal be passed through **preg_quote()** before being placed in a pattern?

**Answer:** To prevent characters in that value from being interpreted as regular-expression syntax.

**Explanation:** Characters such as **.**, **+**, **(**, and **[** have special meanings in patterns. Pass the delimiter too when one is used.

### 29. Why is a regular expression not a complete parser for formats such as HTML?

**Answer:** Nested and context-sensitive structure can exceed what a simple pattern safely describes.

**Explanation:** Use a parser designed for the format. Regex is useful for local patterns and text transformations, not every structured language.

## True or false

### 30. A PHP PCRE pattern must have delimiters.

**Answer:** True.

**Explanation:** The delimiters mark the start and end of the pattern; modifiers can follow the closing delimiter.

### 31. **\d+** means exactly one digit.

**Answer:** False.

**Explanation:** **\d** matches one digit and **+** means one or more occurrences.

### 32. **preg_match()** returns **false** when there is simply no match.

**Answer:** False.

**Explanation:** No match returns **0**. **false** signals a matching failure.

### 33. **preg_match() === 1** is a clear way to test for a match.

**Answer:** True.

**Explanation:** The strict check distinguishes the match result from zero and false.

### 34. The **s** modifier makes **.** match newline characters.

**Answer:** True.

**Explanation:** Without **s**, dot does not match newline by default.

### 35. The **u** modifier converts a non-UTF-8 subject to valid UTF-8.

**Answer:** False.

**Explanation:** It requests UTF-8 matching. It does not convert the input; invalid UTF-8 causes matching failure.

### 36. **preg_quote()** should be applied to replacement text passed to **preg_replace()**.

**Answer:** False.

**Explanation:** **preg_quote()** escapes text for use in a pattern, not for use as a replacement string.

### 37. **preg_split()** returns an array of substrings separated by pattern matches.

**Answer:** True.

**Explanation:** It is the regular-expression counterpart to simple delimiter splitting such as **explode()**.

## Predict the output

### 38. What does this print?

~~~php
<?php

$result = preg_match('/\d+/', 'Order 42');

echo $result;
~~~

**Answer:**

~~~text
1
~~~

**Explanation:** The subject contains one or more digits, so **preg_match()** returns **1**.

### 39. What does this print?

~~~php
<?php

$result = preg_match('/\Acat\z/', 'a cat');

var_dump($result);
~~~

**Answer:**

~~~text
int(0)
~~~

**Explanation:** The whole subject is not exactly **cat**, so the pattern does not match. **preg_match()** returns integer zero.

### 40. What does this print?

~~~php
<?php

preg_match('/\A(?<area>\d{3})-(?<line>\d{4})\z/', '555-0198', $matches);

echo $matches['area'] . ' / ' . $matches['line'];
~~~

**Answer:**

~~~text
555 / 0198
~~~

**Explanation:** The named capture groups store the digits before and after the hyphen.

### 41. What does this print?

~~~php
<?php

$text = "red\tgreen\nblue";
echo preg_replace('/\s+/', ' ', $text);
~~~

**Answer:**

~~~text
red green blue
~~~

**Explanation:** **\s+** matches each run of one or more whitespace characters, and the replacement is one space.

### 42. What does this print?

~~~php
<?php

$parts = preg_split('/\s*[,;]\s*/', 'red, green;blue');
echo implode('|', $parts);
~~~

**Answer:**

~~~text
red|green|blue
~~~

**Explanation:** The pattern splits on either delimiter and consumes optional spaces around it.

## Coding questions

### 43. Write a pattern and check that a product code is exactly two uppercase ASCII letters, a hyphen, and four digits.

**Answer:**

~~~php
<?php

$code = 'AB-2048';
$pattern = '/\A[A-Z]{2}-[0-9]{4}\z/';

if (preg_match($pattern, $code) === 1) {
    echo 'Valid product code';
} else {
    echo 'Invalid product code';
}
~~~

**Expected output:**

~~~text
Valid product code
~~~

**Explanation:** The anchors require the entire subject to match. The character classes and quantifiers define the required structure.

### 44. Find all integer sequences in a sentence and print them.

**Answer:**

~~~php
<?php

$text = 'There are 12 books and 3 notebooks.';
$count = preg_match_all('/\d+/', $text, $matches);

if ($count === false) {
    echo 'Matching failed';
} else {
    echo 'Matches: ' . implode(', ', $matches[0]);
}
~~~

**Expected output:**

~~~text
Matches: 12, 3
~~~

**Explanation:** **preg_match_all()** stores each full match in **$matches[0]** and returns the number of matches, or false on failure.

### 45. Search for a runtime phrase as literal text, even if it contains regex metacharacters.

**Answer:**

~~~php
<?php

$phrase = 'price: $5.00';
$text = 'Today price: $5.00 is shown.';

$pattern = '~' . preg_quote($phrase, '~') . '~';

if (preg_match($pattern, $text) === 1) {
    echo 'Found literal phrase';
} else {
    echo 'Not found';
}
~~~

**Expected output:**

~~~text
Found literal phrase
~~~

**Explanation:** **preg_quote()** makes the dollar sign and period literal in the pattern and escapes the chosen **~** delimiter if it occurs in the phrase.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [049 - Working with Arrays](../lessons/049-working-with-arrays.md) | [Read lesson](../lessons/050-regular-expressions-with-pcre.md) | [Open example](../examples/050-regular-expressions-with-pcre.md) | [051 - Dates, Times, and Time Zones](../lessons/051-dates-times-and-time-zones.md) |
