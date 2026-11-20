# 036 - Returning Values and Ending Execution: Questions and Answers

Review [036 - Returning Values and Ending Execution](../lessons/036-returning-values-and-ending-execution.md). Each answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. What does **return $value;** do inside a function?

A. Prints the value to the screen<br>
B. Ends the function and passes the value to its caller<br>
C. Ends every PHP process on the server<br>
D. Repeats the function

**Answer: B.** **return** ends the current function call and makes the value available to the caller.

### 2. Which statement sends text to output?

A. **return**<br>
B. **echo**<br>
C. **break**<br>
D. **continue**

**Answer: B.** **echo** produces output. **return** sends a value back from a function or ends the current file.

### 3. What value does an untyped function call produce if the function reaches its end without a return statement?

A. **false**<br>
B. **0**<br>
C. **null**<br>
D. An empty array

**Answer: C.** A function with no explicit return value produces **null**.

### 4. What does a function declared with the **void** return type promise?

A. It returns a string<br>
B. It does not return a value<br>
C. It stops the whole script<br>
D. It always returns **false**

**Answer: B.** A **void** function performs work without returning a value.

### 5. How can a function return multiple related pieces of data?

A. Return several values separated by commas<br>
B. Return one array or object containing those pieces<br>
C. Call **echo** several times<br>
D. Use **exit**

**Answer: B.** A function returns one value; an array or object can group related data into that value.

### 6. What happens when **return** is reached inside a function?

A. Only the current statement is skipped<br>
B. The current function ends immediately<br>
C. The entire PHP script always ends<br>
D. The function starts again

**Answer: B.** Statements later in that function call are not executed after a return is reached.

### 7. What does a top-level **return** do in a file loaded by **require**?

A. Ends the whole calling script<br>
B. Ends the included file and returns control to the caller<br>
C. Restarts the included file<br>
D. Prints the return value automatically

**Answer: B.** At file scope, **return** ends the current file. When it is included, the caller continues after the require statement.

### 8. What does **exit** do?

A. Ends only the current function<br>
B. Terminates the entire current script<br>
C. Returns a value to the caller<br>
D. Skips one loop iteration

**Answer: B.** **exit** ends the script, even when it is called from a function or included file.

### 9. What is **die** in PHP?

A. A different spelling of **return**<br>
B. An alias of **exit**<br>
C. A loop statement<br>
D. A type declaration

**Answer: B.** **die** and **exit** have the same behavior.

### 10. If **exit** receives a string, what is the normal exit status?

A. **0**<br>
B. **1**<br>
C. The string's length<br>
D. No status is set

**Answer: A.** A string is printed before termination, and PHP exits with status **0**.

### 11. If **exit** receives an integer, what does that integer represent?

A. Text to print<br>
B. The command-line exit status<br>
C. A loop count<br>
D. A function return type

**Answer: B.** An integer argument is used as the script's exit status; PHP does not automatically print it.

### 12. What happens to a pending **finally** block when **exit** is called?

A. It always runs before the script stops<br>
B. It does not run<br>
C. It runs twice<br>
D. It becomes a shutdown function

**Answer: B.** PHP does not execute pending **finally** blocks when **exit** terminates the script.

### 13. Which cleanup still happens during **exit**?

A. Registered shutdown functions and object destructors run<br>
B. Every pending **finally** block runs<br>
C. The caller resumes after the **exit** call<br>
D. The script continues after **exit**

**Answer: A.** PHP runs shutdown functions and object destructors, although it does not continue normal execution.

### 14. Which status conventionally indicates success to a command-line shell?

A. **0**<br>
B. **1**<br>
C. **255**<br>
D. Any string

**Answer: A.** A zero exit status conventionally means success; a non-zero status indicates a failure or other condition.

### 15. Which is the better way for a CLI program to report a failure?

A. Print an error-like message using **exit('Failed')** and assume it is a failure status<br>
B. Write a diagnostic to **STDERR** and exit with a non-zero integer<br>
C. Return a string from the main file<br>
D. Use **die** in every helper function

**Answer: B.** A string passed to **exit** uses status **0**. Writing to **STDERR** and exiting with a non-zero code clearly reports failure.

## Short-answer questions

### 16. Write a function that returns the square of an integer.

**Answer:**

~~~php
<?php
function square(int $number): int
{
    return $number * $number;
}
~~~

**Explanation:** The returned integer becomes the value of a call to **square()**.

### 17. What is the difference between **echo** and **return**?

**Answer:** **echo** sends output. **return** passes a value to a caller and ends the current function or file.

**Explanation:** A displayed value is not automatically a returned value.

### 18. What happens to statements in a function after a reached **return** statement?

**Answer:** They are skipped for that function call.

**Explanation:** **return** immediately ends the current function.

### 19. How can a function provide a name, score, and status as one result?

**Answer:** Put the related values into an array or object and return that single value.

**Explanation:** PHP functions return one value at a time; a composite value can hold several fields.

### 20. What value does a function return when it reaches its end without an explicit return and has no declared return type?

**Answer:** **null**.

**Explanation:** The caller can observe this, for example with **var_dump()**.

### 21. Can a **void** function return a value such as a string?

**Answer:** No.

**Explanation:** A **void** function may use **return;** to stop early, but it cannot return an expression.

### 22. What is the purpose of an early return?

**Answer:** It handles a case and ends the function before the rest of its logic runs.

**Explanation:** This can make the remaining path simpler and reduce unnecessary nesting.

### 23. What does top-level **return** do in an included file?

**Answer:** It ends that included file and transfers control back to the including file.

**Explanation:** It does not terminate the entire caller script.

### 24. Can a top-level return value from a required file be assigned to a variable?

**Answer:** Yes.

**Explanation:** If the included file explicitly returns a value, the **include** or **require** expression can provide it to the caller.

### 25. How is **exit** different from **return** in an included file?

**Answer:** **return** ends only the included file; **exit** ends the entire script.

**Explanation:** With **return**, the including file resumes after the include statement.

### 26. What does **die** mean in PHP?

**Answer:** It is an alias of **exit**.

**Explanation:** Both terminate the current script.

### 27. What happens when a string is passed to **exit**?

**Answer:** PHP prints the string and terminates with exit status **0**.

**Explanation:** Use a non-zero integer status for a command-line failure.

### 28. How can a PHP CLI script report an error with exit status **1**?

**Answer:**

~~~php
<?php
fwrite(STDERR, "An error occurred\n");
exit(1);
~~~

**Explanation:** The diagnostic goes to standard error, and the integer sets the process status.

### 29. What cleanup behavior should you remember about **exit**?

**Answer:** Registered shutdown functions and object destructors run, but pending **finally** blocks do not.

**Explanation:** Avoid calling **exit** deep inside code that depends on **finally** for required cleanup.

## True or false

### 30. **return** prints its value automatically.

**Answer: False.** **return** passes the value to the caller; use **echo** to display it.

**Explanation:** Returning data and producing output are separate actions.

### 31. A function ends immediately when PHP reaches **return**.

**Answer: True.** Execution leaves that function call at that point.

**Explanation:** Later statements in the same function call are skipped.

### 32. A function without a return type or an explicit return value produces **null**.

**Answer: True.** The call evaluates to **null** if execution reaches the end without returning something.

**Explanation:** It does not automatically return an empty string or zero.

### 33. A **void** function may return a string as its result.

**Answer: False.** A **void** function cannot return a value.

**Explanation:** It can use **return;** with no expression to stop early.

### 34. A top-level **return** in a required file always stops the main calling script.

**Answer: False.** It ends the included file and returns control to the caller.

**Explanation:** **exit** would terminate the whole script.

### 35. **exit('Failed')** sets a non-zero command-line exit status.

**Answer: False.** A string is printed, but the exit status is **0**.

**Explanation:** Use an integer such as **exit(1)** to signal failure.

### 36. **die** and **exit** have different termination behavior.

**Answer: False.** **die** is an alias of **exit**.

**Explanation:** They are interchangeable, though **exit** often communicates intent more clearly.

### 37. A **finally** block is guaranteed to run if **exit** is called inside its **try** block.

**Answer: False.** Pending **finally** blocks are not executed when **exit** terminates the script.

**Explanation:** Do not use **exit** where cleanup in **finally** is required.

## Predict the output

### 38. What does this print?

~~~php
<?php
function square(int $number): int
{
    return $number * $number;
}

echo square(4), "\n";
echo "After call\n";
~~~

**Answer:**

~~~text
16
After call
~~~

**Explanation:** The function returns **16**. The caller resumes after the call and prints the next line.

### 39. What does this print?

~~~php
<?php
function shippingCost(int $subtotal): int
{
    if ($subtotal >= 50) {
        return 0;
    }

    return 5;
}

echo shippingCost(75), "\n";
echo shippingCost(20), "\n";
~~~

**Answer:**

~~~text
0
5
~~~

**Explanation:** The first call returns from the **if** branch; the second reaches the final return.

### 40. What does this print?

~~~php
<?php
function showNotice(): void
{
    echo "Notice\n";
}

$result = showNotice();
var_dump($result);
~~~

**Answer:**

~~~text
Notice
NULL
~~~

**Explanation:** The function prints **Notice** but returns no value, so the assigned result is **null**.

### 41. If **data.php** returns the string **ready**, what does the following main file print?

~~~php
<?php
$status = require __DIR__ . '/data.php';
echo $status, "\n";
echo "Main file continues\n";
~~~

**Answer:**

~~~text
ready
Main file continues
~~~

**Explanation:** The included file's top-level **return** provides a value to the **require** expression, then the main file continues.

### 42. What is printed, and what is the process exit status?

~~~php
<?php
echo "Starting\n";
exit(7);
echo "This is not printed\n";
~~~

**Answer:** It prints:

~~~text
Starting
~~~

The process exit status is **7**.

**Explanation:** The integer passed to **exit** is the status code. Script execution stops before the last echo.

## Write the code

### 43. Write a function named **add** that accepts two integers, returns their sum, and then print the result for **4** and **6**.

**Answer:**

~~~php
<?php
function add(int $left, int $right): int
{
    return $left + $right;
}

echo add(4, 6), "\n";
~~~

**Explanation:** The function returns the integer sum; **echo** displays the call's result.

### 44. Write a function named **makeUserSummary** that returns an associative array with the supplied **name** and **active** values.

**Answer:**

~~~php
<?php
function makeUserSummary(string $name, bool $active): array
{
    return [
        'name' => $name,
        'active' => $active,
    ];
}
~~~

**Explanation:** The function returns one array containing both related values.

### 45. Write CLI failure handling that prints **Invalid input** to standard error and exits with status **1**.

**Answer:**

~~~php
<?php
fwrite(STDERR, "Invalid input\n");
exit(1);
~~~

**Explanation:** The message is diagnostic output, and the integer makes the process report failure to its caller.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [035 - Including and Requiring PHP Files](../lessons/035-including-and-requiring-php-files.md) | [Read this lesson](../lessons/036-returning-values-and-ending-execution.md) | [Open example](../examples/036-returning-values-and-ending-execution.md) | [037 - Defining and Calling Functions](../lessons/037-defining-and-calling-functions.md) |
