# 015 - Strings and Numeric Strings: Complete Example

This command-line example creates messages with different PHP string syntaxes, joins text, displays multiline heredoc and nowdoc values, and inspects numeric strings without confusing their contents with their type.

## Question

How can a PHP program format text, preserve literal placeholders, and recognize numeric input while still validating what the application actually needs?

## Goal

Create this project:

~~~text
string-practice/
|-- README.md
|-- strings.php
~~~

The **strings.php** script demonstrates:

- Single-quoted strings and literal variable names
- Double-quoted strings, interpolation, and escape sequences
- Concatenation with **.** and **.=**
- Multiline strings with heredoc and nowdoc
- The byte count of UTF-8 text
- **is_numeric()** checks and arithmetic with numeric strings
- The difference between numeric recognition and full input validation

## Complete code: strings.php

Save this file at the project root:

~~~php
<?php

$name = "Asha";
$topic = "strings";

echo 'Single quoted: $name\n' . PHP_EOL;
echo "Double quoted: {$name}\n";

$fullName = $name . " Ranjan";
echo "Concatenated name: " . $fullName . PHP_EOL;

$fullName .= " (PHP learner)";
echo "Appended label: " . $fullName . PHP_EOL;

$profile = <<<PROFILE
Profile
Name: {$name}
Topic: {$topic}
PROFILE;

echo $profile;

$template = <<<'TEMPLATE'
Placeholder: $name
Keep \n unchanged.
TEMPLATE;

echo $template;

$unicodeText = "cafe\u{00E9}";
echo "UTF-8 byte length: " . strlen($unicodeText) . PHP_EOL;

$numericText = "12.5";
$ordinaryText = "12 items";
$calculatedValue = $numericText + 2;

var_dump($numericText);
var_dump(is_numeric($numericText));
var_dump($ordinaryText);
var_dump(is_numeric($ordinaryText));
var_dump($calculatedValue);

if (is_numeric($numericText)) {
    echo "PHP recognizes the text as numeric." . PHP_EOL;
}
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# String Practice

A small PHP CLI script for practicing string syntax and numeric strings.

## Check and run

~~~sh
php -l strings.php
php strings.php
~~~

## What it demonstrates

- Single and double quotes
- Variable interpolation and escape sequences
- Concatenation and appending
- Heredoc and nowdoc
- UTF-8 byte length
- Numeric-string checks and arithmetic
- Why numeric recognition is not complete input validation
~~~~

This Markdown block is the README file to create in the project. Keep the PHP source in **strings.php**.

## Check and run the program

Open a terminal in **string-practice/** and check the syntax:

~~~sh
php -l strings.php
~~~

For a valid file, PHP reports that no syntax errors were detected. Run the script with:

~~~sh
php strings.php
~~~

Expected output:

~~~text
Single quoted: $name\n
Double quoted: Asha
Concatenated name: Asha Ranjan
Appended label: Asha Ranjan (PHP learner)
Profile
Name: Asha
Topic: strings
Placeholder: $name
Keep \n unchanged.
UTF-8 byte length: 5
string(4) "12.5"
bool(true)
string(8) "12 items"
bool(false)
float(14.5)
PHP recognizes the text as numeric.
~~~

The Unicode escape in **$unicodeText** creates the letter with an acute accent using UTF-8. Its visible text has four characters, while **strlen()** reports five bytes.

## Understand the output

### Single quotes preserve literal text

The first line contains **$name** rather than **Asha**, because the string uses single quotes. Its **\n** is also literal text. **PHP_EOL** after the string adds the actual line break for the command-line output.

### Double quotes interpolate variables

The second line contains **Asha** because the double-quoted string expands **$name**. Its **\n** escape creates a line break.

### Concatenation joins values

The dot operator combines the greeting parts into **$fullName**. The **.=** operator appends another label to that same variable.

### Heredoc expands values; nowdoc preserves them

The heredoc profile expands **$name** and **$topic** while keeping the text on separate lines. The nowdoc template prints **$name** and **\n** literally because its body is not interpolated.

### String length is measured in bytes

The source uses a Unicode escape so the example does not depend on the editor saving a particular accented character literally. The escape represents a UTF-8 character that takes two bytes. Thus the ASCII letters **c**, **a**, **f**, and the accented final letter form four visible characters but five bytes.

### Numeric strings remain strings when assigned

**$numericText** is displayed as **string(4)** and **is_numeric()** returns true. The text **"12 items"** is not entirely numeric, so **is_numeric()** returns false.

When **$numericText** is used in arithmetic, PHP interprets the value numerically for that expression. The result is **float(14.5)**, while the original variable remains a string.

The final message only means PHP recognizes a numeric value. A real application still needs checks for the required format, range, and meaning.

## Make a change

1. Change **$name** and observe every place where the heredoc and double-quoted message expand it.
2. Replace the single quotes around the first output string with double quotes. Compare how the variable and **\n** behave.
3. Change **$topic** and inspect the heredoc output.
4. Add another line to the nowdoc containing **$topic**. Confirm that it stays literal.
5. Change **$numericText** to **"15"**, **"-2.5"**, and **"1.5e2"**. Inspect the numeric checks and arithmetic results.
6. Change **$ordinaryText** to **"12 "**. On PHP 8.0 and newer, trailing whitespace is accepted by **is_numeric()**; older versions differ.
7. Check the UTF-8 string with **mb_strlen()** if the mbstring extension is enabled. Compare its character count with **strlen()**'s byte count.
8. Imagine a quantity field that accepts only whole numbers from **1** through **100**. Add checks for that exact rule; **is_numeric()** alone is insufficient.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Expecting **$name** to expand inside the single-quoted message | Single-quoted strings preserve variable text literally. | Use double quotes or concatenate the variable. |
| Expecting **\n** to add a newline in single quotes | The escape is not interpreted there. | Use a double-quoted string or append **PHP_EOL**. |
| Using **+** to join the name and label | **+** performs arithmetic rather than string concatenation. | Use **.** or **.=**. |
| Expecting nowdoc to expand **$name** | Nowdoc keeps its body literal. | Use heredoc or a double-quoted string when interpolation is intended. |
| Reading **strlen()** as a count of visible UTF-8 characters | It returns the number of bytes. | Use encoding-aware functions when available and appropriate. |
| Assuming **is_numeric()** validates a quantity completely | It does not enforce integer-only or range requirements. | Validate the precise constraints the application needs. |
| Assuming a numeric operation changes the original string's type | The value is interpreted for that expression; no assignment changed the variable. | Inspect both the original variable and the expression result. |
| Treating concatenated user input as safe output | Joining strings does not escape them for HTML, SQL, or another context. | Use context-appropriate escaping or parameterized APIs. |

## Key takeaways

- Single-quoted strings keep variables and most escapes literal; double-quoted strings interpolate variables and interpret common escapes.
- The dot operator joins strings, and **.=** appends to an existing string.
- Heredoc interpolates variables in multiline text; nowdoc keeps its contents literal.
- PHP strings are byte sequences; **strlen()** counts bytes rather than UTF-8 characters.
- A numeric string is still a string when assigned.
- **is_numeric()** checks numeric interpretation but does not validate application-specific rules.
- Arithmetic may interpret a numeric string for an expression without changing the original variable.
- Concatenation does not escape data for a particular output context.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [014 - Null, booleans, integers, and floating-point numbers](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) | [Read this lesson](../lessons/015-strings-and-numeric-strings.md) | [Open Q&A](../qnas/015-strings-and-numeric-strings.md) | [016 - Indexed and associative arrays](../lessons/016-indexed-and-associative-arrays.md) |
