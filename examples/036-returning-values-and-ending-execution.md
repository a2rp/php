# 036 - Returning Values and Ending Execution: Complete Example

This command-line project demonstrates function return values, early returns, a **void** function, a value returned from a required file, and a separate script that stops with a non-zero exit status.

## Question

How can PHP functions return useful data to their callers, how can an included file return a value, and how can a command-line program report failure while stopping execution?

## Goal

Create this project:

~~~text
return-and-exit-example/
├── index.php
├── exit-demo.php
└── includes/
    └── settings.php
~~~

The main program will:

- Calculate and return a value from a function
- Use an early return for a condition
- Return related values in an array
- Print a message from a **void** function
- Load a returned array from a required file
- Continue the main file after that included file returns

The separate **exit-demo.php** script will write an error message to standard error and terminate with a non-zero status.

## Complete code

### index.php

~~~php
<?php
declare(strict_types=1);

function calculateTotal(int $price, int $quantity): int
{
    return $price * $quantity;
}

function shippingCost(int $subtotal): int
{
    if ($subtotal >= 50) {
        return 0;
    }

    return 5;
}

function makeSummary(string $name, int $score): array
{
    return [
        'name' => $name,
        'score' => $score,
        'passed' => $score >= 50,
    ];
}

function printSavedMessage(): void
{
    echo "Saved\n";
}

echo "=== 1. Return a calculated value ===\n";

$total = calculateTotal(12, 3);
echo "Product: {$total}\n";

echo "\n=== 2. Return early for a condition ===\n";

echo "Large order shipping: ", shippingCost(75), "\n";
echo "Small order shipping: ", shippingCost(20), "\n";

echo "\n=== 3. Return related values in an array ===\n";

$summary = makeSummary('Asha', 82);
$result = $summary['passed'] ? 'passed' : 'not passed';

echo "{$summary['name']} scored {$summary['score']}: {$result}\n";

echo "\n=== 4. Perform an action with a void function ===\n";

printSavedMessage();

echo "\n=== 5. Return a value from an included file ===\n";

$settings = require __DIR__ . '/includes/settings.php';

echo "Site: {$settings['siteName']}\n";
echo "Release year: {$settings['year']}\n";
echo "The main script continues\n";
~~~

### includes/settings.php

~~~php
<?php
return [
    'siteName' => 'Learn PHP',
    'year' => 2026,
];
~~~

The top-level **return** ends this included file and gives the array back to the **require** expression in **index.php**.

### exit-demo.php

~~~php
<?php
fwrite(STDERR, "Invalid input\n");
exit(1);

echo "This line is never reached\n";
~~~

This is a separate script because **exit** would stop the entire script, including the main file that called it.

## Expected output from index.php

~~~text
=== 1. Return a calculated value ===
Product: 36

=== 2. Return early for a condition ===
Large order shipping: 0
Small order shipping: 5

=== 3. Return related values in an array ===
Asha scored 82: passed

=== 4. Perform an action with a void function ===
Saved

=== 5. Return a value from an included file ===
Site: Learn PHP
Release year: 2026
The main script continues
~~~

## Explanation

### Return a calculated value

**calculateTotal()** returns the multiplication result. The call assigns that value to **$total**, and the caller prints it. The function itself does not print the result.

### Return early

For a subtotal of **75**, **shippingCost()** reaches the first return and gives back **0**. For a subtotal of **20**, the condition is false, so the function reaches the final return and gives back **5**.

Only the first return reached during a call matters. Once PHP returns from the function, it does not execute later statements in that function call.

### Return related data as one value

**makeSummary()** returns one associative array containing the name, score, and pass result. The caller reads each field from that array.

### Print without returning a value

**printSavedMessage()** performs an action by printing **Saved**. Its **void** return type says the function does not provide a result to the caller.

### Return from an included file

The settings file returns an array. **require** evaluates that file, and the returned array becomes the value assigned to **$settings**. After the include finishes, the main script continues to the next echo statement.

A top-level **return** in an included file only ends that file. **exit** would stop the complete script.

### Report a CLI error and stop

Run the separate script:

~~~sh
php exit-demo.php
~~~

It writes **Invalid input** to standard error and terminates with status **1**. In PowerShell, inspect the result with **$LASTEXITCODE** after the command. In a Unix-like shell, inspect it with **echo $?**.

The echo after **exit(1)** is not reached. In a web application, avoid using **exit** throughout helper code; return control to a central handler where the application can decide how to respond.

## Run the main example

From the project directory, run:

~~~sh
php index.php
~~~

Its output should match the expected-output block above.

## Make a change

1. Change the price and quantity passed to **calculateTotal()**.
2. Change the free-shipping threshold in **shippingCost()**.
3. Return another field such as **'level'** from **makeSummary()**.
4. Change the score and observe the **passed** result.
5. Add another line to **settings.php** and print its value in **index.php**.
6. Change the status in **exit-demo.php** from **1** to **2**, then inspect the shell status.

## Common mistakes

- Expecting a function's **return** value to print automatically.
- Using **echo** when a caller needs a value for later calculations.
- Forgetting to return a value on one path through a typed, non-void function.
- Returning a value from a function declared **void**.
- Assuming a top-level **return** in an included file terminates its caller.
- Assuming **exit('Invalid input')** signals failure to a shell; a string exits with status **0**.
- Calling **exit** deep in reusable code and preventing the application from handling the problem.
- Expecting a pending **finally** block to execute after **exit**.

## Key takeaways

- **return** passes data to a caller and ends the current function or file.
- **echo** produces output and is separate from returning a value.
- Arrays and objects can group related pieces into one returned value.
- A top-level return from a required file can provide the value of the **require** expression.
- **exit** and **die** stop the entire script; use an integer status to report success or failure to a CLI caller.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [035 - Including and Requiring PHP Files](../lessons/035-including-and-requiring-php-files.md) | [Read this lesson](../lessons/036-returning-values-and-ending-execution.md) | [Open Q&A](../qnas/036-returning-values-and-ending-execution.md) | [037 - Defining and Calling Functions](../lessons/037-defining-and-calling-functions.md) |
