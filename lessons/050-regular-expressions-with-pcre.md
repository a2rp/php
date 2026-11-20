# 050 - Regular Expressions with PCRE

A **regular expression** describes a pattern for finding or transforming text. PHP provides the **preg_*** functions, powered by PCRE2, for matching, extracting, replacing, and splitting strings.

Regular expressions are useful when the text follows a pattern. For exact literal search or replacement, ordinary string functions are often clearer.

## What you will learn

- Read a delimited PCRE pattern and choose common modifiers.
- Use character classes, anchors, quantifiers, groups, and alternation.
- Match once or find every match with **preg_match()** and **preg_match_all()**.
- Replace text with **preg_replace()** and **preg_replace_callback()**.
- Split text with **preg_split()** and quote dynamic literal text with **preg_quote()**.
- Distinguish no match from an invalid pattern and use Unicode mode deliberately.

## Pattern structure

A PHP PCRE pattern has a delimiter at the beginning and end. The pattern content goes between them; modifiers follow the closing delimiter:

~~~php
<?php

$pattern = '~\Aphp\z~i';
~~~

Here **~** is the delimiter, **\A** and **\z** anchor the match to the whole subject, and **i** makes the match case-insensitive. Forward slashes are common delimiters too. Choose a delimiter that does not occur often in the pattern, or escape it when it does.

Patterns are PHP strings as well as regular expressions. A single-quoted PHP string usually makes backslashes easier to read:

~~~php
<?php

$pattern = '/\d+/';
~~~

In a double-quoted PHP string, backslashes may first be interpreted by PHP before PCRE sees them. Use single quotes for most pattern literals unless interpolation is needed.

## Common pattern building blocks

### Character classes

Square brackets match one character from a set. A caret immediately after the opening bracket negates the set.

| Pattern | Meaning |
| --- | --- |
| **[aeiou]** | One listed vowel |
| **[A-Z]** | One uppercase ASCII letter |
| **[0-9]** | One digit |
| **[^,]** | One character other than a comma |

A hyphen indicates a range when placed between characters in a class. Put it first or last, or escape it, when you mean a literal hyphen.

Common shorthand classes include **\d** for a digit, **\s** for whitespace, and **\w** for a word character. For patterns that must be limited to ASCII, an explicit class such as **[A-Za-z0-9_]** makes that intention clear. Unicode character properties such as **\p{L}** can match letters across scripts when the pattern uses UTF-8 mode.

### Quantifiers

Quantifiers specify how many times the previous token may appear:

| Quantifier | Meaning |
| --- | --- |
| **?** | Zero or one |
| ***** | Zero or more |
| **+** | One or more |
| **{3}** | Exactly three |
| **{2,5}** | From two through five |
| **{2,}** | At least two |

By default, quantifiers are **greedy**: they try to consume as much text as possible while allowing the full pattern to match. Add **?** after a quantifier to make it **lazy**, so it tries to consume as little as possible.

For example, **<.+>** can match from the first **<** to the last **>** on a line. **<.+?>** asks for the shortest match that satisfies the pattern. Lazy matching is not a substitute for a parser when the input is structured markup.

### Anchors and boundaries

Anchors match positions rather than characters:

- **\A** matches the absolute start of the subject.
- **\z** matches the absolute end of the subject.
- **^** and **$** match the start and end of the subject by default. With the **m** modifier, they match line boundaries.
- **\b** matches a word boundary.

For validating a complete identifier, use **\A** and **\z** so a valid substring inside longer text does not count as a full match:

~~~php
<?php

$usernamePattern = '/\A[A-Za-z][A-Za-z0-9_]{2,15}\z/';
~~~

This example allows an ASCII letter first, followed by two to fifteen ASCII letters, digits, or underscores. It therefore permits a total length of three through sixteen characters.

### Groups and alternation

Parentheses group parts of a pattern and capture matched text. A vertical bar means “or”:

~~~php
<?php

$pattern = '/\A(cat|dog)\z/';
~~~

This matches exactly **cat** or **dog**. A capturing group is available in the matches array. Use **(?:...)** when grouping is needed but the group does not need to be captured.

Named groups make extracted values easier to identify:

~~~php
<?php

$datePattern = '/\A(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})\z/';
~~~

A matching group is available by its name and numeric position in the matches array. A shape-matching regular expression only confirms the text's format; it does not prove a calendar date is valid.

## Modifiers

Modifiers change how the pattern is interpreted. Common ones include:

| Modifier | Effect |
| --- | --- |
| **i** | Case-insensitive matching |
| **m** | Make **^** and **$** match line boundaries |
| **s** | Let **.** match newline characters too |
| **x** | Ignore pattern whitespace outside character classes and allow pattern comments |
| **u** | Treat the pattern and subject as UTF-8 |

Modifiers go after the closing delimiter:

~~~php
<?php

$caseInsensitive = '/\Aphp\z/i';
$multiline = '/^Title:/m';
$utf8Letters = '/\A\p{L}+\z/u';
~~~

Use **u** for UTF-8 text patterns and subjects. The subject must contain valid UTF-8 for Unicode-mode matching. PCRE modifiers affect matching behavior; they do not convert text encodings or make an arbitrary regex a complete validator.

With **x**, whitespace in the pattern is ignored outside character classes. An unescaped **#** outside a character class begins a comment through the end of that line.

## Match once with preg_match()

**preg_match()** searches for the first match. It returns **1** when it finds a match, **0** when there is no match, and **false** if matching fails, such as when the pattern is invalid. Compare its result strictly:

~~~php
<?php

$emailLike = 'asha@example.test';
$pattern = '/\A[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\z/';

$result = preg_match($pattern, $emailLike);

if ($result === 1) {
    echo 'Pattern matched';
} elseif ($result === 0) {
    echo 'No match';
} else {
    echo 'Regex execution failed';
}
~~~

The pattern above checks a simple email-like shape; it is not a complete email-address validator. Use a purpose-built validation facility when validating a real email address.

Pass a matches array to retrieve the full match and captured groups:

~~~php
<?php

$pattern = '/\A(?<area>\d{3})-(?<number>\d{4})\z/';
$phonePart = '555-0198';

if (preg_match($pattern, $phonePart, $matches) === 1) {
    echo $matches['area'];
    echo $matches['number'];
}
~~~

By default, **$matches[0]** contains the entire match, and captured groups are also available by their numeric indexes. Named groups are available by name.

## Find all matches with preg_match_all()

**preg_match_all()** returns the number of matches, or **false** if the operation fails. Its matches array contains the matches found:

~~~php
<?php

$text = 'Order 18, shelf 205, box 7';
$count = preg_match_all('/\d+/', $text, $matches);

echo $count;
print_r($matches[0]);
~~~

The default result groups captured values by pattern position. Use the matches array only when the function returns a positive match count.

## Replace with preg_replace()

**preg_replace()** replaces text that matches a pattern and returns the resulting string:

~~~php
<?php

$message = "PHP   has\tmany\nfeatures.";
$clean = preg_replace('/\s+/', ' ', $message);

echo $clean;
~~~

The pattern **\s+** matches one or more whitespace characters, so repeated whitespace is replaced with one space. For plain exact-text replacement, prefer **str_replace()** because it communicates the intent more directly.

Captured groups can be reused in a replacement string:

~~~php
<?php

$label = 'Last, First';
$formatted = preg_replace(
    '/\A([^,]+),\s*(.+)\z/',
    '$2 $1',
    $label
);

echo $formatted;
~~~

When the replacement needs calculations or conditional logic, use **preg_replace_callback()**:

~~~php
<?php

$text = '2 apples and 3 pears';

$updated = preg_replace_callback(
    '/\d+/',
    fn (array $matches): string => (string) ((int) $matches[0] * 2),
    $text
);

echo $updated;
~~~

## Split with preg_split()

**preg_split()** splits a string at every pattern match and returns an array:

~~~php
<?php

$line = 'red, green; blue';
$parts = preg_split('/\s*[,;]\s*/', $line);

print_r($parts);
~~~

This pattern splits on either a comma or semicolon, including optional surrounding whitespace. If the separator is a fixed literal string, **explode()** may be simpler.

## Quote dynamic literal text with preg_quote()

If a runtime value must be searched literally, quote it before inserting it into a pattern. Otherwise, characters such as **.**, **+**, and **(** will be treated as regex syntax:

~~~php
<?php

$literalSearch = 'price: $5.00';
$pattern = '~' . preg_quote($literalSearch, '~') . '~';

$text = 'The price: $5.00 is listed.';
echo preg_match($pattern, $text) === 1 ? 'Found' : 'Not found';
~~~

Pass the delimiter as the second argument so **preg_quote()** also escapes that delimiter. Do not use **preg_quote()** on a replacement string; it is for pattern text.

## Unicode and offsets

The **u** modifier makes PCRE interpret the pattern and subject as UTF-8. It does not normalize Unicode text, convert encodings, or guarantee that an offset is a user-perceived character index. PCRE offsets are byte offsets.

For Unicode letters, **\p{L}** can be useful in a **u** pattern. For example, **/\A\p{L}+\z/u** matches a string containing one or more Unicode letters. It does not accept spaces, apostrophes, or hyphens unless the pattern is extended to allow them.

## Safer pattern design

- Prefer **str_contains()**, **str_starts_with()**, **str_ends_with()**, or **str_replace()** for simple literal tasks.
- Keep validation patterns anchored when the entire input must match.
- Build patterns from fixed syntax; use **preg_quote()** for user-supplied text that should be literal.
- Limit the input length when the application has a reasonable maximum.
- Avoid needlessly ambiguous nested repetition, which can cause excessive backtracking on long inputs.
- Treat a **false** result as a matching error, not as an ordinary “does not match” result.

A regular expression checks text shape. It does not replace parsing for complex structured formats such as HTML, JSON, or CSV.

## Common mistakes

- Forgetting that PHP patterns require delimiters.
- Using double-quoted PHP strings and accidentally changing backslashes before PCRE receives them.
- Checking **preg_match()** with a loose truth test instead of distinguishing **1**, **0**, and **false**.
- Forgetting anchors when the whole subject must satisfy a format.
- Treating **.** as matching newline without the **s** modifier.
- Assuming **u** converts non-UTF-8 input into UTF-8.
- Concatenating untrusted text into a pattern without **preg_quote()**.
- Using a regex where a simple string function is clearer.

## Try it yourself

1. Match a product code with two uppercase letters, a hyphen, and four digits.
2. Extract the area and local parts from a hyphen-separated phone-like value with capture groups.
3. Find every integer in a sentence with **preg_match_all()**.
4. Replace runs of tabs and spaces with one space.
5. Split a list on commas or semicolons with **preg_split()**.
6. Search for a user-provided phrase literally by using **preg_quote()**.
7. Add the **i** modifier and compare case-sensitive and case-insensitive matching.
8. Explain why matching an email-like shape is not the same as complete email validation.

## Key takeaways

- PCRE patterns are delimited PHP strings, and modifiers follow the closing delimiter.
- Character classes, quantifiers, anchors, and groups express reusable text patterns.
- **preg_match()** returns **1**, **0**, or **false**; handle all three outcomes.
- Use **preg_match_all()**, **preg_replace()**, **preg_replace_callback()**, and **preg_split()** for their distinct jobs.
- Use **preg_quote()** when dynamic text must be treated literally inside a pattern.
- Use UTF-8 mode deliberately and remember that offsets are bytes.
- Prefer simpler string functions when no pattern matching is needed.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [049 - Working with Arrays](049-working-with-arrays.md) | [Open Q&A](../qnas/050-regular-expressions-with-pcre.md) | [Open example](../examples/050-regular-expressions-with-pcre.md) | [051 - Dates, Times, and Time Zones](051-dates-times-and-time-zones.md) |
