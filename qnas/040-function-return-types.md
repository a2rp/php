# 040 - Function Return Types: Q&A

Review how PHP functions return values, how return types describe the result, and what **void** and **never** mean. Try each question before reading its answer. See the [chapter lesson](../lessons/040-function-return-types.md).

## Multiple-choice questions

### 1. What does a function's **return** statement do?

A. Sends a value back to the caller and ends the function
B. Prints a value automatically
C. Starts the function again
D. Declares a parameter

**Answer:** A. It sends a value back to the caller and ends the function.

**Explanation:** The caller can store or use the returned value.

### 2. Where is a return type declared?

A. Before the **function** keyword
B. After the parameter list, preceded by a colon
C. Inside every **return** statement
D. After the function body

**Answer:** B. After the parameter list, preceded by a colon.

**Explanation:** For example, **function count(): int** declares an integer return type.

### 3. What does **: int** promise?

A. The function accepts only integer arguments
B. The function returns a value compatible with **int**
C. The function prints an integer
D. The function has no parameters

**Answer:** B. It promises a value compatible with **int**.

**Explanation:** Parameter types are declared inside the parentheses; return type follows them.

### 4. What happens when PHP executes **return** inside a function?

A. The function ends immediately
B. The script always exits
C. The function continues after the return
D. The return type changes

**Answer:** A. The function ends immediately.

**Explanation:** Statements after a reached return are skipped for that call.

### 5. Does **return $total;** print **$total**?

A. Yes, always
B. No, it gives the value to the caller
C. Only if the return type is **int**
D. Only inside a web page

**Answer:** B. It gives the value to the caller.

**Explanation:** Use **echo** or another output operation to display it.

### 6. What does **void** mean as a return type?

A. The function returns null as a value the caller can use
B. The function is intended not to return a value
C. The function never ends
D. The function must return a string

**Answer:** B. It is intended not to return a value.

**Explanation:** An action-only function can use **void**. Use **never** for a function that cannot complete normally.

### 7. Which is permitted in a function declared **void**?

A. **return 5;**
B. **return 'done';**
C. **return;**
D. **return [];**

**Answer:** C. **return;**

**Explanation:** A void function may use an empty return to stop early, but it must not return a value.

### 8. What does the **never** return type describe?

A. A function that returns null
B. A function that cannot complete normally
C. A function that returns any type
D. A function without parameters

**Answer:** B. A function that cannot complete normally.

**Explanation:** It must throw, exit, or otherwise never reach a normal completion.

### 9. Since which PHP version is the **never** return type available?

A. PHP 5.0
B. PHP 7.1
C. PHP 8.0
D. PHP 8.1

**Answer:** D. PHP 8.1.

**Explanation:** PHP 8.1 introduced **never** as a return-only type.

### 10. If a function has no return type and reaches its end without **return**, what does the call produce?

A. **false**
B. **null**
C. **0**
D. An empty array

**Answer:** B. **null**.

**Explanation:** An untyped function that falls through returns null implicitly.

### 11. If a function declared **: string** returns an incompatible value, what can PHP raise?

A. **TypeError**
B. **ArgumentCountError**
C. **ParseWarning**
D. No error

**Answer:** A. **TypeError**.

**Explanation:** PHP checks returned values against the declared return type.

### 12. What must a nullable return type such as **?string** allow?

A. A string or null
B. A string or integer
C. No return value
D. Only an empty string

**Answer:** A. A string or null.

**Explanation:** A nullable type is the union of that type and **null**.

### 13. How can a function provide several related pieces of data as its return?

A. Return several expressions at once
B. Return an array or object containing them
C. Use several return types in separate statements
D. Echo each value and claim it was returned

**Answer:** B. Return an array or object containing them.

**Explanation:** A PHP function returns one value; a collection can group multiple fields into that value.

### 14. What happens if a typed non-void function reaches its end without returning a value?

A. PHP invents a value of the declared type
B. PHP raises a **TypeError**
C. It automatically becomes **void**
D. It repeats the last statement

**Answer:** B. PHP raises a **TypeError**.

**Explanation:** A non-void return contract must be fulfilled on every normal path.

### 15. Which type form allows either an integer or a string return?

A. **int|string**
B. **int,string**
C. **?int|string** only
D. **void|int**

**Answer:** A. **int|string**.

**Explanation:** A union return type lists its alternatives with the pipe symbol.

## Short-answer questions

### 16. What is the difference between a return type and a parameter type?

**Answer:** A parameter type describes an input. A return type describes the function's output.

**Explanation:** Parameter types appear inside the parentheses; the return type follows the closing parenthesis.

### 17. How can a caller use a returned value?

**Answer:** It can assign it to a variable, pass it to another function, or use it in an expression.

**Explanation:** A return value becomes part of the caller's code after the function finishes.

### 18. Does **echo** return its printed text to the caller?

**Answer:** No.

**Explanation:** Displaying output and returning data are separate actions.

### 19. What happens to statements after a **return** that was executed?

**Answer:** They are skipped for that call.

**Explanation:** Returning immediately ends the current function execution.

### 20. How can a function choose between different return values?

**Answer:** Use conditional branches with a compatible return in each path.

**Explanation:** For example, an **if** branch can return one value and the later path another.

### 21. What must a typed non-void function do on every normal path?

**Answer:** Return a value compatible with its declared return type.

**Explanation:** Falling through without a value violates the contract and raises a **TypeError**.

### 22. What does **?string** mean as a return type?

**Answer:** The function may return either a string or null.

**Explanation:** It is shorthand for **string|null**.

### 23. Does a nullable return type let a function omit its return statement?

**Answer:** No.

**Explanation:** The function must explicitly return a compatible value, which may be null.

### 24. How can a function return both a name and a score?

**Answer:** Return one array or object containing both fields.

**Explanation:** PHP returns one value from a function; the array or object is that single value.

### 25. What is the purpose of **void**?

**Answer:** It declares that a function performs an action but does not return a value.

**Explanation:** It is appropriate for effects such as printing a notice.

### 26. What kind of return statement may a **void** function use to stop early?

**Answer:** An empty **return;** statement.

**Explanation:** A void function must not return an expression such as **return 5;**.

### 27. How is **void** different from an untyped function that happens to return null?

**Answer:** **void** explicitly promises that the caller has no result to use; an untyped function has no declared return contract and may produce null when it falls through.

**Explanation:** Explicit types communicate intent and let PHP enforce the contract.

### 28. What does **never** promise?

**Answer:** The function cannot complete normally and return control to its caller.

**Explanation:** It must throw, exit, or otherwise never reach the end.

### 29. When should you use a union return type?

**Answer:** When the function can legitimately return values of more than one listed type.

**Explanation:** Keep the set narrow so callers know what values to handle.

## True or false

### 30. A function can return a value to its caller without printing it.

**Answer:** True.

**Explanation:** **return** passes data back; **echo** prints output.

### 31. Executing a return statement ends the current function immediately.

**Answer:** True.

**Explanation:** Later statements in that execution path do not run.

### 32. A function declared **: void** may return an integer value.

**Answer:** False.

**Explanation:** It may use **return;** but cannot return a value.

### 33. A function declared **: ?string** may explicitly return null.

**Answer:** True.

**Explanation:** The nullable type permits both string and null.

### 34. A typed non-void function may safely reach its end without returning anything.

**Answer:** False.

**Explanation:** PHP raises a **TypeError** when the required return value is missing.

### 35. A function can return an array that contains multiple fields.

**Answer:** True.

**Explanation:** The array is one returned value containing related data.

### 36. The **never** return type means the function returns null.

**Answer:** False.

**Explanation:** **never** means normal completion is impossible; null is a value and is not **never**.

### 37. If a function is untyped and has no executed return statement, its call produces null.

**Answer:** True.

**Explanation:** That is PHP's implicit result for an untyped function that falls through.

## Predict the output

### 38. What is printed?

~~~php
<?php
function square(int $number): int
{
    return $number * $number;
}

echo square(4), "\n";
~~~

**Answer:**

~~~text
16
~~~

**Explanation:** The function returns **16**, and **echo** displays that value.

### 39. What is printed?

~~~php
<?php
function chooseLabel(bool $ready): string
{
    if ($ready) {
        return 'Ready';
    }

    return 'Waiting';
}

echo chooseLabel(false), "\n";
~~~

**Answer:**

~~~text
Waiting
~~~

**Explanation:** The condition is false, so the second return supplies the string.

### 40. What is printed?

~~~php
<?php
function findCode(bool $found): ?int
{
    if (!$found) {
        return null;
    }

    return 25;
}

var_dump(findCode(false));
~~~

**Answer:**

~~~text
NULL
~~~

**Explanation:** The nullable integer return type permits an explicit null result.

### 41. What is printed?

~~~php
<?php
function makePair(): array
{
    return ['left', 'right'];
}

[$first, $second] = makePair();
echo "{$first} {$second}\n";
~~~

**Answer:**

~~~text
left right
~~~

**Explanation:** The function returns one array, then the caller unpacks its two elements.

### 42. What happens when this function is called?

~~~php
<?php
function getCount(): int
{
    // No return statement
}

var_dump(getCount());
~~~

**Answer:** PHP raises a **TypeError** because the function promised an integer but did not return a value.

**Explanation:** A typed non-void function cannot reach its end without a compatible return.

## Write-code questions

### 43. Define **double(int $number): int** and return twice the argument.

**Answer:**

~~~php
<?php
function double(int $number): int
{
    return $number * 2;
}

echo double(6), "\n";
~~~

**Explanation:** The function returns an integer value, matching **: int**.

### 44. Define **findNickname(bool $found): ?string** to return **'Mina'** when found and null otherwise.

**Answer:**

~~~php
<?php
function findNickname(bool $found): ?string
{
    if ($found) {
        return 'Mina';
    }

    return null;
}
~~~

**Explanation:** Both return values are allowed by the nullable string type.

### 45. Define a **void** function that prints **Saved** and call it.

**Answer:**

~~~php
<?php
function showSavedNotice(): void
{
    echo "Saved\n";
}

showSavedNotice();
~~~

**Explanation:** The function performs an output action and does not return a value.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [039 - Default, Named, Variadic, and Unpacked Arguments](../lessons/039-default-named-variadic-and-unpacked-arguments.md) | [Read lesson](../lessons/040-function-return-types.md) | [Open example](../examples/040-function-return-types.md) | [041 - Anonymous Functions and Closures](../lessons/041-anonymous-functions-and-closures.md) |
