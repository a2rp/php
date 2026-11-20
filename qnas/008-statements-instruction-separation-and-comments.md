# 008 - Statements, Instruction Separation, and Comments: Questions and Answers

Use this Q&A to review [008 - Statements, Instruction Separation, and Comments](../lessons/008-statements-instruction-separation-and-comments.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. Which punctuation normally ends a simple PHP statement such as an assignment or an **echo** statement?

A. A semicolon<br>
B. A comma at the end of the line<br>
C. A question mark<br>
D. A blank line

**Answer: A.** Simple PHP statements such as assignments and output end with a semicolon. A newline alone does not separate them.

### 2. Which code correctly separates two simple statements?

A. **$name = "Learner"; echo $name;**<br>
B. **$name = "Learner" echo $name**<br>
C. **$name = "Learner" : echo $name**<br>
D. **$name = "Learner" / echo $name**

**Answer: A.** The assignment and the **echo** statement each have a semicolon. Formatting them on separate lines is clearer, but the semicolons mark the statement boundaries.

### 3. What does a closing **?>** do to the final PHP statement immediately before it?

A. It acts as a statement terminator<br>
B. It turns the statement into a comment<br>
C. It starts a new PHP file<br>
D. It sends the statement to the browser unchanged

**Answer: A.** PHP treats the closing tag as terminating the final statement in that block. It is still clearer to include the semicolon before the tag.

### 4. Which comment marker is a clear default for a one-line PHP comment?

A. **//**<br>
B. **<!--**<br>
C. **<comment>**<br>
D. **/***

**Answer: A.** Use **//** for a single-line PHP comment. The **<!-- ... -->** form is an HTML comment, not a PHP comment.

### 5. Which syntax creates a PHP block comment?

A. **/* note */**<br>
B. **// note //**<br>
C. **<!-- note -->**<br>
D. **#* note *#**

**Answer: A.** A PHP block comment starts with **/* ** and ends at the first ** */** closing marker.

### 6. What happens when **?>** appears after **//** inside a PHP block?

A. PHP exits the block at **?>**, so later text is no longer part of the line comment<br>
B. The line comment hides the rest of the file regardless of tags<br>
C. PHP ignores the closing tag as part of the comment<br>
D. The PHP runtime deletes the whole file

**Answer: A.** The closing PHP tag changes the parser back to non-PHP output. A line comment does not continue through the rest of the mixed file.

### 7. Which statement about HTML comments is correct?

A. They do not disable PHP code inside a PHP-enabled page<br>
B. They prevent the PHP server from parsing the file<br>
C. They are another spelling of **//**<br>
D. They are required to start every PHP file

**Answer: A.** HTML comments are passed through as markup. PHP code inside PHP tags is still processed on the server before the response reaches the browser.

### 8. What is a risk of nesting **/* ... */** comments?

A. The first ***/** closes the block, leaving later text outside the comment<br>
B. PHP automatically pairs every opening marker with the last closing marker<br>
C. Nested comments execute their contents twice<br>
D. PHP converts the outer comment into HTML

**Answer: A.** PHP block comments do not nest. The first closing marker ends the comment.

### 9. What does **php -l file.php** do?

A. Checks PHP syntax without executing the file<br>
B. Runs the file in a browser<br>
C. Starts a local web server<br>
D. Adds comments to the file

**Answer: A.** The **-l** option checks syntax only. It does not show the program's runtime output or prove its behavior is correct.

### 10. In PHP 8 and later, what does **#[** begin?

A. Attribute syntax<br>
B. A block comment<br>
C. An HTML comment<br>
D. A string literal

**Answer: A.** PHP 8 introduced attributes using **#[ ... ]**. For ordinary one-line comments, use **//** to avoid confusion.

### 11. Which description of a PHP comment is correct?

A. It is ignored as PHP source and is not emitted as PHP output<br>
B. It always appears in the browser as visible text<br>
C. It runs after all PHP statements<br>
D. It ends every PHP file automatically

**Answer: A.** PHP comments are notes in the source. They do not execute as PHP statements or become output simply because they are comments.

### 12. Which code keeps the statement boundary clear in a mixed PHP and HTML file?

A. **<?php echo "Hello"; ?>**<br>
B. **<?php echo "Hello" ?>**<br>
C. **<?php echo "Hello" ?>;**<br>
D. **<?php echo "Hello"**

**Answer: A.** The semicolon before the closing tag is allowed and recommended for clarity, even though the closing tag can terminate the final statement.

## True or false

### 1. A newline by itself ends a simple PHP statement.

**Answer: False.** Use the required semicolon. Newlines and indentation help readability but do not replace statement separation.

### 2. Simple statements can appear on the same line if they are separated correctly.

**Answer: True.** Semicolons mark the boundaries, though separate lines are usually easier to read.

### 3. A PHP closing tag can terminate the final statement in that PHP block.

**Answer: True.** The closing tag acts as a terminator for the final statement immediately before it. Keeping the semicolon is still clearer.

### 4. A PHP-only file that ends while still in PHP mode should normally have a final **?>**.

**Answer: False.** The final closing tag is optional and is usually omitted in a PHP-only source file.

### 5. PHP supports both single-line comments and block comments.

**Answer: True.** **//** and **#** are single-line forms, and **/* ... */** is the block form.

### 6. PHP block comments can be nested safely.

**Answer: False.** The first ***/** closes the block, which can make the rest of the source parse unexpectedly.

### 7. An HTML comment prevents PHP inside it from executing on the server.

**Answer: False.** PHP runs on the server before the HTML response reaches the browser. HTML comment markers do not disable PHP execution.

### 8. A PHP line comment continues through HTML that follows after **?>**.

**Answer: False.** **?>** closes the PHP block. Text after it is outside the PHP line comment and is output as non-PHP content.

### 9. Comments should generally explain non-obvious intent rather than repeat exactly what the next line does.

**Answer: True.** Useful comments add context. Repeating obvious code can add clutter and may become stale.

## Short-answer questions

### 1. What is a PHP statement?

**Answer:** It is an instruction in PHP code, such as assigning a value or outputting text. Simple statements normally end with semicolons.

### 2. Why can PHP statements not rely on line breaks as separators?

**Answer:** PHP source can place statements across lines or on one line. The semicolon marks the end of a simple statement; a newline mainly helps people read the code.

### 3. Why keep a semicolon before **?>** if the closing tag can terminate the final statement?

**Answer:** It makes the statement boundary explicit and keeps the code style consistent. The closing-tag rule is a special case for the final statement in that block.

### 4. Name the PHP comment forms covered in this chapter.

**Answer:** **//** and **#** start single-line comments. **/* ... */** creates a block comment. Use **//** as a clear default for ordinary line comments.

### 5. What is the difference between a PHP comment and an HTML comment?

**Answer:** A PHP comment is interpreted by PHP inside a PHP block. An HTML comment is markup passed through in the response; it does not prevent PHP code from executing on the server.

### 6. What happens to HTML after a PHP closing tag on the same line as a **//** comment?

**Answer:** The closing tag exits PHP mode, so the later HTML is outside the PHP line comment and becomes output.

### 7. Why should block comments not be nested?

**Answer:** PHP ends a block comment at the first ***/** marker. An inner closing marker can therefore terminate the outer comment earlier than intended.

### 8. What can **php -l file.php** tell you, and what does it not tell you?

**Answer:** It can report PHP syntax errors in the checked file without executing it. It does not tell you whether runtime behavior or output is correct.

### 9. What is a useful purpose for a comment?

**Answer:** Explaining why a non-obvious decision was made, documenting an assumption, or clarifying a tricky part of the code.

## Troubleshooting practice

### 1. PHP reports a parse error on the second assignment, even though each assignment is on its own line. What should you check?

**Answer:** Check that the first assignment ends with a semicolon.

**Explanation:** A line break does not terminate the assignment statement.

### 2. You omit the semicolon before **?>** and the page still parses. Is it a good style to omit it everywhere?

**Answer:** No. The closing tag can terminate the final statement in that PHP block, but it does not replace semicolons between earlier statements. Keeping a semicolon before the tag is clearer.

### 3. An HTML paragraph appears after a PHP line that ends with **?>**, even though the line began with **//**. Why?

**Answer:** The closing tag ended the PHP block. The line comment did not continue through the following HTML.

### 4. A PHP statement inside **<!-- ... -->** still ran on the server. Why did the HTML comment not stop it?

**Answer:** HTML comments affect how markup is treated by the browser; they do not disable PHP parsing. The server processed the PHP block before sending the response.

### 5. Source after a block comment is reported as invalid PHP. What should you inspect?

**Answer:** Look for an earlier ***/** that closed the block comment. PHP block comments do not nest, so a closing marker inside an attempted nested comment can expose the remaining text to the parser.

### 6. A file passes **php -l**, but the output is not what you expected. What should you do?

**Answer:** Run the file in the intended context and inspect the actual output. The lint command checks syntax; it does not execute the program or verify its behavior.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [007 - PHP tags and embedding PHP in HTML](../lessons/007-php-tags-and-embedding-php-in-html.md) | [Read this lesson](../lessons/008-statements-instruction-separation-and-comments.md) | [Open example](../examples/008-statements-instruction-separation-and-comments.md) | [009 - Identifiers, keywords, reserved words, and naming](../lessons/009-identifiers-keywords-reserved-words-and-naming.md) |
