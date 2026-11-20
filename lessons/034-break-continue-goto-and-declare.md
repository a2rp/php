# 034 - break, continue, goto, and declare

Loops normally finish when their condition becomes false. PHP also provides statements that change normal control flow: **break** exits a loop or **switch**, **continue** skips to another loop iteration, **goto** jumps to a label, and **declare** sets a language directive.

Use **break** and **continue** when they make a loop clearer. **goto** exists but is rarely the clearest choice. In modern PHP, **strict_types** is the **declare** directive beginners are most likely to use.

## What you will learn

- Stop a loop early with **break**
- Skip an iteration with **continue**
- Use numeric levels in nested loops and switches
- Handle **continue** carefully inside a **switch**
- Understand the syntax and restrictions of **goto**
- Use **declare(strict_types=1)** for stricter scalar type checks

## Stop a loop with break

**break** immediately exits the nearest **for**, **foreach**, **while**, **do-while**, or **switch** structure. PHP continues with the statement after that structure.

~~~php
<?php
$temperatures = [18, 20, 21, -999, 19];

foreach ($temperatures as $temperature) {
    if ($temperature === -999) {
        echo "Reading unavailable\n";
        break;
    }

    echo "Temperature: {$temperature}\n";
}

echo "Finished reading data\n";
~~~

Output:

~~~text
Temperature: 18
Temperature: 20
Temperature: 21
Reading unavailable
Finished reading data
~~~

The value **-999** is a sentinel that means the reading is unavailable. When PHP finds it, **break** ends the **foreach**. The final message is outside the loop, so it still runs.

Use **break** when continuing the current loop no longer makes sense, such as when you have found a result. It does not skip just one item; it exits the current structure.

### Stop nested loops

A plain **break** exits only the nearest enclosing loop or switch. A positive integer says how many enclosing loop or switch structures to exit:

~~~php
<?php
$found = false;
$rows = [
    ['red', 'green'],
    ['blue', 'gold'],
];

foreach ($rows as $rowIndex => $row) {
    foreach ($row as $columnIndex => $color) {
        if ($color === 'blue') {
            echo "Found blue at {$rowIndex}, {$columnIndex}\n";
            $found = true;
            break 2;
        }
    }
}

echo $found ? "Search complete\n" : "Color not found\n";
~~~

Output:

~~~text
Found blue at 1, 0
Search complete
~~~

**break 2** exits both **foreach** loops. A level counts enclosing breakable structures, not **if** statements. Use the smallest level that expresses the intended exit. If a deeply nested search needs a large level, a helper function with an early **return** may be easier to follow.

## Skip an iteration with continue

**continue** skips the remaining statements in the current loop iteration and moves to the next iteration. The loop itself keeps running.

~~~php
<?php
$values = [12, 0, 8, -3, 5];

foreach ($values as $value) {
    if ($value <= 0) {
        continue;
    }

    echo "Positive value: {$value}\n";
}
~~~

Output:

~~~text
Positive value: 12
Positive value: 8
Positive value: 5
~~~

For a **for** loop, the update expression still runs after **continue**, and then PHP checks the condition. In a **while** or **do-while** loop, PHP moves to the condition check. In **foreach**, PHP moves to the next element.

Put the condition before the statements that should be skipped. Any work performed before **continue** has already happened.

## Continue an outer loop

**continue** can also take a positive numeric level. It skips to the next iteration of the selected enclosing loop.

~~~php
<?php
$groups = [
    ['Asha', 'Dev'],
    [],
    ['Mina'],
];

foreach ($groups as $groupNumber => $group) {
    if ($group === []) {
        echo "Skipping empty group {$groupNumber}\n";
        continue;
    }

    foreach ($group as $name) {
        echo "{$name}\n";
    }
}
~~~

Output:

~~~text
Asha
Dev
Skipping empty group 1
Mina
~~~

This **continue** is outside the inner loop, so it continues the outer **foreach**. From inside a nested loop, **continue 2** can skip directly to the next outer iteration. Use numeric levels sparingly and make the intended target clear.

## Continue inside a switch

PHP treats **switch** as a looping structure for **continue**. Inside a **switch**, plain **continue** behaves like **break** for that switch and raises a warning because this is often a mistake. If the switch is inside a loop and the goal is to start the next outer iteration, use **continue 2**.

~~~php
<?php
$commands = ['add', 'skip', 'remove'];

foreach ($commands as $command) {
    switch ($command) {
        case 'skip':
            echo "Skipping this loop iteration\n";
            continue 2;

        case 'remove':
            echo "Removing item\n";
            break;

        default:
            echo "Adding item\n";
    }

    echo "Finished command: {$command}\n";
}
~~~

Output:

~~~text
Adding item
Finished command: add
Skipping this loop iteration
Removing item
Finished command: remove
~~~

**continue 2** leaves the **switch** and continues the surrounding **foreach**, so the finished-command line is skipped for **skip**. The **break** in the **remove** case exits only the **switch**; execution then reaches the line after it.

## Jump to a label with goto

A label is a case-sensitive name followed by a colon. **goto** transfers execution to a label in the same file and the same function or method context.

~~~php
<?php
$attempts = 0;

retry:
$attempts++;
echo "Attempt {$attempts}\n";

if ($attempts < 3) {
    goto retry;
}

echo "Done\n";
~~~

Output:

~~~text
Attempt 1
Attempt 2
Attempt 3
Done
~~~

PHP restricts **goto**. A jump cannot enter a loop or **switch**, enter or leave a function or method, or jump into another file. It can jump out of a loop or switch.

This small example demonstrates the syntax. In ordinary application code, a loop or a function is usually easier to follow than a backward jump. Prefer **break**, **continue**, **return**, and structured conditions when they express the same behavior.

## Set directives with declare

**declare** sets a language directive for code. Directive values must be literals because PHP processes them while compiling the file.

~~~php
<?php
declare(ticks=1);

// Code in this scope uses the directive.
~~~

PHP recognizes **ticks**, **encoding**, and **strict_types** directives. **ticks** is an advanced feature that can trigger registered tick functions after groups of statements. **encoding** is a specialized script-encoding directive. Most day-to-day application code does not need either one; source files are ordinarily saved as UTF-8.

The directive most application developers should recognize is **strict_types**.

## Enable strict scalar typing

PHP's scalar type declarations can accept compatible values by coercion in the default mode. To request stricter checks for scalar type declarations in a file, put this directive at the top of the file:

~~~php
<?php
declare(strict_types=1);

function formatQuantity(int $quantity): string
{
    return "Quantity: {$quantity}";
}

echo formatQuantity(4), "\n";
~~~

Output:

~~~text
Quantity: 4
~~~

With strict scalar typing enabled, a mismatched scalar argument for a typed function call results in a **TypeError** rather than being silently converted. One exception is that an **int** value is accepted for a **float** declaration.

~~~php
<?php
declare(strict_types=1);

function double(int $number): int
{
    return $number * 2;
}

echo double(4), "\n";
// double("4"); // TypeError in strict mode
~~~

Output:

~~~text
8
~~~

The directive must be the first PHP statement in its file. Put it before other executable PHP statements. It is a per-file setting, not a global switch for the whole project. For scalar function arguments, the calling file's mode matters: a call made from a file without strict typing uses that file's coercive mode, even if the function was declared in a strict file. Strict typing applies to scalar type declarations; it does not turn every variable assignment into a type check.

Only literal values are valid in a **declare** directive. This is valid:

~~~php
<?php
declare(strict_types=1);
~~~

A variable cannot supply a directive value:

~~~php
<?php
$strictMode = 1;
// declare(strict_types=$strictMode); // Invalid: directive values must be literals.
~~~

## Choose the right flow-control tool

| Statement | Effect | Common use |
| --- | --- | --- |
| **break** | Exit the nearest loop or switch | Stop after finding a result |
| **break 2** | Exit two enclosing loop or switch structures | Stop a nested search |
| **continue** | Skip to the next iteration of the nearest loop | Ignore an item and process the rest |
| **continue 2** | Continue an outer loop | Skip an outer item from inside a switch or nested loop |
| **goto** | Jump to a label in the same file and context | Rare cases where a structured alternative is unsuitable |
| **declare** | Set a language directive | Enable strict scalar typing for a file |

## Common mistakes

- Using **break** when only the current iteration should be skipped. Use **continue**.
- Using **continue** when the whole loop should stop. Use **break**.
- Counting an **if** statement as a level for **break 2** or **continue 2**. Only enclosing loops and switches count.
- Writing plain **continue** in a **switch** when the intention is to continue the surrounding loop. Use **continue 2**.
- Assuming **break** in a **switch** also exits its surrounding loop. It exits only the switch unless a higher level is used.
- Adding **goto** jumps when a loop, conditional, function, or early **return** is clearer.
- Placing **declare(strict_types=1)** after executable statements.
- Assuming **strict_types** checks all values in a program. It changes scalar type-declaration behavior, not untyped assignments.

## Try it yourself

1. Stop a **foreach** loop when it reaches a chosen value.
2. Search a nested array and use **break 2** after finding a match.
3. Skip non-positive numbers with **continue**.
4. Put a **switch** inside a **foreach** and use **continue 2** for one case.
5. Explain why **break** in another switch case does not exit the **foreach**.
6. Create a label and use **goto** to repeat a short demonstration, then rewrite it as a loop.
7. Add **declare(strict_types=1)** to a file and compare a valid call with a mismatched scalar argument.
8. Move the **declare** statement below a PHP statement and observe the placement error.

## Key takeaways

- **break** exits the nearest loop or switch; numeric levels can exit multiple enclosing structures.
- **continue** skips the current iteration; a **for** loop still runs its update expression.
- A **switch** counts as a level for **continue**, so use **continue 2** to continue an outer loop from inside a switch.
- **goto** is restricted to a label in the same file and context and cannot jump into a loop or switch.
- **declare** configures language directives; **strict_types** is the directive most application developers should recognize.
- **declare(strict_types=1)** belongs at the top of a file and changes scalar type-declaration behavior, not every assignment.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [033 - foreach Loops and Key-Value Iteration](033-foreach-loops-and-key-value-iteration.md) | [Open Q&A](../qnas/034-break-continue-goto-and-declare.md) | [Open example](../examples/034-break-continue-goto-and-declare.md) | [035 - Including and Requiring PHP Files](035-including-and-requiring-php-files.md) |
