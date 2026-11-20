# 001 - Introduction to PHP: Complete Example

This example turns a few PHP values into a complete HTML page. It demonstrates the main idea from [001 - Introduction to PHP](../lessons/001-introduction-to-php.md): PHP runs before the response reaches the browser.

## Question

How can a PHP file use values from PHP variables to create a greeting inside an HTML page?

## Goal

Create a small PHP page that displays a person's name and the course they are learning. The PHP code runs on the server and generates the HTML response. The browser displays that response.

## Complete code

Save this as **index.php**:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>My first PHP page</title>
</head>
<body>
    <main>
        <?php
        $name = 'Asha';
        $courseName = 'PHP';
        ?>
        <h1>Hello, <?php echo htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8'); ?>!</h1>
        <p>Welcome to the <?php echo htmlspecialchars($courseName, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8'); ?> course.</p>
    </main>
</body>
</html>
~~~

GitHub displays this code as text. It does not execute PHP or create a live page in this Markdown file.

## Expected browser output

The page body displays:

~~~text
Hello, Asha!
Welcome to the PHP course.
~~~

The browser tab displays the title **My first PHP page**. The title is in the document head, so it is not part of the visible page body.

## How the example works

### 1. PHP stores the values

~~~php
$name = 'Asha';
$courseName = 'PHP';
~~~

These assignment statements create two variables. Change the quoted strings to change the generated greeting.

### 2. The PHP block ends before the document markup

The file begins with its HTML structure. Inside the main element, a PHP block creates the values; its closing tag lets the file continue as HTML. PHP can mix server-side code and HTML in one file.

### 3. PHP writes escaped values into the HTML

~~~php
<?php echo htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8'); ?>
~~~

The **echo** statement writes the result into the response. The **htmlspecialchars** function converts characters that have special meaning in HTML into safe text representations. This is a useful habit when placing dynamic values into HTML. The encoding argument says the text is UTF-8; the flags handle quotes and invalid sequences consistently.

The browser receives the generated HTML with the values in place. It does not receive the PHP instructions that produced them.

## Run it locally

To see this example in a browser, use a PHP installation and a local development server:

1. Save the code in a file named **index.php** inside a folder.
2. Open a terminal in that folder.
3. Start PHP's built-in development server:

~~~sh
php -S 127.0.0.1:8000
~~~

4. Open **http://127.0.0.1:8000/index.php** in a browser.
5. Stop the server with **Ctrl+C** in the terminal when you are finished.

Opening the PHP source directly as a file in a browser does not run the PHP code. The request needs to go through a PHP-capable server. PHP's built-in server is intended for local development, not as a production hosting server.

## Try changing it

1. Change **Asha** to your own name and refresh the page.
2. Change **PHP** to another course name and refresh the page.
3. Add a third variable for a favorite topic and print it in a paragraph.
4. Identify which text is written as HTML and which values are inserted by PHP.
5. Explain why the browser displays the greeting but does not show the PHP source.

## Key takeaways

- A PHP file can combine PHP instructions with HTML markup.
- The PHP runtime executes the PHP blocks before the response is sent.
- The browser displays the generated response, not the PHP source code.
- Variables let the page include values that can be changed in PHP.
- Escape dynamic text for the context where it will be displayed. Here, **htmlspecialchars** prepares text for HTML.
- A local PHP server lets a browser request and display a PHP page during development.

## Chapter navigation

| Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- |
| [001 - Introduction to PHP](../lessons/001-introduction-to-php.md) | [Open Q&A](../qnas/001-introduction-to-php.md) | [002 - How PHP fits into a web application](../lessons/002-how-php-fits-into-a-web-application.md) |
