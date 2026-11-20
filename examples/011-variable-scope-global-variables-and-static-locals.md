# 011 - Variable Scope, Global Variables, and Static Locals: Complete Example

This command-line example compares global variables, function-local variables, parameters, and static locals in one small PHP script.

## Question

How can you tell which variable a function uses, and how does a static local differ from a normal local variable?

## Goal

Create this project:

~~~text
scope-practice/
|-- README.md
|-- scope.php
~~~

The **scope.php** program will:

- Print a value passed into a function as an argument
- Show that a same-named local variable does not overwrite the global variable
- Deliberately update a global variable with **global**
- Compare a normal local counter with a static local counter

## Complete code: scope.php

Save this file at the project root:

~~~php
<?php

$courseTitle = "PHP Fundamentals";
$completedLessons = 9;

function showCourseTitle($title)
{
    $message = "Course from parameter: " . $title;
    echo $message, PHP_EOL;
}

function showLocalTitle()
{
    $courseTitle = "Function-local title";
    echo "Title inside function: ", $courseTitle, PHP_EOL;
}

function completeOneLesson()
{
    global $completedLessons;
    $completedLessons++;
}

function nextOrdinaryCounter()
{
    $counter = 0;
    $counter++;
    return $counter;
}

function nextStaticCounter()
{
    static $counter = 0;
    $counter++;
    return $counter;
}

echo "Global course title before calls: ", $courseTitle, PHP_EOL;

showCourseTitle($courseTitle);
showLocalTitle();

echo "Global course title after local function: ", $courseTitle, PHP_EOL;

echo "Completed lessons before update: ", $completedLessons, PHP_EOL;
completeOneLesson();
echo "Completed lessons after global update: ", $completedLessons, PHP_EOL;

echo "Ordinary local counter: ", nextOrdinaryCounter(), PHP_EOL;
echo "Ordinary local counter: ", nextOrdinaryCounter(), PHP_EOL;

echo "Static local counter: ", nextStaticCounter(), PHP_EOL;
echo "Static local counter: ", nextStaticCounter(), PHP_EOL;
echo "Static local counter: ", nextStaticCounter(), PHP_EOL;
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Scope Practice

A small PHP CLI script demonstrating global scope, function scope, and static locals.

## Check and run

~~~sh
php -l scope.php
php scope.php
~~~

## What it demonstrates

- Passing a value into a function parameter
- Creating a local variable that has the same name as a global
- Using global when a function should change global state
- Comparing a fresh local variable with a static local variable
~~~~

This Markdown page is the README text to create in the project. The PHP program is saved separately as **scope.php**.

## Check and run the program

Open a terminal in **scope-practice/** and check the syntax:

~~~sh
php -l scope.php
~~~

For a valid file, PHP reports that no syntax errors were detected.

Run the program:

~~~sh
php scope.php
~~~

Expected terminal output:

~~~text
Global course title before calls: PHP Fundamentals
Course from parameter: PHP Fundamentals
Title inside function: Function-local title
Global course title after local function: PHP Fundamentals
Completed lessons before update: 9
Completed lessons after global update: 10
Ordinary local counter: 1
Ordinary local counter: 1
Static local counter: 1
Static local counter: 2
Static local counter: 3
~~~

## Understand the output

### The parameter receives a value from the caller

The call **showCourseTitle($courseTitle)** passes the current title to the function's **$title** parameter. The function uses the value it receives; it does not need direct access to the global variable.

### A local variable can have the same name as a global

**showLocalTitle()** assigns **"Function-local title"** to a local **$courseTitle**. The final global title is still **"PHP Fundamentals"** because that local variable is separate from the top-level variable.

### **global** deliberately changes the shared value

**completeOneLesson()** contains **global $completedLessons;**. That makes the function's name refer to the global variable, so the update from **9** to **10** remains visible after the function returns.

### A normal local starts fresh; a static local remembers

**nextOrdinaryCounter()** assigns **0** on every call, increments it to **1**, and returns it. Each call prints **1**.

**nextStaticCounter()** initializes its local **$counter** once and preserves its updated value between calls. The three calls print **1**, **2**, and **3**. The static variable is still private to that function.

In this CLI example, the static counter lasts for the current PHP script run. Running **php scope.php** again starts a new process and a fresh counter. In a typical web request, a static local also does not store data between separate requests.

## Make a change

1. Change **$courseTitle** at the top of the file. Predict what **showCourseTitle()** prints, then run the program.
2. Change the local **$courseTitle** inside **showLocalTitle()**. Check whether the global course title printed afterward changes.
3. Remove **global $completedLessons;** in a scratch copy. Observe that the function no longer updates the top-level value as intended, then restore the declaration.
4. Call **nextOrdinaryCounter()** five times. Explain why each result is **1**.
5. Call **nextStaticCounter()** two more times. Predict the next values.
6. Replace the body of **nextStaticCounter()** with an ordinary local variable. Compare the results and explain which assignment resets the count.
7. Rewrite **completeOneLesson()** to accept a number and return the incremented number instead of using a global variable.

A parameter-and-return version of the final change could look like this:

~~~php
function addOneLesson($completedLessons)
{
    return $completedLessons + 1;
}

$completedLessons = addOneLesson($completedLessons);
~~~

This version makes the input and output visible at the call site. It does not modify global state from inside the function.

## Common mistakes

| Mistake | What happens | Correction |
| --- | --- | --- |
| Reading **$courseTitle** inside a function without passing or importing it | The function does not automatically see the global variable. | Pass it as an argument or use **global** deliberately. |
| Assuming the local **$courseTitle** changes the top-level variable | The same spelling refers to separate variables in separate scopes. | Use **global** only when a function is intentionally meant to update the global. |
| Removing **global $completedLessons;** and expecting the top-level count to change | The function no longer refers to the global variable. | Restore the **global** declaration, or pass the count in and return an updated value. |
| Expecting the ordinary counter to increase across calls | It is assigned **0** again for each call. | Use a static local only when state should belong to the function, or pass state explicitly. |
| Expecting the static counter to be accessible outside its function | Static changes how long the local value is retained, not where its name is available. | Return the value if the caller needs to use it. |
| Assuming a static local stores state between separate PHP requests | Static locals retain state across calls during a script execution, not as persistent storage between ordinary requests. | Use an appropriate storage mechanism for state that must survive separate requests. |

## Key takeaways

- A parameter receives a value from the function caller and is local to the function.
- A same-named local variable does not automatically refer to a global variable.
- **global** intentionally connects a function's local name to a global variable.
- A normal local variable is initialized again on each function call.
- A static local keeps its value between calls but remains accessible only inside its function.
- Prefer arguments and return values for ordinary data flow; use hidden state deliberately.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [010 - Variables and assignment](../lessons/010-variables-and-assignment.md) | [Read this lesson](../lessons/011-variable-scope-global-variables-and-static-locals.md) | [Open Q&A](../qnas/011-variable-scope-global-variables-and-static-locals.md) | [012 - Superglobals and external input](../lessons/012-superglobals-and-external-input.md) |
