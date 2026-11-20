# 011 - Variable Scope, Global Variables, and Static Locals: Questions and Answers

Use this Q&A to review [011 - Variable Scope, Global Variables, and Static Locals](../lessons/011-variable-scope-global-variables-and-static-locals.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does a variable's scope describe?

A. How many times the variable is assigned<br>
B. Where the variable can be used by its name<br>
C. Which data type PHP must use<br>
D. Whether the variable is printed

**Answer: B.** Scope is the context in which a variable is available. PHP function scope and global scope are separate.

### 2. A variable assigned at the top level of a PHP file belongs to which scope?

A. Function scope<br>
B. Global scope<br>
C. Static scope only<br>
D. Parameter scope

**Answer: B.** A variable assigned outside a function belongs to global scope.

### 3. Can a function automatically read a top-level variable just by writing its name inside the function?

A. Yes, every variable is available everywhere in PHP<br>
B. No, the function has its own scope<br>
C. Yes, but only when the variable is a string<br>
D. No, because PHP has no global variables

**Answer: B.** A function does not automatically inherit a top-level variable. Pass the value as a parameter or deliberately import it with **global**.

### 4. Which choice makes a value available to a function without relying on a global variable?

A. Pass it as an argument to a parameter<br>
B. Rename it inside the function<br>
C. Print it before the function call<br>
D. Add a semicolon to its global assignment

**Answer: A.** A function parameter receives an argument from the caller. This makes the dependency visible where the function is called.

### 5. What does **global $completedLessons;** do inside a function?

A. Creates a constant<br>
B. Makes the function's local name refer to the global variable<br>
C. Copies the global value into a new, unrelated local variable<br>
D. Makes the variable available in every PHP file automatically

**Answer: B.** The local name refers to the global variable. Assigning through it changes the value in global scope.

### 6. What is the main behavior of a static local variable?

A. It becomes available in every function<br>
B. It retains its value between calls but remains local to its function<br>
C. It is reset to its initial value on every call<br>
D. It can only store strings

**Answer: B.** A static local variable keeps its value between calls while remaining accessible only in the function where it is declared.

### 7. Which declaration creates a static local counter?

A. **global $counter = 0;**<br>
B. **static $counter = 0;**<br>
C. **local static($counter);**<br>
D. **$counter static = 0;**

**Answer: B.** The **static** keyword before a local variable declaration gives that function-local variable persistent state across calls.

### 8. What happens to an ordinary local variable after the function call ends?

A. It automatically becomes global<br>
B. Its function call ends, and a later call starts with a fresh local variable<br>
C. It becomes a static local variable<br>
D. PHP saves it for the next call automatically

**Answer: B.** An ordinary local variable belongs to one call. It does not retain its value for the next call.

### 9. Which is usually the clearest way to provide ordinary input to a function?

A. Use a global variable for every input<br>
B. Pass the needed value as an argument<br>
C. Store it in a static local variable<br>
D. Read an undefined variable inside the function

**Answer: B.** Parameters make the function's inputs explicit and make it easier to reuse the function.

## True or false

### 10. A function can automatically use a global variable whenever the spelling matches a local variable name.

**Answer: False.** A function has its own scope. It must deliberately access the global variable, for example with **global**, or receive a value through a parameter.

### 11. If a function uses **global $total;** and then changes **$total**, the global value changes.

**Answer: True.** The local name brought in by **global** refers to the global variable, so assignments affect that shared value.

### 12. A static local variable remains available after its function returns, but only inside that function on a later call.

**Answer: True.** It retains its value between calls and stays private to its function.

### 13. A static local variable and a global variable have identical scope.

**Answer: False.** A global variable belongs to global scope. A static local variable persists between calls but can only be accessed in its function.

### 14. Function parameters are local variables.

**Answer: True.** A parameter receives the caller's argument and is available inside that function's scope.

## Read and predict

### 15. What does this code print?

~~~php
$courseTitle = "Global title";

function showCourseTitle()
{
    $courseTitle = "Local title";
    echo $courseTitle, PHP_EOL;
}

showCourseTitle();
echo $courseTitle, PHP_EOL;
~~~

**Answer:**

~~~text
Local title
Global title
~~~

The function assigns a local variable named **$courseTitle**. It is separate from the global variable with the same spelling.

### 16. What does this code print?

~~~php
$completedLessons = 9;

function addLesson()
{
    global $completedLessons;
    $completedLessons++;
}

addLesson();
echo $completedLessons, PHP_EOL;
~~~

**Answer:**

~~~text
10
~~~

The **global** declaration makes the function's **$completedLessons** refer to the global variable, so incrementing it changes the global value.

### 17. What does this function print over three calls?

~~~php
function nextVisitNumber()
{
    static $visitNumber = 0;
    $visitNumber++;
    echo $visitNumber, PHP_EOL;
}

nextVisitNumber();
nextVisitNumber();
nextVisitNumber();
~~~

**Answer:**

~~~text
1
2
3
~~~

The static local variable is initialized on its first use and keeps its updated value for later calls.

### 18. What does this function print over three calls?

~~~php
function showLocalCounter()
{
    $counter = 0;
    $counter++;
    echo $counter, PHP_EOL;
}

showLocalCounter();
showLocalCounter();
showLocalCounter();
~~~

**Answer:**

~~~text
1
1
1
~~~

The ordinary local variable is assigned **0** at the start of each call, then incremented once. Each call therefore prints **1**.

## Find and correct the mistake

### 19. Why can this function not print the top-level **$courseTitle** as written?

~~~php
$courseTitle = "PHP Fundamentals";

function showCourseTitle()
{
    echo $courseTitle;
}
~~~

**Answer:** The function cannot automatically read the global variable. Pass it into a parameter:

~~~php
$courseTitle = "PHP Fundamentals";

function showCourseTitle($title)
{
    echo $title, PHP_EOL;
}

showCourseTitle($courseTitle);
~~~

### 20. What is risky about this function?

~~~php
$totalLessons = 9;

function addLesson()
{
    global $totalLessons;
    $totalLessons++;
}
~~~

**Answer:** The function changes shared global state, but its dependency is not visible in the call **addLesson()**. For ordinary data flow, prefer a function that receives the value and returns the updated result:

~~~php
function addLesson($totalLessons)
{
    return $totalLessons + 1;
}

$totalLessons = 9;
$totalLessons = addLesson($totalLessons);
~~~

### 21. Why does this function print **0** on every call?

~~~php
function showCounter()
{
    $counter = 0;
    echo $counter, PHP_EOL;
    $counter++;
}
~~~

**Answer:** The ordinary local variable is assigned **0** again each time the function is called. If the function is intentionally responsible for remembering a count, use a static local variable. Otherwise, pass the count in and return the updated value.

## Coding practice

### 22. Write a function that receives a course title and prints it without using a global variable.

**Answer:**

~~~php
<?php

function showCourseTitle($title)
{
    echo "Course: ", $title, PHP_EOL;
}

$courseTitle = "PHP Fundamentals";
showCourseTitle($courseTitle);
~~~

The caller passes the value as an argument, and the function receives it in its local **$title** parameter.

### 23. Write a function that increments a global lesson count using **global**.

**Answer:**

~~~php
<?php

$completedLessons = 9;

function completeOneLesson()
{
    global $completedLessons;
    $completedLessons++;
}

completeOneLesson();
echo $completedLessons, PHP_EOL;
~~~

The output is **10**. The function deliberately changes the shared global variable.

### 24. Write a function that returns a visit number which increases across calls by using a static local.

**Answer:**

~~~php
<?php

function nextVisitNumber()
{
    static $visitNumber = 0;
    $visitNumber++;
    return $visitNumber;
}

echo nextVisitNumber(), PHP_EOL;
echo nextVisitNumber(), PHP_EOL;
echo nextVisitNumber(), PHP_EOL;
~~~

The output is **1**, **2**, and **3** on separate lines. The static local retains its value between calls but is still private to the function.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [010 - Variables and assignment](../lessons/010-variables-and-assignment.md) | [Read this lesson](../lessons/011-variable-scope-global-variables-and-static-locals.md) | [Open example](../examples/011-variable-scope-global-variables-and-static-locals.md) | [012 - Superglobals and external input](../lessons/012-superglobals-and-external-input.md) |
