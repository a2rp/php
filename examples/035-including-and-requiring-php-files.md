# 035 - Including and Requiring PHP Files: Complete Example

This command-line project splits a page into reusable PHP files. It demonstrates **require**, **include**, the **_once** forms, **__DIR__**, variables shared through include scope, and a fixed allowlist for choosing a page.

## Question

How can a PHP page reuse layout fragments and helper functions from other files while keeping file paths predictable and page selection limited to known files?

## Goal

Create this project:

~~~text
include-and-require-example/
├── index.php
├── includes/
│   ├── footer.php
│   ├── header.php
│   ├── helpers.php
│   ├── notice.php
│   └── title.php
└── pages/
    ├── contact.php
    └── home.php
~~~

The program will:

- Load a required header and optional footer
- Read a variable from the scope that includes a template
- Load an optional notice only once
- Load helper functions with **require_once**
- Select a page only from a fixed list of known paths

## Complete code

### index.php

~~~php
<?php
declare(strict_types=1);

echo "=== 1. Shared page layout ===\n";

$pageTitle = 'Home';

require __DIR__ . '/includes/header.php';
include __DIR__ . '/includes/title.php';

echo "Main page content\n";

include __DIR__ . '/includes/footer.php';

echo "\n=== 2. Optional notice, once ===\n";

include_once __DIR__ . '/includes/notice.php';
include_once __DIR__ . '/includes/notice.php';

echo "\n=== 3. Shared helper functions ===\n";

require_once __DIR__ . '/includes/helpers.php';
require_once __DIR__ . '/includes/helpers.php';

echo formatGreeting('Asha'), "\n";

echo "\n=== 4. Select from known pages ===\n";

$pages = [
    'home' => __DIR__ . '/pages/home.php',
    'contact' => __DIR__ . '/pages/contact.php',
];

$pageName = 'contact';

if (isset($pages[$pageName])) {
    require $pages[$pageName];
} else {
    echo "Page not found\n";
}
~~~

### includes/header.php

~~~php
<?php
echo "<header>Shared header</header>\n";
~~~

### includes/title.php

~~~php
<?php
echo "Title from included file: {$pageTitle}\n";
~~~

This file can read **$pageTitle** because **index.php** includes it after setting that variable.

### includes/footer.php

~~~php
<?php
echo "<footer>Shared footer</footer>\n";
~~~

### includes/notice.php

~~~php
<?php
echo "Optional notice\n";
~~~

The two **include_once** statements in **index.php** evaluate this file only once.

### includes/helpers.php

~~~php
<?php
function formatGreeting(string $name): string
{
    return "Hello, {$name}!";
}
~~~

The **require_once** statements prevent this function declaration file from being evaluated repeatedly.

### pages/home.php

~~~php
<?php
echo "Home page selected\n";
~~~

### pages/contact.php

~~~php
<?php
echo "Contact page selected\n";
~~~

The **$pages** array in **index.php** is the allowlist. The selected name chooses one of these fixed file paths.

## Expected output

~~~text
=== 1. Shared page layout ===
<header>Shared header</header>
Title from included file: Home
Main page content
<footer>Shared footer</footer>

=== 2. Optional notice, once ===
Optional notice

=== 3. Shared helper functions ===
Hello, Asha!

=== 4. Select from known pages ===
Contact page selected
~~~

## Explanation

### Required and optional files

The header is necessary for this page, so it is loaded with **require**. The footer is an optional fragment in this small example, so it uses **include**. If the header file is missing, the script reports an error and cannot finish normally. A missing footer produces a warning, and execution normally continues.

The example uses paths based on **__DIR__**, so every file is located relative to **index.php**, regardless of the process's current working directory.

### Variable scope

**index.php** assigns **$pageTitle** before including **title.php**. The included code runs in the scope of the include statement, so it can read that variable. If an include were placed inside a function, the included code would instead run in that function's local scope.

### Loading files only once

The notice is included twice in the source, but **include_once** evaluates it only on the first statement. The helper file is loaded with **require_once** for the same reason: it declares a function, and declaring the same function again would cause a problem.

The **_once** suffix prevents repeated evaluation. It does not validate the path or make an untrusted path safe.

### Selecting an allowed page

The program uses **$pageName** to look up a path in **$pages**. Only the paths listed in that array can be loaded. The request value itself is not appended to a directory path.

To try the not-found branch, change **$pageName** to a value such as **settings**. Because that key is not in the map, the program prints **Page not found** and does not try to load a file with that name.

## Run the example

From the project directory, run:

~~~sh
php index.php
~~~

The output should match the block above.

## Make a change

1. Change **$pageTitle** to **About** and observe that the included title changes.
2. Add a line to **header.php** and confirm it appears before the page content.
3. Change the optional notice text and confirm it still prints only once.
4. Add a **formatFarewell()** function to **helpers.php** and call it from **index.php**.
5. Set **$pageName** to **home** and observe the selected file's output.
6. Set **$pageName** to **unknown** and check the safe not-found branch.
7. Temporarily change the header path to a nonexistent filename and observe how **require** fails. Restore it after the demonstration.

## Common mistakes

- Using a bare relative path and assuming it starts from the directory of **index.php**. Build a project path with **__DIR__**.
- Using **include** for a file that the application cannot operate without.
- Using **require** for content that is genuinely optional.
- Loading a helper declaration file repeatedly and causing a redeclaration error.
- Assuming **include_once** makes a path safe. It only avoids repeated loading.
- Appending a request value directly to an include path instead of selecting from known paths.
- Forgetting that an included file uses the variable scope where its include statement runs.

## Key takeaways

- Use **require** for essential files and **include** for optional files.
- Use the **_once** forms when repeated evaluation could cause duplicate output or declarations.
- **__DIR__** anchors local paths to the source file.
- Included files run in the scope of the include statement and can produce output at that point.
- A fixed path map restricts page selection to files the application intentionally allows.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [034 - break, continue, goto, and declare](../lessons/034-break-continue-goto-and-declare.md) | [Read this lesson](../lessons/035-including-and-requiring-php-files.md) | [Open Q&A](../qnas/035-including-and-requiring-php-files.md) | [036 - Returning values and ending execution](../lessons/036-returning-values-and-ending-execution.md) |
