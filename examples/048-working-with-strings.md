# 048 - Working with Strings: Complete Example

This example puts PHP strings into one small command-line program. It demonstrates literal and interpolated strings, multiline strings, concatenation, everyday string functions, byte-aware versus multibyte text handling, and escaping text for an HTML context.

## Goal

Create a small report for a person named Asha. The program will:

- Show how single quotes, double quotes, heredoc, and nowdoc treat variables.
- Join a first and last name and append text.
- Clean, search, replace, split, and join strings.
- Compare the byte length of UTF-8 text with its character length when mbstring is enabled.
- Escape a string before placing it in HTML.

## Requirements

- PHP 8.0 or later for **str_contains()**, **str_starts_with()**, and **str_ends_with()**.
- The **mbstring** extension is optional. The program checks for it before using **mb_strlen()** and **mb_substr()**.
- Save the file as UTF-8 so the **é** in **café** is encoded as UTF-8.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

$name = 'Asha';

// Single-quoted strings keep the variable name literal.
$literalGreeting = 'Hello, $name';

// Double-quoted strings interpolate supported variables.
$interpolatedGreeting = "Hello, {$name}!";

// Heredoc is a multiline string that interpolates variables.
$notice = <<<NOTICE
Hello, {$name}.
This block interpolates the name.
NOTICE;

// Nowdoc keeps its contents literal.
$template = <<<'TEMPLATE'
Hello, $name.
This block stays literal.
TEMPLATE;

echo "Single-quoted: {$literalGreeting}\n";
echo "Double-quoted: {$interpolatedGreeting}\n";
echo "Heredoc:\n{$notice}\n";
echo "Nowdoc:\n{$template}\n";

// Join strings with a dot and append with dot-equals.
$firstName = 'Asha';
$lastName = 'Rao';
$fullName = $firstName . ' ' . $lastName;
$fullName .= ' (PHP learner)';

echo "Combined name: {$fullName}\n";

// Trim removes surrounding whitespace, not spaces between words.
$rawTitle = '  Working with PHP strings  ';
$title = trim($rawTitle);

echo "Trimmed title: {$title}\n";
echo 'Lowercase: ' . strtolower($title) . "\n";
echo 'Uppercase: ' . strtoupper($title) . "\n";

// Search helpers are case-sensitive.
$filename = 'strings-guide.md';

echo 'Contains a dot: ';
echo str_contains($filename, '.') ? "yes\n" : "no\n";

echo 'Starts with "strings": ';
echo str_starts_with($filename, 'strings') ? "yes\n" : "no\n";

echo 'Ends with ".md": ';
echo str_ends_with($filename, '.md') ? "yes\n" : "no\n";

// str_replace() returns a new string.
$sentence = 'I am learning HTML.';
$updatedSentence = str_replace('HTML', 'PHP', $sentence);

echo "Replaced text: {$updatedSentence}\n";

// explode() splits at a separator; implode() joins array items.
$tagText = 'html,css,php';
$tags = explode(',', $tagText);

echo 'Joined tags: ' . implode(' | ', $tags) . "\n";

// strpos() can return 0 for a match at the start. Compare strictly.
$description = 'PHP strings are byte sequences.';

if (strpos($description, 'PHP') !== false) {
    echo "strpos check: found PHP\n";
}

// strlen() counts bytes, so a UTF-8 character may count as more than one.
$word = 'café';
echo 'Byte length: ' . strlen($word) . "\n";

if (function_exists('mb_strlen')) {
    echo 'Character length: ' . mb_strlen($word, 'UTF-8') . "\n";
    echo 'First three characters: ' . mb_substr($word, 0, 3, 'UTF-8') . "\n";
} else {
    echo "Character-aware functions: enable the mbstring extension\n";
}

// Escape text for an HTML text context before inserting it into a page.
$userText = '<strong>Asha & friends</strong>';
$htmlText = htmlspecialchars(
    $userText,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);

echo "HTML-escaped text: {$htmlText}\n";
~~~

## Expected output

When mbstring is enabled:

~~~text
Single-quoted: Hello, $name
Double-quoted: Hello, Asha!
Heredoc:
Hello, Asha.
This block interpolates the name.
Nowdoc:
Hello, $name.
This block stays literal.
Combined name: Asha Rao (PHP learner)
Trimmed title: Working with PHP strings
Lowercase: working with php strings
Uppercase: WORKING WITH PHP STRINGS
Contains a dot: yes
Starts with "strings": yes
Ends with ".md": yes
Replaced text: I am learning PHP.
Joined tags: html | css | php
strpos check: found PHP
Byte length: 5
Character length: 4
First three characters: caf
HTML-escaped text: &lt;strong&gt;Asha &amp; friends&lt;/strong&gt;
~~~

If mbstring is not enabled, the three character-aware lines are replaced by:

~~~text
Character-aware functions: enable the mbstring extension
~~~

## How it works

### Literal and interpolated strings

The single-quoted **$literalGreeting** contains the characters **$name** literally. The double-quoted **$interpolatedGreeting** substitutes the value of the variable. Braces in **{$name}** mark the variable boundary.

The heredoc block also interpolates **$name**, while the quoted nowdoc identifier makes the template literal. These forms are especially useful when a string spans several lines.

### Concatenation

The **.** operator joins **$firstName**, a space, and **$lastName**. The **.=** operator then appends the learner label to **$fullName**. PHP does not use **+** to join strings.

### String helpers

**trim()** removes whitespace around the title. The case conversion functions change the basic English text in this example.

The PHP 8 search helpers return booleans and make common checks readable. They are case-sensitive. **str_replace()** returns the updated sentence; **$sentence** itself still contains its original value.

**explode()** turns the comma-separated text into an array. **implode()** then combines those items with a different separator. For real CSV input, use a CSV-aware parser because a quoted field may contain a comma.

### Searching safely with strpos()

**strpos()** returns the byte position of a match or **false** when there is no match. Since the word **PHP** is at the beginning, the matching position is **0**. The strict **!== false** comparison treats that position as a successful match.

### UTF-8 and length

In this UTF-8 source file, **café** uses five bytes: **c**, **a**, **f**, and the two-byte **é**. Therefore **strlen()** prints **5**. When mbstring is enabled, **mb_strlen()** counts four encoded characters and **mb_substr()** returns the first three without cutting a UTF-8 character in half.

If a feature must count user-perceived grapheme clusters—for example, a combined accent or emoji sequence—look at grapheme-aware functions from the **intl** extension. A code point count and a visible-character count are not always the same.

### HTML output

**htmlspecialchars()** converts markup delimiters and the ampersand into entities for an HTML text context. A browser displays the escaped value as literal text rather than creating a **strong** element from the input. HTML escaping is specific to HTML output; it is not a general-purpose sanitizer for JavaScript, CSS, URLs, or SQL.

## Try it yourself

1. Change the name and inspect the outputs from each string form.
2. Add a literal dollar sign to a double-quoted string by escaping it.
3. Change the filename and check how the three search helpers respond.
4. Replace a different word, then compare the original string with the returned value.
5. Add another tag to the array before calling **implode()**.
6. Try a UTF-8 word with more than one non-ASCII character and compare byte and character lengths.
7. Add a user-supplied apostrophe to **$userText** and observe the effect of **ENT_QUOTES**.

## Common mistakes

- Expecting single-quoted strings to substitute variables.
- Using **+** instead of **.** to join strings.
- Assuming **trim()** removes internal whitespace.
- Treating **strlen()** or **substr()** as character-aware for UTF-8.
- Checking **strpos()** with a loose truth test and missing a match at offset zero.
- Assuming HTML escaping is correct for every output context.
- Assuming comma splitting is sufficient to parse CSV.

## Key takeaways

- Choose single or double quotes based on whether interpolation and escapes are needed.
- Use heredoc for multiline interpolated text and nowdoc for multiline literal text.
- Join and append with **.** and **.=**.
- PHP string helpers return values; assign a result when you want to keep it.
- Distinguish byte length from encoded-character or grapheme length.
- Escape text for the specific context where it will be rendered.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [047 - Iterators, Traversable, and iterable Values](../lessons/047-iterators-traversable-and-iterable-values.md) | [Read lesson](../lessons/048-working-with-strings.md) | [Open Q&A](../qnas/048-working-with-strings.md) | [049 - Working with Arrays](../lessons/049-working-with-arrays.md) |
