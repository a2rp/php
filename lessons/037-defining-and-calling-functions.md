# 037 - Defining and Calling Functions

A function is a named, reusable block of code. Define it once, then call it whenever the program needs that behavior. This chapter introduces named functions without parameters; the next chapter explains how to pass information into them.

## What you will learn

- Recognize a PHP function declaration
- Call a function by its name and parentheses
- Follow execution through a function call
- Use functions to avoid repeating the same behavior
- Choose clear names and avoid duplicate declarations
- Understand when a conditional function declaration becomes available

## Why use a function?

Copying the same statements in several places makes later edits easy to miss. A function gives repeated behavior one name:

~~~php
<?php
function showWelcome(): void
{
    echo "Welcome to the PHP course!\n";
}

showWelcome();
showWelcome();
~~~

Output:

~~~text
Welcome to the PHP course!
Welcome to the PHP course!
~~~

The declaration defines what **showWelcome()** does. Each call runs its body once. Calling it twice runs the body twice.

## Declare a named function

The basic syntax is:

~~~php
function functionName(): void
{
    // Statements that run when the function is called
}
~~~

- **function** starts a declaration.
- **functionName** is the name used to call the function.
- **()** contains the parameter list. It is empty in this chapter.
- **: void** says the function does not return a value.
- **{ }** enclose the function body.

A declaration makes the function available; it does not run the body. The body runs only when execution reaches a call.

~~~php
<?php
function printDivider(): void
{
    echo "--------------------\n";
}

echo "Report\n";
printDivider();
echo "End of report\n";
~~~

Output:

~~~text
Report
--------------------
End of report
~~~

Execution reaches **printDivider()**, enters its body, prints the divider, then resumes after the call.

## Call a function

Write its name followed by parentheses and a semicolon:

~~~php
showWelcome();
~~~

The parentheses are required even when the function accepts no information.

This is a declaration:

~~~php
function showWelcome(): void
{
    echo "Welcome!\n";
}
~~~

This is a call:

~~~php
showWelcome();
~~~

A declaration creates the operation; a call asks PHP to perform it. Writing a declaration alone does not print anything.

## Follow execution flow

When PHP reaches a call, it runs the function body and then continues after the call:

~~~php
<?php
function announceStart(): void
{
    echo "Inside the function\n";
}

echo "Before the call\n";
announceStart();
echo "After the call\n";
~~~

Output:

~~~text
Before the call
Inside the function
After the call
~~~

A function call is an instruction in the surrounding program. This execution order matters when a function prints text, changes state, or performs another action.

## Keep repeated behavior in one place

~~~php
<?php
function printSectionHeading(): void
{
    echo "\n====================\n";
}

echo "Profile\n";
printSectionHeading();
echo "Name: Asha\n";

echo "Settings\n";
printSectionHeading();
echo "Theme: Dark\n";
~~~

Output:

~~~text
Profile

====================
Name: Asha
Settings

====================
Theme: Dark
~~~

If the divider changes, edit the function once. Every call uses the updated body. Functions are most useful when they group a meaningful action or make repeated logic easier to maintain. Do not create a function for every single line by default.

## Choose clear names

A function name should describe its action. Common PHP style uses **camelCase** for user-defined function names:

~~~php
function calculateSubtotal(): void
{
    echo "Subtotal calculated\n";
}
~~~

PHP treats function names without regard to letter case, so **calculateSubtotal()** and **calculatesubtotal()** refer to the same function. Still, keep the spelling and capitalization consistent so the code is easy to search and review.

A function name begins with a letter or underscore and can contain letters, numbers, and underscores. Names cannot contain spaces. Avoid PHP keywords and names already used by other functions.

## Declaration order and conditional declarations

A normal, unconditional named function can be declared later in a file and called earlier:

~~~php
<?php
printMessage();

function printMessage(): void
{
    echo "The function was declared later in the file.\n";
}
~~~

Output:

~~~text
The function was declared later in the file.
~~~

A declaration nested inside a conditional is different: PHP creates that function only if execution reaches the declaration.

~~~php
<?php
$enabled = true;

if ($enabled) {
    function printEnabledMessage(): void
    {
        echo "Feature enabled\n";
    }
}

printEnabledMessage();
~~~

Here the condition is true, so the declaration runs before the call. If it were false, the function would not exist and the call would fail with an **Error** for an undefined function. For beginner code, declare ordinary functions at the top level, outside conditionals.

## Function names must be unique

A function name identifies one function in its namespace. PHP does not let you declare two functions with the same name and choose between them by changing their parameter lists. A duplicate declaration causes a fatal error when PHP processes it.

~~~php
<?php
function printStatus(): void
{
    echo "Ready\n";
}

// Declaring another printStatus() here is invalid.
~~~

Later lessons cover namespaces, methods, and other callable forms. For now, give every named function a unique, descriptive name.

## Common mistakes

- Writing **showWelcome;** instead of calling **showWelcome()**.
- Expecting the body to run just because the function was declared.
- Forgetting parentheses on a no-parameter call.
- Calling a conditionally declared function when its declaration branch did not run.
- Declaring the same function name more than once.
- Changing capitalization between declaration and call, even though PHP ignores case in function names.
- Making tiny functions that obscure rather than clarify the program.
- Expecting printed text to be a returned value. Output and return values are separate.

## Try it yourself

1. Define **showCourseName()** so it prints a course name.
2. Call it once before and once after another message.
3. Define **printLine()** that prints ten asterisks, then use it around a heading.
4. Change a function body and observe how every call uses the updated behavior.
5. Explain which lines declare a function and which lines call one.

## Key takeaways

- A named function packages reusable behavior behind a descriptive name.
- A declaration uses **function**, a name, parentheses, and a braced body.
- The body runs when called, not merely when declared.
- A call uses the function name with parentheses.
- An unconditional function declaration can appear later in the same file; a conditional declaration must execute before its function is called.
- Function names are case-insensitive in PHP, but consistent capitalization improves clarity.
- Function names must be unique; PHP does not overload functions by parameter list.
- The next chapter covers parameters and arguments so functions can receive caller-provided values.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [036 - Returning Values and Ending Execution](036-returning-values-and-ending-execution.md) | [Open Q&A](../qnas/037-defining-and-calling-functions.md) | [Open example](../examples/037-defining-and-calling-functions.md) | [038 - Function Parameters and Arguments](038-function-parameters-and-arguments.md) |
