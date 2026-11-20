# 012 - Superglobals and External Input

Web applications receive values from URLs, forms, cookies, uploaded files, server configuration, and other sources. PHP exposes many of these values through **superglobals**: predefined variables that are available in every scope. Their availability is convenient, but values supplied by a browser are still untrusted input.

## What you will learn

- What makes a PHP variable a superglobal
- What common superglobals contain
- Where query-string and form values appear
- Why browser-supplied values must be checked before use
- How to distinguish missing values, validation, normalization, and safe output
- Why **$_REQUEST** can hide where a value came from

## What is a superglobal?

A superglobal is a predefined PHP variable that is available throughout a script, including inside functions. Unlike an ordinary global variable, a superglobal does not need a **global** declaration before a function can read it.

The standard superglobals include:

| Superglobal | Common purpose |
| --- | --- |
| **$GLOBALS** | Access variables in the global symbol table. Its contents are application state, not automatically user input. |
| **$_SERVER** | Server and request environment information. Available entries vary by server and some values can reflect client-supplied request data. |
| **$_GET** | Values parsed from the URL query string. A query string can be present with different HTTP methods. |
| **$_POST** | Values parsed from supported form data sent in an HTTP POST request. |
| **$_FILES** | Information about files submitted through an HTTP file upload. Names and other metadata still need to be treated carefully. |
| **$_COOKIE** | Cookie values sent by the browser. A user can change cookie values. |
| **$_SESSION** | Session data for the current request. Start or resume the session before relying on its values. |
| **$_REQUEST** | A configuration-dependent combination of request values, commonly from GET, POST, and sometimes cookies. |
| **$_ENV** | Environment values provided to the PHP process; which values are available depends on configuration and deployment. |

These variables are arrays, so a named value is commonly read with a key:

~~~php
$nameFromUrl = $_GET['name'];
~~~

For example, a request to **/profile.php?name=Asha** sends the query-string key **name**. PHP makes that value available through **$_GET['name']**. PHP's array syntax is covered in more detail in a later chapter.

## Query-string and form input

### Values from the URL

**$_GET** contains values parsed from the URL query string. For a URL such as:

~~~text
/profile.php?name=Asha&lesson=12
~~~

the script can read **$_GET['name']** and **$_GET['lesson']**. Query-string values are visible in the address bar and may be copied, bookmarked, logged, or shared. Do not put passwords, private tokens, or other secrets in a URL.

PHP populates **$_GET** when a query string is present; it is not restricted to requests whose method is GET.

### Values from a form submitted with POST

For supported form encodings, **$_POST** contains the submitted form values. A POST request sends its data in the request body rather than putting those form fields in the URL query string.

POST does not make data secret or trustworthy. A user can edit form controls, send a request without using your page, or provide a value that the interface would normally prevent. Use HTTPS to protect data in transit, and validate values on the server.

A JSON request body is not automatically parsed into **$_POST**. Reading and decoding JSON request bodies is covered later with APIs.

## A value being present does not make it valid

External data can be missing, empty, malformed, unexpectedly long, or the wrong shape. A query parameter that looks like one string may instead be submitted as an array. For example, **?name[]=Asha** does not have the same shape as **?name=Asha**.

Check that a value exists and has the type your code expects before using it:

~~~php
<?php

$displayName = "Visitor";

if (isset($_GET["name"]) && is_string($_GET["name"])) {
    $displayName = trim($_GET["name"]);
}
~~~

This example uses **isset()** to check that the key has a non-null value, and **is_string()** to reject a value with an unexpected shape. The **if** statement is introduced in detail later in the course; here it shows why input needs a guard. Calling **trim()** removes surrounding whitespace. That is normalization, not a complete check that the name follows every rule your application may require.

A reliable input process separates three jobs:

1. **Check and validate** that the value is present and allowed for the application's purpose.
2. **Normalize** it when appropriate, such as trimming whitespace or converting a value to a consistent form.
3. **Encode it for the output context** when displaying it, so the value is treated as data rather than executable markup.

Validation does not make a value safe in every output context. HTML text, HTML attributes, JavaScript, URLs, SQL, and shell commands each have different rules. Use the correct handling for the specific destination.

## Escape a value when showing it as HTML text

When a user-supplied value is placed inside HTML text, encode HTML special characters at the point where the page is built:

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

Here the value is placed in an HTML text node, so **htmlspecialchars()** is used for that output context. It does not validate whether a name is acceptable, and it is not a universal escaping function for JavaScript, URLs, SQL, or shell commands. More detailed output safety is covered in the security chapters.

## Common superglobals and their boundaries

### **$_COOKIE**

Cookie data is stored by the browser and sent back with requests. A user can change it, so do not trust a cookie merely because your application originally set it. A cookie value is not proof that the user is authorized.

### **$_SESSION**

Session data is maintained by the server and associated with a session identifier sent by the browser. Use **session_start()** to start or resume a session before reading or writing session values. Validate important state and permissions on the server; do not assume a session key exists or has the expected type.

Session lifecycle and security are covered in later chapters.

### **$_FILES**

**$_FILES** contains upload metadata and temporary-file information for submitted files. A provided filename or declared content type is not proof that the file is safe. Upload error checks, content inspection, storage locations, and safe file handling are covered in dedicated chapters.

### **$_SERVER**

**$_SERVER** contains server and execution details, but its exact entries vary with the web server and execution environment. Some entries are derived from request headers or other client-supplied information. Do not treat every entry as trusted configuration or use a request-derived host value as an authority without deliberate validation.

Most **$_SERVER** entries are not meaningful when a script runs from the command line.

### **$_ENV** and **$GLOBALS**

**$_ENV** exposes environment data when the process and PHP configuration provide it. A value may be absent, so code should handle that case and keep secrets out of source files.

**$GLOBALS** provides access to variables in global scope. It is useful to recognize, but using it to hide a function's dependencies makes code harder to follow. Prefer passing values into a function when that is practical.

## Why **$_REQUEST** is often unclear

By default, **$_REQUEST** is commonly assembled from **$_GET**, **$_POST**, and sometimes **$_COOKIE**. Which sources it contains and which value wins when keys collide depend on PHP configuration.

For example, a value named **name** could have come from the URL, a form body, or a cookie. Reading **$_REQUEST['name']** hides that distinction. Prefer **$_GET['name']** or **$_POST['name']** when the source matters, and do not use **$_REQUEST** as a shortcut around deciding what kind of request your code expects.

None of these superglobals automatically validates or safely encodes its values. The filter extension also does not filter data by default when **FILTER_DEFAULT** is used; choose an explicit validation or sanitization plan for the value's purpose.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming data from **$_GET** or **$_POST** is valid | A user can construct a request with arbitrary values. | Check presence, expected shape, and allowed values on the server. |
| Assuming POST data is secret or protected from editing | The request method does not prevent a user from changing the body. | Use HTTPS for transport protection and validate the request server-side. |
| Reading a missing key directly | The value may be absent and trigger a diagnostic or produce an unexpected result. | Check that the key is present before reading it. |
| Assuming every request value is a string | Bracket syntax can cause a parameter to arrive as an array. | Check the expected type and reject an unexpected shape. |
| Printing user input directly into HTML | Special characters may be interpreted as markup. | Encode for the exact output context; for HTML text, use **htmlspecialchars()**. |
| Treating **$_REQUEST** as a predictable source | Its contents and key precedence depend on configuration. | Use the specific source, such as **$_GET** or **$_POST**. |
| Trusting a cookie, filename, or request header because PHP exposes it in a superglobal | The value can still originate from or be changed by a client. | Treat it as input and validate it for its intended use. |
| Expecting **filter_input()** to validate data with its default filter | **FILTER_DEFAULT** does not apply a validation filter. | Select an explicit validation or sanitization rule and handle failure. |

## Try it yourself

1. In a browser, open **profile.php?name=Asha** and identify the query-string value that PHP exposes through **$_GET**.
2. Open **profile.php?name=%3Cscript%3Ealert(1)%3C%2Fscript%3E**. Observe that the greeting displays the characters as text rather than running a script.
3. Open **profile.php** without a **name** parameter. Explain why the script uses its fallback value.
4. Try **profile.php?name%5B%5D=Asha**. Explain why the type check rejects this array-shaped value.
5. Describe one difference between **$_GET** and **$_POST**, and one reason neither should be trusted without validation.
6. Explain why **$_REQUEST['name']** does not clearly identify where **name** came from.
7. List two kinds of values that may appear in **$_SERVER**, then name one reason a request-derived entry needs careful handling.
8. Explain the difference between validating a name and encoding it for HTML text.
9. Identify which later chapters cover forms, file uploads, sessions, and HTML output safety.

## Key points

- Superglobals are predefined variables available in every scope without a **global** declaration.
- **$_GET**, **$_POST**, **$_COOKIE**, and request-derived **$_SERVER** values can contain client-controlled input.
- Input can be missing or have a different type or shape than expected.
- Validate and normalize values for their purpose, then encode them for the output context.
- **$_REQUEST** combines sources according to configuration, so prefer a specific input source when it matters.
- **$_FILES**, cookies, sessions, server data, and environment data each have their own trust and lifecycle considerations.
- PHP does not automatically validate or safely encode values from a superglobal.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [011 - Variable scope, global variables, and static locals](011-variable-scope-global-variables-and-static-locals.md) | [Open Q&A](../qnas/012-superglobals-and-external-input.md) | [Open example](../examples/012-superglobals-and-external-input.md) | [013 - The PHP type system and value model](013-the-php-type-system-and-value-model.md) |
