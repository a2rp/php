# 035 - Including and Requiring PHP Files: Questions and Answers

Review [035 - Including and Requiring PHP Files](../lessons/035-including-and-requiring-php-files.md). Each answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. What happens when PHP reaches an **include** statement?

A. It creates a new PHP process<br>
B. It loads and evaluates the specified file at that point<br>
C. It turns the file into a string without evaluating it<br>
D. It always redirects the browser

**Answer: B.** PHP evaluates the included file when execution reaches the statement. Any output appears at that position.

### 2. What is the usual difference if a file cannot be loaded?

A. **include** warns; **require** raises an error<br>
B. **include** stops; **require** silently skips<br>
C. Both silently skip the file<br>
D. **require** only prints a warning

**Answer: A.** A missing **include** produces a warning and execution normally continues. A missing **require** raises an error; if uncaught, it stops the script.

### 3. Which form should load a file that the page cannot work without?

A. **include**<br>
B. **require**<br>
C. **echo**<br>
D. **goto**

**Answer: B.** **require** makes the failure visible when an essential file is unavailable.

### 4. What does **include_once** add to normal **include** behavior?

A. It encrypts the included file<br>
B. It avoids evaluating a file again if it was already included<br>
C. It changes the file extension<br>
D. It prevents all PHP warnings

**Answer: B.** The **_once** form prevents repeated evaluation during the same script execution.

### 5. What does **require_once** combine?

A. Required-file failure behavior and once-only loading<br>
B. Optional loading and repeated evaluation<br>
C. Output buffering and file deletion<br>
D. A loop and a conditional

**Answer: A.** It behaves like **require** when loading fails and avoids loading the same file more than once.

### 6. What does **__DIR__** contain?

A. The current date<br>
B. The directory of the file where the constant appears<br>
C. The web server's document title<br>
D. The contents of the current file

**Answer: B.** **__DIR__** makes a path relative to the file containing that expression.

### 7. Why is **__DIR__ . '/includes/header.php'** often preferable to a bare relative path?

A. It makes the path relative to the current file instead of depending on the working directory<br>
B. It automatically creates the file<br>
C. It makes **include** load the file only once<br>
D. It hides PHP errors

**Answer: A.** The base directory is explicit, which makes the path more predictable.

### 8. What variable scope does an included file use?

A. Always a brand-new global scope<br>
B. The scope of the line where the include statement occurs<br>
C. Always the scope of the file that was included most recently<br>
D. The browser's JavaScript scope

**Answer: B.** Included code inherits the variable scope of the location where it is included.

### 9. If a file is included from inside a function, which scope does its code use?

A. The local scope of that function<br>
B. The global scope only<br>
C. A separate process scope<br>
D. No scope

**Answer: A.** The included code runs in the scope of the include statement, which here is the function's local scope.

### 10. What does the **_once** suffix guarantee?

A. The path is safe and trusted<br>
B. The file will not be evaluated more than once during the script execution<br>
C. The file cannot output HTML<br>
D. The file will load even when missing

**Answer: B.** It prevents repeated evaluation; it does not validate paths or guarantee that a file exists.

### 11. What normally happens to output from an included file?

A. It is discarded<br>
B. It appears where the include statement is executed<br>
C. It is always saved to a separate file<br>
D. It is shown before all other output

**Answer: B.** Included code executes at the include point, so its output appears in that order.

### 12. What is a common risk of building an include path directly from a request parameter?

A. PHP automatically converts the path to a safe one<br>
B. A user might influence which file the script loads<br>
C. PHP will ignore the request parameter<br>
D. It disables all includes

**Answer: B.** A user-controlled path may select an unintended file. Use a fixed mapping of allowed names to known paths.

### 13. Which path form explicitly starts from the current source file's directory?

A. **__DIR__ . '/includes/menu.php'**<br>
B. **menu.php** with no path<br>
C. **$_GET['page']**<br>
D. **include_path**

**Answer: A.** **__DIR__** identifies the directory of the file where the expression is written.

### 14. When can **include_once** be useful?

A. When the same helper declaration file may be reached through multiple code paths<br>
B. Whenever any variable is assigned<br>
C. To convert a PHP file into JSON<br>
D. To stop a loop

**Answer: A.** The once form helps avoid duplicate side effects or function and class redeclarations.

### 15. Which is a good way to let a request select a page file?

A. Append the raw request value to a filesystem path<br>
B. Map allowed page names to fixed paths and select from that map<br>
C. Include every file in the directory<br>
D. Use **goto** with the request value

**Answer: B.** A fixed allowlist prevents the request value from becoming an arbitrary path.

## Short-answer questions

### 16. Write a basic **include** statement for **partials/footer.php** relative to the current file.

**Answer:**

~~~php
<?php
include __DIR__ . '/partials/footer.php';
~~~

**Explanation:** The path starts from the directory containing the current PHP file.

### 17. Write a **require** statement for a required settings file in the same directory.

**Answer:**

~~~php
<?php
require __DIR__ . '/settings.php';
~~~

**Explanation:** **require** is appropriate when the script must stop if the settings file is unavailable.

### 18. Explain the difference between **include** and **require** on failure.

**Answer:** **include** produces a warning and execution normally continues. **require** raises an error and an uncaught error stops the script.

**Explanation:** Choose based on whether the missing file is optional or essential.

### 19. What do the **include_once** and **require_once** forms prevent?

**Answer:** They prevent a file from being evaluated again if it has already been loaded in the current script execution.

**Explanation:** This can prevent repeated output and duplicate function or class declarations.

### 20. Why use **__DIR__** in project paths?

**Answer:** It anchors a relative path to the file containing the expression.

**Explanation:** This avoids relying on the process's current working directory.

### 21. If a file is included at top level, can it read a variable defined immediately before the include?

**Answer:** Yes.

**Explanation:** The included code inherits the variable scope of the include statement.

### 22. If a file is included inside a function, does its code automatically use the caller's global scope?

**Answer:** No. It runs in the local scope of that function.

**Explanation:** Include scope is determined by where the include statement appears.

### 23. Where does included-file output appear?

**Answer:** At the point where PHP executes the include statement.

**Explanation:** PHP evaluates included code in sequence with the caller's code.

### 24. Does **include_once** make an untrusted file path safe?

**Answer:** No.

**Explanation:** It controls repeated evaluation only. The application must still restrict which files can be loaded.

### 25. What happens to **include_path** when an explicit relative path starts with **./** or **../**?

**Answer:** PHP ignores **include_path** for that explicitly specified path.

**Explanation:** For project files, a path built from **__DIR__** makes the intended base directory even clearer.

### 26. Give one suitable use for **require_once**.

**Answer:** Load an essential file of shared function declarations that may be reached through more than one code path.

**Explanation:** The file is required for the program, and repeated evaluation could cause redeclaration problems.

### 27. Is **include** appropriate for an optional page fragment?

**Answer:** Yes, if the script can still provide a useful result when the fragment is absent.

**Explanation:** A missing **include** warns but normally allows execution to continue.

### 28. Why can including a function-declaration file twice cause a problem?

**Answer:** The second evaluation may try to declare the same function again and produce a redeclaration error.

**Explanation:** Use **include_once** or **require_once** when repeated loading is possible and should be avoided.

### 29. Name one safer alternative to appending raw request input to an include path.

**Answer:** Use an associative array that maps a small set of allowed names to fixed file paths.

**Explanation:** The request can select only known entries, rather than supplying a filesystem path.

## True or false

### 30. **include** and **require** have identical behavior when the requested file is missing.

**Answer: False.** **include** warns, while **require** raises an error.

**Explanation:** The difference determines whether a missing file is treated as optional or essential.

### 31. A missing **include** normally allows the script to continue after the warning.

**Answer: True.** **include** emits a warning rather than the failure behavior of **require**.

**Explanation:** Execution can continue, though the application should still handle the missing optional content appropriately.

### 32. An uncaught error from a missing **require** stops the script.

**Answer: True.** The required file could not be loaded, so continuing may not be safe.

**Explanation:** In current PHP, the failure raises an **Error**.

### 33. **__DIR__** always means the process's current working directory.

**Answer: False.** It is the directory of the source file where **__DIR__** appears.

**Explanation:** The working directory can differ from the file's directory.

### 34. An included file inherits the variable scope of the include statement.

**Answer: True.** An include at top level has top-level scope; an include inside a function uses that function's local scope.

**Explanation:** The include executes in the surrounding scope.

### 35. **require_once** can prevent the same file from being evaluated repeatedly in one script execution.

**Answer: True.** The once form checks whether the file has already been loaded.

**Explanation:** It is useful for essential helper or declaration files.

### 36. **include_once** validates that the selected path came from an allowlist.

**Answer: False.** It only controls whether a file is loaded again.

**Explanation:** Path validation and selection are separate responsibilities.

### 37. A path beginning with **../** bypasses **include_path** lookup.

**Answer: True.** An explicitly specified relative path is resolved directly rather than searched through **include_path**.

**Explanation:** Paths based on **__DIR__** make project file locations predictable.

## Predict the output

### 38. The included file **header.php** contains the following. What does the main script print?

**header.php:**

~~~php
<?php
echo "Header\n";
~~~

**index.php:**

~~~php
<?php
echo "Before\n";
include __DIR__ . '/header.php';
echo "After\n";
~~~

**Answer:**

~~~text
Before
Header
After
~~~

**Explanation:** The included file is evaluated between the two echo statements.

### 39. The included file **message.php** contains the following. What does the main script print?

**message.php:**

~~~php
<?php
echo $greeting, "\n";
~~~

**index.php:**

~~~php
<?php
$greeting = 'Hello';
include __DIR__ . '/message.php';
~~~

**Answer:**

~~~text
Hello
~~~

**Explanation:** The include occurs at top level, so the included code can access the variable in that scope.

### 40. If **repeat.php** prints the word **loaded** followed by a newline, what does this print?

~~~php
<?php
include __DIR__ . '/repeat.php';
include __DIR__ . '/repeat.php';
~~~

**Answer:**

~~~text
loaded
loaded
~~~

**Explanation:** Plain **include** evaluates the file each time the statement is reached.

### 41. If **repeat.php** prints the word **loaded** followed by a newline, what does this print?

~~~php
<?php
include_once __DIR__ . '/repeat.php';
include_once __DIR__ . '/repeat.php';
~~~

**Answer:**

~~~text
loaded
~~~

**Explanation:** The first statement evaluates the file. The second sees that it has already been included and skips its code.

### 42. If **status.php** returns the string **ready**, what does this print?

~~~php
<?php
$status = require __DIR__ . '/status.php';
echo $status, "\n";
~~~

**Answer:**

~~~text
ready
~~~

**Explanation:** An included file can explicitly return a value, and the include or require expression can be assigned that result.

## Write the code

### 43. Require **includes/bootstrap.php** using a path based on the current file.

**Answer:**

~~~php
<?php
require __DIR__ . '/includes/bootstrap.php';
~~~

**Explanation:** **require** expresses that the bootstrap file is essential, and **__DIR__** anchors its path to this source file.

### 44. Use a fixed mapping so the value in **$pageName** can select only **home.php** or **contact.php** from a **pages** directory. Print **Page not found** for any other name.

**Answer:**

~~~php
<?php
$pages = [
    'home' => __DIR__ . '/pages/home.php',
    'contact' => __DIR__ . '/pages/contact.php',
];

if (isset($pages[$pageName])) {
    require $pages[$pageName];
} else {
    http_response_code(404);
    echo 'Page not found';
}
~~~

**Explanation:** The input selects from known paths instead of being concatenated into a filesystem path.

### 45. Load **includes/helpers.php** only once, even if this code path is reached more than once.

**Answer:**

~~~php
<?php
require_once __DIR__ . '/includes/helpers.php';
~~~

**Explanation:** **require_once** makes the file essential and avoids evaluating it again during the same script execution.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [034 - break, continue, goto, and declare](../lessons/034-break-continue-goto-and-declare.md) | [Read this lesson](../lessons/035-including-and-requiring-php-files.md) | [Open example](../examples/035-including-and-requiring-php-files.md) | [036 - Returning values and ending execution](../lessons/036-returning-values-and-ending-execution.md) |
