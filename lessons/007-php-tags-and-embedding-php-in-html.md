# 007 - PHP Tags and Embedding PHP in HTML

A PHP file can contain PHP code, HTML, or both. PHP tags mark where PHP code begins and ends. In a web page, the PHP runtime executes code inside those tags and returns the resulting output; the browser receives the response, not the PHP source code.

## What you will learn

- How the standard PHP opening and closing tags work
- How to mix PHP code with HTML in a **.php** file
- How to output an expression with **echo** or the short echo tag
- When to close a PHP block and when to omit a closing tag
- Why legacy short opening tags should be avoided
- How to distinguish a PHP source file from a file the browser can process

## The standard PHP tags

The standard opening tag is **<?php**. It tells PHP to begin interpreting the following source as PHP code. The closing tag **?>** ends that PHP block. When PHP processes a file, content outside a PHP block is passed through as output.

~~~php
<?php

echo "Hello from PHP!";
?>
~~~

The opening tag must be followed by whitespace, such as a space or newline, before PHP code begins. The examples in this course put the tag on its own line so the boundary is easy to see.

A **.php** file may contain a PHP block, plain HTML, or alternating sections of both. The file extension tells the server which kind of file to process only when the server is configured to handle PHP; renaming a file alone does not make a browser execute PHP.

## Put PHP inside an HTML document

Save a mixed PHP and HTML page with a **.php** extension, such as **index.php**:

~~~php
<?php
$pageTitle = "PHP and HTML";
$message = "PHP generates this message on the server.";
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?php echo $pageTitle; ?></title>
</head>
<body>
    <main>
        <h1><?php echo $pageTitle; ?></h1>
        <p><?php echo $message; ?></p>
    </main>
</body>
</html>
~~~

PHP begins at the first **<?php** tag, assigns the two values, and then ends that block at **?>**. The HTML that follows is part of the response. The PHP blocks inside the **<title>**, **<h1>**, and **<p>** elements insert the values into the page.

The web server executes the PHP file before returning the response. A browser displays the resulting title and text; it does not see or execute the PHP statements.

## Resume HTML after PHP

When PHP is writing a mixed HTML page, close the current PHP block with **?>** at the point where the HTML should continue. Later in the same file, open another PHP block when PHP code is needed again.

~~~php
<?php
$greeting = "Welcome";
?>
<h1><?php echo $greeting; ?></h1>
<p>This paragraph is ordinary HTML in the PHP response.</p>
~~~

The first PHP block prepares a value. The closing tag returns the file to HTML output. The later block outputs the greeting inside the heading, then the remaining paragraph is output as HTML.

Do not place raw HTML inside an open PHP block as if it were PHP syntax. Either output the markup as a string from PHP or close the PHP block, write the HTML, and open a PHP block again when needed. For larger HTML sections, writing the HTML outside PHP tags is usually easier to read than building a long string with **echo**.

## Output an expression with the short echo tag

The **<?= ... ?>** tag is shorthand for **<?php echo ...; ?>**. Use it when you want to place one expression directly into HTML:

~~~php
<h1><?= $pageTitle ?></h1>
~~~

This is the **short echo tag**. It is different from the legacy short opening tag **<?**. The short echo tag is available even when the **short_open_tag** configuration is disabled. It is a practical option in templates where small values are inserted into markup.

## Avoid legacy short opening tags

Do not rely on the legacy opening tag **<?**:

~~~php
<? echo "This may not work on every PHP setup."; ?>
~~~

Whether **<?** is recognized can depend on the PHP build or the **short_open_tag** setting. It can also be confused with markup such as an XML declaration. Use **<?php** for a PHP block and **<?= ... ?>** for a short echo instead.

## Should a PHP file end with ?>?

If a file contains only PHP code through its end, omit the final closing tag:

~~~php
<?php
$message = "This file contains only PHP code.";
echo $message;
~~~

Omitting the final **?>** avoids accidentally sending whitespace or a newline after the PHP code. This convention is useful for PHP-only source files.

In a mixed PHP and HTML template, use **?>** when the file needs to return to HTML. The recommendation to omit a closing tag applies when the PHP code is finished at the end of a PHP-only file; it does not mean that a template must remain in PHP mode while its HTML is written.

## Output is not automatically safe HTML

The **echo** statement outputs a value; it does not automatically make that value safe for an HTML context. This lesson uses fixed, trusted strings to teach the tag boundaries. Later lessons cover output escaping and how to safely display values that may come from a user or another external source.

For now, do not copy user input into a page simply by echoing it. The page's HTML context matters, and output must be handled appropriately before it is used in markup.

## PHP in the browser and PHP on the server

PHP runs on the server. To see a PHP-generated page, request a **.php** file through a PHP-enabled web server, such as the local development server from [chapter 005](005-run-php-from-the-command-line-and-built-in-server.md). Opening the source file directly with a **file://** URL does not run PHP.

The browser receives the generated response, which may contain HTML, text, or another content type. The PHP opening and closing tags and PHP statements are processed on the server; they are not a browser scripting language like JavaScript.

## Common mistakes

| Mistake | Why it causes trouble | What to do |
| --- | --- | --- |
| Using **<?** as the opening tag | Legacy short tags may be disabled or unavailable. | Use **<?php**. |
| Expecting a browser to execute a PHP file opened from disk | The browser does not run server-side PHP from a **file://** URL. | Request the file through a PHP-enabled server using **http://**. |
| Writing raw HTML while still inside a PHP block | PHP tries to parse those characters as PHP code. | Close the PHP block before writing HTML, then reopen it where PHP is needed. |
| Forgetting **?>** before a section of HTML | The HTML is still inside the PHP block and may cause a syntax error. | Close the PHP block at the transition to HTML. |
| Adding **?>** to the end of a PHP-only file | Extra whitespace after it can become output. | Omit the final closing tag in PHP-only files. |
| Assuming **echo** escapes a value for HTML | **echo** outputs its value without context-sensitive escaping. | Use trusted fixed values for this exercise; learn escaping before displaying external data. |
| Expecting PHP code to appear in the browser's page | PHP is executed on the server before the response is returned. | Inspect the rendered result; use developer tools or view-source only to inspect the response, not to run the PHP. |

## Try it yourself

1. Create **index.php** with one PHP block that assigns a page title, followed by a complete HTML document.
2. Output the title in the document's **<title>** and **<h1>** using **<?php echo ...; ?>**.
3. Replace one output block with the short echo form **<?= ... ?>** and compare readability.
4. Add a paragraph written as plain HTML between two PHP blocks.
5. Run **php -l index.php** to check the syntax, then serve the page through the local PHP server and view it in a browser.
6. Save a copy as a PHP-only file and omit its final **?>** tag.
7. Explain why **<?** is less portable than **<?php**.
8. Describe why a PHP page should be requested through a server instead of opened using **file://**.

## Key points

- **<?php** opens a standard PHP block; **?>** closes it and returns a mixed file to non-PHP output.
- HTML can be written outside PHP blocks in a **.php** template.
- Use **echo** or **<?= ... ?>** to output an expression.
- **<?= ... ?>** is the short echo tag; do not confuse it with legacy **<?**.
- Omit the final closing tag when a PHP-only source file ends in PHP code.
- A PHP-enabled server executes the PHP and returns a response for the browser to display.
- **echo** does not automatically escape values for HTML; external values need appropriate handling.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [006 - Set up the editor, terminal, and project files](006-set-up-the-editor-terminal-and-project-files.md) | [Open Q&A](../qnas/007-php-tags-and-embedding-php-in-html.md) | [Open example](../examples/007-php-tags-and-embedding-php-in-html.md) | [008 - Statements, instruction separation, and comments](008-statements-instruction-separation-and-comments.md) |
