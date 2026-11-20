# 007 - PHP Tags and Embedding PHP in HTML: Questions and Answers

Use this Q&A to review [007 - PHP Tags and Embedding PHP in HTML](../lessons/007-php-tags-and-embedding-php-in-html.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which tag opens a standard PHP code block?

A. **<?php**<br>
B. **<php>**<br>
C. **<script php>**<br>
D. **<?**

**Answer: A.** Use **<?php** as the standard opening tag. The legacy short opening tag **<?** may not be enabled on every PHP installation.

### 2. What does **?>** do in a mixed PHP and HTML file?

A. Closes the current PHP block so the file can continue with non-PHP output<br>
B. Ends the browser session<br>
C. Starts a PHP comment<br>
D. Sends the file to the editor

**Answer: A.** PHP stops interpreting source as PHP at the closing tag. In a template, this lets the file continue with HTML output.

### 3. What is sent as the response for HTML text written outside PHP tags in a processed **.php** page?

A. The HTML text is part of the output returned to the browser<br>
B. The HTML text is always deleted<br>
C. The HTML text is run as a CLI command<br>
D. The PHP source is sent for the browser to execute

**Answer: A.** In a mixed file, PHP processes PHP blocks and passes the surrounding text through as output.

### 4. What does this syntax do?

~~~php
<?= $pageTitle ?>
~~~

A. Outputs the value of the expression<br>
B. Opens a PHP comment<br>
C. Defines a new HTML element<br>
D. Disables PHP output

**Answer: A.** **<?= ... ?>** is shorthand for **<?php echo ...; ?>** and is useful for inserting a value into a template.

### 5. Which statement about the short echo tag is correct?

A. **<?= ... ?>** works even when **short_open_tag** is disabled<br>
B. It is the same as the legacy **<?** tag<br>
C. It can only be used in command-line scripts<br>
D. It turns PHP into JavaScript

**Answer: A.** The short echo tag is distinct from the legacy short opening tag **<?** and is available independently of the **short_open_tag** configuration.

### 6. Why should a beginner avoid using **<?**?

A. It may be unavailable depending on the PHP build or configuration<br>
B. It always closes an HTML document<br>
C. It only works inside CSS<br>
D. It cannot be typed in a text editor

**Answer: A.** Use **<?php** for a portable standard opening tag. The legacy short tag can be disabled.

### 7. What is the recommended ending for a PHP-only source file?

A. Omit the final **?>** closing tag<br>
B. Add a second **<?php** tag at the very end<br>
C. End with an HTML **</body>** element<br>
D. Replace the PHP file extension with **.html**

**Answer: A.** Omitting the final closing tag helps avoid accidental whitespace or a newline being emitted after PHP-only code.

### 8. What should you do before writing ordinary HTML after a PHP block?

A. Close the PHP block with **?>**<br>
B. Put the HTML inside a PHP variable in every case<br>
C. Rename the file to **.txt**<br>
D. Add **<?** before each HTML element

**Answer: A.** Closing the PHP block returns the file to non-PHP output, where you can write the HTML markup directly.

### 9. What does **echo** do to a value?

A. Outputs it; it does not automatically escape it for HTML<br>
B. Always validates it as safe HTML<br>
C. Converts it into a PHP comment<br>
D. Hides it from the server

**Answer: A.** **echo** outputs a value. It does not perform context-sensitive escaping, so untrusted values need appropriate handling before being inserted into markup.

### 10. How should you normally view a PHP-generated web page?

A. Request it through a PHP-enabled web server using an **http://** URL<br>
B. Open its source file with **file://** and expect PHP to execute<br>
C. Paste its source code into the browser address bar<br>
D. Rename it to **.jpg**

**Answer: A.** The server executes PHP and returns a response. A browser does not execute server-side PHP from a local file URL.

### 11. A file contains a PHP block and HTML after it. Which extension is appropriate for this lesson's page?

A. **.php**<br>
B. **.txt**<br>
C. **.php.txt**<br>
D. **.docx**

**Answer: A.** Save the mixed template with a **.php** extension and request it through a PHP-enabled server. A filename alone still does not configure an unrelated server to process PHP.

## True or false

### 1. The standard PHP opening tag is **<?php**.

**Answer: True.** This is the normal portable tag used to begin a PHP block.

### 2. HTML written outside PHP tags in a processed PHP template is sent as part of the output.

**Answer: True.** PHP code is interpreted inside its tags; the surrounding text is passed through as output.

### 3. The short echo tag **<?= ... ?>** is the same as legacy **<?**.

**Answer: False.** The short echo tag outputs an expression. The legacy **<?** is a short opening tag whose availability may depend on configuration.

### 4. A pure PHP source file should normally include **?>** as its last characters.

**Answer: False.** It is preferable to omit the final closing tag in a PHP-only file to avoid accidental trailing output.

### 5. To resume writing HTML after an open PHP block, the file must leave PHP mode.

**Answer: True.** Use **?>** at the boundary, then write the HTML outside the PHP block.

### 6. **echo** automatically escapes a value so it is always safe to insert into HTML.

**Answer: False.** **echo** only outputs the value. Appropriate escaping depends on the output context.

### 7. Opening a PHP source file directly with **file://** runs its PHP code.

**Answer: False.** PHP runs on the server. Use the HTTP address of a PHP-enabled server to request the page.

### 8. The browser normally receives the PHP source statements from a PHP-enabled server and executes them itself.

**Answer: False.** The server processes PHP first and returns the resulting response for the browser to render.

### 9. The opening tag **<?php** must be followed by whitespace before PHP code begins.

**Answer: True.** A space, tab, or newline separates the tag from the PHP code that follows.

## Short-answer questions

### 1. What do the standard PHP opening and closing tags mark?

**Answer:** **<?php** marks the start of a PHP block, and **?>** marks its end. In a mixed file, text outside the block can be returned as output.

### 2. What is the difference between **<?php echo $title; ?>** and **<?= $title ?>**?

**Answer:** Both output the value of **$title**. The second is the short echo form and is often convenient inside a template.

### 3. Why should you use **<?php** instead of **<?**?

**Answer:** **<?php** is the standard portable opening tag. The legacy **<?** tag can depend on the PHP build or **short_open_tag** configuration.

### 4. When should a mixed PHP and HTML page use a closing tag?

**Answer:** Use **?>** when the source needs to leave PHP mode and continue with HTML output. A PHP-only file that ends while still in PHP code should normally omit its final closing tag.

### 5. Why does PHP run on a server rather than in the browser?

**Answer:** PHP is a server-side language. The PHP runtime processes the file and returns a response, such as HTML, which the browser then renders.

### 6. Why is it inaccurate to say that **echo** makes a value safe for HTML?

**Answer:** **echo** only outputs a value. It does not escape special characters or adapt the value to the context where it is inserted.

### 7. What does the PHP runtime do with HTML text outside a PHP block in a mixed template?

**Answer:** It passes that text through as part of the output response while executing the PHP blocks in the file.

### 8. Why may a PHP file that opens correctly in an editor still not work in a browser?

**Answer:** An editor only edits or displays the source. The page must be requested through a server configured to run PHP; opening the file directly does not execute server-side code.

## Troubleshooting practice

### 1. PHP reports a syntax error after an HTML heading in your **.php** file. What should you check?

**Answer:** Check whether the preceding PHP block was closed with **?>** before the heading.

**Explanation:** If PHP mode is still open, the parser treats the heading as PHP source instead of HTML output.

### 2. A page displays the literal text **<?php echo $name; ?>**. What might be wrong?

**Answer:** The file may not be processed by a PHP-enabled server, or it may have been opened directly as a local file. Request it through the local PHP server using its **http://** address.

### 3. A file containing **<? echo $message; ?>** works on one machine but not another. Why?

**Answer:** The legacy short opening tag may be enabled on one PHP setup and disabled on the other. Replace it with **<?php echo $message; ?>** or **<?= $message ?>**.

### 4. A PHP-only file unexpectedly sends whitespace before a later response header. What convention can help avoid this?

**Answer:** Omit the final **?>** closing tag in PHP-only source files and check for whitespace outside any PHP block.

### 5. A value from an external source is inserted using **<?= $value ?>**. Is it automatically safe for HTML?

**Answer:** No. The short echo tag outputs the expression, but does not escape it. Apply the appropriate output handling for the context before displaying external data.

### 6. Your template has a large block of HTML, but the PHP code builds the entire document as one long quoted string. What is a clearer approach?

**Answer:** Close the PHP block, write the HTML directly, and reopen PHP only where dynamic PHP output is needed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [006 - Set up the editor, terminal, and project files](../lessons/006-set-up-the-editor-terminal-and-project-files.md) | [Read this lesson](../lessons/007-php-tags-and-embedding-php-in-html.md) | [Open example](../examples/007-php-tags-and-embedding-php-in-html.md) | [008 - Statements, instruction separation, and comments](../lessons/008-statements-instruction-separation-and-comments.md) |
