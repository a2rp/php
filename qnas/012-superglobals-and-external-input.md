# 012 - Superglobals and External Input: Questions and Answers

Use this Q&A to review [012 - Superglobals and External Input](../lessons/012-superglobals-and-external-input.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What makes a PHP variable a superglobal?

A. It can only be used at the top level of a file<br>
B. It is available in all scopes without a **global** declaration<br>
C. It is always safe to print<br>
D. It can only contain strings

**Answer: B.** PHP makes predefined superglobals available throughout the script, including inside functions.

### 2. Which superglobal contains values parsed from a URL query string?

A. **$_POST**<br>
B. **$_FILES**<br>
C. **$_GET**<br>
D. **$_SESSION**

**Answer: C.** A URL such as **profile.php?name=Asha** provides the query-string key **name** through **$_GET['name']**.

### 3. Which statement about **$_POST** is correct?

A. It guarantees that a value came from a trusted user<br>
B. It contains supported form data sent in a POST request body<br>
C. It always contains JSON request bodies automatically<br>
D. It makes request data secret

**Answer: B.** PHP parses supported form encodings into **$_POST**. A JSON body is not automatically placed there, and POST data still needs validation.

### 4. Why should an application avoid putting a password in a URL query string?

A. **$_GET** cannot contain text<br>
B. URLs may be copied, bookmarked, shared, or logged<br>
C. PHP removes all query values automatically<br>
D. Query strings are only available to command-line scripts

**Answer: B.** Query-string values are visible in the URL and may be recorded in browser history, logs, or other places.

### 5. Which statement about browser-supplied values is safest?

A. Values sent with POST are trusted<br>
B. Hidden form fields cannot be changed<br>
C. Cookies cannot be changed after PHP creates them<br>
D. Treat browser-supplied values as untrusted and check them on the server

**Answer: D.** Users can construct or edit requests, form fields, and cookies. The server must check values before relying on them.

### 6. What does **$_REQUEST** commonly combine by default?

A. **$_GET**, **$_POST**, and sometimes **$_COOKIE**<br>
B. **$_FILES** and **$_ENV** only<br>
C. All PHP variables in every script<br>
D. Only query-string values

**Answer: A.** The exact contents and key precedence depend on PHP configuration. This can hide where a value came from.

### 7. Why is using **$_GET['name']** often clearer than **$_REQUEST['name']**?

A. It identifies the intended input source<br>
B. It validates the value automatically<br>
C. It makes the value impossible to change<br>
D. It converts the value to an integer

**Answer: A.** Naming the source makes it clear that the application expects query-string input. **$_REQUEST** can merge multiple sources according to configuration.

### 8. What is one important fact about **$_SERVER**?

A. Every entry exists on every server and is trustworthy<br>
B. Its entries can vary by server, and some can reflect request data<br>
C. It only contains submitted form fields<br>
D. It is unavailable to PHP

**Answer: B.** Server and execution environments differ. Some entries are derived from request headers or other client-controlled data.

### 9. What should a program assume about a filename in **$_FILES**?

A. It has already been verified as safe<br>
B. Its extension proves what content it contains<br>
C. It is metadata that still needs careful validation and handling<br>
D. It is always a complete path chosen by the server

**Answer: C.** Uploaded filenames and declared content types are not proof that a file is safe. Dedicated upload checks are needed.

### 10. What does **isset($_GET['name'])** check?

A. Whether the key has a value that is set and not **null**<br>
B. Whether the value is a valid person's name<br>
C. Whether the value is safe for HTML output<br>
D. Whether the current method is POST

**Answer: A.** **isset()** checks for a set, non-null value. It does not validate the value's meaning or encode it for output.

### 11. A request uses **?name[]=Asha**. What should code that expects one string do?

A. Assume the value is a string because the key is named **name**<br>
B. Check the type and reject or handle the unexpected array shape<br>
C. Print it directly into the page<br>
D. Convert every array to an HTML tag

**Answer: B.** Bracket syntax can make a request parameter an array. Check that the value matches the shape your code expects.

### 12. Which PHP function is suitable for encoding a value placed in HTML text?

A. **trim()**<br>
B. **htmlspecialchars()**<br>
C. **isset()**<br>
D. **session_start()**

**Answer: B.** **htmlspecialchars()** encodes HTML-special characters. Use it for an HTML output context; it is not a universal encoding function for every context.

## True or false

### 13. Superglobals can be read inside a function without writing a **global** declaration for them.

**Answer: True.** Superglobals are available in all scopes automatically.

### 14. A POST request guarantees that its values cannot be edited by a user.

**Answer: False.** The request method does not prevent a user from constructing or changing the request body. Validate values on the server.

### 15. Trimming whitespace from a string proves that it is a valid name.

**Answer: False.** **trim()** normalizes surrounding whitespace. Validation must still check the application's rules for an acceptable name.

### 16. Escaping a value for HTML text also makes it safe to use in SQL or JavaScript.

**Answer: False.** Output handling depends on the destination context. HTML encoding is not SQL parameterization or JavaScript encoding.

### 17. The default **FILTER_DEFAULT** filter performs input validation.

**Answer: False.** It is an alias for **FILTER_UNSAFE_RAW**, so it does not validate by default. Choose a specific validation or sanitization rule when using the filter extension.

## Short-answer questions

### 18. Name four commonly used superglobals and state what each contains.

**Answer:** Examples include **$_GET** for query-string values, **$_POST** for supported form data in POST requests, **$_COOKIE** for browser-sent cookie values, and **$_FILES** for upload metadata and temporary-file information. Other common superglobals include **$_SERVER**, **$_SESSION**, **$_REQUEST**, **$_ENV**, and **$GLOBALS**.

### 19. What is the difference between validation and output encoding?

**Answer:** Validation checks whether a value is allowed for the application's purpose. Output encoding changes how a value is represented for a particular destination, such as HTML text. A value can pass validation and still need output encoding.

### 20. Why should an application check a query parameter's type as well as whether it exists?

**Answer:** The key can be present but have an unexpected shape, such as an array instead of one string. Checking presence alone does not establish the value's type or validity.

### 21. What should code do before reading a value that may not have been submitted?

**Answer:** Check whether the expected key is present and handle the missing case deliberately, such as using a safe default or reporting a validation error. Do not depend on an undefined array key behaving like a valid value.

### 22. Why can a value from **$_SERVER** still need validation?

**Answer:** Available entries vary by server, and some are built from request headers or other client-supplied data. Do not assume every entry is trusted configuration.

## Read and predict

### 23. If the request is **profile.php?name=Asha**, which value does **$_GET['name']** represent?

**Answer:** It represents the query-string value **Asha**. The same query-string parsing can occur when a query string accompanies a request method other than GET.

### 24. What does this code do when **name** is missing or submitted as an array?

~~~php
$displayName = "Visitor";

if (isset($_GET["name"]) && is_string($_GET["name"])) {
    $displayName = trim($_GET["name"]);
}
~~~

**Answer:** It keeps **"Visitor"** as the fallback unless **name** is set and is a string. When it is a string, the code trims surrounding whitespace. Trimming is normalization, not a full validation rule.

### 25. Why does the example call **htmlspecialchars()** after assigning a name?

**Answer:** The value is going to be placed in HTML text. Encoding its special characters prevents the browser from treating those characters as markup. This does not determine whether the name is acceptable, and it is not meant for unrelated output contexts.

## Find and correct the mistake

### 26. What is the problem with this code?

~~~php
echo $_GET["name"];
~~~

**Answer:** The key might be missing, have an unexpected type, or contain text that the browser interprets as HTML. Check the expected value and encode it for the output context before displaying it.

### 27. What is risky about using this value to choose a file path?

~~~php
$fileName = $_GET["file"];
readfile($fileName);
~~~

**Answer:** A user could submit a path the application did not intend to read. Validate against an allowed set of files or safe identifiers, and construct paths on the server. Do not use arbitrary request text directly as a file path.

### 28. Why is this source ambiguous?

~~~php
$name = $_REQUEST["name"];
~~~

**Answer:** Depending on configuration, **name** may have come from GET, POST, or a cookie, and precedence can vary. Use the specific superglobal that matches the expected source.

## Coding practice

### 29. Write a small HTML response that uses a fallback, accepts only a string query value, trims it, and encodes it for HTML text.

**Answer:**

~~~php
<?php

$displayName = "Visitor";

if (isset($_GET["name"]) && is_string($_GET["name"])) {
    $displayName = trim($_GET["name"]);
}

$safeDisplayName = htmlspecialchars(
    $displayName,
    ENT_QUOTES | ENT_SUBSTITUTE,
    "UTF-8"
);
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Profile</title>
</head>
<body>
    <p>Hello, <?php echo $safeDisplayName; ?>.</p>
</body>
</html>
~~~

The code checks for a string and encodes it for the HTML text node. An application with stricter name rules must add its own validation.

### 30. Explain why this code does not make an unsafe input safe merely by calling **trim()**.

~~~php
$comment = trim($_POST["comment"]);
~~~

**Answer:** **trim()** only removes surrounding whitespace. The value may still contain disallowed content, have an unexpected type, or need context-specific encoding before output. Check the value according to its purpose.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [011 - Variable scope, global variables, and static locals](../lessons/011-variable-scope-global-variables-and-static-locals.md) | [Read this lesson](../lessons/012-superglobals-and-external-input.md) | [Open example](../examples/012-superglobals-and-external-input.md) | [013 - The PHP type system and value model](../lessons/013-the-php-type-system-and-value-model.md) |
