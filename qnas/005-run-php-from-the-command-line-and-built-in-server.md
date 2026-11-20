# 005 - Run PHP from the Command Line and Built-in Server: Questions and Answers

Use this Q&A to review [005 - Run PHP from the Command Line and Built-in Server](../lessons/005-run-php-from-the-command-line-and-built-in-server.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which command executes a PHP file named **hello.php** from the current folder?

A. **php hello.php**<br>
B. **browser hello.php**<br>
C. **html hello.php**<br>
D. **php -S hello.php**

**Answer: A.** The PHP CLI reads and executes the named file. Its output appears in the terminal.

### 2. Where does output from **php hello.php** normally appear?

A. In the terminal<br>
B. In a browser tab automatically<br>
C. In the PHP source file<br>
D. In the web server's document root

**Answer: A.** A CLI script runs in the terminal and writes its output there. It does not create a browser request automatically.

### 3. What does a relative script path such as **hello.php** depend on?

A. The terminal's current working directory<br>
B. The browser's download folder<br>
C. The PHP version number<br>
D. The server's port

**Answer: A.** Relative paths are resolved from the folder where the command is run. Use the right working directory or pass the correct path to the file.

### 4. What does this command start?

~~~sh
php -S 127.0.0.1:8000
~~~

A. PHP's built-in web server on the local computer at port 8000<br>
B. A production server on every network interface<br>
C. A database server<br>
D. A PHP script named **127.0.0.1**

**Answer: A.** The **-S** option starts the built-in server. The address **127.0.0.1** is the local computer, and **8000** is the port.

### 5. What does **-t public** select in the command below?

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

A. The document root named **public**<br>
B. A PHP test suite<br>
C. A terminal window named public<br>
D. A PHP version called public

**Answer: A.** The **-t** option sets the directory from which the development server serves requested files. Here, **public** is relative to the current working directory.

### 6. What URL should you visit when the server is running at **127.0.0.1:8000**?

A. **http://127.0.0.1:8000/**<br>
B. **file://127.0.0.1:8000/**<br>
C. **php://127.0.0.1:8000/**<br>
D. **ftp://127.0.0.1:8000/**

**Answer: A.** The browser makes an HTTP request to the PHP server. A **file://** URL opens a local file and does not ask PHP's web server to execute it.

### 7. How do you stop the built-in server in the usual terminal session?

A. Press **Ctrl+C** in the terminal running it<br>
B. Delete **index.php**<br>
C. Close the browser tab only<br>
D. Run **php -v** in another terminal

**Answer: A.** The server keeps running in the terminal process. Pressing **Ctrl+C** interrupts that process and stops the server.

### 8. What is the built-in web server intended for?

A. Local development, testing, and controlled demonstrations<br>
B. Hosting every public production website<br>
C. Replacing every database server<br>
D. Compiling PHP into HTML files

**Answer: A.** It is a convenient development tool. It is not a full production web server and should not be exposed on a public network.

### 9. What can the root URL show when the document root contains **index.php**?

A. The response produced by that index page<br>
B. The terminal's current folder listing in every setup<br>
C. The PHP source code unconditionally<br>
D. The PHP installation program

**Answer: A.** The server uses an index file for the root request when one is present. PHP code in a requested PHP file is executed before the response is sent.

### 10. Why could command-line PHP and a hosted website behave differently?

A. They may use different PHP versions, configuration, or extensions<br>
B. A browser changes the PHP source file automatically<br>
C. PHP always disables extensions in a browser<br>
D. CLI commands cannot execute PHP

**Answer: A.** The PHP executable and configuration used by a web server can be separate from the one the terminal resolves.

### 11. Which statement about the command below is correct?

~~~sh
php -S 127.0.0.1:8000 -t public
~~~

A. The terminal should be kept open while the server is running<br>
B. It only runs a PHP file called **public** once<br>
C. It creates a public internet website automatically<br>
D. It runs without PHP installed

**Answer: A.** The server runs as a process attached to that terminal. Keep it running while making browser requests, then stop it with **Ctrl+C**.

### 12. What happens if port **8000** is already in use?

A. The server may fail to bind to that port; choose another available port<br>
B. PHP changes the document root automatically<br>
C. The browser will execute the PHP source directly<br>
D. The port is replaced with a random domain name

**Answer: A.** Two processes generally cannot listen on the same address and port at the same time. If the start command reports a bind error, try another available port and use that port in the browser URL.

## True or false

### 1. Running **php hello.php** requires a browser.

**Answer: False.** It executes the script through the CLI runtime, and output normally appears in the terminal.

### 2. A relative path is interpreted from the terminal's current working directory.

**Answer: True.** The same relative filename can refer to different locations if the command is run from different folders.

### 3. Opening **hello.php** with a **file://** URL is the same as requesting it from PHP's built-in server.

**Answer: False.** A **file://** URL reads a local file; an HTTP request to a PHP server lets PHP process a PHP page before returning a response.

### 4. Starting the server from a project folder without **-t** uses that current folder as the default document root.

**Answer: True.** The **-t** option is used when you want to specify a different document root explicitly.

### 5. A PHP source file requested through the PHP web server is sent to the browser unchanged for the browser to execute.

**Answer: False.** PHP executes server-side code and returns the response. The browser receives the response, such as generated HTML, rather than executing the PHP source.

### 6. The built-in server is intended for public production hosting.

**Answer: False.** It is intended for development, testing, or controlled demonstrations. Do not expose it on a public network.

### 7. The server's terminal can be closed or stopped while you continue expecting requests to succeed.

**Answer: False.** The server process must keep running to accept requests. Stopping it ends that local server session.

### 8. A PHP web server and PHP CLI can use different configuration files or extensions.

**Answer: True.** They can be separate PHP runtimes. Check the runtime actually used by the environment that runs the application.

## Short-answer questions

### 1. What is the difference between running PHP from the CLI and requesting a PHP page in a browser?

**Answer:** CLI execution runs a PHP file as a terminal process and writes output to the terminal. A browser request goes over HTTP to a server, which runs the PHP page and returns a response to the browser.

### 2. What does **127.0.0.1** mean in the local server address?

**Answer:** It is the loopback address for the same computer. It lets you practice by connecting to the server locally.

### 3. What is the document root?

**Answer:** It is the folder the web server uses as the base for mapping URL paths to files. The built-in server uses the current folder by default; **-t** can select another folder.

### 4. Why might a beginner put web-accessible pages in a **public** folder?

**Answer:** It makes the intended document root explicit and keeps other project files outside the folder the local server is serving as the website root.

### 5. What does the built-in server do with an **index.php** file at the document root?

**Answer:** It can use that index file for a request to the root URL, execute its PHP code, and return the resulting response.

### 6. Why is the built-in server not appropriate for public production hosting?

**Answer:** It is a development and testing tool, not a full production web server. It normally handles one request at a time in one process and should not be exposed on a public network.

### 7. Where can command-line script arguments be read in PHP?

**Answer:** In the **$argv** array. The script name is at index zero, and additional arguments follow it.

### 8. What should you check if **php hello.php** says it cannot open the file?

**Answer:** Check the terminal's current working directory and confirm the path to the file. Run the command in the file's folder or pass a correct relative or absolute path.

## Troubleshooting practice

### 1. You run **php hello.php**, but the shell says it cannot find **php**. What should you check?

**Answer:** Check that PHP CLI is installed and that the folder containing the PHP executable is on PATH. Open a new terminal after changing PATH and confirm the command resolves.

**Explanation:** Chapter 004 covers installing PHP and checking which executable the terminal can find.

### 2. The command runs, but PHP reports that **hello.php** cannot be opened. What is likely wrong?

**Answer:** The command is being run from a different folder than expected, or the path is incorrect.

**Explanation:** Show the current working directory, then either move into the folder containing the file or pass its correct path.

### 3. The browser says it cannot connect to **http://127.0.0.1:8000/**. What should you verify?

**Answer:** Confirm the development server is still running in its terminal, check the address and port printed by the server, and see whether another program already uses port 8000.

### 4. The root URL returns a not-found response even though **index.php** exists in the project. What should you inspect?

**Answer:** Check the document root and the location of **index.php**. If the server was started from the project root without **-t public**, a file under **public** is not at the root of the served directory. Start with **-t public** or request the matching URL path.

### 5. The browser displays raw PHP source after you open a file. What should you check?

**Answer:** Check whether you opened the file directly with **file://** or reached a server that is not configured to run PHP. Use the HTTP address of the running PHP server and make sure it serves the requested PHP file.

### 6. The local example works, but the deployed website reports a missing PHP extension. What could explain this?

**Answer:** The deployment environment may use a different PHP runtime or configuration from the local CLI or built-in server. Check the extensions available to the runtime that handles deployed web requests.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [004 - Install PHP and inspect the runtime](../lessons/004-install-php-and-inspect-the-runtime.md) | [Read this lesson](../lessons/005-run-php-from-the-command-line-and-built-in-server.md) | [Open example](../examples/005-run-php-from-the-command-line-and-built-in-server.md) | [006 - Set up the editor, terminal, and project files](../lessons/006-set-up-the-editor-terminal-and-project-files.md) |
