# 007 - PHP Tags and Embedding PHP in HTML: Complete Example

This example creates one PHP-only command-line file and one page that mixes HTML with PHP. It shows where each kind of code belongs and what the browser receives after the PHP server processes the page.

## Question

How can a PHP file combine ordinary HTML with PHP-generated values, and how is that different from a file containing only PHP code?

## Goal

Create this small project:

~~~text
php-template-practice/
├── README.md
├── greeting.php
└── public/
    └── index.php
~~~

- **greeting.php** contains only PHP and prints a terminal greeting.
- **public/index.php** contains HTML with PHP blocks that insert page values.
- **README.md** records how to check and run the files.

## Complete code: PHP-only file

Save this as **greeting.php** at the project root:

~~~php
<?php

$greeting = "Hello from a PHP-only file!";
echo $greeting, PHP_EOL;
~~~

This file begins with the standard **<?php** tag and ends while PHP code is still active. There is no final **?>** tag because the file contains only PHP.

## Complete code: mixed HTML and PHP page

Create a **public** folder and save this as **public/index.php**:

~~~php
<?php
$pageTitle = "PHP and HTML";
$visitorName = "Learner";
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= $pageTitle ?></title>
</head>
<body>
    <main>
        <h1><?= $pageTitle ?></h1>
        <p><?php echo "Welcome, ", $visitorName, "!"; ?></p>
        <p>This paragraph is ordinary HTML outside a PHP block.</p>
    </main>
</body>
</html>
~~~

The first PHP block assigns fixed values and closes before the HTML document. The **<?= ... ?>** tags output the page title. The **<?php echo ...; ?>** block outputs the greeting inside a paragraph. The last paragraph is ordinary HTML outside a PHP block.

These values are fixed for the example. Do not replace them with user input and assume **echo** makes that input safe; output escaping is a separate topic.

## Complete code: README.md

Save this as **README.md** at the project root:

~~~markdown
# PHP Template Practice

A small project showing a PHP-only script and an HTML page with embedded PHP.

## Files

- `greeting.php` is a PHP-only command-line script.
- `public/index.php` is a mixed HTML and PHP page.

## Check the PHP syntax

```sh
php -l greeting.php
php -l public/index.php
```

## Run the CLI script

```sh
php greeting.php
```

## View the page locally

```sh
php -S 127.0.0.1:8000 -t public
```

Open `http://127.0.0.1:8000/` in a browser. Press Ctrl+C in the server terminal when finished.
~~~

This Markdown page displays the example code as text. It does not create or execute the PHP files for you.

## Check syntax without running the files

Open a terminal in **php-template-practice/**, then run:

~~~sh
php -l greeting.php
php -l public/index.php
~~~

For each valid file, PHP reports that no syntax errors were detected. The **-l** option checks syntax only; it does not print the greeting or render the page.

## Run the PHP-only file

From the project root, run:

~~~sh
php greeting.php
~~~

Expected terminal output:

~~~text
Hello from a PHP-only file!
~~~

The PHP runtime starts at **<?php**, executes the statements, and writes the result to the terminal. Since there is no HTML after the PHP block, the file does not need a closing tag at the end.

## View the mixed page in a browser

From the project root, start the local server with **public** as its document root:

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

Keep the terminal open and visit:

~~~text
http://127.0.0.1:8000/
~~~

The browser should display a page with this content:

~~~text
PHP and HTML

Welcome, Learner!

This paragraph is ordinary HTML outside a PHP block.
~~~

The first **PHP and HTML** is the page title shown in the browser tab; it is also the page heading. The page body contains the greeting and the ordinary HTML paragraph. Browser fonts, spacing, and title-bar appearance depend on the browser.

When finished, return to the terminal and press **Ctrl+C** to stop the local server.

## Inspect the generated response

Conceptually, the response contains HTML and the inserted values, such as:

~~~html
<title>PHP and HTML</title>
<h1>PHP and HTML</h1>
<p>Welcome, Learner!</p>
<p>This paragraph is ordinary HTML outside a PHP block.</p>
~~~

This is an illustrative excerpt, not the complete response. The server processes the PHP blocks before returning the page. The browser does not receive the PHP statements **$pageTitle = ...** or **echo ...** as executable PHP source.

## How the tags work in this example

1. **<?php** starts the first PHP block.
2. PHP assigns values to **$pageTitle** and **$visitorName**.
3. **?>** ends that block, so the HTML document can continue.
4. **<?= $pageTitle ?>** outputs the title value in the HTML.
5. **<?php echo ...; ?>** outputs the greeting inside the paragraph.
6. The final paragraph is outside PHP tags, so it is returned as HTML.

The short echo form is useful for inserting a small expression in a template. It is not the same as the legacy **<?** opening tag.

## Common mistakes

- **Using `<?` instead of `<?php`:** The legacy short tag may be disabled. Use the standard opening tag.
- **Forgetting `?>` before the HTML document:** PHP continues to parse the file as PHP, so HTML may cause a syntax error.
- **Adding a closing tag to every PHP-only file:** Omit the final **?>** when the file ends in PHP code.
- **Opening the page with `file://`:** This does not run PHP. Start the local server and use its **http://** address.
- **Echoing external input as if it were safe:** **echo** does not escape values for HTML. This example uses fixed values only.
- **Expecting PHP statements to appear in the rendered page:** The server processes the PHP before returning the response.

## Try changing it

1. Change **$pageTitle** and refresh the browser page.
2. Replace one **<?= ... ?>** expression with **<?php echo ...; ?>** and compare the two forms.
3. Add another static HTML paragraph outside PHP tags.
4. Add a PHP-only file for a different terminal message and omit its final closing tag.
5. Temporarily remove the closing **?>** before the HTML in **public/index.php**, then restore it after observing the syntax check.
6. Explain why the PHP source is not displayed as PHP code in the browser.

## Key takeaways

- **<?php** and **?>** mark PHP blocks in a mixed PHP file.
- HTML can be written directly outside PHP blocks in a **.php** template.
- Use **<?= ... ?>** or **<?php echo ...; ?>** to insert output.
- A PHP-only file should normally omit its final closing tag.
- PHP executes on the server; the browser receives and renders the generated response.
- **echo** does not automatically make external values safe for HTML.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [006 - Set up the editor, terminal, and project files](../lessons/006-set-up-the-editor-terminal-and-project-files.md) | [Read this lesson](../lessons/007-php-tags-and-embedding-php-in-html.md) | [Open Q&A](../qnas/007-php-tags-and-embedding-php-in-html.md) | [008 - Statements, instruction separation, and comments](../lessons/008-statements-instruction-separation-and-comments.md) |
