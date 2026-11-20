# 005 - Run PHP from the Command Line and Built-in Server: Complete Example

This example creates one small PHP script for terminal output and one PHP page for a browser. It shows how the same PHP runtime can execute code directly or handle a local HTTP request.

## Question

How do you run a PHP script in a terminal, and how do you view a PHP-generated page in a browser without opening the source file directly?

## Goal

Create this project structure:

~~~text
php-practice/
├── hello.php
└── public/
    └── index.php
~~~

The terminal script prints a short message. The page in **public** returns an HTML document that contains PHP-generated text.

## Complete code: terminal script

Save this as **hello.php** in the **php-practice** folder:

~~~php
<?php

echo "Hello from PHP!", PHP_EOL;
~~~

## Complete code: browser page

Create a folder named **public** inside **php-practice**. Save this as **public/index.php**:

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
        <h1>My first PHP page</h1>
        <p>This page was served by PHP's built-in development server.</p>
        <p>Generated message: <?php echo "PHP is running"; ?></p>
    </main>
</body>
</html>
~~~

GitHub displays these code blocks as text. It does not run the PHP programs inside this Markdown page.

## Run the terminal script

1. Open a terminal in the **php-practice** folder.
2. Run:

~~~sh
php hello.php
~~~

Expected terminal output:

~~~text
Hello from PHP!
~~~

PHP executes the file and writes the message to the terminal. No browser or web server is involved in this step.

If the command cannot find the file, check that the terminal is in **php-practice**. A relative path is resolved from the current working directory.

## Start the local web server

Keep the terminal open and, from the **php-practice** folder, run:

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

The **-S** option starts PHP's built-in web server. The address binds the practice server to this computer, and **-t public** sets **public** as its document root.

Open this address in a browser:

~~~text
http://127.0.0.1:8000/
~~~

The server finds **index.php** in the document root, executes its PHP code, and returns the resulting HTML response. The browser should show:

~~~text
My first PHP page

This page was served by PHP's built-in development server.

Generated message: PHP is running
~~~

The exact font, margins, and spacing depend on the browser's default styles. The text should be the same.

When you finish, return to the terminal running the server and press **Ctrl+C**. The browser can no longer request pages from that stopped server process.

## What is happening?

### The terminal script

**php hello.php** asks the PHP CLI to execute **hello.php**. The **echo** statement writes text, and **PHP_EOL** adds the platform's usual line ending.

### The browser page

The browser sends an HTTP request to the local server. The server locates **index.php** under the **public** document root and executes its PHP code. The response contains HTML, including the text produced by **echo**. The browser renders that HTML; it does not execute the PHP source.

The local address **127.0.0.1** refers to this computer. Port **8000** identifies the local server endpoint. If that port is busy, start the server on another available port, for example **8001**, and browse to **http://127.0.0.1:8001/**.

## Compare the two outputs

| Program | Command or action | Result |
| --- | --- | --- |
| **hello.php** | Run **php hello.php** | Plain text appears in the terminal. |
| **public/index.php** | Start the server and visit its HTTP address | The browser renders an HTML page containing PHP output. |

The two programs use PHP, but the runtime context and destination of the output are different. A production web server may also use a PHP version or configuration separate from the CLI used in this example.

## Common mistakes

- **Opening index.php with file://:** This bypasses the PHP server. Start the server and visit its **http://** address instead.
- **Starting in the wrong folder:** The relative path **public** is resolved from the current working directory. Run the server command from **php-practice**.
- **Forgetting -t public:** Without that option, the current folder becomes the document root. The page would then be at **/public/** instead of the root URL.
- **Stopping the terminal process:** The server must remain running while the browser makes requests. Press **Ctrl+C** only when finished.
- **Using a busy port:** Choose a different available port and use the same port in the browser address.
- **Treating the built-in server as production hosting:** It is intended for local development and testing, not for a publicly exposed production site.

## Try changing it

1. Change the terminal message and run **php hello.php** again.
2. Change the **<title>** or heading in **public/index.php**, refresh the browser, and observe the updated response.
3. Add a second paragraph containing a different PHP **echo** expression.
4. Start the server without **-t public** and request **http://127.0.0.1:8000/public/**. Compare it with the root URL.
5. Stop the server and explain why the browser can no longer load the local page.
6. Explain why this example's output does not prove which PHP runtime a separately hosted website uses.

## Key takeaways

- **php hello.php** executes a script and prints its output in the terminal.
- The current working directory affects relative file paths.
- **php -S 127.0.0.1:8000 -t public** starts a local development server with **public** as the document root.
- Use the server's **http://** address so PHP can process the page before the browser receives it.
- Keep the server terminal open during practice and press **Ctrl+C** to stop it.
- The built-in server is for development and testing, not public production hosting.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [004 - Install PHP and inspect the runtime](../lessons/004-install-php-and-inspect-the-runtime.md) | [Read this lesson](../lessons/005-run-php-from-the-command-line-and-built-in-server.md) | [Open Q&A](../qnas/005-run-php-from-the-command-line-and-built-in-server.md) | [006 - Set up the editor, terminal, and project files](../lessons/006-set-up-the-editor-terminal-and-project-files.md) |
