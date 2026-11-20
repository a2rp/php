# 011 - Variable Scope, Global Variables, and Static Locals

A variable's **scope** is where PHP code can use that variable. Variables created at the top level of a script and variables created inside functions do not automatically share the same scope. Understanding this boundary helps prevent accidental reads and changes.

## What you will learn

- What global and function scope mean
- Why a function cannot automatically read a top-level variable
- How function parameters and local variables work
- How **global** makes a global variable available inside a function
- How a static local variable keeps its value between function calls
- When to keep data local instead of relying on global state

## What is variable scope?

A variable's scope is the part of a program where that variable is available by its name. In this chapter, we will work with two common scopes:

- **Global scope:** variables assigned outside a function.
- **Function scope:** variables and parameters that belong to a function.

A variable does not become available everywhere just because it was assigned earlier in the file.

## Global variables are outside functions

A variable assigned outside a function belongs to global scope:

~~~php
<?php
$courseTitle = "PHP Fundamentals";
~~~

A named function has its own local scope. It cannot automatically read a global variable by writing the same name:

~~~php
<?php
$courseTitle = "PHP Fundamentals";

function showCourseTitle()
{
    echo $courseTitle;
}

showCourseTitle();
~~~

Inside **showCourseTitle()**, **$courseTitle** refers to a function-local variable. No local value was assigned, so PHP reports an undefined-variable diagnostic when the function runs. The top-level variable with the same spelling is a different variable in this context.

## Function variables and parameters are local

Variables assigned inside a function belong to that function call. Function parameters are also local variables; they receive the argument provided by the caller.

~~~php
<?php
function showCourseTitle($title)
{
    $message = "Course: " . $title;
    echo $message, PHP_EOL;
}

showCourseTitle("PHP Fundamentals");
~~~

Here, **$title** receives the argument, and **$message** is created inside the function. Both names are local to **showCourseTitle()**. The function does not need to read a global variable.

A local variable created during a normal function call does not keep its value for the next call. Each call starts a new function scope.

## Use **global** to access a global variable in a function

PHP's **global** keyword brings a global variable into the function's local scope. The local name then refers to the global variable, so assigning to it changes the global value:

~~~php
<?php
$completedLessons = 9;

function completeOneMoreLesson()
{
    global $completedLessons;
    $completedLessons++;
}

completeOneMoreLesson();

echo $completedLessons, PHP_EOL;
~~~

The output is **10**. Without **global $completedLessons;**, the function would not automatically use the top-level variable.

You can list more than one variable after **global**, separated by commas, but a function that needs many globals is hard to understand and reuse. Prefer passing the required value as an argument and returning a result when that design fits:

~~~php
<?php
function addOne($number)
{
    return $number + 1;
}

$completedLessons = 9;
$completedLessons = addOne($completedLessons);

echo $completedLessons, PHP_EOL;
~~~

This function makes its input and result visible at the call site. Function arguments and return values are covered in more detail later.

## Use a static local when a function must remember a value

A **static local variable** stays local to its function, but it retains its value between calls. Initialize it with the **static** keyword:

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

The output is:

~~~text
1
2
3
~~~

The variable **$visitNumber** is available only inside **nextVisitNumber()**. Unlike a normal local variable, its initialization and value persist from one call to the next. A later call continues from the value left by the previous call.

A static local variable is not the same thing as a global variable: it does not become available throughout the script. It is useful for a small piece of state that belongs to one function. Do not use it as hidden storage when callers should be able to provide or inspect that state.

This course uses simple literal initial values for static locals. More advanced initialization rules depend on the PHP version and belong outside this beginner example.

## Compare the three approaches

| Kind of variable | Where it is assigned | Where it is available | What happens across function calls |
| --- | --- | --- | --- |
| Global variable | Outside a function | Global scope; inside a function only when explicitly brought in | Its value remains available in global scope. |
| Ordinary local variable | Inside a function or as a parameter | That function's scope | A fresh local variable is used for each call. |
| Static local variable | Inside a function with **static** | Only that function's scope | It keeps its value between calls. |

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming a function can automatically read a top-level variable | Function scope is separate from global scope. | Pass the value as an argument, or deliberately use **global** when appropriate. |
| Assigning to a same-named local and expecting the global to change | The local and global names refer to separate variables unless explicitly connected. | Use **global** if the function is intentionally meant to change the global value. |
| Expecting a normal local variable to keep its value after the function returns | A normal local belongs to that call. | Return the result, pass state in, or use a static local only when function-owned persistent state is appropriate. |
| Treating a static local as a global variable | Static locals remain private to their function. | Keep the value inside its function and return any result callers need. |
| Using many globals inside one function | Dependencies and side effects become difficult to see. | Prefer parameters and return values for ordinary data flow. |
| Adding **global** without realizing it changes shared state | Assignments through the imported name update the global variable. | Use **global** deliberately and make the side effect clear. |

## Try it yourself

1. Create a variable at the top level and write a function that tries to print it without **global**. Explain the diagnostic, then remove the incorrect call.
2. Add a parameter to that function and pass the value in from the caller. Print the parameter inside the function.
3. Create **$completedLessons** globally and change it inside a function using **global**. Print it after the call.
4. Rewrite the previous exercise so the function accepts a number and returns the incremented result instead of changing a global.
5. Write a function with an ordinary local counter initialized to zero. Call it three times and note the returned values.
6. Change that counter to a static local and call the function three times again. Explain why the results differ.
7. Identify which names are available outside a function: its parameter, its ordinary local variable, its static local variable, and a top-level variable.
8. Read a function that uses **global** and list every global variable it depends on. Consider whether passing these values as arguments would make the function clearer.

## Key points

- Global scope and function scope are separate.
- A function cannot automatically use a top-level variable by writing its name.
- Parameters and ordinary variables inside a function are local to that function.
- **global $name;** makes a function-local name refer to a variable in global scope; changes affect the global variable.
- A static local variable remains private to its function but retains its value between calls.
- Prefer clear inputs and outputs over hidden global state for ordinary functions.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [010 - Variables and assignment](010-variables-and-assignment.md) | [Open Q&A](../qnas/011-variable-scope-global-variables-and-static-locals.md) | [Open example](../examples/011-variable-scope-global-variables-and-static-locals.md) | [012 - Superglobals and external input](012-superglobals-and-external-input.md) |
