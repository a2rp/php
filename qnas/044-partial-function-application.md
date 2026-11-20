# 044 - Partial Function Application: Q&A

Review PHP 8.6 partial function application, including **?** and **...** placeholders, callback adaptation, named placeholders, and evaluation timing. Try each question before reading its answer. See the [chapter lesson](../lessons/044-partial-function-application.md).

## Multiple-choice questions

### 1. Which PHP version introduced partial function application?

A. PHP 7.4
B. PHP 8.0
C. PHP 8.1
D. PHP 8.6

**Answer:** D. PHP 8.6.

**Explanation:** Partial function application is a PHP 8.6 feature. Earlier PHP versions do not support the placeholder syntax.

### 2. What does a **?** placeholder represent?

A. Zero or more arguments
B. Exactly one argument for the returned closure
C. An optional argument that can be omitted
D. A literal question mark passed to the function

**Answer:** B. Exactly one argument for the returned closure.

**Explanation:** Each **?** creates one required parameter in the closure.

### 3. What does the final **...** placeholder represent?

A. All remaining arguments to forward
B. A comment saying more code follows
C. A variadic argument that is always empty
D. Exactly one required argument

**Answer:** A. All remaining arguments to forward.

**Explanation:** The placeholder keeps unmentioned parameters available to the closure and forwards arguments supplied to it.

### 4. Where may the partial-application **...** placeholder appear?

A. Anywhere, any number of times
B. At most once, and it must be last
C. Only as the first argument
D. Only before a named argument

**Answer:** B. At most once, and it must be last.

**Explanation:** PHP requires the remaining-arguments placeholder to be final.

### 5. What does this expression do?

~~~php
$forLengthTwo = volume(2, ?, ?);
~~~

A. Calls **volume()** immediately with zeroes
B. Creates a closure with two required parameters
C. Creates an array with two placeholders
D. Changes the **volume()** function definition

**Answer:** B. Creates a closure with two required parameters.

**Explanation:** The placeholders leave the width and height to be supplied later.

### 6. When does the target function run after partial application?

A. When the closure is created
B. When the returned closure is invoked
C. When PHP parses the file
D. It never runs

**Answer:** B. When the returned closure is invoked.

**Explanation:** Partial application creates a closure but delays the original call until its missing arguments are supplied.

### 7. How many times is a supplied argument expression evaluated?

A. Every time the returned closure is called
B. Once, when the closure is created
C. Twice, at creation and invocation
D. It is never evaluated

**Answer:** B. Once, when the closure is created.

**Explanation:** A provided value is stored in the created closure.

### 8. What happens to extra arguments passed to a closure created without **...**?

A. They are forwarded to the original function.
B. They are ignored.
C. They cause partial application to restart.
D. They change the default values.

**Answer:** B. They are ignored.

**Explanation:** To forward extra arguments to the original callable, include a final **...** placeholder.

### 9. What is the main benefit of a named placeholder such as **subject: ?**?

A. It creates an optional argument.
B. It lets you refer to the original parameter by name and choose closure parameter order.
C. It declares a new function.
D. It forces an argument to be passed by reference.

**Answer:** B. It lets you refer to the original parameter by name and choose closure parameter order.

**Explanation:** The resulting closure parameters follow the order in which the named placeholders appear.

### 10. Which is a valid partial application for removing hyphens from a later-provided subject?

A. **str_replace(subject: ?, search: ?, replace: '')**
B. **str_replace(?, subject: '-', replace: '')**
C. **str_replace(..., ?, '')**
D. **str_replace(new ?, '-')**

**Answer:** A. **str_replace(subject: ?, search: ?, replace: '')**

**Explanation:** The subject and search string are left as placeholders, while the replacement is fixed as an empty string.

### 11. What does **volume(2, 3, 4, ...)** create?

A. The integer **24** immediately
B. A closure that takes no additional arguments and will call **volume()** when invoked
C. A closure that requires three arguments
D. A syntax error because every argument must be a placeholder

**Answer:** B. A closure that takes no additional arguments and will call **volume()** when invoked.

**Explanation:** All function arguments are supplied, but the final **...** delays execution until the closure is called.

### 12. Which syntax is unsupported?

A. **strlen(?)**
B. **$object->method(?, ...)**
C. **new Example(?, ...)**
D. **volume(2, ...)**

**Answer:** C. **new Example(?, ...)**.

**Explanation:** Partial application applies to callable expressions, not object-construction expressions.

### 13. Can partial-application placeholders be combined with argument unpacking such as **...$values**?

A. Yes, anywhere
B. Yes, if **?** is named
C. No
D. Only for built-in functions

**Answer:** C. No.

**Explanation:** Partial-application placeholders cannot be combined with argument unpacking.

### 14. What happens if **?** stands for a parameter that had a default in the original function?

A. The new closure parameter is optional.
B. The new closure parameter is required.
C. The placeholder uses the default immediately.
D. The placeholder is removed.

**Answer:** B. The new closure parameter is required.

**Explanation:** **?** means exactly one argument must be supplied to the returned closure.

### 15. Which set contains functions the PHP manual lists as incompatible with partial application because they inspect calling scope?

A. **trim()**, **strlen()**, **strtoupper()**
B. **compact()**, **extract()**, **func_get_args()**, **get_defined_vars()**
C. **array_map()**, **array_filter()**, **array_keys()**
D. **print_r()**, **var_dump()**, **count()**

**Answer:** B. **compact()**, **extract()**, **func_get_args()**, **get_defined_vars()**.

**Explanation:** These functions inspect the calling scope, which partial application changes.

## Short-answer questions

### 16. What is partial function application?

**Answer:** It creates a closure by supplying some arguments to a callable now and leaving placeholders for arguments that will be supplied later.

**Explanation:** The target callable runs only when the returned closure is invoked.

### 17. Write a partial application that fixes **$greeting** to **'Hello'** and leaves the name for later.

Assume **greet(string $greeting, string $name)** exists.

**Answer:**

~~~php
$hello = greet('Hello', ?);
~~~

**Explanation:** Calling **$hello('Asha')** later invokes **greet('Hello', 'Asha')**.

### 18. How many closure parameters does **volume(2, ?, ?)** have?

**Answer:** Two required parameters.

**Explanation:** Each **?** stands for exactly one argument.

### 19. What is the difference between **?** and **...**?

**Answer:** **?** leaves exactly one required argument. **...** leaves all remaining arguments to be supplied and forwarded.

**Explanation:** The **...** placeholder can appear once at the end; **?** may appear at each argument position that should be filled later.

### 20. What does **volume(2, ...)** leave for the returned closure?

**Answer:** The width and height arguments.

**Explanation:** The length is fixed at **2**, while all remaining parameters are forwarded when the closure is called.

### 21. How do named placeholders affect the returned closure's parameter order?

**Answer:** Its parameters follow the order in which the named placeholders are written.

**Explanation:** Named placeholders can make the closure's argument order more useful than the original function's order.

### 22. What happens when **greetWithDefault(?)** receives two arguments even though the closure has one placeholder?

**Answer:** Without **...**, the extra closure argument is ignored; the original function uses its default for the unprovided parameter.

**Explanation:** Include **...** if extra arguments must reach the original callable.

### 23. What does **greetWithDefault(?, ...)** do with extra arguments?

**Answer:** It forwards them to the original callable's remaining parameters.

**Explanation:** If a second argument is passed to the closure, it can supply the original function's optional greeting parameter.

### 24. When is an expression such as **currentGreeting()** evaluated in **greet(currentGreeting(), ?)**?

**Answer:** When the partial closure is created.

**Explanation:** The resulting value is stored in the closure and reused on later calls.

### 25. How does that timing differ from an arrow function wrapping **greet()**?

**Answer:** An expression inside the arrow function's body is evaluated whenever the arrow function is called.

**Explanation:** A provided partial-application argument is evaluated once at closure creation.

### 26. How can partial function application help with **array_map()**?

**Answer:** It can fix configuration arguments and leave one placeholder for each array value.

**Explanation:** The resulting closure accepts the unary callback argument **array_map()** supplies.

### 27. Why must **...** be the last argument?

**Answer:** It represents all arguments not otherwise supplied or left by earlier placeholders.

**Explanation:** Nothing can follow the placeholder that stands for all remaining arguments.

### 28. Name two syntax combinations partial application does not support.

**Answer:** It cannot be combined with argument unpacking or the nullsafe operator. It also does not support constructor expressions.

**Explanation:** The feature applies to callable expressions and has specific syntax restrictions.

### 29. Which PHP version is required for partial function application?

**Answer:** PHP 8.6 or later.

**Explanation:** Earlier versions do not recognize the placeholder syntax.

## True or false

### 30. **volume(2, ?, ?)** immediately calculates the volume.

**Answer:** False.

**Explanation:** It creates a closure. The calculation happens after the two remaining arguments are supplied.

### 31. Each **?** becomes exactly one required parameter of the returned closure.

**Answer:** True.

**Explanation:** A question-mark placeholder represents precisely one future argument.

### 32. The **...** placeholder may appear in the middle of the arguments if another argument follows.

**Answer:** False.

**Explanation:** It must appear once at most and be last.

### 33. A parameter's default value makes a corresponding **?** optional in the returned closure.

**Answer:** False.

**Explanation:** A placeholder always creates a required closure parameter.

### 34. Without a final **...**, extra arguments passed to the closure are forwarded to the target function.

**Answer:** False.

**Explanation:** They are ignored. Add the final **...** to forward remaining arguments.

### 35. Fixed argument expressions are evaluated every time the returned closure is invoked.

**Answer:** False.

**Explanation:** They are evaluated once when the closure is created.

### 36. Named placeholders can reorder the parameters of the returned closure.

**Answer:** True.

**Explanation:** The closure parameter order follows the order in which the named placeholders appear.

### 37. Partial function application was added in PHP 8.1.

**Answer:** False.

**Explanation:** PHP 8.1 added first-class callable syntax. Partial function application arrived in PHP 8.6.

## Predict the output

### 38. What does this code print?

~~~php
<?php
function volume(int $length, int $width, int $height): int
{
    return $length * $width * $height;
}

$fixedLength = volume(2, ?, ?);

echo $fixedLength(3, 4), "\n";
~~~

**Answer:**

~~~text
24
~~~

**Explanation:** The closure supplies **3** as width and **4** as height, so **volume(2, 3, 4)** returns **24**.

### 39. What does this code print?

~~~php
<?php
function greetWithDefault(
    string $name,
    string $greeting = 'Hello',
): string {
    return "{$greeting}, {$name}!";
}

$greet = greetWithDefault(?);

echo $greet('Asha', 'Goodbye'), "\n";
~~~

**Answer:**

~~~text
Hello, Asha!
~~~

**Explanation:** The closure has one placeholder and no final **...**, so the extra **'Goodbye'** argument is ignored. The original function uses its default greeting.

### 40. What does this code print?

~~~php
<?php
function greetWithDefault(
    string $name,
    string $greeting = 'Hello',
): string {
    return "{$greeting}, {$name}!";
}

$greet = greetWithDefault(?, ...);

echo $greet('Asha', 'Goodbye'), "\n";
~~~

**Answer:**

~~~text
Goodbye, Asha!
~~~

**Explanation:** The final **...** forwards the second argument to the original function's greeting parameter.

### 41. What does this code print?

~~~php
<?php
function replaceText(
    string $search,
    string $replace,
    string $subject,
): string {
    return str_replace($search, $replace, $subject);
}

$removeDashes = replaceText(
    subject: ?,
    search: '-',
    replace: '',
);

echo $removeDashes('a-b-c'), "\n";
~~~

**Answer:**

~~~text
abc
~~~

**Explanation:** The named placeholder leaves **subject** as the closure's only parameter. The other arguments are fixed when the closure is created.

### 42. What does this code print?

~~~php
<?php
function announce(): string
{
    echo "Selected\n";
    return 'Ready';
}

function label(string $prefix, string $name): string
{
    return "{$prefix}: {$name}";
}

$makeLabel = label(announce(), ?);

echo "Created\n";
echo $makeLabel('PHP'), "\n";
echo $makeLabel('HTML'), "\n";
~~~

**Answer:**

~~~text
Selected
Created
Ready: PHP
Ready: HTML
~~~

**Explanation:** **announce()** runs once while **$makeLabel** is created. The captured string is reused for both later calls.

## Write code

### 43. Use partial application and **array_map()** to remove spaces from **['red apple', 'green pear']**.

**Answer:**

~~~php
<?php
$removeSpaces = str_replace(' ', '', ?);

$words = ['red apple', 'green pear'];
$compact = array_map($removeSpaces, $words);

echo implode(', ', $compact), "\n";
~~~

Output:

~~~text
redapple, greenpear
~~~

**Explanation:** The search and replacement values are fixed; the **?** receives each subject string.

### 44. Create a closure that checks whether each value is strictly in **$allowed**.

Assume **$allowed** is an array and use **in_array()**.

**Answer:**

~~~php
<?php
$allowed = ['draft', 'published'];

$isAllowed = in_array(?, $allowed, strict: true);

var_dump($isAllowed('draft'));
var_dump($isAllowed('archived'));
~~~

Output:

~~~text
bool(true)
bool(false)
~~~

**Explanation:** The candidate value is left as the closure's parameter. The allowed list and strict comparison flag are fixed.

### 45. Delay a call to **volume(2, 3, 4)** until the returned closure is invoked.

**Answer:**

~~~php
<?php
function volume(int $length, int $width, int $height): int
{
    return $length * $width * $height;
}

$calculate = volume(2, 3, 4, ...);

echo "Ready\n";
echo $calculate(), "\n";
~~~

Output:

~~~text
Ready
24
~~~

**Explanation:** All target arguments are provided, but the final **...** delays the function call. Invoking the zero-argument closure performs the calculation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [043 - Callable Values and First-Class Callable Syntax](../lessons/043-callable-values-and-first-class-callable-syntax.md) | [Read lesson](../lessons/044-partial-function-application.md) | [Open example](../examples/044-partial-function-application.md) | [045 - Passing and Returning Values by Reference](../lessons/045-passing-and-returning-values-by-reference.md) |
