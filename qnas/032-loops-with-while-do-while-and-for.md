# 032 - Loops with while, do-while, and for: Questions and Answers

Review [032 - Loops with while, do-while, and for](../lessons/032-loops-with-while-do-while-and-for.md). Every answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. When does a `while` loop check its condition?

A. After every two iterations<br>
B. Before each iteration<br>
C. Only after the loop ends<br>
D. Only once before the script starts

**Answer: B.** PHP checks the condition before the body on every iteration.

### 2. If a `while` condition is false initially, how many times does its body run?

A. Zero<br>
B. One<br>
C. Two<br>
D. Until `break` is reached

**Answer: A.** The condition is checked first, so a false condition prevents the body from running.

### 3. Which loop always runs its body at least once?

A. `while`<br>
B. `for`<br>
C. `do-while`<br>
D. `foreach`

**Answer: C.** A `do-while` checks its condition after the body.

### 4. Which punctuation is required after a `do-while` condition?

A. A comma<br>
B. A colon<br>
C. A semicolon<br>
D. No punctuation

**Answer: C.** The syntax ends with `} while ($condition);`.

### 5. What are the three parts inside a `for` loop's parentheses?

A. Start, body, finish<br>
B. Initialization, condition, update<br>
C. Key, value, array<br>
D. Try, catch, finally

**Answer: B.** Initialization runs once, the condition is checked before each iteration, and update runs after the body.

### 6. How many times does the initialization part of a `for` loop normally run?

A. Once<br>
B. Before every body statement<br>
C. After each iteration<br>
D. Only when the condition is false

**Answer: A.** Initialization runs once before the first condition check.

### 7. What happens to a `for` loop's update expression when `continue` is reached in its body?

A. It is skipped forever.<br>
B. It runs before the next condition check.<br>
C. It runs before the body starts.<br>
D. It exits the loop.

**Answer: B.** In a `for` loop, continue moves to the update expression, then PHP checks the condition again.

### 8. What does `continue` do in a loop?

A. Exits every enclosing loop<br>
B. Skips the rest of the current iteration<br>
C. Repeats the current statement<br>
D. Resets the counter

**Answer: B.** It skips the remaining body statements and moves toward the next iteration.

### 9. What does `break` do when used in a single loop?

A. Exits the nearest enclosing loop<br>
B. Skips one body statement<br>
C. Restarts the loop<br>
D. Sets the condition to true

**Answer: A.** By default, break exits the immediately enclosing loop structure.

### 10. What does `break 2` do inside two nested loops?

A. Exits the inner and outer loops<br>
B. Exits only the inner loop<br>
C. Skips two iterations<br>
D. Breaks two variables

**Answer: A.** The numeric level says how many enclosing structures to exit.

### 11. What is a common cause of an accidental infinite `while` loop?

A. Using braces<br>
B. Forgetting to change the value used by its condition<br>
C. Printing a newline<br>
D. Starting a counter at zero

**Answer: B.** If the condition remains true and there is no exit, the loop keeps running.

### 12. How many values does `for ($i = 0; $i < 4; $i++)` visit in its body?

A. Three: 1, 2, 3<br>
B. Four: 0, 1, 2, 3<br>
C. Five: 0, 1, 2, 3, 4<br>
D. Four: 1, 2, 3, 4

**Answer: B.** The counter starts at zero and stops before reaching four.

### 13. What does `continue` do to the update expression in a `for` loop?

A. It skips the update expression.<br>
B. It proceeds to the update expression.<br>
C. It runs initialization again.<br>
D. It converts the loop to a `while`.

**Answer: B.** The update still runs, which helps the loop make progress.

### 14. Which loop is often convenient when the counter's start, stopping condition, and update are known together?

A. `for`<br>
B. `do-while` only<br>
C. `switch`<br>
D. `if`

**Answer: A.** A `for` loop keeps the counter setup, condition, and update together.

### 15. Which loop is introduced in the next chapter for iterating through arrays and other iterables?

A. `repeat`<br>
B. `foreach`<br>
C. `until`<br>
D. `loop`

**Answer: B.** The next chapter covers `foreach` loops and key-value iteration.

## Short-answer questions

### 16. What is the basic shape of a `while` loop?

**Answer:**

~~~php
<?php
while ($condition) {
    // Repeated statements.
}
~~~

**Explanation:** PHP checks the condition before running the body each time.

### 17. What is the key difference between `while` and `do-while`?

**Answer:** `while` checks before the body and may run zero times; `do-while` checks after the body and runs at least once.

**Explanation:** Choose `do-while` when an initial attempt must happen before deciding whether to repeat.

### 18. In what order do the parts of a `for` loop run?

**Answer:** Initialization once; condition; body; update; then condition and body repeat as long as the condition is true.

**Explanation:** The update follows each completed body, then the condition controls the next iteration.

### 19. How can a loop body make a `while` condition eventually become false?

**Answer:** Change the variable or state that the condition depends on.

**Explanation:** A counter can increment, input can advance, or the program can reach a deliberate `break`.

### 20. What does a false initial condition do to a `do-while` loop?

**Answer:** The body still runs once, then the condition is checked.

**Explanation:** The test is at the end of the first iteration.

### 21. What does `break` do by default?

**Answer:** It exits the nearest enclosing loop or switch.

**Explanation:** Use a numeric level such as `break 2` only when you intentionally need to exit more than one nested structure.

### 22. What does `continue` do by default?

**Answer:** It skips the rest of the current iteration and proceeds toward the next one.

**Explanation:** It does not terminate the loop.

### 23. What happens after `continue` in a `for` loop?

**Answer:** PHP runs the update expression and then checks the condition.

**Explanation:** This differs from a `while` body where an update placed after the `continue` would be skipped.

### 24. Why can `continue` inside a `while` loop cause an infinite loop?

**Answer:** It can skip a counter update placed later in the body, leaving the condition true forever.

**Explanation:** Update the counter before the continue branch or restructure the loop.

### 25. How many iterations does a counter from `0` with condition `$i < 4` run?

**Answer:** Four iterations, for values `0`, `1`, `2`, and `3`.

**Explanation:** The condition excludes `4`.

### 26. What does `break 2` mean in two nested loops?

**Answer:** Exit both the inner and outer loop structures.

**Explanation:** The number is the count of enclosing structures to break out of.

### 27. Name two reasons a loop might accidentally run forever.

**Answer:** Its condition variable never changes, or `continue` skips the only update that would make the condition false.

**Explanation:** Check that every path through the body either progresses the loop state or exits.

### 28. When is `do-while` a useful choice?

**Answer:** When the body must run once before the program checks whether another iteration is needed.

**Explanation:** A prompt or an initial attempt can happen first, with the condition based on its result afterward.

### 29. Which loop should normally be used to visit each value in an array?

**Answer:** `foreach`.

**Explanation:** Chapter 033 covers that loop. A counter loop is better suited to a numeric range or a known step pattern.

## True or false

### 30. A `while` loop tests its condition before its first body iteration.

**Answer: True.** If the condition is false initially, the body runs zero times.

### 31. A `do-while` loop can run zero times.

**Answer: False.** Its body runs once before the first condition check.

### 32. A `do-while` statement ends with a semicolon after its condition.

**Answer: True.** The syntax is `} while ($condition);`.

### 33. A `for` loop's initialization normally runs after each body iteration.

**Answer: False.** Initialization runs once before the first condition check.

### 34. In a `for` loop, `continue` proceeds to the update expression.

**Answer: True.** PHP runs the update and then checks the loop condition again.

### 35. A plain `break` in a nested loop exits all enclosing loops.

**Answer: False.** It exits only the nearest one; a level such as `break 2` targets two structures.

### 36. If a `while` condition is false before entering the loop, the body does not run.

**Answer: True.** The test happens before the body.

### 37. PHP automatically increments a variable used as a loop counter.

**Answer: False.** The program must update it, usually in the loop body or the `for` update section.

## Predict the output

### 38. What does this print?

~~~php
<?php
$count = 1;

while ($count <= 3) {
    echo $count, "\n";
    $count++;
}
~~~

**Answer:**

~~~text
1
2
3
~~~

**Explanation:** The counter starts at one and increments until the condition becomes false at four.

### 39. What does this print?

~~~php
<?php
$count = 5;

do {
    echo "Once\n";
    $count++;
} while ($count < 5);
~~~

**Answer:**

~~~text
Once
~~~

**Explanation:** The body runs before the condition is checked, so it runs once.

### 40. What does this print?

~~~php
<?php
for ($index = 0; $index < 4; $index++) {
    echo $index, " ";
}
~~~

**Answer:**

~~~text
0 1 2 3
~~~

**Explanation:** The counter begins at zero and stops when it reaches four.

### 41. What does this print?

~~~php
<?php
for ($number = 1; $number <= 5; $number++) {
    if ($number === 3) {
        continue;
    }

    echo $number, " ";
}
~~~

**Answer:**

~~~text
1 2 4 5
~~~

**Explanation:** The `continue` skips printing three. The `for` update still increments the counter.

### 42. What does this print?

~~~php
<?php
for ($row = 1; $row <= 2; $row++) {
    for ($column = 1; $column <= 3; $column++) {
        if ($row === 2 && $column === 2) {
            break 2;
        }

        echo "{$row},{$column}\n";
    }
}
~~~

**Answer:**

~~~text
1,1
1,2
1,3
2,1
~~~

**Explanation:** When the counters reach row 2 and column 2, `break 2` exits both loops before that pair is printed.

## Write the code

### 43. Use a `while` loop to print the integers from 1 through 5, updating the counter so the loop ends.

**Answer:**

~~~php
<?php
$count = 1;

while ($count <= 5) {
    echo $count, "\n";
    $count++;
}
~~~

**Explanation:** Incrementing `$count` makes the condition false after the fifth value.

### 44. Write a `do-while` loop whose body runs once even though the condition is false after the first iteration.

**Answer:**

~~~php
<?php
$count = 1;

do {
    echo "Attempted\n";
    $count++;
} while ($count < 1);
~~~

**Explanation:** The body runs first. After incrementing, `$count < 1` is false, so the loop ends.

### 45. Use a `for` loop to print the even integers from 2 through 10.

**Answer:**

~~~php
<?php
for ($number = 2; $number <= 10; $number += 2) {
    echo $number, "\n";
}
~~~

**Explanation:** The update adds two each time, and the inclusive condition allows the final value `10`.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [031 - The match expression](../lessons/031-the-match-expression.md) | [Read this lesson](../lessons/032-loops-with-while-do-while-and-for.md) | [Open example](../examples/032-loops-with-while-do-while-and-for.md) | [033 - foreach loops and key-value iteration](../lessons/033-foreach-loops-and-key-value-iteration.md) |
