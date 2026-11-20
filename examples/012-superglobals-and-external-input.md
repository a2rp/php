# 012 - Superglobals and External Input: Complete Example

This example is a small PHP page that receives a name from the URL query string, checks its shape, and displays it as HTML text safely.

## Question

How can a page read a query-string value, handle missing or unexpected input, and display the value without interpreting it as markup?

## Goal

Create this project:

~~~text
request-input-practice/
|-- README.md
|-- request-input.php
~~~

The page demonstrates **$_GET**, a missing-value fallback, a check for a string value, whitespace normalization, and HTML-text output encoding.

## Complete code: request-input.php

Save this file at the project root:

~~~php
<?php

$displayName = "Visitor";
$statusMessage = "No name was provided, so the fallback is shown.";

if (isset($_GET["name"]) && is_string($_GET["name"])) {
    $displayName = trim($_GET["name"]);
    $statusMessage = "A string value was received and trimmed.";
} elseif (isset($_GET["name"])) {
    $statusMessage = "The name value was not a single string, so the fallback is shown.";
}

$safeDisplayName = htmlspecialchars(
    $displayName,
    ENT_QUOTES | ENT_SUBSTITUTE,
    "UTF-8"
);

$safeStatusMessage = htmlspecialchars(
    $statusMessage,
    ENT_QUOTES | ENT_SUBSTITUTE,
    "UTF-8"
);
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Request Input Demo</title>
</head>
<body>
    <main>
        <h1>Request Input Demo</h1>

        <p><?php echo $safeStatusMessage; ?></p>
        <p>Hello, <?php echo $safeDisplayName; ?>.</p>

        <form method="get">
            <label for="name">Name</label>
            <input id="name" name="name" type="text">
            <button type="submit">Send name</button>
        </form>
    </main>
</body>
</html>
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Request Input Practice

A small PHP page demonstrating query-string input and safe HTML text output.

## Start the local PHP server

~~~sh
php -S 127.0.0.1:8000
~~~

Open this page in a browser:

~~~text
http://127.0.0.1:8000/request-input.php
~~~

## What it demonstrates

- Reading a query-string value from $_GET
- Handling a missing value with a fallback
- Checking that the value is a string
- Trimming surrounding whitespace
- Encoding values for their HTML text context
~~~~

This Markdown page is the README text to create in the project. The PHP page is saved separately as **request-input.php**.

## Start and view the example

Open a terminal in **request-input-practice/** and check the PHP syntax:

~~~sh
php -l request-input.php
~~~

Start PHP's built-in development server:

~~~sh
php -S 127.0.0.1:8000
~~~

Keep that terminal running. Open this URL in a browser:

~~~text
http://127.0.0.1:8000/request-input.php
~~~

With no query parameter, the page shows:

~~~text
Request Input Demo
No name was provided, so the fallback is shown.
Hello, Visitor.
~~~

The page also contains a form. Enter **Asha** and submit it. The browser sends a GET request and the address bar includes **?name=Asha**. The page then shows:

~~~text
A string value was received and trimmed.
Hello, Asha.
~~~

The same result can be opened directly at:

~~~text
http://127.0.0.1:8000/request-input.php?name=Asha
~~~

## Try different request values

### Missing value

Open the page without **?name=...**. The code keeps the default **"Visitor"** and explains that no name was provided.

### Whitespace around a string

Open:

~~~text
http://127.0.0.1:8000/request-input.php?name=%20%20Asha%20%20
~~~

The string passes the type check, and **trim()** removes the surrounding spaces. This normalization does not decide whether a name follows every rule your application may have.

### Unexpected array shape

Open:

~~~text
http://127.0.0.1:8000/request-input.php?name%5B%5D=Asha
~~~

The decoded query key uses bracket notation, so PHP provides an array for **$_GET["name"]**. The program expects one string, rejects this shape, and keeps the fallback.

### HTML characters in the input

Open:

~~~text
http://127.0.0.1:8000/request-input.php?name=%3Cscript%3Ealert(1)%3C%2Fscript%3E
~~~

The page displays the submitted characters as text. **htmlspecialchars()** encodes HTML-special characters for the HTML text nodes; the browser does not interpret the value as a script element.

## Follow the input through the program

| Step | Code or behavior | Purpose |
| --- | --- | --- |
| Set a fallback | **$displayName = "Visitor";** | The page has a value to display when no usable string is supplied. |
| Check presence and shape | **isset($_GET["name"]) && is_string($_GET["name"])** | Avoids assuming the key exists or contains one string. |
| Normalize the string | **trim($_GET["name"])** | Removes surrounding whitespace; it does not validate all application rules. |
| Encode for HTML text | **htmlspecialchars(..., "UTF-8")** | Displays special characters as text in this page's HTML text nodes. |
| Write the result | **echo $safeDisplayName;** | Outputs the encoded value in the greeting. |

## Important limits

- The example checks only that **name** is a string and trims it. A real application may need length limits and rules for acceptable characters.
- HTML encoding is applied because the values are placed in HTML text. Use output handling appropriate to the actual destination; HTML encoding is not for SQL, JavaScript, shell commands, or every URL context.
- A GET request is useful for this harmless demonstration. Do not put passwords, private tokens, or other secrets in the URL.
- The built-in PHP server is for local development and learning, not a production web server.
- Form handling, complete validation, uploads, sessions, and output security are covered in later lessons.

## Make a change

1. Change the default **"Visitor"** text and reload the page without a query parameter.
2. Submit a name with spaces at the beginning and end. Observe what **trim()** changes.
3. Submit an HTML-looking value and observe how the page displays it.
4. Change the URL key from **name** to **course** in the request, then notice that the page continues using its fallback because it expects the **name** key.
5. Temporarily remove the **is_string()** check in a scratch copy and request **?name[]=Asha**. Restore the check afterward; the code expects one string.
6. Explain why passing this value through **htmlspecialchars()** does not prove that it is a valid name.
7. Add a length rule for the name before displaying it. Decide what the page should do when the value is too long.

## Common mistakes

| Mistake | Why it is a problem | Better approach |
| --- | --- | --- |
| Reading **$_GET["name"]** without checking for a missing key | The request may not contain that key. | Check presence and provide a deliberate fallback or validation error. |
| Assuming a query value is always a string | Bracket syntax can make it an array. | Check the expected shape with **is_string()**. |
| Thinking **trim()** validates a name | It only removes surrounding whitespace. | Add validation rules that match the application's requirements. |
| Echoing the raw name into the page | HTML characters can be interpreted as markup. | Encode at the point of output for the correct context. |
| Assuming HTML encoding validates the value | An encoded value can still violate business rules. | Validate separately, then encode for output. |
| Reusing HTML-text encoding for SQL or JavaScript | Those contexts have different handling requirements. | Use the correct API and encoding for each destination. |
| Using **$_REQUEST["name"]** without deciding the source | The value may come from different request mechanisms, depending on configuration. | Read **$_GET** or **$_POST** explicitly when the source matters. |

## Key takeaways

- **$_GET** exposes query-string values, which are controlled by the request sender.
- Check for missing values and unexpected shapes before using request data.
- Normalization, validation, and output encoding solve different problems.
- Encode user input for the context where it is displayed; this example uses HTML text.
- A request method does not make input trustworthy or private.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [011 - Variable scope, global variables, and static locals](../lessons/011-variable-scope-global-variables-and-static-locals.md) | [Read this lesson](../lessons/012-superglobals-and-external-input.md) | [Open Q&A](../qnas/012-superglobals-and-external-input.md) | [013 - The PHP type system and value model](../lessons/013-the-php-type-system-and-value-model.md) |
