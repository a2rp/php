# 008 - Statements, Instruction Separation, and Comments

PHP statements tell the runtime to do something, such as assign a value or output text. In PHP code, ordinary statements are separated with semicolons. Comments let you leave notes in the source without making those notes part of the program's output.

## What you will learn

- What a PHP statement is
- Where semicolons belong
- Why a newline is not a substitute for a semicolon
- How the closing PHP tag affects the last statement in a block
- How to write single-line and block comments
- How PHP comments differ from HTML comments

## Statements and semicolons

A statement is an instruction in a PHP program. Assigning a value and calling **echo** are examples:

~~~php
<?php
$name = "Learner";
echo "Welcome, ", $name, "!";
~~~

The assignment statement ends with a semicolon. The **echo** statement also ends with a semicolon. In ordinary PHP code, put a semicolon at the end of each statement that requires one; do not rely on line breaks to separate statements.

This is valid even though the statements share one line:

~~~php
<?php $first = "Hello"; $second = "PHP"; echo $first, " ", $second;
~~~

Line breaks and indentation make source easier to read, but they do not replace the statement terminator. If you leave out a required semicolon between statements, PHP will usually report a syntax error near the following statement.

Compound statements such as conditionals and loops have their own syntax and are covered in later chapters. This chapter focuses on simple statements such as assignments and output.

## The closing PHP tag and the last statement

When a PHP block ends with **?>**, that closing tag also terminates the final PHP statement in that block. The semicolon before the closing tag is therefore optional for that final statement:

~~~php
<?php
echo "This statement is followed by a closing tag"
?>
~~~

Even though PHP allows this, keep the semicolon before **?>** in normal examples and templates. It makes the statement boundary visible and keeps the style consistent:

~~~php
<?php
echo "This statement ends clearly";
?>
<p>HTML follows the PHP block.</p>
~~~

If a file contains only PHP code through its end, omit the final **?>** tag and write the semicolon normally:

~~~php
<?php
echo "The file ends while PHP is active";
~~~

The closing-tag rule applies only to the final statement immediately before that tag. It does not let you omit semicolons between earlier statements in the same block.

## Single-line comments

A comment is a note for people reading the source. PHP ignores a comment while executing the code; it does not appear in the output just because it is a comment.

Use **//** for a single-line comment:

~~~php
<?php
// Store the name that the greeting will use.
$name = "Learner";

echo "Welcome, ", $name, "!";
~~~

PHP also accepts **#** for a single-line comment:

~~~php
<?php
# This is also a single-line PHP comment.
echo "Hello!";
~~~

For everyday PHP comments, **//** is a clear default. In PHP 8 and later, **#[** begins attribute syntax, so do not use **#** as a comment marker immediately before **[**.

A single-line comment continues to the end of the line or to the end of the current PHP block, whichever comes first. In a mixed file, **?>** exits PHP mode even if it follows a **//** or **#** comment on the same line. Text after that closing tag is outside the comment and becomes output.

## Block comments

Use **/* ... */** for a comment that spans one or more lines:

~~~php
<?php
/*
This note spans multiple lines.
It is still ignored as PHP code.
*/
echo "The comment is not part of this output.";
~~~

A block comment ends at the first ***/** closing marker. Do not nest **/* ... */** comments inside one another; the first closing marker ends the comment and can leave unexpected text to be parsed as PHP.

Use comments to explain intent, assumptions, or a non-obvious decision. A comment that merely repeats the next line can become stale and add clutter:

~~~php
<?php
// Set the name to Learner.
$name = "Learner";
~~~

That note does not add much because the assignment already says it. A short explanation of why a value is chosen or how a tricky part works is more useful.

## PHP comments and HTML comments are different

PHP comments only affect source while PHP is parsing a PHP block. HTML comments use **<!-- ... -->** and are passed through as HTML for the browser to interpret.

An HTML comment does not prevent PHP code inside a PHP-enabled page from running on the server. For example, PHP still executes the PHP block in this file before the response reaches the browser:

~~~php
<!--
<?php echo "This PHP code still runs on the server."; ?>
-->
~~~

The browser may hide the returned text because it is inside an HTML comment, but that does not undo the server-side execution. Use PHP comments to comment out PHP source; do not use HTML comments as a way to disable server-side code.

## Common mistakes

| Mistake | Why it causes trouble | What to do |
| --- | --- | --- |
| Leaving out a semicolon between simple PHP statements | A newline alone does not separate the statements. | End each simple statement with **;**. |
| Thinking every line must end in a semicolon | Some compound syntax has different rules. | Learn the syntax for each statement type; this lesson covers ordinary assignments and output. |
| Omitting a semicolon before every **?>** | PHP only implies a terminator for the final statement before the closing tag. | Keep the semicolon for readability and consistency. |
| Putting **//** before HTML and expecting the HTML to be commented out | **?>** exits PHP mode; the PHP line comment does not continue through following HTML. | Use an HTML comment for browser markup, or structure the PHP and HTML blocks deliberately. |
| Using an HTML comment to disable PHP | PHP inside the HTML comment is still processed by the server. | Use PHP comments when commenting out PHP source. |
| Nesting block comments | The first ***/** closes the block. | Keep block comments separate and do not nest them. |
| Writing a comment that repeats the next statement | The comment can become stale without adding useful context. | Explain the reason or intent when it is not already clear. |

## Try it yourself

1. Write two assignments and one **echo** statement, each ending with a semicolon.
2. Remove one semicolon between simple statements and run **php -l filename.php** to read the syntax error; restore it afterward.
3. Put two valid simple statements on one line, separated with semicolons.
4. Add a **//** comment and a **/* ... */** comment to a PHP-only file.
5. In a mixed PHP and HTML file, close a PHP block and write a paragraph; identify which text is inside PHP and which is HTML output.
6. Compare an HTML comment with a PHP comment and explain why they have different effects.
7. Keep the final PHP-only file open, without a closing **?>** tag, and make sure its statements still end with semicolons.
8. Rewrite one comment so it explains why a statement is needed instead of repeating what it does.

## Key points

- Simple PHP statements such as assignments and **echo** end with semicolons.
- A newline is not a substitute for a semicolon between statements.
- A closing **?>** implies the terminator for the final statement immediately before it; keeping the semicolon is clearer.
- **//** and **#** begin single-line comments; **/* ... */** makes a block comment.
- A single-line PHP comment ends with the line or when the PHP block closes.
- HTML comments do not prevent PHP code inside them from running on the server.
- Comments should clarify intent rather than restate obvious code.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [007 - PHP tags and embedding PHP in HTML](007-php-tags-and-embedding-php-in-html.md) | [Open Q&A](../qnas/008-statements-instruction-separation-and-comments.md) | [Open example](../examples/008-statements-instruction-separation-and-comments.md) | [009 - Identifiers, keywords, reserved words, and naming](009-identifiers-keywords-reserved-words-and-naming.md) |
