# 006 - Set Up the Editor, Terminal, and Project Files: Complete Example

This example sets up a small PHP project that has a clear root folder, one command-line script, one page served from **public/**, and a short README with run instructions.

## Question

How can you organize a beginner PHP project so that its files are easy to edit, check, run, and share?

## Goal

Create this project:

~~~text
php-practice/
├── README.md
├── hello.php
└── public/
    └── index.php
~~~

The **php-practice/** folder is the project root. **hello.php** is run from the terminal. **public/index.php** is the page served to a browser.

## Create the project files

1. Create a folder named **php-practice** and open it in your editor.
2. Create a folder named **public** inside it.
3. Create **hello.php** and **README.md** at the project root.
4. Create **index.php** inside **public/**.
5. Confirm that the filenames end with **.php** and **.md**, not an extra **.txt** extension.

To create **public/** from a terminal already opened at the project root, use PowerShell:

~~~powershell
New-Item -ItemType Directory -Force public
~~~

Or use macOS or Linux:

~~~sh
mkdir -p public
~~~

## Complete code: hello.php

Save this as **hello.php** in the project root:

~~~php
<?php

$person = "Learner";
echo "Hello, ", $person, "!", PHP_EOL;
~~~

This is a terminal script. It writes a greeting to the command line.

## Complete code: public/index.php

Save this as **public/index.php**:

~~~php
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PHP practice project</title>
</head>
<body>
    <main>
        <h1>PHP practice project</h1>
        <p>This page is organized inside the public folder.</p>
        <p><?php echo "PHP generated this line."; ?></p>
    </main>
</body>
</html>
~~~

This page contains HTML and a small PHP output expression. The next lesson explains PHP tags and embedding PHP in HTML in more detail.

## Complete code: README.md

Save this as **README.md** at the project root:

~~~markdown
# PHP Practice Project

A small beginner project with one CLI script and one local PHP page.

## Project files

- `hello.php` prints a greeting in the terminal.
- `public/index.php` is the page served by PHP's local development server.

## Check and run the CLI script

From this project folder:

```sh
php -l hello.php
php hello.php
```

## Check and serve the web page

From this project folder:

```sh
php -l public/index.php
php -S 127.0.0.1:8000 -t public
```

Open `http://127.0.0.1:8000/` in a browser. Press Ctrl+C in the server terminal when finished.

The built-in server is for local development and testing.
~~~

GitHub shows this README code block as text here. The Markdown page does not create or run these files for you; save each code block under the filename shown above.

## Work from the project root

Open a terminal at **php-practice/**. Confirm the current location before using relative paths.

In PowerShell:

~~~powershell
Get-Location
~~~

In macOS or Linux:

~~~sh
pwd
~~~

The final path should end in **php-practice**. From that folder, these paths refer to the project files:

- **hello.php**
- **public/index.php**

## Check the PHP syntax

Ask PHP to check both files without executing them:

~~~sh
php -l hello.php
php -l public/index.php
~~~

For each valid file, PHP reports that no syntax errors were detected. A syntax check does not print the greeting, render the page, or guarantee that the program behaves correctly.

## Run the CLI script

From **php-practice/**, run:

~~~sh
php hello.php
~~~

Expected terminal output:

~~~text
Hello, Learner!
~~~

The PHP runtime executes the source file and prints the greeting. If the file cannot be found, verify that the terminal is at the project root and that the filename is spelled correctly.

## Serve the page locally

From the same project root, start the local development server:

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

Keep the terminal open, then visit:

~~~text
http://127.0.0.1:8000/
~~~

The browser should show:

~~~text
PHP practice project

This page is organized inside the public folder.

PHP generated this line.
~~~

The browser receives and renders the page response. It does not execute the PHP source in your editor. When finished, stop the server by pressing **Ctrl+C** in its terminal.

## Why these files are separated

- The project root keeps the files for this exercise together.
- **hello.php** shows that PHP can run as a CLI script without a browser.
- **public/index.php** is under the local server's document root, so it is reachable at the root URL.
- **README.md** records the file purposes and the commands needed to work with the project.

This structure is intentionally small. Larger applications may add folders such as **src/** for application code and other folders for assets, tests, or configuration. Add them when the project needs them.

## Common mistakes

- **The terminal cannot find hello.php:** Run the command from **php-practice/** or pass the correct path.
- **The page returns not found:** Confirm the server was started from the project root with **-t public**, and confirm **public/index.php** exists.
- **The file is named index.php.txt:** Show file extensions in the file browser and rename it with the intended **.php** extension.
- **The browser cannot connect:** Keep the server process running and use the address and port from the start command.
- **Syntax check succeeds, but output is wrong:** The syntax check does not execute the code; run the CLI script or request the page through the local server.
- **The page's PHP source appears as text:** Make sure the browser is using the running PHP server's **http://** address rather than opening the source file directly.

## Try changing it

1. Change **$person** in **hello.php**, save, then run **php hello.php** again.
2. Change the heading in **public/index.php**, save, and refresh the browser page.
3. Add a short instruction to **README.md** and check that it remains readable Markdown.
4. Run **php -l** on each PHP file after making a change.
5. Run the server from the wrong folder once, note the path problem, and then return to the project root.
6. Explain what the editor, terminal, and PHP runtime each do in this project.

## Key takeaways

- Keep related files inside a named project folder and open that folder in the editor.
- Use **.php** for PHP source and check that the actual filename has no hidden extra extension.
- Run commands from the project root so relative paths such as **public/index.php** are clear.
- Use **php -l filename.php** for a syntax check; it does not execute the file.
- Keep browser pages under the document root selected with **-t public**.
- Use **README.md** to record how to check and run a small project.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [005 - Run PHP from the command line and built-in server](../lessons/005-run-php-from-the-command-line-and-built-in-server.md) | [Read this lesson](../lessons/006-set-up-the-editor-terminal-and-project-files.md) | [Open Q&A](../qnas/006-set-up-the-editor-terminal-and-project-files.md) | [007 - PHP tags and embedding PHP in HTML](../lessons/007-php-tags-and-embedding-php-in-html.md) |
