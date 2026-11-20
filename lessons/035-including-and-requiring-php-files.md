# 035 - Including and Requiring PHP Files

PHP can load another file while a script runs. The file's PHP code is evaluated at the point where it is included, and any output it produces appears at that position.

Use this feature to share page sections, configuration, helper functions, and other code instead of copying the same code into many scripts.

## What you will learn

- Include a file with **include** or **require**
- Choose between warning-level and fatal missing-file behavior
- Prevent a file from being loaded more than once
- Build predictable local paths with **__DIR__**
- Understand the variable scope of included files
- Avoid common path, duplication, and security mistakes

## A small project with shared files

A simple project can keep reusable page sections in a directory:

~~~text
my-site/
├── index.php
└── includes/
    ├── header.php
    └── footer.php
~~~

The shared files can contain HTML, PHP, or both.

**includes/header.php:**

~~~php
<header>
    <h1>My PHP site</h1>
</header>
~~~

**includes/footer.php:**

~~~php
<footer>
    <p>Learning PHP one page at a time.</p>
</footer>
~~~

Use an include statement in **index.php** to insert those sections:

~~~php
<?php
$pageTitle = 'Home';

require __DIR__ . '/includes/header.php';

echo "<main>\n";
echo "    <h2>{$pageTitle}</h2>\n";
echo "    <p>This page reuses its header and footer.</p>\n";
echo "</main>\n";

include __DIR__ . '/includes/footer.php';
~~~

Output:

~~~html
<header>
    <h1>My PHP site</h1>
</header>
<main>
    <h2>Home</h2>
    <p>This page reuses its header and footer.</p>
</main>
<footer>
    <p>Learning PHP one page at a time.</p>
</footer>
~~~

The content from **header.php** appears where **require** is written. The footer appears where **include** is written. PHP evaluates each included file as the program reaches the include statement.

## include and require

Both **include** and **require** load and evaluate a file. The main difference is what happens when PHP cannot load it:

- **include** emits a warning. In ordinary execution, the script continues after the include statement.
- **require** raises an error if the file cannot be loaded. If that error is not caught, the script stops.

Choose **require** when the current page cannot work without the file, such as a required bootstrap or core configuration file. Choose **include** when the file is optional and the rest of the script can still provide a useful result without it.

~~~php
<?php
require __DIR__ . '/includes/settings.php';
include __DIR__ . '/includes/optional-banner.php';
~~~

A file needed for correct operation should not be silently treated as optional. Match the include form to how important that file is.

## Load a file only once

A plain include can evaluate the same file more than once if the program reaches the statement multiple times. That can repeat output or side effects. If the file declares a function or class, loading it again can cause a redeclaration error.

Use the **_once** forms when a file must not be evaluated more than once during the same script execution:

- **include_once** behaves like **include**, but skips a file that has already been included.
- **require_once** behaves like **require**, but skips a file that has already been required or included.

~~~php
<?php
require_once __DIR__ . '/includes/helpers.php';
require_once __DIR__ . '/includes/helpers.php';

// helpers.php is evaluated only once in this script execution.
~~~

Use **require_once** for essential files that may be reached through multiple code paths, such as a file containing shared function declarations. Use **include_once** when such a file is optional. Do not add **_once** automatically everywhere; first decide whether repeated evaluation would actually be a problem.

## Build reliable file paths with __DIR__

A relative path such as **includes/header.php** can be affected by the script's working directory or PHP's include path. **__DIR__** is a magic constant containing the directory of the file where it appears. Join it with the relative path to make the location clear:

~~~php
<?php
require __DIR__ . '/includes/header.php';
~~~

This is more reliable than depending on which directory the command or web server used as its current working directory.

PHP can search its configured **include_path** when a bare filename is used. An explicit path that starts with a directory, such as **./** or **../**, or an absolute path bypasses **include_path**. For application files in your own project, a path based on **__DIR__** makes the intended file location easier to review.

## Included code and variable scope

An included file inherits the variable scope of the line where the include happens. If the include is at the top level of a script, variables defined there are available to the included file:

~~~php
<?php
$pageTitle = 'Contact';
include __DIR__ . '/includes/title.php';
~~~

In **includes/title.php**, **$pageTitle** is available:

~~~php
<h1><?= $pageTitle ?></h1>
~~~

The include also runs in the surrounding scope. Variables created by the included code become available in that scope. If the include statement is inside a function, the included code uses that function's local scope instead of automatically receiving the caller's global variables.

This is one reason to keep included files understandable: they can read and change variables from the place that includes them.

## Included files can contain PHP and HTML

PHP can switch between PHP code and plain text in an included file. For example, a shared template can print markup directly:

~~~php
<section>
    <p>This markup came from an included file.</p>
</section>
~~~

If an included file contains PHP tags, PHP evaluates the PHP code and sends any surrounding text as output. Keep presentation fragments in templates and reusable logic in helper files so each file has a clear purpose.

## Choosing between the four forms

| Form | If the file is missing | Prevents loading it again |
| --- | --- | --- |
| **include** | Warning; script normally continues | No |
| **require** | Error; uncaught error stops the script | No |
| **include_once** | Warning; script normally continues | Yes |
| **require_once** | Error; uncaught error stops the script | Yes |

The **_once** suffix changes repeated-loading behavior. It does not change the basic difference between optional **include** and essential **require**.

## Keep include paths under your control

An include statement executes the selected file as PHP code. Do not construct the file path directly from untrusted request data:

~~~php
<?php
// Avoid this pattern:
$page = $_GET['page'] ?? '';
include __DIR__ . '/pages/' . $page . '.php';
~~~

A user-controlled path can select an unintended file if it is not carefully constrained. Prefer a fixed mapping from allowed names to known files:

~~~php
<?php
$pages = [
    'home' => __DIR__ . '/pages/home.php',
    'contact' => __DIR__ . '/pages/contact.php',
];

$pageName = $_GET['page'] ?? 'home';

if (isset($pages[$pageName])) {
    require $pages[$pageName];
} else {
    http_response_code(404);
    echo 'Page not found';
}
~~~

The array provides the only valid paths. The request value selects one of those known entries; it is not appended to a filesystem path.

## Common mistakes

- Using **include** for a file that the script cannot work without. Use **require** for essential files.
- Using **require** for an optional fragment and stopping the page when it is absent.
- Using a bare relative path and assuming it is resolved from the directory you expect. Prefer **__DIR__** for project files.
- Repeatedly loading a file that declares functions or classes. Use the relevant **_once** form when repeated loading is possible.
- Assuming **include_once** makes a path safe. It prevents repeat evaluation; it does not validate the path.
- Building an include path directly from a query parameter, form field, or other untrusted input.
- Forgetting that included code runs in the variable scope where the include statement appears.
- Putting unrelated responsibilities in one shared file. Keep templates, configuration, and helper logic organized by purpose.

## Try it yourself

1. Create **includes/header.php** and **includes/footer.php**, then include them from **index.php**.
2. Replace a bare relative path with a path that starts from **__DIR__**.
3. Make a required file unavailable and compare **require** with **include**.
4. Include a file twice, then change the statements to **require_once** and observe the difference.
5. Set a variable before including a small template and print it from that template.
6. Move an include statement into a function and compare the variable scope.
7. Use a fixed page map so a request can choose only from known files.

## Key takeaways

- **include** and **require** evaluate another file at the point where they are used.
- Use **require** when the file is essential; use **include** when it is optional.
- The **_once** forms prevent the same file from being evaluated more than once in a script execution.
- **__DIR__** helps build predictable paths relative to the current file.
- Included code inherits the variable scope where the include statement occurs.
- Treat include paths as security-sensitive because loading a file executes its code.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [034 - break, continue, goto, and declare](034-break-continue-goto-and-declare.md) | [Open Q&A](../qnas/035-including-and-requiring-php-files.md) | [Open example](../examples/035-including-and-requiring-php-files.md) | [036 - Returning values and ending execution](036-returning-values-and-ending-execution.md) |
