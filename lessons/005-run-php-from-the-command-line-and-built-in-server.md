# 005 - Run PHP from the Command Line and Built-in Server

After installing PHP, you can run a PHP file directly in a terminal or serve a small project through PHP's built-in development server. These are two different ways to execute PHP: one produces terminal output, and the other handles HTTP requests from a browser.

## What you will learn

- How to run a PHP script from a terminal
- How the command's working directory affects file paths
- How to start and stop PHP's built-in web server
- How to choose a document root
- How to distinguish CLI output from a web response
- Why the built-in server is for local development and testing

## Run a PHP file from a terminal

Create a file named **hello.php**:

~~~php
<?php

echo "Hello from PHP!", PHP_EOL;
~~~

Open a terminal in the folder that contains the file, then run:

~~~sh
php hello.php
~~~

The PHP command reads the source file, executes its PHP code, and writes the result to the terminal. The expected output is:

~~~text
Hello from PHP!
~~~

A browser is not involved. This way of running PHP is useful for small scripts, command-line tools, maintenance tasks, and learning the language.

### Check the current folder

A relative path such as **hello.php** is resolved from the terminal's current working directory. If PHP says it cannot open the file, check that the terminal is in the folder containing the file or pass the correct path.

For example, if the terminal is one folder above **project**, run:

~~~sh
php project/hello.php
~~~

On Windows PowerShell, **Get-Location** shows the current folder. On macOS or Linux, use **pwd**. Chapter 004 covered finding the PHP executable; if the shell cannot find **php**, check that installation and PATH setup first.

You can also pass values after the script filename. PHP makes script arguments available through **$argv**. In this chapter, focus on running a file; command-line arguments are covered later with command-line applications.

## Run a PHP page in a browser

A browser does not execute PHP source code by opening a local **.php** file. PHP must run in a PHP runtime that handles the request and sends a response to the browser. For practice, PHP includes a small built-in web server.

Create this project structure:

~~~text
php-practice/
├── hello.php
└── public/
    └── index.php
~~~

Save this as **public/index.php**:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PHP practice page</title>
</head>
<body>
    <main>
        <h1>Hello from PHP</h1>
        <p>This HTML page was returned by PHP's development server.</p>
        <p>PHP can insert a value here: <?php echo "PHP is running"; ?></p>
    </main>
</body>
</html>
~~~

Open a terminal in the **php-practice** project folder and start the server with **public** as its document root:

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

Keep that terminal open while the server is running. Visit **http://127.0.0.1:8000/** in a browser. The server finds **index.php**, executes its PHP code, and returns the resulting page. The browser displays the HTML response; it does not receive the PHP source code shown above.

To stop the server, return to its terminal and press **Ctrl+C**. If the address is already in use, choose another available port, such as **8001**, and visit **http://127.0.0.1:8001/** instead.

## Understand the address and document root

In **127.0.0.1:8000**, **127.0.0.1** means this computer and **8000** is the port used by the development server. The address is intended for local access. A URL beginning with **http://** makes a request to the running server; opening the same PHP file through a **file://** URL does not run it as a server-side PHP page.

The directory from which you start the server is the default document root. The **-t public** option explicitly makes the **public** folder the document root. This lets the project keep its web-accessible pages in one folder while other project files stay outside it.

If you leave off **-t public**, the server uses the current folder as its document root. Then the URL path maps to files under that folder. For example, when starting in **php-practice**, a file at **public/index.php** would be requested at **/public/index.php** unless **public** is selected as the document root.

An **index.php** file is served for the root URL when it exists in the document root. PHP executes PHP code in a requested PHP file and serves ordinary static files, such as CSS, as files. A path that does not match a file or route returns a not-found response.

## CLI and browser execution compared

| Run style | Example | Where output appears | Typical purpose |
| --- | --- | --- | --- |
| Command line | **php hello.php** | Terminal | Scripts and command-line tools |
| Built-in server | **php -S 127.0.0.1:8000 -t public** | Browser receives an HTTP response; server messages appear in the terminal | Local web-page development and testing |

Both use PHP code, but their environment and output differ. CLI output is not automatically an HTML page. A web request has HTTP details such as a URL, request method, headers, and a response. Later chapters explain the request and response lifecycle.

The command-line runtime and a web server can also use different PHP installations, configuration, or extensions. The built-in server uses the PHP executable that starts it, but a separate hosted website may use another runtime.

## Safe and appropriate use of the built-in server

PHP's built-in server is designed for development, testing, and controlled demonstrations. It is not a full production web server and should not be exposed on a public network. Bind to **127.0.0.1** for local practice, and do not use **0.0.0.0** to make it publicly reachable.

The server normally handles one request at a time in one process. A request that blocks can delay other requests. Use the production web-server setup appropriate to the application when deploying a site; that is covered later in this course.

## Common problems

| Symptom | What to check |
| --- | --- |
| The terminal cannot find **php**. | Check the installation and PATH setup from chapter 004, then open a fresh terminal. |
| PHP cannot open the script file. | Check the current working directory and the relative or absolute path you passed. |
| The browser downloads or displays PHP source instead of executing it. | Confirm that you opened an **http://** address served by PHP, not the file directly, and that the request reaches the PHP server. |
| The browser cannot connect. | Confirm the server is still running, use the same address and port shown in the command, and check whether another program is using that port. |
| The root URL does not show the intended page. | Confirm the document root is correct and that it contains **index.php** or **index.html**. |
| The browser shows a not-found response. | Check the URL path and that the requested file exists inside the document root. |
| A hosted site behaves differently from local CLI PHP. | The hosted web server may use a different PHP version, configuration, or set of extensions. |

## Try it yourself

1. Create **hello.php** and run it using **php hello.php**.
2. Run the same script from a different current directory by passing its relative path.
3. Create **public/index.php** with the sample HTML and PHP output.
4. Start the development server with **php -S 127.0.0.1:8000 -t public** and open the local URL in a browser.
5. Stop the server with **Ctrl+C**, then try the browser address again. Describe what changed.
6. Start the server without **-t public** and compare which URL reaches the page.
7. Explain why opening a PHP file through **file://** is different from requesting it through the PHP server.
8. In your own words, explain why the built-in server is suitable for practice but not public production hosting.

## Key points

- Use **php filename.php** to execute a PHP script in the terminal.
- Relative file paths are resolved from the current working directory.
- Use **php -S 127.0.0.1:8000** to start PHP's built-in development server.
- Use **-t directory** to choose an explicit document root, such as **public**.
- Browse to the server's **http://** address; directly opening a PHP source file does not execute it.
- Press **Ctrl+C** in the server terminal to stop the development server.
- The built-in server is for local development and testing, not public production use.
- CLI and web-server PHP may use different configurations and extensions.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [004 - Install PHP and inspect the runtime](004-install-php-and-inspect-the-runtime.md) | [Open Q&A](../qnas/005-run-php-from-the-command-line-and-built-in-server.md) | [Open example](../examples/005-run-php-from-the-command-line-and-built-in-server.md) | [006 - Set up the editor, terminal, and project files](006-set-up-the-editor-terminal-and-project-files.md) |
