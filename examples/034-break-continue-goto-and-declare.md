# 034 - break, continue, goto, and declare: Complete Example

This command-line example demonstrates how **break**, **continue**, **goto**, and **declare(strict_types=1)** change program flow.

## Question

How can a PHP program stop a search early, skip selected values, continue a loop from a switch case, use a label jump, and enable strict scalar type checks?

## Goal

Create this project:

~~~text
control-flow-example/
└── index.php
~~~

The program will:

- Stop a **foreach** loop when it reaches a sentinel value
- Skip non-positive values with **continue**
- Show that a **for** loop still updates its counter after **continue**
- Exit nested loops with **break 2**
- Use **continue 2** inside a switch nested in a loop
- Demonstrate a **goto** label
- Enable strict scalar typing at the top of the file

## Complete code

Create **index.php** with this code:

~~~php
<?php
declare(strict_types=1);

echo "=== 1. Stop on a sentinel ===\n";

$readings = [4, 7, -1, 9];

foreach ($readings as $reading) {
    if ($reading === -1) {
        echo "stop value found\n";
        break;
    }

    echo "value: {$reading}\n";
}

echo "\n=== 2. Skip non-positive values ===\n";

$values = [3, 0, -2, 9];

foreach ($values as $value) {
    if ($value <= 0) {
        continue;
    }

    echo "positive: {$value}\n";
}

echo "\n=== 3. Continue in a for loop ===\n";

for ($number = 0; $number < 4; $number++) {
    if ($number === 1) {
        continue;
    }

    echo "for value: {$number}\n";
}

echo "\n=== 4. Exit nested loops ===\n";

$grid = [
    ['red', 'green'],
    ['blue', 'gold'],
];

foreach ($grid as $rowIndex => $row) {
    foreach ($row as $columnIndex => $color) {
        if ($color === 'blue') {
            echo "match at {$rowIndex}, {$columnIndex}\n";
            break 2;
        }
    }
}

echo "search complete\n";

echo "\n=== 5. Continue from a switch ===\n";

$commands = ['add', 'skip', 'remove'];

foreach ($commands as $command) {
    switch ($command) {
        case 'skip':
            echo "skip outer iteration\n";
            continue 2;

        case 'remove':
            echo "remove\n";
            break;

        default:
            echo "add\n";
    }

    echo "finished: {$command}\n";
}

echo "\n=== 6. Use a goto label ===\n";

$attempts = 0;

retry_demo:
$attempts++;
echo "attempt {$attempts}\n";

if ($attempts < 2) {
    goto retry_demo;
}

echo "goto demo complete\n";

echo "\n=== 7. Strict scalar typing ===\n";

function describeQuantity(int $quantity): string
{
    return "Quantity: {$quantity}";
}

echo describeQuantity(3), "\n";

// Uncomment to observe a TypeError in strict mode:
// echo describeQuantity("3"), "\n";
~~~

## Expected output

~~~text
=== 1. Stop on a sentinel ===
value: 4
value: 7
stop value found

=== 2. Skip non-positive values ===
positive: 3
positive: 9

=== 3. Continue in a for loop ===
for value: 0
for value: 2
for value: 3

=== 4. Exit nested loops ===
match at 1, 0
search complete

=== 5. Continue from a switch ===
add
finished: add
skip outer iteration
remove
finished: remove

=== 6. Use a goto label ===
attempt 1
attempt 2
goto demo complete

=== 7. Strict scalar typing ===
Quantity: 3
~~~

## Explanation

### 1. Stop at a sentinel

The **foreach** processes **4** and **7**. When it reaches **-1**, **break** exits the loop, so **9** is never printed. The code after the loop still runs.

### 2. Skip selected values

The **continue** statement skips the echo for **0** and **-2**. It does not stop the loop, so the later positive value **9** is still processed.

### 3. Keep the for-loop update

When **$number** is **1**, **continue** skips the echo. The **for** loop still increments the counter, so it proceeds to **2** instead of repeating **1** forever.

### 4. Exit both loops

The match is in row **1**, column **0**. **break 2** exits the inner and outer **foreach** loops. The following **search complete** line then runs.

### 5. Continue the outer loop from switch

The **add** and **remove** cases finish the **switch**, then print their **finished** messages. The **skip** case uses **continue 2**, which leaves both the **switch** and the current **foreach** iteration. Therefore, **finished: skip** is not printed.

The **break** in the **remove** case exits only the **switch**. It does not exit the surrounding **foreach**.

### 6. Jump to a label

The program increments **$attempts**, then jumps back to **retry_demo** while the value is less than **2**. Once the condition is false, execution continues after the **if** statement.

This is a syntax demonstration. A loop is clearer for normal repetition, so prefer a loop in application code.

### 7. Use strict scalar typing

**declare(strict_types=1)** is the first PHP statement in the file. The function declares an integer parameter and a string return type. Passing **3** works. Uncommenting the string call passes a mismatched scalar value and raises **TypeError** in strict mode.

Strict typing applies to scalar type declarations; it does not automatically type-check every variable. It is a per-file setting.

## Run the example

From the project folder, run:

~~~sh
php index.php
~~~

The output should match the block above. The invalid call remains commented out so the program can complete normally.

## Make a change

1. Replace the sentinel **-1** with another value and update the condition.
2. Add another positive and negative number to **$values**.
3. Change the skipped **for** value from **1** to **2** and predict the output first.
4. Search the grid for **gold** instead of **blue**.
5. Add another command to **$commands** and handle it in the **switch**.
6. Change the **goto** demo to count to **3**, then rewrite it using a loop.
7. Uncomment the mismatched function call and observe the strict type error.

## Common mistakes

- Using **break** where only one iteration should be skipped. Use **continue**.
- Using **continue** when the entire loop should stop. Use **break**.
- Forgetting that a plain **break** inside a switch exits only the switch.
- Using plain **continue** inside a switch when the intended target is the loop around it. Use **continue 2**.
- Forgetting that the **for** update expression still runs after **continue**.
- Placing **declare(strict_types=1)** after executable PHP statements.
- Uncommenting the invalid function call and expecting the rest of the script to continue after the **TypeError**.
- Using **goto** when a normal loop would communicate the repetition more clearly.

## Key takeaways

- **break** stops the nearest loop or switch; a numeric level can exit nested structures.
- **continue** skips the current iteration, and **continue 2** can move past a switch to its surrounding loop.
- **goto** jumps to a label within the same file and function or method context.
- **declare(strict_types=1)** belongs at the top of a file and enables strict scalar type checking for relevant declarations.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [033 - foreach Loops and Key-Value Iteration](../lessons/033-foreach-loops-and-key-value-iteration.md) | [Read this lesson](../lessons/034-break-continue-goto-and-declare.md) | [Open Q&A](../qnas/034-break-continue-goto-and-declare.md) | [035 - Including and Requiring PHP Files](../lessons/035-including-and-requiring-php-files.md) |
