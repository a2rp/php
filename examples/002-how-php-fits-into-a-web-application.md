# 002 - How PHP Fits into a Web Application: Complete Example

This example follows a browser request through a small PHP application. The page reads a name from the URL, prepares an HTML response, and sends it back to the browser.

## Question

How can a PHP application use a value from a request to create a response that changes for each visitor?

## Goal

Build a greeting page that reads a name from the query string. If no usable name is provided, the page greets the visitor as **visitor**. The value is escaped before it is included in HTML.

## Complete code

Save this as **index.php** in a practice folder:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PHP request and response</title>
</head>
<body>
    <main>
        <?php
        $name = $_GET['name'] ?? 'visitor';

        if (!is_string($name)) {
            $name = 'visitor';
        }

        $name = trim($name);

        if ($name === '') {
            $name = 'visitor';
        }
        ?>
        <h1>Hello, <?php echo htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8'); ?>!</h1>
        <p>This greeting was prepared by a PHP application.</p>
    </main>
</body>
</html>
~~~

GitHub displays this code as text. It does not run PHP or make the greeting interactive on this page.

## Run it locally

Use a PHP installation and its local development server:

1. Save the code as **index.php** in a practice folder.
2. Open a terminal in that folder.
3. Start the server:

~~~sh
php -S 127.0.0.1:8000
~~~

4. In a browser, open **http://127.0.0.1:8000/?name=Asha**.
5. Try changing **Asha** to another name. Also open **http://127.0.0.1:8000/** without a name.
6. Stop the local server with **Ctrl+C** in the terminal.

The PHP development server is for local practice. It is not intended to be a production hosting server.

## Expected browser output

With the name **Asha**, the page body displays:

~~~text
Hello, Asha!
This greeting was prepared by a PHP application.
~~~

Without a name, it displays:

~~~text
Hello, visitor!
This greeting was prepared by a PHP application.
~~~

The browser tab displays **PHP request and response**.

## Follow the request through the application

For the URL **http://127.0.0.1:8000/?name=Asha**:

1. The browser sends an HTTP request to the local web server.
2. The query string contains the value **name=Asha**.
3. The web server sends the request to the PHP runtime for this PHP page.
4. PHP reads the query value from **$_GET** and prepares the greeting.
5. PHP produces HTML. The web server returns that response to the browser.
6. The browser displays the returned HTML.

The query value is user input, even when the example URL was typed by you. The application checks that the value is a string and uses a default for missing, non-string, or blank values. A real application may also need length limits and rules specific to the task.

## How the code works

### Read an optional query value

~~~php
$name = $_GET['name'] ?? 'visitor';
~~~

The **$_GET** variable contains values supplied in a URL query string. The **??** operator uses **visitor** when the name key is missing or null. This is a preview of PHP request input; a later lesson covers superglobals in more depth.

### Check and normalize the value

~~~php
if (!is_string($name)) {
    $name = 'visitor';
}

$name = trim($name);

if ($name === '') {
    $name = 'visitor';
}
~~~

Query parameters can have unexpected shapes, including array values. The string check ensures **trim** receives a string. Trimming removes leading and trailing whitespace, and the final condition handles an empty name.

### Escape the value for HTML

~~~php
<?php echo htmlspecialchars($name, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8'); ?>
~~~

The value is inserted into an HTML text node, so **htmlspecialchars** converts special HTML characters to entities. This helps the browser display the value as text rather than interpreting it as markup. Escaping is context-specific; a different output context may need a different approach.

## What this demonstrates

- The browser sends a request with a URL and query string.
- The web server routes the request to the PHP page.
- PHP reads input, applies application logic, and prepares the response.
- The browser receives the generated HTML, not the PHP source.
- A database is unnecessary for this small example.
- Request values need appropriate checks and output handling.

## Try changing it

1. Change the default greeting from **visitor** to another word.
2. Add a second query parameter called **course** and display it in a paragraph.
3. Explain what happens when the name is missing or blank.
4. Explain why the value is passed through **htmlspecialchars** before it is written into the page.
5. Identify which part of the example is the browser, the web server, the PHP application, and the response.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [001 - Introduction to PHP](../lessons/001-introduction-to-php.md) | [Read this lesson](../lessons/002-how-php-fits-into-a-web-application.md) | [Open Q&A](../qnas/002-how-php-fits-into-a-web-application.md) | [003 - PHP releases, versions, and compatibility](../lessons/003-php-releases-versions-and-compatibility.md) |
