# 032 - Loops with while, do-while, and for

Loops repeat a block of code while a condition allows it. PHP's `while` and `do-while` loops are useful when repetition depends on a condition; `for` is convenient when a counter has a clear starting point, stopping condition, and update.

Every loop needs a clear way to stop. If its condition never becomes false and the body never exits, the loop can run forever.

## What you will learn

- How `while` checks its condition before each iteration
- How `do-while` always runs its body at least once
- How the three parts of a `for` loop work
- How counters and boundary conditions affect the number of iterations
- How `break` exits a loop and `continue` skips to the next iteration
- How loop control behaves in nested loops
- How to prevent infinite loops and off-by-one errors

## The `while` loop

A `while` loop checks its condition before running the body. If the condition is false at the start, its body runs zero times.

~~~php
<?php
$count = 1;

while ($count <= 3) {
    echo $count, "\n";
    $count++;
}
~~~

Output:

~~~text
1
2
3
~~~

The variable starts at `1`. After each printed value, `$count++` moves it toward the point where `$count <= 3` becomes false.

### Change the loop state

The loop body must update something that affects the condition, unless another action will end the loop. Forgetting the update is a common cause of an infinite loop.

~~~php
<?php
$count = 1;

while ($count <= 3) {
    echo $count, "\n";
    // Without an update, $count remains 1 forever.
    $count++;
}
~~~

A `while` loop is useful when the number of repetitions is not known in advance, such as reading until a stop condition is reached. The condition still needs to change or the loop needs a deliberate exit.

## The `do-while` loop

A `do-while` loop checks its condition after running the body. This guarantees at least one iteration.

~~~php
<?php
$count = 1;

do {
    echo $count, "\n";
    $count++;
} while ($count <= 3);
~~~

Output:

~~~text
1
2
3
~~~

The semicolon after the closing `while (...)` condition is required.

Even a false condition does not prevent the first iteration:

~~~php
<?php
$count = 5;

do {
    echo "The body ran once.\n";
    $count++;
} while ($count < 5);
~~~

Output:

~~~text
The body ran once.
~~~

Use `do-while` when one attempt must happen before PHP decides whether to repeat. Use `while` when the body may need to run zero times.

## The `for` loop

A `for` loop places initialization, condition, and update in one line:

~~~php
for (initialization; condition; update) {
    // Repeated statements.
}
~~~

The parts run in this order:

1. Initialization runs once before the loop starts.
2. The condition is checked before each iteration.
3. If it is true, the body runs.
4. The update runs after the body.
5. PHP checks the condition again.

Example:

~~~php
<?php
for ($count = 1; $count <= 3; $count++) {
    echo $count, "\n";
}
~~~

Output:

~~~text
1
2
3
~~~

The initialization and update can use other counter patterns:

~~~php
<?php
for ($number = 0; $number < 6; $number += 2) {
    echo $number, "\n";
}
~~~

Output:

~~~text
0
2
4
~~~

The condition is checked before the body, so `$number` stops at `4`; after its update to `6`, the condition becomes false.

## Choose the right loop

| Loop | When the condition is checked | Minimum body runs | Typical use |
| --- | --- | --- | --- |
| `while` | Before each iteration | Zero | Repeat while a condition remains true |
| `do-while` | After each iteration | One | Perform one attempt before deciding to repeat |
| `for` | Before each iteration | Zero | Count with an initialization, condition, and update |

Use `foreach` when iterating over an array or other iterable; that loop is covered in the next chapter.

## Watch the boundary condition

A small difference between `<` and `<=` changes the number of iterations. For a counter starting at zero, `$index < 3` runs for `0`, `1`, and `2`. `$index <= 3` also runs for `3`.

~~~php
<?php
for ($index = 0; $index < 3; $index++) {
    echo $index, "\n";
}
~~~

Output:

~~~text
0
1
2
~~~

Before writing a loop, decide whether the start and end values are included. Then choose the comparison operator to match that decision.

## Exit a loop early with `break`

`break` ends the nearest enclosing loop immediately.

~~~php
<?php
for ($number = 1; $number <= 6; $number++) {
    if ($number === 4) {
        break;
    }

    echo $number, "\n";
}
~~~

Output:

~~~text
1
2
3
~~~

The loop stops when `$number` reaches `4`; the `echo` for that value does not run.

## Skip an iteration with `continue`

`continue` skips the rest of the current iteration and proceeds to the next one.

~~~php
<?php
for ($number = 1; $number <= 5; $number++) {
    if ($number === 3) {
        continue;
    }

    echo $number, "\n";
}
~~~

Output:

~~~text
1
2
4
5
~~~

In a `for` loop, `continue` still proceeds through the update expression before the next condition check. In a `while` loop, it goes back to the condition check. If a `while` counter is updated only at the bottom of the body, make sure a `continue` branch updates it before continuing.

~~~php
<?php
$count = 0;

while ($count < 5) {
    $count++;

    if ($count === 3) {
        continue;
    }

    echo $count, "\n";
}
~~~

The counter increments before `continue`, so the loop still progresses and eventually stops.

## Nested loops and numbered control

By default, `break` and `continue` affect the nearest enclosing loop. A positive level can target more than one nested loop.

~~~php
<?php
for ($row = 1; $row <= 3; $row++) {
    for ($column = 1; $column <= 3; $column++) {
        if ($row === 2 && $column === 2) {
            break 2;
        }

        echo "{$row},{$column}\n";
    }
}
~~~

Output:

~~~text
1,1
1,2
1,3
2,1
~~~

`break 2` exits both the inner and outer loops. Use numbered `break` or `continue` sparingly; a small helper function or a clearer condition may make deeply nested control flow easier to follow.

## Avoid infinite loops

A loop is infinite when its condition stays true and no exit is reached. A `for` loop may intentionally omit its condition, but then its body must have a reliable `break` or `return` path.

~~~php
<?php
$attempt = 0;

for (;;) {
    $attempt++;

    if ($attempt === 3) {
        break;
    }
}
~~~

This loop ends because it reaches `break` on the third attempt. Do not use an unbounded loop unless the stop condition and exit path are clear.

Common causes of accidental infinite loops include:

- Forgetting to increment or otherwise change the counter
- Updating the wrong variable
- Using a condition that can never become false
- Calling `continue` before the update in a `while` loop
- Using `<=` when the intended final value should be excluded

## Try it yourself

1. Print the integers from `1` through `10` with a `while` loop.
2. Set a `while` condition to false initially and confirm that its body runs zero times.
3. Write a `do-while` whose condition is false initially and confirm that its body still runs once.
4. Use a `for` loop to print even numbers from `2` through `12`.
5. Compare the output from `$i < 4` and `$i <= 4` when `$i` begins at `1`.
6. Use `continue` to skip the number `5` in a counter loop.
7. Use `break` to stop a loop when its counter reaches `7`.
8. Put a `continue` before the bottom of a `while` body, then make sure the counter still changes.
9. Write two nested loops and use `break 2` to exit both.
10. Write an intentionally unbounded `for (;;)` with a clear exit condition.
11. Decide whether `while`, `do-while`, or `for` fits each small task best.

## Key takeaways

- `while` checks before its body and can run zero times; `do-while` checks after and runs at least once.
- `for` runs initialization once, checks its condition before each iteration, and runs its update after the body.
- Make the loop condition progress toward false to avoid accidental infinite loops.
- Choose `<` or `<=` based on whether the boundary value should be included.
- `break` exits the nearest loop; `continue` skips to the next iteration.
- In a `for` loop, `continue` still reaches the update expression; in a `while` loop, update the counter before continuing.
- Use `foreach` for arrays and other iterables; it is covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [031 - The match expression](031-the-match-expression.md) | [Open Q&A](../qnas/032-loops-with-while-do-while-and-for.md) | [Open example](../examples/032-loops-with-while-do-while-and-for.md) | [033 - foreach loops and key-value iteration](033-foreach-loops-and-key-value-iteration.md) |
