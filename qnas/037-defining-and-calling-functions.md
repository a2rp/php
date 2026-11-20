# 037 - Defining and Calling Functions: Q&A

These questions review named PHP functions, their declarations, calls, execution order, naming, and conditional declarations. Read the [chapter lesson](../lessons/037-defining-and-calling-functions.md) first, then try answering before revealing each answer.

## Multiple-choice questions

### 1. What is the main purpose of a function?

A. To give a reusable block of code a name
B. To turn every PHP file into HTML
C. To make a comment execute
D. To rename a variable

**Answer:** A. To give a reusable block of code a name.

**Explanation:** A function groups behavior so a program can call it by name when needed.

### 2. Which keyword starts a named function declaration?

A. **call**
B. **function**
C. **define**
D. **method**

**Answer:** B. **function**.

**Explanation:** A named function declaration begins with the **function** keyword.

### 3. Which line calls a no-parameter function named **showWelcome**?

A. **function showWelcome(): void**
B. **showWelcome;**
C. **showWelcome();**
D. **call showWelcome()**

**Answer:** C. **showWelcome();**

**Explanation:** A call uses the function name followed by parentheses. The semicolon ends the statement.

### 4. What do the braces in a function declaration enclose?

A. The function body
B. The function name
C. The PHP opening tag
D. The call statement

**Answer:** A. The function body.

**Explanation:** Statements between the braces run when the function is called.

### 5. What happens when PHP processes an ordinary function declaration?

A. Its body always runs immediately
B. The function becomes available; the body waits for a call
C. PHP prints the function name
D. The current script exits

**Answer:** B. The function becomes available; its body waits for a call.

**Explanation:** Declaring a function defines it. Calling it executes its body.

### 6. If a function is called twice, how many times does its body run?

A. Zero
B. Once
C. Twice
D. It depends on the function name's capitalization

**Answer:** C. Twice.

**Explanation:** Each call runs the function body once, unless the program stops before reaching a call.

### 7. What does **: void** express in a function declaration?

A. The function is unavailable
B. The function is allowed to accept no parentheses
C. The function does not return a value
D. The function runs only once

**Answer:** C. The function does not return a value.

**Explanation:** **void** documents that the function performs an action without returning a value.

### 8. What does PHP do after a function body finishes during a call?

A. It resumes after the call
B. It restarts the file from the beginning
C. It deletes the function
D. It exits the script

**Answer:** A. It resumes after the call.

**Explanation:** The caller pauses while the function runs, then execution continues at the next statement.

### 9. How do you call a function that accepts no information?

A. **showWelcome**
B. **showWelcome()**
C. **function showWelcome()**
D. **void showWelcome**

**Answer:** B. **showWelcome()**.

**Explanation:** Parentheses are part of the call syntax, even when the parameter list is empty.

### 10. Are PHP function names case-sensitive?

A. Yes, always
B. No, PHP treats function names without regard to letter case
C. Only names with underscores are case-sensitive
D. Only built-in function names are case-sensitive

**Answer:** B. No, function names are case-insensitive.

**Explanation:** Different capitalization refers to the same function. Consistent capitalization is still easier to read.

### 11. Which is a valid style of function name?

A. **print section**
B. **2printSection**
C. **_printSection2**
D. **print-section**

**Answer:** C. **_printSection2**.

**Explanation:** A function name can start with a letter or underscore and may contain letters, numbers, and underscores. Spaces and hyphens are not valid in a function identifier.

### 12. Can a normal, unconditional named function be called before its declaration appears later in the same file?

A. Yes
B. No, never
C. Only when the name is uppercase
D. Only if the function uses **echo**

**Answer:** A. Yes.

**Explanation:** PHP can resolve an ordinary unconditional function declaration in the file before execution reaches its textual location.

### 13. When is a function declared inside an **if** block created?

A. Whenever PHP parses the file
B. Only when execution reaches that declaration
C. Only after the script exits
D. Only when the function is called with uppercase letters

**Answer:** B. Only when execution reaches that declaration.

**Explanation:** Conditional declarations depend on the branch running. A call made when the branch did not run refers to an undefined function.

### 14. What happens if a file declares the same function name twice?

A. PHP chooses the second declaration
B. PHP combines both bodies
C. A fatal error occurs when the duplicate declaration is processed
D. The first function silently changes its name

**Answer:** C. A fatal error occurs.

**Explanation:** Function names must be unique within their namespace; redeclaring one is invalid.

### 15. Does PHP choose between two same-named user-defined functions by their parameter lists?

A. Yes, that is function overloading
B. No, PHP does not overload functions this way
C. Only if both are declared with **void**
D. Only if the calls use different capitalization

**Answer:** B. No, PHP does not overload functions this way.

**Explanation:** A function name identifies one function. Distinct behavior needs distinct names or a different design.

## Short-answer questions

### 16. What is the difference between declaring a function and calling it?

**Answer:** A declaration defines the function's name and body. A call asks PHP to run that body.

**Explanation:** The declaration does not itself perform the function's action.

### 17. Write the general shape of a no-parameter function that returns no value.

**Answer:**

~~~php
function functionName(): void
{
    // Function body
}
~~~

**Explanation:** The empty parentheses show there are no parameters. The braces contain the statements to run.

### 18. Why do even no-parameter function calls include parentheses?

**Answer:** Parentheses are part of PHP's function-call syntax and show that the function is being invoked.

**Explanation:** Write **functionName()**, not just the bare name.

### 19. If a function prints a message, does merely declaring it print that message?

**Answer:** No. The body prints the message only when the function is called.

**Explanation:** Definition and execution are separate steps.

### 20. What happens to execution when a function call is reached?

**Answer:** PHP runs the function body, then resumes the caller after the call.

**Explanation:** Statements before the call have already run; statements after it wait until the function finishes.

### 21. Why can putting repeated behavior in a function make a program easier to maintain?

**Answer:** The behavior has one implementation, so a change can be made in one place and reused by all calls.

**Explanation:** This reduces duplicated code and makes the purpose clearer when the function has a descriptive name.

### 22. What does the **void** return type communicate?

**Answer:** The function is intended to perform an action without returning a value.

**Explanation:** Printing output is an action; it does not mean the function returned that printed text.

### 23. Give two naming guidelines for user-defined functions.

**Answer:** Start a name with a letter or underscore, and use a descriptive, consistent style such as camelCase.

**Explanation:** Names cannot contain spaces. Avoid PHP keywords and names already used by another function.

### 24. PHP ignores case in function names. Why should a project still use consistent capitalization?

**Answer:** Consistent spelling makes declarations and calls easier to read, search, and review.

**Explanation:** Case-insensitivity is a language behavior, not a reason to vary the spelling.

### 25. When is a conditionally declared function available?

**Answer:** After execution reaches its declaration inside the branch.

**Explanation:** If the condition prevents that code from running, the function is not declared.

### 26. What error occurs if code calls a conditionally declared function whose branch did not run?

**Answer:** PHP raises an **Error** for an undefined function.

**Explanation:** No function with that name was created on that execution path.

### 27. Why should beginner code usually put ordinary function declarations at the top level rather than inside conditions?

**Answer:** Top-level declarations are easier to find and are available without depending on a branch executing first.

**Explanation:** Conditional declarations introduce execution-order pitfalls and are uncommon in ordinary beginner examples.

### 28. Can two functions with the same name be distinguished by giving them different parameter lists?

**Answer:** No.

**Explanation:** PHP does not overload user-defined functions based on parameter count or types. Use a unique name for each function.

### 29. Does text printed by **echo** automatically become a function's return value?

**Answer:** No.

**Explanation:** Output sent by **echo** and a value returned with **return** are different. A later lesson covers return values.

## True or false

### 30. A function body runs as soon as PHP sees its declaration.

**Answer:** False.

**Explanation:** A call runs the body.

### 31. Calling a no-parameter function still requires parentheses.

**Answer:** True.

**Explanation:** For example, call it as **showWelcome()**.

### 32. Each reached call runs the function body once.

**Answer:** True.

**Explanation:** Two reached calls normally run the body twice.

### 33. A function declaration and a function call mean the same thing.

**Answer:** False.

**Explanation:** The declaration defines the operation; the call executes it.

### 34. PHP treats function names without regard to capitalization.

**Answer:** True.

**Explanation:** Consistent capitalization remains good style.

### 35. A function declared inside a conditional exists even if that branch never runs.

**Answer:** False.

**Explanation:** Execution must reach the conditional declaration.

### 36. PHP can overload a user-defined function by selecting a same-named declaration according to its parameter list.

**Answer:** False.

**Explanation:** Duplicate function names are invalid; PHP does not select by signature.

### 37. A descriptive function can help explain what a repeated block of code does.

**Answer:** True.

**Explanation:** A clear name makes the intention visible at each call site.

## Predict the output

### 38. What is printed?

~~~php
<?php
function sayHello(): void
{
    echo "Hello\n";
}

sayHello();
~~~

**Answer:**

~~~text
Hello
~~~

**Explanation:** The declaration defines the function; the call runs its body once.

### 39. What is printed, and in what order?

~~~php
<?php
function inside(): void
{
    echo "B\n";
}

echo "A\n";
inside();
echo "C\n";
~~~

**Answer:**

~~~text
A
B
C
~~~

**Explanation:** PHP prints **A**, runs the function at the call and prints **B**, then continues and prints **C**.

### 40. What is printed?

~~~php
<?php
function printMark(): void
{
    echo "*";
}

printMark();
printMark();
echo "\n";
~~~

**Answer:**

~~~text
**
~~~

**Explanation:** Each call prints one asterisk. The final **echo** prints a newline.

### 41. What is printed?

~~~php
<?php
printLater();

function printLater(): void
{
    echo "Ready\n";
}
~~~

**Answer:**

~~~text
Ready
~~~

**Explanation:** This is a normal unconditional declaration, so PHP can resolve the function before the declaration's textual position.

### 42. If **$enabled** is false, what happens when the last line is reached?

~~~php
<?php
$enabled = false;

if ($enabled) {
    function showFeature(): void
    {
        echo "Enabled\n";
    }
}

showFeature();
~~~

**Answer:** The call fails with an **Error** because **showFeature()** was never declared on this execution path.

**Explanation:** The conditional block did not run, so PHP did not create the function.

## Write-code questions

### 43. Define **showCourseName()** so it prints **PHP Fundamentals** followed by a newline, then call it once.

**Answer:**

~~~php
<?php
function showCourseName(): void
{
    echo "PHP Fundamentals\n";
}

showCourseName();
~~~

**Explanation:** The function definition contains the behavior, and the final line invokes it.

### 44. Define **printLine()** to print ten asterisks and a newline. Call it before and after printing **PHP**.

**Answer:**

~~~php
<?php
function printLine(): void
{
    echo "**********\n";
}

printLine();
echo "PHP\n";
printLine();
~~~

**Explanation:** The same function is reused to print the divider on both sides of the heading.

### 45. Write a short program that prints **Before**, then prints **Inside** from a function call, then prints **After**.

**Answer:**

~~~php
<?php
function printInside(): void
{
    echo "Inside\n";
}

echo "Before\n";
printInside();
echo "After\n";
~~~

**Explanation:** The function runs when execution reaches **printInside()**, then control returns to the next statement.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [036 - Returning Values and Ending Execution](../lessons/036-returning-values-and-ending-execution.md) | [Read lesson](../lessons/037-defining-and-calling-functions.md) | [Open example](../examples/037-defining-and-calling-functions.md) | [038 - Function Parameters and Arguments](../lessons/038-function-parameters-and-arguments.md) |
