# 001 - Introduction to PHP

PHP is a programming language used to create dynamic websites and many other kinds of programs. Its name originally stood for **Personal Home Page**; today PHP is a recursive acronym for **PHP: Hypertext Preprocessor**.

PHP is a general-purpose language. It is especially common on the web because a server can run PHP code to create a response, such as an HTML page or JSON data. PHP can also run from a terminal for scripts and automation. You do not need a framework or a database to write PHP.

This chapter gives you the basic mental model: what PHP does, where it runs, and what a tiny PHP program looks like. Later chapters cover installation, syntax, variables, HTML output, and web requests in detail.

## What you will learn

- What PHP is and what it is used for
- The difference between PHP code and the response a visitor receives
- How PHP fits between a web server and a browser
- What a basic PHP file looks like
- Why PHP can run without a browser

## PHP runs in different environments

A **programming language** gives you rules for writing instructions. An **execution environment** provides a way to run those instructions. PHP code is processed by the PHP runtime, sometimes called the PHP interpreter or engine.

PHP is not only a browser language. A browser does not run PHP code. Common ways to use PHP include:

- Creating HTML on a web server
- Returning JSON from a web application or API
- Running a command-line script
- Automating work with files or other services

A framework can provide helpful structure, but PHP itself is the language. A framework is an optional set of tools built with PHP.

## What happens when a browser requests a PHP page?

For a typical PHP website, the parts work together like this:

1. A browser requests a page from a web server.
2. The web server passes a PHP file to the PHP runtime.
3. PHP executes its instructions and produces a response, often HTML.
4. The web server sends that response to the browser.
5. The browser interprets the HTML and displays the page.

The browser receives the generated response, not the PHP source code. PHP code therefore runs on the server side. A PHP application must be configured correctly so the server processes PHP files instead of serving their source as text.

A simplified picture:

~~~text
Browser -- request --> Web server -- PHP file --> PHP runtime
Browser <-- HTML or other response <-- Web server <-- result
~~~

PHP can also be run directly from a command line. In that case, there may be no browser and no web server involved.

## Your first PHP statement

A PHP file commonly uses the .php extension. PHP code begins with the PHP opening tag. Here is a small command-line script:

~~~php
<?php

$name = 'Asha';
echo 'Hello, ', $name, '!';
~~~

The program produces:

~~~text
Hello, Asha!
~~~

Read it from top to bottom:

- The **PHP opening tag** tells PHP that PHP code begins here.
- **$name** is a variable. PHP variable names begin with a dollar sign.
- **=** assigns a value to the variable.
- Text inside quotes is a string.
- **echo** sends text to the current output.
- A semicolon ends each statement in this example.

This example writes plain text. A PHP program can instead produce HTML, JSON, or another kind of output depending on what the program needs to do.

## PHP can be embedded in HTML

One common use for PHP is to insert generated content into an HTML page:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>My first PHP page</title>
</head>
<body>
    <h1><?php echo 'Hello from PHP'; ?></h1>
</body>
</html>
~~~

The HTML outside the PHP tags is sent as page content. The PHP block runs on the server and writes the heading text into the response. The browser receives the finished HTML, conceptually:

~~~html
<h1>Hello from PHP</h1>
~~~

That final snippet illustrates the output; it is not a second PHP program. Later lessons explain PHP tags and embedding in more detail.

## PHP is more than page templates

PHP is often introduced through web pages, but it can do more than place text inside HTML. A PHP program can:

- Read input and choose what response to create
- Work with arrays, objects, and files
- Communicate with databases and web services
- Return data in formats such as JSON
- Run as a command-line tool

These abilities come from PHP and its libraries. A database, framework, or web server is a separate tool that may be used alongside PHP; none is required for every PHP program.

## Important boundaries

Keep these ideas clear as you continue:

- PHP runs in a PHP-capable environment. A browser alone cannot execute a .php file.
- HTML describes document structure. PHP can generate that HTML, but PHP and HTML are different things.
- JavaScript commonly runs in the browser. PHP commonly runs on the server. A website can use both for different tasks.
- PHP does not make an application secure automatically. Programs must handle input, permissions, and output carefully. Security is covered later in the course.
- A PHP file is source code. Its output is the response produced when that code runs.

## How to get ready

The next lessons explain how PHP fits into a web application, how PHP versions work, and how to install and run the language. You do not need to install anything to understand the ideas in this introduction. When you do run PHP, use a PHP runtime rather than opening the .php source directly as a browser file.

## Try it yourself

1. In your own words, describe what PHP is.
2. Name two things a PHP program can produce besides an HTML page.
3. Explain which part runs PHP code in a typical website.
4. In the first script, identify the variable, its assigned value, and the output statement.
5. Describe why opening a .php file directly in a browser does not demonstrate normal server-side PHP execution.
6. Explain one difference between PHP and a framework written in PHP.

## Key points

- PHP is a general-purpose programming language widely used for server-side web development.
- The PHP runtime executes PHP source code.
- A web server can send a PHP file to the runtime and return the generated response to a browser.
- PHP can also run from a command line without a browser.
- PHP can generate HTML, JSON, plain text, and other output.
- Frameworks, databases, and web servers can work with PHP, but they are separate tools.

## Chapter navigation

| Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- |
| [Open Q&A](../qnas/001-introduction-to-php.md) | [Open example](../examples/001-introduction-to-php.md) | [002 - How PHP fits into a web application](002-how-php-fits-into-a-web-application.md) |
