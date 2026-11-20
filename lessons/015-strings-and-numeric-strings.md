# 015 - Strings and Numeric Strings

A string stores text, but quoted digits are still text until a program deliberately validates or uses them as numbers. This lesson covers PHP string syntax, interpolation, concatenation, multiline strings, byte-oriented string length, and the idea of numeric strings.

## What you will learn

- How single-quoted and double-quoted strings differ
- How escape sequences work in double-quoted strings
- How to place a variable inside a string
- How to join strings with the concatenation operators
- When heredoc and nowdoc are useful
- Why PHP strings are byte sequences and how that affects non-ASCII text
- What makes a string numeric and how **is_numeric()** helps inspect it
- Why numeric-looking input must still be validated for the application's rules

## A string is text

A PHP string is a sequence of bytes used to store text or other byte data. Ordinary quoted text is a string literal:

~~~php
<?php
$courseName = "PHP Fundamentals";
$chapterLabel = 'Chapter 15';

var_dump($courseName);
var_dump($chapterLabel);
~~~

Output:

~~~text
string(16) "PHP Fundamentals"
string(10) "Chapter 15"
~~~

The number inside **string(16)** is the string's length in bytes. For basic ASCII text, each character uses one byte, so the byte count and character count match.

Quotes are part of the PHP source syntax; they are not included in the stored value.

## Single-quoted strings

Single quotes are useful when the string should stay mostly literal. PHP does not expand variables or interpret common escape sequences such as **\n** inside a single-quoted string.

~~~php
<?php
$name = "Asha";

echo 'Hello, $name.\n';
echo PHP_EOL;
echo 'It\'s a PHP lesson.' . PHP_EOL;
echo 'Path: C:\\projects\\php' . PHP_EOL;
~~~

Output:

~~~text
Hello, $name.\n
It's a PHP lesson.
Path: C:\projects\php
~~~

Within a single-quoted string, the two special escapes are **\\** for a backslash and **\'** for a single quote. Other backslashes remain literal.

## Double-quoted strings

Double quotes let PHP interpret common escape sequences and expand variables:

~~~php
<?php
$name = "Asha";
$chapterNumber = 15;

echo "Hello, $name.\n";
echo "You are reading chapter $chapterNumber.\n";
echo "First line\nSecond line\n";
echo "A tab:\tDone\n";
echo "A dollar sign: \$5\n";
~~~

Output:

~~~text
Hello, Asha.
You are reading chapter 15.
First line
Second line
A tab:    Done
A dollar sign: $5
~~~

Common double-quoted escape sequences include:

| Sequence | Meaning |
| --- | --- |
| **\n** | Newline |
| **\t** | Horizontal tab |
| **\\** | A backslash |
| **\"** | A double quote |
| **\$** | A literal dollar sign |

Use braces to make the end of a variable name clear when more text follows:

~~~php
<?php
$language = "PHP";

echo "Learn {$language}15 next." . PHP_EOL;
~~~

Output:

~~~text
Learn PHP15 next.
~~~

Here the braces show that the variable is **$language** and the digits **15** are literal text. For more complex expressions, build the string with concatenation instead of trying to embed the expression in the quoted text.

## Concatenate strings

The dot (**.**) operator joins strings and other values converted for string context. The dot-equals operator (**.=**) appends text to the existing string:

~~~php
<?php
$firstName = "Asha";
$lastName = "Ranjan";

$fullName = $firstName . " " . $lastName;
echo $fullName . PHP_EOL;

$message = "Welcome";
$message .= ", " . $firstName . "!";
echo $message . PHP_EOL;
~~~

Output:

~~~text
Asha Ranjan
Welcome, Asha!
~~~

Concatenation is useful when the parts of a message are clearer as separate values. Use **PHP_EOL** when writing a platform-appropriate newline in command-line output.

## Multiline strings: heredoc and nowdoc

Heredoc syntax is useful when a string spans multiple lines and should still expand variables. The closing identifier must match the opening identifier:

~~~php
<?php
$customer = "Asha";
$itemCount = 3;

$receipt = <<<RECEIPT
Receipt for {$customer}
Items: {$itemCount}
Thank you.
RECEIPT;

echo $receipt;
~~~

Output:

~~~text
Receipt for Asha
Items: 3
Thank you.
~~~

Nowdoc uses a quoted identifier and keeps variables and escape sequences literal. It is useful for multi-line text that should be treated as written:

~~~php
<?php
$customer = "Asha";

$template = <<<'TEMPLATE'
Hello, $customer.
Write \n to show a backslash and the letter n.
TEMPLATE;

echo $template;
~~~

Output:

~~~text
Hello, $customer.
Write \n to show a backslash and the letter n.
~~~

The marker names such as **RECEIPT** and **TEMPLATE** are chosen by the programmer. Use a clear identifier that does not appear by itself at the start of a line inside the string.

## Strings and non-ASCII text

PHP strings are byte sequences; the string type itself does not attach a Unicode character encoding to the bytes. Web applications commonly exchange text encoded as UTF-8, but functions that count or slice string bytes do not automatically count user-perceived characters.

For example, in a UTF-8 source file, **é** is commonly encoded using two bytes:

~~~php
<?php
$text = "café";

echo strlen($text) . PHP_EOL;
~~~

Output for UTF-8 source text:

~~~text
5
~~~

The visible text has four characters, but **strlen()** reports five bytes because **é** uses two bytes in UTF-8. For multibyte text, the **mbstring** extension provides functions such as **mb_strlen()** that can count characters using an encoding. A displayed grapheme, such as a letter combined with a separate accent mark, can consist of more than one Unicode code point, so user-perceived character counting can require more than a basic byte count.

Do not use **strlen()** as a character counter for arbitrary UTF-8 text.

## Numeric strings

A numeric string is still a string value that can be interpreted as an integer or float. Its PHP type does not change just because it contains digits:

~~~php
<?php
$chapterText = "15";
$chapterNumber = 15;

var_dump($chapterText);
var_dump($chapterNumber);
var_dump(is_numeric($chapterText));
~~~

Output:

~~~text
string(2) "15"
int(15)
bool(true)
~~~

**is_numeric()** returns true for numbers and numeric strings. It does not convert a numeric string or tell you whether it satisfies an application's requirements.

~~~php
<?php
var_dump(is_numeric("15"));
var_dump(is_numeric("-2.5"));
var_dump(is_numeric("1.5e2"));
var_dump(is_numeric("fifteen"));
~~~

Output:

~~~text
bool(true)
bool(true)
bool(true)
bool(false)
~~~

Numeric strings can be interpreted as numbers in arithmetic contexts. For example, PHP 8 treats the numeric string **"15"** as a number when used with the addition operator:

~~~php
<?php
$chapterText = "15";
$result = $chapterText + 1;

var_dump($chapterText);
var_dump($result);
~~~

Output:

~~~text
string(2) "15"
int(16)
~~~

The variable **$chapterText** remains a string; the arithmetic expression interprets its value numerically to calculate the result. Automatic conversions have context-specific rules. A later lesson covers type juggling and explicit casting in more detail.

## Validate for the actual requirement

A value passing **is_numeric()** does not mean it is valid for every purpose. It accepts numeric formats such as decimals and exponent notation, while an application may require a whole number within a particular range.

~~~php
<?php
$submittedQuantity = "12";

if (is_numeric($submittedQuantity)) {
    echo "The value is numeric text." . PHP_EOL;
}
~~~

This only answers whether PHP recognizes the value as numeric. A program that needs a positive whole-number quantity must also check that requirement and its allowed limits. Do not treat **is_numeric()** alone as validation for a form, payment, or security-sensitive operation.

When displaying user-provided text in a web page, escaping rules depend on the output context. Concatenating a string does not make it safe HTML, SQL, shell input, or a URL. Those contexts are covered in later lessons.

## Try it yourself

1. Create one single-quoted and one double-quoted string that contain the same variable name. Compare what each prints.
2. Put **\n** in both quote styles. Observe which string contains a line break.
3. Build a full name with **.**, then append a greeting with **.=**.
4. Write a multiline greeting with heredoc and another with nowdoc. Compare variable expansion.
5. Run **strlen("café")** in a UTF-8 file. Explain why the result can be larger than the visible character count.
6. Use **is_numeric()** with an integer string, a decimal string, exponent notation, and ordinary words.
7. Add **1** to the numeric string **"15"**, then inspect both the original variable and the result with **var_dump()**.
8. Design checks for a quantity that must be a positive whole number from **1** to **100**. Explain why **is_numeric()** is only one part of that validation.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Expecting variables to expand inside single quotes | Single-quoted strings preserve variable text literally. | Use double quotes or concatenate the value. |
| Expecting **\n** to create a line break in single quotes | That escape sequence is interpreted in double-quoted strings, not ordinary single-quoted strings. | Use double quotes or concatenate **PHP_EOL** in CLI output. |
| Forgetting to escape a quote that matches the string delimiter | PHP may treat it as the end of the string. | Escape the delimiter or choose the other quote style. |
| Using **+** to join text | **+** is an arithmetic operator. | Use **.** or **.=** for string concatenation. |
| Assuming a numeric string already has an integer type | Quoted digits are still a string until explicitly converted or used in a numeric context. | Inspect with **var_dump()** and convert deliberately when needed. |
| Treating **is_numeric()** as complete input validation | It does not enforce integer-only, range, or business rules. | Validate the precise format and constraints the application requires. |
| Using **strlen()** to count UTF-8 characters | **strlen()** counts bytes. | Use an encoding-aware function when the mbstring extension is available. |
| Assuming concatenation escapes untrusted text | Joining strings does not make them safe for HTML, SQL, shell, or URL contexts. | Use context-appropriate escaping or parameterized APIs in the relevant lesson. |

## Key takeaways

- A PHP string is a sequence of bytes; quoted digits are still strings.
- Single quotes keep variables and most escape sequences literal.
- Double quotes interpret common escapes and interpolate variables.
- Use **.** to concatenate and **.=** to append strings.
- Heredoc expands variables in multiline text; nowdoc keeps the contents literal.
- **strlen()** counts bytes; UTF-8 characters may use multiple bytes.
- **is_numeric()** identifies numbers and numeric strings but does not enforce application rules.
- Numeric strings may be interpreted numerically in arithmetic contexts; full conversion rules come later.
- Concatenation is not a substitute for escaping untrusted data for a particular output context.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [014 - Null, booleans, integers, and floating-point numbers](014-null-booleans-integers-and-floating-point-numbers.md) | [Open Q&A](../qnas/015-strings-and-numeric-strings.md) | [Open example](../examples/015-strings-and-numeric-strings.md) | [016 - Indexed and associative arrays](016-indexed-and-associative-arrays.md) |
