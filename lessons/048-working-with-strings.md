# 048 - Working with Strings

A **string** is a sequence of bytes that PHP commonly uses to represent text. Strings appear in names, messages, form values, file contents, and output. This chapter covers how to create strings, combine and inspect them, and avoid common mistakes when text contains non-ASCII characters.

## What you will learn

- Write single-quoted, double-quoted, heredoc, and nowdoc strings.
- Interpolate variables or join values with the concatenation operator.
- Use common string functions to inspect, trim, split, join, and replace text.
- Understand the difference between bytes, encoded characters, and user-perceived characters.
- Escape text for an HTML text or attribute context when displaying it.

## Create string literals

### Single-quoted strings

Single quotes are useful when the text should stay literal. PHP recognizes only two escape sequences inside a single-quoted string: **\'** for a quote and **\\** for a backslash. Other backslashes remain part of the string.

~~~php
<?php

$plain = 'The label is $label.';
$quoted = 'It\'s a PHP string.';
$path = 'C:\\projects\\php';
~~~

The value of **$plain** includes the dollar sign and the word **$label**; PHP does not substitute the variable.

### Double-quoted strings

Double quotes allow variable interpolation and escape sequences. Common escapes include **\n** for a newline, **\t** for a tab, **\\** for a backslash, **\$** for a literal dollar sign, and **\"** for a double quote.

~~~php
<?php

$name = 'Maya';
$message = "Hello, $name!\nWelcome to PHP.";
$price = "Use \$ before a variable name.";
~~~

Braces make the end of a variable name clear:

~~~php
<?php

$language = 'PHP';
echo "Learning {$language}Strings";
~~~

This prints **Learning PHPStrings**. Without braces, PHP would look for a variable named **$languageStrings**. Curly braces also help when interpolating array elements or object properties:

~~~php
<?php

$user = ['name' => 'Maya'];
echo "Hello, {$user['name']}!";
~~~

Use the braced **{$name}** form for variable interpolation. The older dollar-brace-name interpolation style is deprecated as of PHP 8.2.

A double-quoted string does not evaluate an arbitrary PHP expression. For an expression or a function call, concatenate the result or use a formatting function:

~~~php
<?php

$items = 3;
echo 'Items: ' . ($items + 1);
~~~

### Heredoc and nowdoc

Heredoc is a multiline string form that behaves much like a double-quoted string: variables are interpolated. It is useful for readable blocks of text.

~~~php
<?php

$name = 'Maya';

$notice = <<<NOTICE
Hello, $name.
Your account is ready.
NOTICE;

echo $notice;
~~~

The closing identifier must follow PHP's heredoc indentation rules and should not be followed by extra text. Choose a distinctive identifier so it is not confused with text inside the string.

Nowdoc uses a quoted identifier and keeps the contents literal, like a single-quoted string:

~~~php
<?php

$template = <<<'TEMPLATE'
Hello, $name.
This text is not interpolated.
TEMPLATE;

echo $template;
~~~

Choose single quotes when a short literal should stay simple, double quotes when interpolation or escapes help readability, heredoc for multiline interpolated text, and nowdoc for multiline literal text.

## Combine strings

PHP uses the **.** operator to concatenate strings. It does not use **+** for string joining.

~~~php
<?php

$firstName = 'Maya';
$lastName = 'Patel';

$fullName = $firstName . ' ' . $lastName;
echo $fullName;
~~~

The **.=** assignment operator appends text to an existing variable:

~~~php
<?php

$message = 'Hello';
$message .= ', Maya';
$message .= '!';

echo $message;
~~~

Use parentheses when an arithmetic expression is part of a concatenation. This makes the intended calculation clear:

~~~php
<?php

$quantity = 4;
$price = 2.50;

echo 'Total: $' . ($quantity * $price);
~~~

## Inspect and transform strings

PHP provides many functions for common string tasks. These examples use the original value when it should be preserved and store transformed values in a new variable.

~~~php
<?php

$title = '  PHP Strings  ';

$cleanTitle = trim($title);
$lowercase = strtolower($cleanTitle);
$uppercase = strtoupper($cleanTitle);

echo $cleanTitle;
echo $lowercase;
echo $uppercase;
~~~

**trim()** removes matching characters from the beginning and end of a string. By default, it removes common whitespace characters at the edges; it does not remove text from the middle. It is not a general-purpose Unicode whitespace normalizer.

**strtolower()** and **strtoupper()** are suitable for basic ASCII text. For multibyte text in a known encoding, use the corresponding **mb_strtolower()** and **mb_strtoupper()** functions when the mbstring extension is available.

### Search and replace

In PHP 8 and later, **str_contains()**, **str_starts_with()**, and **str_ends_with()** check whether a string contains, starts with, or ends with another string. These checks are case-sensitive.

~~~php
<?php

$filename = 'notes.md';

var_dump(str_contains($filename, '.'));
var_dump(str_starts_with($filename, 'notes'));
var_dump(str_ends_with($filename, '.md'));
~~~

**str_replace()** replaces matching text and returns a new string. It does not modify the original variable unless you assign the result:

~~~php
<?php

$sentence = 'I study HTML.';
$updated = str_replace('HTML', 'PHP', $sentence);

echo $updated;
~~~

For an earlier PHP version without **str_contains()**, **strpos()** can be used for a substring check. Because a match may be at position zero, compare its result strictly with **false**:

~~~php
<?php

$text = 'PHP strings';

if (strpos($text, 'PHP') !== false) {
    echo 'Found';
}
~~~

### Split and join

**explode()** splits a string at a separator and returns an array. **implode()** joins array items with a separator.

~~~php
<?php

$tags = 'html,css,php';
$tagList = explode(',', $tags);

echo implode(' | ', $tagList);
~~~

Choose a separator that is unambiguous for the data. Splitting a line on commas is not a complete CSV parser: quoted fields can contain commas and require a CSV-aware function such as **str_getcsv()**.

### Extract part of a string

**substr()** extracts a portion of a string using a starting offset and optional length:

~~~php
<?php

$code = 'PHP-048';
echo substr($code, 0, 3);
~~~

A negative offset counts from the end. **substr()** works with byte offsets, so it may split a multibyte character. For text-aware slicing, consider **mb_substr()** with the intended encoding.

## Bytes, characters, and UTF-8

PHP strings are byte sequences. A text file or request commonly uses UTF-8, where one visible character can take more than one byte.

~~~php
<?php

$text = 'café';

echo strlen($text);
~~~

Depending on the source encoding, this UTF-8 string occupies five bytes: the four ASCII letters count as one byte each and **é** uses two. **strlen()** reports bytes, not the number of displayed characters. A string offset such as **$text[0]** and **substr()** also work with bytes.

For multibyte text, the **mbstring** extension provides functions such as **mb_strlen()** and **mb_substr()**. Pass or configure the intended encoding, commonly UTF-8:

~~~php
<?php

$text = 'café';

echo mb_strlen($text, 'UTF-8');
~~~

That reports four encoded characters for this example. Some user-perceived characters, such as an emoji sequence or a letter combined with a separate accent mark, can consist of multiple Unicode code points. When the task is to count grapheme clusters as people perceive them, the **intl** extension provides **grapheme_strlen()** and related functions.

Before measuring or slicing text, identify what is being counted: bytes, encoded characters, or grapheme clusters. Choose the function that matches the requirement.

## Display strings in HTML

Text received from a user is still data; it is not automatically safe HTML. When inserting plain text into an HTML text node or a quoted HTML attribute, escape it for that context with **htmlspecialchars()**:

~~~php
<?php

$comment = '<script>alert("hello")</script>';

echo htmlspecialchars(
    $comment,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
);
~~~

For HTML text, this displays the characters as text rather than interpreting the input as markup. Escaping rules depend on the output context: HTML escaping is not a universal way to make input safe inside JavaScript, CSS, or a URL. Keep data separate from markup and use the right context-specific handling.

## Common mistakes

- Using **+** to join strings instead of **.**.
- Expecting single-quoted strings to interpolate variables.
- Forgetting to use braces when text immediately follows an interpolated variable name.
- Assuming **trim()** removes whitespace from the middle of a string.
- Treating **strlen()** as a character count for UTF-8 text.
- Using byte-based **substr()** or string offsets where a multibyte character boundary matters.
- Escaping text for HTML and assuming the same escaping is correct in every output context.

## Try it yourself

1. Create a greeting using a single-quoted string and another using a double-quoted string. Compare how each treats a variable.
2. Build a full name with **.**, then append a punctuation mark with **.=**.
3. Write a multiline interpolated message with heredoc and a literal template with nowdoc.
4. Trim a title and display lowercase and uppercase versions.
5. Split a comma-separated list with **explode()**, then join it with **implode()** and a different separator.
6. Replace one word in a sentence with **str_replace()**.
7. Compare **strlen('café')** with **mb_strlen('café', 'UTF-8')** when mbstring is available. Explain why the results differ.
8. Display a user-provided string in an HTML text node using **htmlspecialchars()**.

## Key takeaways

- PHP strings are byte sequences; text is commonly encoded as UTF-8.
- Single-quoted strings stay mostly literal, while double-quoted strings support interpolation and escapes.
- Heredoc interpolates variables; nowdoc keeps its content literal.
- Use **.** and **.=** to concatenate and append strings.
- Functions such as **trim()**, **str_replace()**, **explode()**, and **implode()** cover many everyday tasks.
- **strlen()**, string offsets, and **substr()** are byte-based; use multibyte or grapheme functions when that matches the problem.
- Escape untrusted text for its specific output context.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [047 - Iterators, Traversable, and iterable Values](047-iterators-traversable-and-iterable-values.md) | [Open Q&A](../qnas/048-working-with-strings.md) | [Open example](../examples/048-working-with-strings.md) | [049 - Working with Arrays](049-working-with-arrays.md) |
