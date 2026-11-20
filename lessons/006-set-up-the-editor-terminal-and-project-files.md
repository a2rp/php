# 006 - Set Up the Editor, Terminal, and Project Files

A useful PHP workspace has three parts: an editor for changing files, a terminal for running PHP commands, and a project folder that keeps related files together. These are separate tools and responsibilities. The editor does not execute PHP just because it can color PHP syntax.

## What you will learn

- How to choose and prepare an editor for PHP files
- How to open the project folder instead of isolated files
- How to use a terminal from the project root
- How to name and organize starter project files
- How to check that a PHP file parses without running it
- How to spot common file-extension and path mistakes

## The editor writes files; PHP executes them

Use a text editor that can save plain-text files and recognize PHP syntax. PHP syntax highlighting makes code easier to scan, but it is only a display feature: it does not prove that PHP can execute the file or that the file has valid syntax.

PHP itself is the runtime installed in chapter 004. A terminal starts PHP commands, such as **php filename.php** from chapter 005. The editor, terminal, and PHP runtime work together, but they are not the same program.

You can use an editor's integrated terminal or a separate terminal window. Either is fine. The important part is knowing which project folder the terminal is currently using.

## Open a project folder

Create a folder for your work, for example **php-practice**. Open that folder in your editor as the workspace. Opening the whole folder makes it easier to see the project structure and create files in the right place.

A small starter project can use this layout:

~~~text
php-practice/
├── public/
│   └── index.php
├── src/
└── README.md
~~~

- **public/** contains pages and static files the local web server should serve.
- **src/** is a place for PHP application source files as the project grows. It can remain empty in this first setup.
- **README.md** can hold notes about the project and how to run it.
- **public/index.php** is the starter page. The next chapter explains PHP tags and how PHP code can be embedded in an HTML document.

This is a learning layout, not a rule that every PHP project must follow. Do not create folders you do not need yet. As the project grows, its structure can evolve to match the application and tools it uses.

## Create the starter folders

In the editor, you can create **public**, **src**, and **README.md** in the project explorer. Or open a terminal at the **php-practice** project root and create the directories from there.

In PowerShell:

~~~powershell
New-Item -ItemType Directory -Force public, src
~~~

In macOS or Linux terminal:

~~~sh
mkdir -p public src
~~~

Create **public/index.php** and **README.md** in the editor. You do not need a special PHP project generator for this small exercise.

## Save real PHP files

Save PHP source files with the **.php** extension, for example **index.php**. Some operating systems hide known file extensions. If your editor or file browser hides extensions, make sure the filename is not accidentally **index.php.txt**.

Use a plain-text source file rather than a word-processing document. A word processor may add formatting that is not part of the PHP code. For a beginner project, choose a consistent encoding such as UTF-8 and let the editor use its normal source-code line endings.

Keep filenames simple and descriptive. For example:

- **index.php** for a starting page
- **greeting.php** for a file whose purpose is to show a greeting
- **README.md** for project notes

File and directory names are part of paths. Use the exact spelling and capitalization when you refer to them, especially if the project will also run on a case-sensitive system.

## Use the terminal from the project root

The **project root** is the top-level **php-practice** folder. Start the terminal there so relative paths such as **public/index.php** resolve as expected.

To see the current folder in PowerShell, run:

~~~powershell
Get-Location
~~~

On macOS or Linux, run:

~~~sh
pwd
~~~

From the project root, confirm PHP is available:

~~~sh
php -v
~~~

Then ask PHP to check the syntax of the starter page:

~~~sh
php -l public/index.php
~~~

The **-l** option checks PHP syntax without executing the file. A successful check reports that no syntax errors were detected. It does not verify behavior, find every runtime problem, or prove that a web page works in a browser.

When you want to run a PHP script or view a local PHP page, use the approaches from [chapter 005](005-run-php-from-the-command-line-and-built-in-server.md). For example, run **php filename.php** for a CLI script, or start the built-in server with **public** as its document root:

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

Then visit **http://127.0.0.1:8000/** in a browser. Keep the terminal in the project root and leave the server running while you use the page.

## A simple file workflow

Use this repeatable workflow as you learn:

1. Open the **php-practice** folder in the editor.
2. Create or select a **.php** file inside the project.
3. Save the file before running it.
4. Open a terminal in the project root and confirm the current folder.
5. Run **php -l path/to/file.php** to check its PHP syntax.
6. Run the script from the terminal or serve its page through the local PHP server, depending on what the file is for.
7. Read the output and any error message, edit the source, save, and repeat.

If the editor has a PHP language extension, formatter, or debugger, you may add one when you need it. These are optional conveniences. Start with the PHP runtime, a text editor, and a terminal before adding more tools.

## Common setup problems

| Symptom | What to check |
| --- | --- |
| The editor displays the source as plain text. | Confirm the filename ends in **.php** and the editor recognizes the language mode. Syntax colors are helpful but not required to run PHP. |
| The editor says the file is **index.php.txt**. | Show file extensions and rename it to **index.php**. |
| The terminal cannot find **php**. | Check PHP installation and PATH from chapter 004, then open a fresh terminal. |
| PHP cannot find the file passed to it. | Check the current working directory and spelling of the relative path. |
| **php -l** succeeds but the page is wrong. | Syntax checking does not execute the page or verify its behavior. Run it in the appropriate CLI or web context and inspect the result. |
| The local server cannot find **index.php**. | Confirm that the document root points to **public** and that **public/index.php** exists. |
| A path works on one computer but not another. | Check spelling and capitalization. Filesystems can treat letter case differently. |

## Try it yourself

1. Create a **php-practice** folder and open it as the editor workspace.
2. Add **public/**, **src/**, **public/index.php**, and **README.md**.
3. Open a terminal and confirm that its current directory is **php-practice**.
4. Run **php -v** to confirm the terminal can find PHP.
5. Run **php -l public/index.php** and describe what the check does and does not tell you.
6. Rename a temporary file to **index.php.txt**, inspect how your system displays it, then correct its extension.
7. Start the built-in server from the project root with **-t public** and open the local page in a browser.
8. Explain the difference between syntax highlighting in an editor and syntax checking by PHP.

## Key points

- The editor writes source files; the PHP runtime executes them.
- Open the whole project folder so its files and paths are easy to navigate.
- Save PHP source code with the **.php** extension and check that hidden extensions have not added **.txt**.
- Start terminal commands from a known project root so relative paths resolve predictably.
- Use **php -l filename.php** to check syntax without executing the file.
- A successful syntax check does not prove the program behaves correctly.
- Keep the starter project small and add folders or tools as the project needs them.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [005 - Run PHP from the command line and built-in server](005-run-php-from-the-command-line-and-built-in-server.md) | [Open Q&A](../qnas/006-set-up-the-editor-terminal-and-project-files.md) | [Open example](../examples/006-set-up-the-editor-terminal-and-project-files.md) | [007 - PHP tags and embedding PHP in HTML](007-php-tags-and-embedding-php-in-html.md) |
