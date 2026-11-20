# 050 - Regular Expressions with PCRE: Complete Example

This example uses PHP's PCRE functions to check a whole identifier, extract named groups, find repeated matches, normalize whitespace, transform matches, split text, and search for a runtime phrase literally.

## Goal

Build a small text-processing report that demonstrates when a regex is useful, how to read its results, and how to avoid treating dynamic text as pattern syntax.

## Requirements

- PHP 8.0 or later.
- Save the file as UTF-8 for the Unicode-letter example.
- No additional PHP extension is required for the examples shown.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

// Anchors require the complete subject to match.
// This identifier begins with an ASCII letter and is 3 to 16 characters long.
$usernamePattern = '/\A[A-Za-z][A-Za-z0-9_]{2,15}\z/';

foreach (['Asha_7', 'x!'] as $username) {
    $isValid = preg_match($usernamePattern, $username) === 1;

    echo 'Username ' . $username . ': ';
    echo $isValid ? "valid\n" : "invalid\n";
}

// Named groups extract parts of a shape-matched date.
$date = '2026-11-21';
$datePattern = '/\A(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})\z/';

if (preg_match($datePattern, $date, $dateMatches) === 1) {
    echo 'Date: '
        . $dateMatches['year'] . '-'
        . $dateMatches['month'] . '-'
        . $dateMatches['day']
        . ' (year ' . $dateMatches['year']
        . ', month ' . $dateMatches['month']
        . ', day ' . $dateMatches['day'] . ')' . "\n";
}

// Find every number. preg_match_all() returns the count or false on failure.
$textWithNumbers = 'Order 18, shelf 205, box 7';
$matchCount = preg_match_all('/\d+/', $textWithNumbers, $numberMatches);

if ($matchCount === false) {
    throw new RuntimeException('The number pattern could not be applied.');
}

echo 'Numbers found: ' . $matchCount . "\n";
echo 'Numbers: ' . implode(', ', $numberMatches[0]) . "\n";

// Replace repeated whitespace with one space, then trim the edges.
$rawText = "  PHP   has\tmany\nfeatures.  ";
$normalized = preg_replace('/\s+/', ' ', $rawText);

if ($normalized === null) {
    throw new RuntimeException('Whitespace replacement failed.');
}

echo 'Whitespace: ' . trim($normalized) . "\n";

// Capture the two parts and reorder them in the replacement.
$label = 'Patel, Asha';
$formattedLabel = preg_replace(
    '/\A([^,]+),\s*(.+)\z/',
    '$2 $1',
    $label
);

if ($formattedLabel === null) {
    throw new RuntimeException('Label replacement failed.');
}

echo 'Label: ' . $formattedLabel . "\n";

// Use a callback when each match needs a calculation.
$quantities = '2 apples and 3 pears';

$doubledQuantities = preg_replace_callback(
    '/\d+/',
    fn (array $matches): string => (string) ((int) $matches[0] * 2),
    $quantities
);

if ($doubledQuantities === null) {
    throw new RuntimeException('Quantity replacement failed.');
}

echo 'Doubled quantities: ' . $doubledQuantities . "\n";

// Split on a comma or semicolon, allowing spaces around the separator.
$listText = 'red, green; blue';
$list = preg_split('/\s*[,;]\s*/', $listText);

if ($list === false) {
    throw new RuntimeException('List splitting failed.');
}

echo 'Split values: ' . implode(' | ', $list) . "\n";

// Quote runtime text so punctuation is matched literally.
$literalPhrase = 'price: $5.00';
$literalPattern = '~' . preg_quote($literalPhrase, '~') . '~';
$sentence = 'Today price: $5.00 is listed.';

echo 'Literal search: ';
echo preg_match($literalPattern, $sentence) === 1 ? "found\n" : "not found\n";

// Modifiers change how a pattern is interpreted.
$caseInsensitivePattern = '/\Aphp\z/i';
echo 'Case-insensitive search: ';
echo preg_match($caseInsensitivePattern, 'PHP') === 1
    ? "found\n"
    : "not found\n";

$multilineText = "first line\nTitle: Arrays\nlast line";
$multilinePattern = '/^Title:/m';

echo 'Multiline heading: ';
echo preg_match($multilinePattern, $multilineText) === 1
    ? "found\n"
    : "not found\n";

// The u modifier enables UTF-8-aware matching for Unicode properties.
$word = 'café';
$unicodeLetterPattern = '/\A\p{L}+\z/u';

echo 'UTF-8 letters: ';
echo preg_match($unicodeLetterPattern, $word) === 1
    ? "matched\n"
    : "not matched\n";

// Greedy matching takes the longest result that allows the pattern to match.
// Lazy matching takes the shortest result that allows the pattern to match.
$twoTags = '<b>one</b><b>two</b>';

preg_match('/<b>.*<\/b>/', $twoTags, $greedyMatches);
preg_match('/<b>.*?<\/b>/', $twoTags, $lazyMatches);

echo 'Greedy match: ' . $greedyMatches[0] . "\n";
echo 'Lazy match: ' . $lazyMatches[0] . "\n";
~~~

## Expected output

~~~text
Username Asha_7: valid
Username x!: invalid
Date: 2026-11-21 (year 2026, month 11, day 21)
Numbers found: 3
Numbers: 18, 205, 7
Whitespace: PHP has many features.
Label: Asha Patel
Doubled quantities: 4 apples and 6 pears
Split values: red | green | blue
Literal search: found
Case-insensitive search: found
Multiline heading: found
UTF-8 letters: matched
Greedy match: <b>one</b><b>two</b>
Lazy match: <b>one</b>
~~~

## How it works

### Validate a whole value

The username pattern uses **\A** and **\z** so the entire input must match. Its first character must be an ASCII letter; the remaining two through fifteen characters must be an ASCII letter, digit, or underscore. Testing with **=== 1** distinguishes a match from no match or a regex execution failure.

The date pattern uses named capture groups. Those groups extract the year, month, and day from a string with the expected shape. A matching shape alone does not prove that the calendar date is valid.

### Extract and transform matches

**preg_match_all()** returns three for the three number sequences and stores each full match in **$numberMatches[0]**.

**preg_replace()** collapses whitespace and reorders the two captured parts of the label. The capture references **$2** and **$1** in the replacement are substituted with the second and first groups.

**preg_replace_callback()** runs code for each number match. Here the callback converts a matched quantity to an integer, doubles it, and returns the replacement string. The example checks for a null result in case the pattern operation fails.

### Split and search literally

The split pattern matches either a comma or semicolon, plus optional surrounding whitespace. **preg_split()** returns the items as an array.

The price phrase contains a dollar sign and periods, which have special meanings in regex syntax. **preg_quote()** escapes them, and the delimiter is supplied so it can be escaped too if it appears in the runtime value.

### Modifiers and quantifiers

The **i** modifier makes the PHP check case-insensitive. The **m** modifier lets **^** match the start of a line within a multiline subject.

In the final demonstration, the greedy **.* ** expression is intended to match as much text as possible before satisfying the closing tag. The lazy **.*?** expression tries to satisfy the closing tag at the first opportunity. This illustrates quantifier behavior on a simple string; regex is not an HTML parser.

The **u** modifier enables UTF-8 pattern matching, allowing **\p{L}** to match Unicode letters in the example. The source file and subject must be valid UTF-8; this modifier does not convert or normalize text.

## Try it yourself

1. Change the identifier pattern to require exactly two letters and four digits separated by a hyphen.
2. Add a named group to extract each part of that product code.
3. Find every decimal number in a sentence with **preg_match_all()**.
4. Change the whitespace replacement to replace each run with a dash.
5. Add a pipe character as another separator in the **preg_split()** pattern.
6. Search for a literal value containing **+** or parentheses using **preg_quote()**.
7. Compare the greedy and lazy captures after adding another pair of tags.
8. Explain why a pattern that matches a date shape still needs date validation.

## Common mistakes

- Leaving off the pattern delimiters.
- Forgetting that the PHP string parser runs before PCRE sees a double-quoted pattern.
- Using a truthy check that does not distinguish **0**, **1**, and **false**.
- Forgetting anchors when all input must satisfy the pattern.
- Building a pattern from user text without quoting it.
- Treating the **u** modifier as an encoding converter.
- Using a regular expression as a substitute for an HTML or other structured-data parser.

## Key takeaways

- Write patterns as delimited PHP strings and use modifiers deliberately.
- Anchor full-value checks with **\A** and **\z**.
- Capturing groups extract text; non-capturing groups organize patterns without capturing.
- Use **preg_match_all()**, **preg_replace()**, callbacks, and **preg_split()** for their distinct purposes.
- Quote runtime literal text with **preg_quote()** before inserting it into a pattern.
- A regex validates or transforms text patterns; it does not automatically validate a real-world semantic value.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [049 - Working with Arrays](../lessons/049-working-with-arrays.md) | [Read lesson](../lessons/050-regular-expressions-with-pcre.md) | [Open Q&A](../qnas/050-regular-expressions-with-pcre.md) | [051 - Dates, Times, and Time Zones](../lessons/051-dates-times-and-time-zones.md) |
