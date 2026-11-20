# 006 - Set Up the Editor, Terminal, and Project Files: Questions and Answers

Use this Q&A to review [006 - Set Up the Editor, Terminal, and Project Files](../lessons/006-set-up-the-editor-terminal-and-project-files.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does a text editor do in a beginner PHP project?

A. It creates and edits source files<br>
B. It replaces the PHP runtime automatically<br>
C. It turns every file into a web page<br>
D. It starts a public production server

**Answer: A.** The editor is for authoring files. PHP must be installed separately to execute PHP code.

### 2. What does PHP syntax highlighting in an editor tell you?

A. The editor recognizes parts of the file as PHP for display<br>
B. The PHP file is guaranteed to run without errors<br>
C. A web server is already running<br>
D. The browser has executed the code

**Answer: A.** Syntax coloring helps people read the source. It does not execute or validate the program by itself.

### 3. Why is it useful to open the whole project folder in the editor?

A. It makes related files and their structure visible in one workspace<br>
B. It installs PHP automatically<br>
C. It prevents the need for a terminal<br>
D. It publishes the folder to the internet

**Answer: A.** Opening the folder makes it easier to navigate files and create them in the intended locations.

### 4. Which filename has the intended PHP source extension?

A. **index.php**<br>
B. **index.php.txt**<br>
C. **index.docx**<br>
D. **index.html.txt**

**Answer: A.** PHP source files should use the **.php** extension. On some systems, hidden extensions can make a file named **index.php.txt** look like **index.php**.

### 5. What does this command do?

~~~sh
php -l public/index.php
~~~

A. Checks the file's PHP syntax without executing it<br>
B. Starts a local web server<br>
C. Creates the **public** directory<br>
D. Runs the page in a browser

**Answer: A.** PHP's **-l** option performs a syntax check. It does not execute the program or confirm that its behavior is correct.

### 6. From which folder should you normally run the chapter's project commands?

A. The **php-practice** project root<br>
B. The editor's installation folder<br>
C. The browser's cache folder<br>
D. Any folder, because relative paths never depend on location

**Answer: A.** Starting at the project root makes paths such as **public/index.php** resolve in the way the lesson expects.

### 7. What is the purpose of the **public/** folder in the starter layout?

A. It holds pages and static files served by the local web server<br>
B. It stores the PHP executable<br>
C. It holds the editor's settings for every project<br>
D. It stores the project's operating system

**Answer: A.** The local server can use **public/** as its document root. The folder is a useful convention, not a requirement for every PHP project.

### 8. Which command displays the current folder in PowerShell?

A. **Get-Location**<br>
B. **php -l**<br>
C. **php -m**<br>
D. **New-Item**

**Answer: A.** **Get-Location** prints PowerShell's current working directory. On macOS or Linux, the similar command is **pwd**.

### 9. Which command starts the chapter's local server from the project root?

A. **php -S 127.0.0.1:8000 -t public**<br>
B. **php -l 127.0.0.1:8000**<br>
C. **public -S php**<br>
D. **php index.php -t 8000**

**Answer: A.** The **-S** option starts PHP's built-in server, and **-t public** selects the **public** directory as its document root.

### 10. Which statement about the starter project structure is correct?

A. It is a simple learning layout that can grow as needed<br>
B. Every PHP project must use exactly these folders<br>
C. The **src/** folder executes PHP automatically<br>
D. The **README.md** file is required by the PHP runtime

**Answer: A.** The layout is a helpful starting point, not a universal PHP requirement. Add folders when the project needs them.

### 11. What does a successful **php -l** check not guarantee?

A. That the program's behavior is correct at runtime<br>
B. That the parser found no syntax error in the checked source<br>
C. That the named file could be checked<br>
D. That PHP ran the syntax-check command

**Answer: A.** Syntax checking does not execute the program. It cannot guarantee correct output or detect every runtime problem.

### 12. Why choose a plain-text editor instead of a word processor for PHP source?

A. A word processor may add formatting that is not PHP source code<br>
B. PHP only runs if the editor has a paid license<br>
C. Word processors install a PHP web server<br>
D. PHP code cannot contain text

**Answer: A.** Source code should be saved as plain text. Rich-text formatting can add data that is not part of the program.

## True or false

### 1. An editor with PHP syntax colors has executed the file.

**Answer: False.** Highlighting changes how the source is displayed; the PHP runtime executes files.

### 2. A terminal's current working directory affects how relative paths are resolved.

**Answer: True.** A relative path is interpreted from the current folder, so commands should start from a known project root.

### 3. A file named **index.php.txt** is definitely a PHP file because its name contains **.php**.

**Answer: False.** Its final extension is **.txt**. Show file extensions and rename it to **index.php** if that was the intended name.

### 4. The project root in this lesson is the top-level **php-practice** folder.

**Answer: True.** It contains the project folders and README. Commands that use relative paths are run from there.

### 5. **php -l public/index.php** executes the page so you can check its output.

**Answer: False.** It checks PHP syntax without executing the file. Use CLI execution or the development server to inspect behavior.

### 6. A successful syntax check proves that every function used in the program exists and will work.

**Answer: False.** Some problems require running the program to discover. Syntax checking is not a complete static analysis or runtime test.

### 7. A **src/** directory is mandatory in every PHP project.

**Answer: False.** It is a common place to keep application source as a project grows, but a tiny practice project can start without it.

### 8. An editor's integrated terminal is required; a separate terminal cannot be used.

**Answer: False.** Either an integrated or separate terminal can run PHP commands. Know which folder that terminal is using.

### 9. The **public/** folder can be chosen as the development server's document root with **-t public**.

**Answer: True.** The path is relative to the folder from which the server command is started.

### 10. A project needs every possible folder and tool before it can run its first PHP file.

**Answer: False.** Start with the files and tools the exercise needs. Add structure and optional tooling when there is a reason.

## Short-answer questions

### 1. What are the three separate parts of the workflow introduced in this lesson?

**Answer:** The editor changes source files, the terminal starts commands, and the PHP runtime executes PHP code.

### 2. Why open a project folder instead of only opening one file?

**Answer:** The project view shows related files and folders together and makes it easier to create files in the correct place.

### 3. What is the project root in the example, and why does it matter?

**Answer:** The project root is **php-practice**, the top-level folder. Running commands there makes the example's relative paths predictable.

### 4. What is the difference between syntax highlighting and **php -l**?

**Answer:** Syntax highlighting colors text in the editor to aid reading. **php -l** asks PHP to parse the file for syntax errors without executing it.

### 5. What does a successful **php -l** result tell you, and what does it not tell you?

**Answer:** It tells you that PHP did not detect a syntax error in the checked file. It does not prove the program's output, runtime behavior, or web-server setup is correct.

### 6. Why should a beginner check the full filename and extension?

**Answer:** Some file browsers hide known extensions, so a file intended to be **index.php** may actually be **index.php.txt**. PHP tooling and servers expect the actual path and extension to match the intended file.

### 7. What does **public/** contain in this starter layout?

**Answer:** It contains the page and static files intended to be served by the local web server. The server can use it as the document root.

### 8. Why is **src/** included even though it is empty in the starter layout?

**Answer:** It shows a possible place for PHP application source as the project grows. The folder is optional and can be omitted until there are source files to put in it.

### 9. Which commands show the current directory in PowerShell and in macOS or Linux terminals?

**Answer:** Use **Get-Location** in PowerShell and **pwd** in macOS or Linux.

### 10. Why should source code be stored as plain text?

**Answer:** PHP tools expect source text. A word processor can add rich-text formatting that is not part of the PHP program.

## Troubleshooting practice

### 1. The editor colors **index.php**, but the terminal reports a syntax error. Does the highlighting prove the file is valid?

**Answer:** No. Syntax highlighting only affects display. Read the PHP parser message and inspect the indicated line and nearby code.

### 2. The file browser shows **index.php**, but PHP says the file cannot be found. What should you check?

**Answer:** Check whether the actual filename is **index.php.txt**, show file extensions, and verify the current directory and path passed to PHP.

**Explanation:** A hidden extension and an unexpected working directory can each make the displayed name or relative path misleading.

### 3. The command **php -l public/index.php** says the file cannot be opened, even though the file exists in the editor. What is likely wrong?

**Answer:** The terminal is probably not at the **php-practice** project root, or the relative path or filename spelling is wrong.

**Explanation:** Check **Get-Location** or **pwd**, then run the command from the project root or supply a path that matches the terminal's current folder.

### 4. **php -l** reports no syntax errors, but a browser page does not load. What should you do next?

**Answer:** Check that the development server is running, that it was started from the correct project folder with **-t public**, and that **public/index.php** exists. Then visit the server's HTTP address.

**Explanation:** Linting checks syntax only; it does not start a server or verify a browser request.

### 5. PHP commands work in a separate terminal but not in the editor's integrated terminal. What might differ?

**Answer:** The integrated terminal may have started before PATH was changed or may use a different shell environment. Restart it and check the PHP command and current folder again.

### 6. A teammate cannot run a project page after copying only **index.php**. What project setup might be missing?

**Answer:** They may also need the rest of the project files and the expected folder layout, including the **public/** document root. Share the project folder structure and the correct run instructions, not only one source file.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [005 - Run PHP from the command line and built-in server](../lessons/005-run-php-from-the-command-line-and-built-in-server.md) | [Read this lesson](../lessons/006-set-up-the-editor-terminal-and-project-files.md) | [Open example](../examples/006-set-up-the-editor-terminal-and-project-files.md) | [007 - PHP tags and embedding PHP in HTML](../lessons/007-php-tags-and-embedding-php-in-html.md) |
