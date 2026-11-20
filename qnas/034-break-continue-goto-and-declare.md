# 034 - break, continue, goto, and declare: Questions and Answers

Review [034 - break, continue, goto, and declare](../lessons/034-break-continue-goto-and-declare.md). Each answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. What does **break** do inside a loop?

A. Skips only the current iteration<br>
B. Exits the nearest loop or switch<br>
C. Restarts the loop from the beginning<br>
D. Exits the PHP script

**Answer: B.** By default, **break** exits the nearest enclosing loop or **switch**. Execution continues after that structure.

### 2. What does a plain **break** do when a **switch** is inside a loop?

A. Exits both the switch and the loop<br>
B. Exits only the switch<br>
C. Skips to the next loop iteration<br>
D. Exits the PHP file

**Answer: B.** A plain **break** exits only the nearest structure, which is the **switch**. Use a numeric level such as **break 2** to leave both.

### 3. What does **break 2** mean inside two nested loops?

A. Break after two iterations<br>
B. Exit two enclosing loop or switch structures<br>
C. Exit the current function<br>
D. Skip two values

**Answer: B.** The numeric argument is a nesting level. **break 2** exits the two nearest enclosing breakable structures.

### 4. Does an **if** statement count as a level for **break 2**?

A. Yes, every block counts<br>
B. No, only enclosing loops and switches count<br>
C. Only when it has an **else** branch<br>
D. Only in a **foreach**

**Answer: B.** The level counts loops and **switch** structures, not conditional blocks such as **if**.

### 5. What does **continue** do inside a loop?

A. Exits the loop completely<br>
B. Skips the rest of the current iteration<br>
C. Restarts the PHP program<br>
D. Exits the nearest function

**Answer: B.** **continue** skips the remaining body for the current iteration and moves toward the next iteration.

### 6. What happens to the update expression in a **for** loop after **continue**?

A. It is skipped permanently<br>
B. It runs before PHP checks the next loop condition<br>
C. It runs only if the loop has an **else**<br>
D. The loop exits before the update

**Answer: B.** The normal **for** loop update step still runs, followed by the condition check.

### 7. What does plain **continue** do inside a **switch** in PHP?

A. Always continues the surrounding loop without a warning<br>
B. Behaves like **break** for the switch and raises a warning<br>
C. Exits the entire script<br>
D. Jumps to the next switch case

**Answer: B.** PHP counts **switch** as a looping structure for **continue**. Plain **continue** there behaves like **break** and warns because this is often unintended.

### 8. A **switch** is inside a **foreach**. Which statement skips to the next **foreach** item from a switch case?

A. **break**<br>
B. **continue**<br>
C. **continue 2**<br>
D. **goto 2**

**Answer: C.** **continue 2** leaves the switch level and continues the enclosing loop.

### 9. Which statement about a **goto** label is correct?

A. Labels are case-insensitive variables<br>
B. A label is a case-sensitive name followed by a colon<br>
C. A label must always be inside a loop<br>
D. A label is declared with the **function** keyword

**Answer: B.** A label has a name followed by a colon, and matching is case-sensitive.

### 10. Which jump is not allowed in PHP?

A. Jumping out of a loop<br>
B. Jumping to a label in the same function<br>
C. Jumping into a loop or switch<br>
D. Jumping forward to a label in the same file

**Answer: C.** PHP does not allow **goto** to enter a loop or **switch** structure.

### 11. Where must a **goto** target label be?

A. In any file in the project<br>
B. In the same file and function or method context<br>
C. In a global variable<br>
D. In a different PHP process

**Answer: B.** PHP restricts the label to the same file and execution context; a jump cannot cross into or out of a function or method.

### 12. What is the purpose of **declare**?

A. Declare a new variable type<br>
B. Set language directives for code<br>
C. Import a PHP file<br>
D. Create a loop

**Answer: B.** **declare** configures directives such as **ticks**, **encoding**, and **strict_types**.

### 13. What does **declare(strict_types=1)** primarily change?

A. It forces every variable to have a type<br>
B. It enables stricter behavior for scalar type declarations in that file<br>
C. It makes all arrays immutable<br>
D. It disables function calls

**Answer: B.** Strict typing affects scalar type declarations. It does not type-check every untyped variable assignment.

### 14. Where should **declare(strict_types=1)** be placed?

A. After the first function call<br>
B. As the first PHP statement in the file<br>
C. Inside every function<br>
D. At the end of the file

**Answer: B.** Strict typing is a per-file directive and must appear before other PHP statements.

### 15. In strict scalar typing, which value can satisfy a **float** type declaration?

A. A numeric string<br>
B. A boolean<br>
C. An integer<br>
D. Any value PHP can convert

**Answer: C.** An **int** value is accepted for a **float** declaration even with strict scalar typing enabled.

## Short-answer questions

### 16. Write the statement that exits the nearest loop.

**Answer:**

~~~php
break;
~~~

**Explanation:** A plain **break** exits the nearest enclosing loop or switch.

### 17. When is **break 2** useful?

**Answer:** It is useful when code inside a nested loop needs to exit both the inner and outer loop.

**Explanation:** The number counts enclosing loop or switch structures. An **if** block does not add a level.

### 18. Explain the difference between **break** and **continue**.

**Answer:** **break** exits the current loop or switch. **continue** skips the rest of the current loop iteration and proceeds toward the next one.

**Explanation:** Use **break** when the loop should stop; use **continue** when only the current item should be skipped.

### 19. What does a **for** loop do after its body reaches **continue**?

**Answer:** It runs the update expression and then checks the loop condition.

**Explanation:** **continue** does not permanently skip the normal update step of a **for** loop.

### 20. A **switch** is inside a **while** loop. What does **break** in a switch case exit by default?

**Answer:** It exits only the **switch**.

**Explanation:** The nearest enclosing structure is the switch. The while loop continues unless a higher break level is used.

### 21. How can a switch case skip the next iteration of its surrounding loop?

**Answer:** Use **continue 2** inside that switch case.

**Explanation:** The switch is one level and the surrounding loop is the second level.

### 22. Show the basic syntax for a label and a jump to it.

**Answer:**

~~~php
<?php
goto finish;

finish:
echo "Finished\n";
~~~

**Explanation:** The label is a case-sensitive name ending with a colon. The **goto** statement transfers execution to it.

### 23. Name two restrictions on **goto**.

**Answer:** It cannot jump into a loop or switch, and it cannot jump into or out of a function or method.

**Explanation:** The target must also be in the same file and context.

### 24. Why is **goto** usually avoided in ordinary application code?

**Answer:** Unnecessary jumps make the execution path harder to follow.

**Explanation:** Loops, conditions, helper functions, **break**, **continue**, and **return** usually show the program's structure more clearly.

### 25. What does **declare(strict_types=1)** enable?

**Answer:** It enables strict handling for scalar type declarations in that file.

**Explanation:** A mismatched scalar argument or return value can result in a **TypeError** instead of being coerced. It does not type-check untyped assignments.

### 26. Which directives does PHP recognize for **declare**?

**Answer:** **ticks**, **encoding**, and **strict_types**.

**Explanation:** **strict_types** is commonly useful in modern application code. **ticks** and **encoding** are more specialized.

### 27. Is **strict_types** a project-wide setting?

**Answer:** No. It is enabled per file.

**Explanation:** For scalar function arguments, the calling file's setting determines whether coercive or strict behavior applies to that call.

### 28. Does strict mode reject an integer passed to a parameter declared as **float**?

**Answer:** No. An integer is accepted for a **float** declaration.

**Explanation:** This is a documented exception to strict scalar matching.

### 29. Why must a **declare** directive use a literal value?

**Answer:** PHP processes directive values while compiling the file, so variables and constants cannot provide those values.

**Explanation:** For example, write **declare(strict_types=1)** directly instead of passing a variable that contains **1**.

## True or false

### 30. A plain **break** exits every nested loop.

**Answer: False.** A plain **break** exits only the nearest loop or switch.

**Explanation:** Use a numeric level such as **break 2** when more than one enclosing structure must be exited.

### 31. An **if** statement adds one level to **break 2**.

**Answer: False.** Only enclosing loops and switches count toward the level.

**Explanation:** Conditional blocks do not change the breakable-structure count.

### 32. **continue** ends the entire loop.

**Answer: False.** It skips the rest of the current iteration, then the loop can continue.

**Explanation:** Use **break** to end the loop.

### 33. A **for** loop still performs its update expression after **continue**.

**Answer: True.** The update expression runs before the next condition check.

**Explanation:** This preserves the normal order of a **for** loop.

### 34. PHP treats **switch** as a looping structure for **continue**.

**Answer: True.** This is why plain **continue** inside a switch behaves like **break** and raises a warning.

**Explanation:** Use **continue 2** to continue an enclosing loop from inside a switch.

### 35. **continue 2** inside a switch nested in a loop can move to the next outer-loop iteration.

**Answer: True.** It continues through two enclosing levels: the switch and then the loop.

**Explanation:** The same level-counting principle applies to nested loops.

### 36. PHP allows **goto** to jump into the middle of a **while** loop.

**Answer: False.** Jumping into a loop or switch is disallowed.

**Explanation:** A **goto** can jump out of such a structure, but it cannot jump into one.

### 37. **declare(strict_types=1)** makes every variable assignment in a file type-safe.

**Answer: False.** It affects scalar type declarations, not untyped assignments.

**Explanation:** It is not a replacement for declaring function parameter, return, or property types.

## Predict the output

### 38. What does this print?

~~~php
<?php
for ($number = 0; $number < 5; $number++) {
    if ($number === 2) {
        continue;
    }

    echo $number;
}
~~~

**Answer:**

~~~text
0134
~~~

**Explanation:** The iteration where **$number** is **2** skips the echo. The **for** update still runs, so the loop proceeds to **3**.

### 39. What does this print?

~~~php
<?php
$number = 1;

while ($number <= 4) {
    if ($number === 3) {
        break;
    }

    echo $number;
    $number++;
}
~~~

**Answer:**

~~~text
12
~~~

**Explanation:** The loop prints **1** and **2**. When **$number** becomes **3**, **break** exits before the echo.

### 40. What does this print?

~~~php
<?php
for ($number = 1; $number <= 2; $number++) {
    switch ($number) {
        case 1:
            echo "one ";
            break;
        case 2:
            echo "two ";
            break;
    }

    echo "done ";
}
~~~

**Answer:**

~~~text
one done two done
~~~

**Explanation:** Each **break** exits only the **switch**. After the switch, the loop prints **done**.

### 41. What does this print?

~~~php
<?php
foreach ([['a', 'b'], ['c', 'd']] as $rowIndex => $row) {
    foreach ($row as $columnIndex => $value) {
        if ($value === 'c') {
            echo "{$rowIndex},{$columnIndex}\n";
            break 2;
        }
    }
}

echo "end\n";
~~~

**Answer:**

~~~text
1,0
end
~~~

**Explanation:** The matching value **c** is at row key **1** and column key **0**. **break 2** exits both loops, then execution reaches the final echo.

### 42. What does this print?

~~~php
<?php
foreach (['keep', 'skip', 'keep'] as $word) {
    switch ($word) {
        case 'skip':
            continue 2;
        default:
            echo $word, "\n";
    }

    echo "checked\n";
}
~~~

**Answer:**

~~~text
keep
checked
keep
checked
~~~

**Explanation:** For **skip**, **continue 2** skips the remaining switch and loop body and proceeds to the next **foreach** iteration.

## Write the code

### 43. Given an array named **$scores**, print only values greater than zero. Skip other values.

**Answer:**

~~~php
<?php
foreach ($scores as $score) {
    if ($score <= 0) {
        continue;
    }

    echo $score, "\n";
}
~~~

**Explanation:** **continue** skips the echo for non-positive values while allowing the loop to process the remaining scores.

### 44. Search a two-dimensional array named **$rows** for the string **target**. Print **Found** and exit both loops when it is found.

**Answer:**

~~~php
<?php
foreach ($rows as $row) {
    foreach ($row as $value) {
        if ($value === 'target') {
            echo "Found\n";
            break 2;
        }
    }
}
~~~

**Explanation:** The match is found inside the inner loop. **break 2** exits both the inner and outer loops.

### 45. Write a strict-typing file header and a function that accepts an integer quantity and returns a string. Call it with a valid integer.

**Answer:**

~~~php
<?php
declare(strict_types=1);

function describeQuantity(int $quantity): string
{
    return "Quantity: {$quantity}";
}

echo describeQuantity(3), "\n";
~~~

**Explanation:** The directive is at the top of the file. The function has scalar parameter and return declarations, and the call supplies an integer value.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [033 - foreach Loops and Key-Value Iteration](../lessons/033-foreach-loops-and-key-value-iteration.md) | [Read this lesson](../lessons/034-break-continue-goto-and-declare.md) | [Open example](../examples/034-break-continue-goto-and-declare.md) | [035 - Including and Requiring PHP Files](../lessons/035-including-and-requiring-php-files.md) |
