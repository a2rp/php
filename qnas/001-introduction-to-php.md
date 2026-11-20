# 001 - Introduction to PHP: Questions and Answers

This Q&A reviews the ideas from [001 - Introduction to PHP](../lessons/001-introduction-to-php.md). Each answer follows its question so you can check your understanding as you go.

## Multiple-choice questions

### 1. What is PHP?

A. A browser that displays websites  <br>
B. A general-purpose programming language often used for server-side web development  <br>
C. A database  <br>
D. A web browser styling format<br>

**Answer: B.** PHP is a programming language. It can create web responses and can also run scripts outside a browser.

### 2. What does the modern PHP acronym stand for?

A. Personal Home Page  <br>
B. Program Hosting Pages  <br>
C. PHP: Hypertext Preprocessor  <br>
D. Page Handler Protocol<br>

**Answer: C.** PHP is a recursive acronym for PHP: Hypertext Preprocessor. Personal Home Page was the original name.

### 3. In a typical PHP website, where does the PHP source code run?

A. In the visitor's browser  <br>
B. In the PHP runtime on the server side  <br>
C. In the HTML parser  <br>
D. In the CSS engine<br>

**Answer: B.** The web server arranges for the PHP runtime to execute the PHP file. The server then sends the generated response to the browser.

### 4. What does the browser normally receive from a PHP page?

A. The PHP source exactly as it was written  <br>
B. The PHP runtime itself  <br>
C. The response produced by PHP, often HTML  <br>
D. A database server<br>

**Answer: C.** PHP runs before the response reaches the browser. For a normal page, the browser receives the resulting HTML, not the PHP source code.

### 5. Which is a valid PHP variable name from the introduction?

A. name  <br>
B. $name  <br>
C. name$  <br>
D. #name<br>

**Answer: B.** PHP variable names begin with a dollar sign, as in **$name**.

### 6. What is the role of echo in the first example?

A. It assigns a value to a variable  <br>
B. It sends text to the current output  <br>
C. It starts the PHP runtime  <br>
D. It creates an HTML file on disk<br>

**Answer: B.** The example uses **echo** to write the greeting to the program's output.

### 7. Which statement about PHP is correct?

A. Every PHP program needs a framework.  <br>
B. Every PHP program needs a database.  <br>
C. PHP can also run from a command line.  <br>
D. PHP is only used to style HTML.<br>

**Answer: C.** PHP can run in a command-line environment without a browser or web server. Frameworks and databases are optional tools.

### 8. What is a framework?

A. The PHP language itself  <br>
B. A set of tools and conventions that can be built with PHP  <br>
C. A browser request  <br>
D. A PHP variable<br>

**Answer: B.** A framework is an additional tool that can help organize an application. It is separate from the PHP language.

### 9. Which file is normally treated as PHP source by a PHP-capable server?

A. A file with the .php extension  <br>
B. A file with the .jpg extension  <br>
C. A file with the .css extension  <br>
D. Any file opened directly in a browser<br>

**Answer: A.** PHP files commonly use the .php extension. A PHP-capable server must be configured to pass the file to PHP for execution.

### 10. Which output could a PHP program produce?

A. HTML only  <br>
B. JSON only  <br>
C. Plain text only  <br>
D. HTML, JSON, plain text, or other output<br>

**Answer: D.** PHP can produce different kinds of output. The program and its environment determine what response or result it creates.

## True or false

### 1. A browser can execute PHP source code by itself.

**Answer: False.** PHP must run in a PHP-capable environment. In a typical website that happens on the server; the browser receives the generated response.

### 2. PHP can be used without a web browser.

**Answer: True.** PHP can run from the command line, for example as a script or automation tool.

### 3. HTML and PHP are the same language.

**Answer: False.** HTML describes document structure. PHP is a programming language that can generate HTML among other kinds of output.

### 4. A PHP application automatically becomes secure because the code runs on the server.

**Answer: False.** Server-side execution does not make unsafe input handling or output safe. Developers still need to apply security practices.

### 5. A PHP framework and the PHP language are the same thing.

**Answer: False.** A framework is an optional set of tools built with or for PHP. The language can be used without a framework.

## Short-answer questions

### 1. In one or two sentences, what is PHP?

**Answer:** PHP is a general-purpose programming language commonly used to generate dynamic web responses. It can also run command-line programs and other scripts.

### 2. Describe the path from a browser request to a generated PHP page.

**Answer:** The browser requests a page from a web server. The server passes the PHP file to the PHP runtime, which executes it and produces a response. The server sends that response back to the browser to display.

### 3. Why does the browser normally not receive the PHP source?

**Answer:** The PHP runtime processes the source on the server before the response is sent. The browser receives the generated output, such as HTML, rather than the source instructions.

### 4. What does the dollar sign mean in $name?

**Answer:** It marks the name as a PHP variable. PHP variable names begin with a dollar sign.

### 5. Name two kinds of output a PHP program can produce.

**Answer:** For example, HTML and JSON. A PHP program can also produce plain text or other output.

### 6. What is one difference between PHP and a PHP framework?

**Answer:** PHP is the programming language. A framework is an additional collection of tools and conventions that may help build an application with PHP.

## Read the code

### 1. What does this program output?

~~~php
<?php

$name = 'Asha';
echo 'Hello, ', $name, '!';
~~~

**Answer:**

~~~text
Hello, Asha!
~~~

**Explanation:** The assignment stores the string **Asha** in **$name**. The **echo** statement writes the surrounding text and the variable's value in order.

### 2. What appears inside the heading after PHP processes this page?

~~~php
<h1><?php echo 'Welcome'; ?></h1>
~~~

**Answer:** The heading contains **Welcome**. PHP executes the code block and writes that text into the response; a browser then interprets the resulting HTML.

## Practice exercises with answers

### 1. Write a script that prints your name.

**Answer:**

~~~php
<?php

$name = 'Riya';
echo $name;
~~~

**Explanation:** Replace **Riya** with a name of your choice. The variable stores the string, and **echo** sends it to output.

### 2. Write a script that prints a short two-part greeting.

**Answer:**

~~~php
<?php

$name = 'Riya';
echo 'Hello, ', $name, '!';
~~~

**Expected output:**

~~~text
Hello, Riya!
~~~

**Explanation:** **echo** can write multiple values in sequence. This example keeps the greeting text and variable value as separate arguments.

### 3. In a typical website, identify which parts belong to the server and which part belongs to the visitor.

**Answer:** The web server and PHP runtime process the PHP source and create a response. The visitor's browser receives and displays that response.

**Explanation:** This distinction helps explain why changing PHP code requires running it through a PHP-capable environment before the page output changes.

## Chapter navigation

| Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- |
| [001 - Introduction to PHP](../lessons/001-introduction-to-php.md) | [Open example](../examples/001-introduction-to-php.md) | [002 - How PHP fits into a web application](../lessons/002-how-php-fits-into-a-web-application.md) |
