# 036 - Returning Values and Ending Execution

A function can calculate a value and send it back to the code that called it. PHP's **return** statement also ends the current function. At file scope, **return** ends the current file; **exit** ends the entire script.

These statements have different jobs. Use **return** to give a result back to a caller or leave the current function or included file. Use **exit** only when the whole script must stop.

## What you will learn

- Return a calculated value from a function
- Store or use the returned value in an expression
- Use early **return** to finish a function when a condition is met
- Return an array when a result has multiple related values
- Understand the default **null** result and the **void** return type
- Use **return** in an included file
- Compare **return**, **exit**, and **die**
- Choose an exit status for a command-line program

## Return a value from a function

A function can return a value with **return**. The caller can assign that value to a variable, pass it to another function, or use it in an expression.

~~~php
<?php
function calculateTotal(int $price, int $quantity): int
{
    return $price * $quantity;
}

$total = calculateTotal(12, 3);

echo "Total: {$total}\n";
~~~

Output:

~~~text
Total: 36
~~~

The function calculates a result and hands it back. **return** does not print the value. The **echo** statement prints it after the call finishes.

~~~php
<?php
function double(int $number): int
{
    return $number * 2;
}

echo double(5), "\n";
~~~

Output:

~~~text
10
~~~

A return value can be used anywhere an expression is accepted. If the function has a declared return type, the returned value must satisfy that type.

## return ends the current function

When PHP executes **return** inside a function, the function stops immediately. Any statements after that return in the same execution path are skipped.

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

Output:

~~~text
0
5
~~~

The first call returns from the function inside the **if**. The second call continues to the final return. This style is called an early return: handle a case, return its result, and keep the remaining path simple.

A function can have multiple return statements, but only the first one reached during a particular call is used.

## Return more than one related value

A function returns one value. If that result needs to contain several related pieces, return an array:

~~~php
<?php
function makeSummary(string $name, int $score): array
{
    return [
        'name' => $name,
        'score' => $score,
        'passed' => $score >= 50,
    ];
}

$summary = makeSummary('Asha', 82);

echo "{$summary['name']}: {$summary['score']}\n";
echo $summary['passed'] ? "Passed\n" : "Try again\n";
~~~

Output:

~~~text
Asha: 82
Passed
~~~

The function returns one array value containing the name, score, and result. Arrays and objects are both common ways to group related output.

## Functions without an explicit return

If a function finishes without a **return** statement, its call produces **null**:

~~~php
<?php
function showNotice(): void
{
    echo "Notice shown\n";
}

$result = showNotice();
var_dump($result);
~~~

Output:

~~~text
Notice shown
NULL
~~~

The function printed text, but it did not return that text. Output and return values are separate.

A **void** return type makes this intent explicit: the function performs an action and does not return a value. A **void** function can use **return;** without an expression to stop early, but it cannot return a value.

~~~php
<?php
function printStatus(bool $ready): void
{
    if (!$ready) {
        echo "Not ready\n";
        return;
    }

    echo "Ready\n";
}

printStatus(false);
~~~

Output:

~~~text
Not ready
~~~

If a function declares a non-void return type, make sure each normal path returns a value of that type. PHP raises a **TypeError** if a typed function finishes without returning the required value.

## Return from an included file

At the top level of a file, **return** ends that file. If the file was included or required, execution continues in the calling file after the include statement.

The included file can also return a value to that statement:

**data.php:**

~~~php
<?php
return [
    'siteName' => 'Learning PHP',
    'year' => 2026,
];
~~~

**index.php:**

~~~php
<?php
$site = require __DIR__ . '/data.php';

echo $site['siteName'], "\n";
echo $site['year'], "\n";
echo "The main file continues\n";
~~~

Output:

~~~text
Learning PHP
2026
The main file continues
~~~

The **return** inside **data.php** ends only that included file and provides the array as the value of the **require** expression. The main file then continues.

This is different from **exit**: exiting from an included file terminates the whole script, including the file that included it.

## Stop the entire script with exit

**exit** terminates the current PHP script. It does not merely end a function or an included file.

~~~php
<?php
echo "Before exit\n";

exit;

echo "This line is never reached\n";
~~~

Output:

~~~text
Before exit
~~~

PHP also accepts **die**, which is an alias of **exit**:

~~~php
<?php
die;
~~~

They have the same behavior. Prefer **exit** in new code because its purpose is clear to more readers.

### Exit with a message or status code

When **exit** receives a string, PHP prints that string and exits with status code **0**. When it receives an integer, the integer is the command-line exit status and is not printed automatically.

~~~php
<?php
exit('Could not continue');
~~~

This prints **Could not continue**, then exits with status **0**. That may be misleading for scripts and shell commands because the message sounds like an error while the status says success.

For a command-line script, write an error to standard error and use a non-zero status:

~~~php
<?php
fwrite(STDERR, "Could not continue\n");
exit(1);
~~~

Exit status **0** conventionally means success. A non-zero status signals failure to a calling shell or automation. Use a meaningful status code within PHP's supported range.

Do not use **exit** as a replacement for ordinary function returns or structured error handling. It prevents the rest of the application from responding or cleaning up normally.

### Cleanup and finally blocks

PHP runs registered shutdown functions and object destructors when the script exits. However, **exit** does not run pending **finally** blocks. Avoid calling **exit** deep inside code that relies on a **finally** block to release resources or perform required cleanup. Return to a higher-level controller or throw an exception when the application needs to handle the problem.

## Compare return and exit

| Statement | What it ends | What happens next |
| --- | --- | --- |
| **return value** inside a function | The current function call | The caller receives the value and continues |
| **return;** inside a function | The current function call | The caller continues; no value is returned |
| **return** at file scope in an included file | The included file | Control returns to the including file |
| **return** in the main file | The current script file | The script ends |
| **exit** or **die** | The entire script | No caller continues; shutdown processing runs |

Use the statement that matches the scope you intend to end.

## Common mistakes

- Expecting **return** to print a value. Use **echo** when output is needed.
- Expecting **echo** to provide a value to the caller. Use **return** to send data back from a function.
- Forgetting that a function stops as soon as a return statement runs.
- Forgetting a return value on a path through a function with a non-void return type.
- Returning a value from a function declared **void**.
- Using **exit** inside a helper function when only that function should end.
- Using a string argument to **exit** for an error and assuming it creates a non-zero exit status.
- Calling **exit** where a **finally** block must run.
- Treating **die** as different from **exit**. It is an alias.

## Try it yourself

1. Write a function that multiplies two integers and returns the product.
2. Assign the function result to a variable, then print it.
3. Add an early return for a free-shipping threshold.
4. Return an associative array containing a name and a status.
5. Call a **void** function that prints a message and inspect its result.
6. Create an included file that returns an array, then read that value in the main file.
7. Add a **return** after including the file and observe that the caller continues.
8. Place **exit** in the included file and compare how it changes the whole script.
9. Write a CLI error to **STDERR** and end with status **1**.

## Key takeaways

- A function's **return** value is data for its caller; **echo** sends output.
- **return** ends the current function or file, depending on where it runs.
- A function without a return value produces **null** unless it declares **void**.
- An included file can **return** a value to the include or require expression.
- **exit** and **die** stop the entire script; a string message exits with status **0**.
- Use **exit** at the application's outer boundary, and use returns or exceptions inside reusable code.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [035 - Including and Requiring PHP Files](035-including-and-requiring-php-files.md) | [Open Q&A](../qnas/036-returning-values-and-ending-execution.md) | [Open example](../examples/036-returning-values-and-ending-execution.md) | [037 - Defining and Calling Functions](037-defining-and-calling-functions.md) |
