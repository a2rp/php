# 032 - Loops with while, do-while, and for: Complete Example

This command-line program uses `while`, `do-while`, and `for` loops to count, skip an iteration, stop early, and exit nested loops. Each section prints predictable output so you can trace when PHP checks a condition and updates a counter.

## Question

How can PHP repeat a task with different loop forms while ensuring each loop progresses and stops at the intended point?

## Goal

Create this small command-line project:

~~~text
loop-control-example/
â””â”€â”€ index.php
~~~

The program will:

- Count with a `while` loop
- Show that `do-while` runs once even when its final condition is false
- Count through a numeric range with `for`
- Skip a value with `continue` and stop with `break`
- Exit two nested loops with `break 2`
- Give an unbounded `for (;;)` loop a clear exit condition

## Complete code

Create `index.php` with this code:

~~~php
<?php
echo "=== 1. while counter ===\n";

$count = 1;

while ($count <= 3) {
    echo $count, "\n";
    $count++;
}

echo "\n=== 2. do-while runs once ===\n";

$attempt = 1;

do {
    echo "The body ran once.\n";
    $attempt++;
} while ($attempt < 1);

echo "\n=== 3. for counter ===\n";

for ($index = 0; $index < 5; $index++) {
    echo $index, "\n";
}
echo "\n";

echo "\n=== 4. continue and break ===\n";

for ($number = 1; $number <= 8; $number++) {
    if ($number === 3) {
        continue;
    }

    if ($number === 7) {
        break;
    }

    echo $number, "\n";
}
echo "\n";

echo "\n=== 5. break out of nested loops ===\n";

for ($row = 1; $row <= 3; $row++) {
    for ($column = 1; $column <= 3; $column++) {
        if ($row === 2 && $column === 2) {
            break 2;
        }

        echo "{$row},{$column}\n";
    }
}

echo "\n=== 6. for loop with an explicit exit ===\n";

$attempts = 0;

for (;;) {
    $attempts++;

    if ($attempts === 3) {
        break;
    }
}

echo "Attempts: {$attempts}\n";
~~~

## Expected output

~~~text
=== 1. while counter ===
1
2
3

=== 2. do-while runs once ===
The body ran once.

=== 3. for counter ===
0
1
2
3
4

=== 4. continue and break ===
1
2
4
5
6

=== 5. break out of nested loops ===
1,1
1,2
1,3
2,1

=== 6. for loop with an explicit exit ===
Attempts: 3
~~~

## How it works

### 1. The `while` counter

The condition `$count <= 3` is checked before every iteration. The body prints the current value, then increments it. When `$count` becomes `4`, the condition is false and the loop ends.

### 2. The `do-while` counter

The body prints before PHP checks `$attempt < 1`. The counter then becomes `2`, the condition is false, and the loop stops after that one iteration.

### 3. The `for` counter

Initialization sets `$index` to zero once. PHP checks `$index < 5`, runs the body, increments the index, and checks again. The body sees values zero through four; it does not see five.

### 4. `continue` and `break`

When `$number` is `3`, `continue` skips the echo. In a `for` loop, PHP still runs `$number++` before checking the condition again. When `$number` reaches `7`, `break` ends the loop, so seven and eight are not printed.

### 5. Nested loops

The nested counters print coordinate pairs. At row 2, column 2, `break 2` exits both loops. The triggering pair is checked before the echo, so `2,2` does not appear.

### 6. An unbounded `for` loop

The empty condition in `for (;;)` does not stop the loop. The body increments `$attempts` and reaches `break` when it becomes `3`. Without that exit, the loop would continue indefinitely.

## Run it

~~~sh
php --version
php index.php
~~~

The code is displayed here as a learning example. Run it locally if you want to compare your output with the expected output above.

## Make a change

1. Change the `while` counter to print from `1` through `5`.
2. Set the `do-while` condition to true once, then update the counter so it ends after a few iterations.
3. Change the `for` condition from `$index < 5` to `$index <= 5` and count the extra value.
4. Skip both `3` and `5` with `continue`.
5. Move the `break` threshold from `7` to `5` and predict the output.
6. Change `break 2` to `break` and observe which loop continues.
7. Add a condition to the `for (;;)` body that exits at five attempts.
8. Remove a counter update temporarily and identify which loop could become infinite.

## Common mistakes

- Forgetting to increment a variable used by a `while` condition
- Expecting a `do-while` condition to prevent the first body execution
- Forgetting the final semicolon in `do-while`
- Using `<=` when the endpoint should be excluded
- Assuming `continue` skips the `for` update expression
- Placing a `while` counter update after a `continue` path
- Assuming plain `break` exits every nested loop
- Writing `for (;;)` without an intentional `break` or `return`

## Key takeaways

- `while` tests before the body; `do-while` tests after it and runs at least once.
- `for` keeps initialization, condition, and update together.
- `continue` skips the current body remainder; `break` exits the nearest loop.
- Use `break 2` only when the intent is to exit two nested structures.
- Every loop needs a progress update or a clear exit path.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [031 - The match expression](../lessons/031-the-match-expression.md) | [Read this lesson](../lessons/032-loops-with-while-do-while-and-for.md) | [Open Q&A](../qnas/032-loops-with-while-do-while-and-for.md) | [033 - foreach loops and key-value iteration](../lessons/033-foreach-loops-and-key-value-iteration.md) |
