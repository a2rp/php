# 042 - Arrow Functions: Q&A

Use these questions to review arrow-function syntax, automatic variable capture, callbacks, and when a regular closure is a better fit. Try each question before reading its answer. See the [chapter lesson](../lessons/042-arrow-functions.md).

## Multiple-choice questions

### 1. Which keyword begins a PHP arrow function?

A. **function**
B. **fn**
C. **arrow**
D. **lambda**

**Answer:** B. **fn**.

**Explanation:** PHP arrow functions begin with **fn**, followed by the parameter list and **=>**.

### 2. Which is the valid basic arrow-function form?

A. **fn ($value) { $value * 2; }**
B. **fn ($value) => $value * 2**
C. **arrow ($value) return $value * 2**
D. **function fn ($value) => $value * 2**

**Answer:** B. **fn ($value) => $value * 2**.

**Explanation:** An arrow function has a parameter list, the **=>** operator, and one expression.

### 3. What happens to the expression's value in an arrow function?

A. It is printed automatically.
B. It is returned automatically.
C. It is discarded.
D. It must be assigned to **$this**.

**Answer:** B. It is returned automatically.

**Explanation:** The expression after **=>** is the function's result. No explicit **return** statement is needed.

### 4. How many expressions can an arrow function body contain?

A. Exactly one expression
B. Any number of statements in braces
C. Zero expressions only
D. Two expressions separated by a comma

**Answer:** A. Exactly one expression.

**Explanation:** For a multi-statement body, use a regular anonymous function instead.

### 5. How are outer variables captured by an arrow function?

A. They must be listed in **use**.
B. Referenced outer variables are captured automatically by value.
C. They are always captured by reference.
D. They are unavailable.

**Answer:** B. Referenced outer variables are captured automatically by value.

**Explanation:** Arrow functions do not need a **use** clause. Their automatic capture is by value.

### 6. Which syntax assigns an arrow function to **$double**?

A. **$double = fn (int $n): int => $n * 2;**
B. **fn $double (int $n) => $n * 2;**
C. **$double := fn (int $n) { $n * 2; }**
D. **function $double => $n * 2;**

**Answer:** A. **$double = fn (int $n): int => $n * 2;**

**Explanation:** The arrow function is a value assigned to a variable, so the assignment ends with a semicolon.

### 7. What is the correct syntax for an arrow function with no parameters?

A. **fn => 'ready'**
B. **fn () => 'ready'**
C. **fn (null) => 'ready'**
D. **fn (void) => 'ready'**

**Answer:** B. **fn () => 'ready'**.

**Explanation:** The parentheses are still required; they are simply empty.

### 8. Which callback transforms each array value into a doubled value?

A. **array_map(fn ($n) => $n * 2, $values)**
B. **array_filter(fn ($n) => $n * 2, $values)**
C. **array_map($values, fn ($n) => $n * 2)**
D. **array_reduce(fn ($n) => $n * 2, $values)**

**Answer:** A. **array_map(fn ($n) => $n * 2, $values)**.

**Explanation:** **array_map()** calls its callback for each input and collects the returned values.

### 9. Which function is commonly used to keep only array values that pass a condition?

A. **array_map()**
B. **array_filter()**
C. **array_keys()**
D. **array_merge()**

**Answer:** B. **array_filter()**.

**Explanation:** **array_filter()** uses a callback's truthy result to decide which values to keep.

### 10. What is the minimum PHP version that supports arrow-function syntax?

A. PHP 5.6
B. PHP 7.0
C. PHP 7.4
D. PHP 8.3

**Answer:** C. PHP 7.4.

**Explanation:** Arrow functions were introduced in PHP 7.4. Older runtimes cannot parse this syntax.

### 11. Which arrow function declares an integer return type?

A. **fn (int $n): int => $n + 1**
B. **fn (int $n) int: => $n + 1**
C. **fn int ($n) => $n + 1**
D. **fn ($n): => int $n + 1**

**Answer:** A. **fn (int $n): int => $n + 1**.

**Explanation:** As with other PHP functions, the return type follows the parameter list after a colon.

### 12. Which statement about **use** is correct for arrow functions?

A. Every arrow function must include **use**.
B. Arrow functions use **use (&$value)** for reference capture.
C. Arrow functions automatically capture referenced outer variables by value.
D. **use** is required only when the function has a return type.

**Answer:** C. Arrow functions automatically capture referenced outer variables by value.

**Explanation:** Use a regular closure if you need an explicit capture list or a capture by reference.

### 13. Which is a good reason to use a regular closure instead of an arrow function?

A. The callback accepts a parameter.
B. The callback returns a value.
C. The function needs multiple statements and a local block.
D. The function is assigned to a variable.

**Answer:** C. The function needs multiple statements and a local block.

**Explanation:** Arrow functions have a single-expression body. Regular closures support a statement block.

### 14. What does this expression create?

~~~php
$twice = fn (int $n): int => $n * 2;
~~~

A. A class named **twice**
B. A callable function value stored in **$twice**
C. An array with two entries
D. A named global function

**Answer:** B. A callable function value stored in **$twice**.

**Explanation:** An arrow function is an anonymous function value and can be stored in a variable.

### 15. Which statement correctly distinguishes arrow functions from first-class callable syntax?

A. They are two names for the same syntax.
B. Arrow functions define a new function expression; first-class callable syntax converts an existing function or method into a callable.
C. Arrow functions work only with methods.
D. First-class callable syntax was introduced in PHP 7.4 along with arrow functions.

**Answer:** B. Arrow functions define a new function expression; first-class callable syntax converts an existing function or method into a callable.

**Explanation:** They solve related but different problems. First-class callable syntax is covered in the next chapter and requires PHP 8.1 or later.

## Short-answer questions

### 16. Write the general syntax of an arrow function.

**Answer:**

~~~php
fn (parameters): returnType => expression
~~~

**Explanation:** The return type is optional. The expression's value is returned automatically.

### 17. Why is an arrow function useful for a short callback?

**Answer:** It keeps a small function near the operation that uses it, without a separate **function** body and **use** list.

**Explanation:** This is especially convenient with functions such as **array_map()** and **array_filter()**.

### 18. How do you call an arrow function stored in **$square**?

**Answer:** Call it like **$square(5)**, supplying arguments in parentheses.

**Explanation:** The variable contains a callable function value.

### 19. What does automatic capture by value mean?

**Answer:** A referenced outer variable's value is made available to the arrow function when that arrow function is created.

**Explanation:** A later reassignment of the outer variable does not replace the value already captured by the arrow function.

### 20. Can an arrow function capture an outer variable without listing it in **use**?

**Answer:** Yes. Referenced outer variables are captured automatically by value.

**Explanation:** Arrow functions do not use an explicit **use** clause.

### 21. What should you use if a callback needs to increment an outer scalar counter?

**Answer:** Use a regular closure with **use (&$counter)**.

**Explanation:** The **&** captures the variable by reference so changes made in the closure are visible in the surrounding scope.

### 22. Can an arrow function contain a **foreach** statement?

**Answer:** No. A **foreach** loop is a statement, while an arrow function body is one expression.

**Explanation:** Put multi-step logic in a regular closure or a named function.

### 23. Can a ternary operator appear in an arrow function?

**Answer:** Yes.

~~~php
$label = fn (int $score): string =>
    $score >= 50 ? 'Pass' : 'Try again';
~~~

**Explanation:** A ternary expression is still one expression.

### 24. Are parameter types and return types allowed?

**Answer:** Yes. Arrow functions can declare parameter types and a return type in the same positions used by other PHP functions.

**Explanation:** For example: **fn (int $n): int => $n + 1**.

### 25. What is the purpose of **array_map()** when given an arrow-function callback?

**Answer:** It calls the callback for each input value and returns an array containing the callback results.

**Explanation:** The arrow function decides how each value is transformed.

### 26. What is the purpose of **array_filter()** when given an arrow-function callback?

**Answer:** It keeps values whose callback result is truthy.

**Explanation:** The arrow function expresses the condition for each value.

### 27. When was arrow-function syntax introduced in PHP?

**Answer:** PHP 7.4.

**Explanation:** A runtime older than PHP 7.4 cannot parse the syntax.

### 28. Why does this fail as arrow-function syntax?

~~~php
fn (int $n) {
    return $n + 1;
}
~~~

**Answer:** It uses a brace-delimited body. Arrow functions require **=>** followed by one expression.

**Explanation:** For braces and multiple statements, use **function (int $n) { ... }**.

### 29. When should a short arrow function be replaced with a named function or regular closure?

**Answer:** Replace it when the expression becomes hard to understand, the logic has multiple steps, or the function needs by-reference capture.

**Explanation:** Choose the form that makes the behavior easiest to read and maintain.

## True or false

### 30. An arrow function can be assigned to a variable.

**Answer:** True.

**Explanation:** It is an anonymous function value, so it can be stored and called through that variable.

### 31. An arrow function requires an explicit **return** statement.

**Answer:** False.

**Explanation:** Its single expression is returned automatically.

### 32. An arrow function can have an empty parameter list.

**Answer:** True.

**Explanation:** Use parentheses with nothing between them: **fn () => expression**.

### 33. A regular arrow function captures referenced outer variables by reference.

**Answer:** False.

**Explanation:** Automatic capture is by value. Use a regular closure with **use (&$value)** for reference capture.

### 34. A later reassignment of an outer scalar variable changes the value already captured by an arrow function.

**Answer:** False.

**Explanation:** The arrow function keeps the value captured when it was created.

### 35. A single ternary expression is allowed in an arrow function.

**Answer:** True.

**Explanation:** The body is one expression, and a ternary expression satisfies that requirement.

### 36. An arrow function can have a statement block with several independent statements.

**Answer:** False.

**Explanation:** Use a regular closure or named function when you need a block.

### 37. PHP versions before 7.4 support arrow functions.

**Answer:** False.

**Explanation:** Arrow functions were added in PHP 7.4.

## Predict the output

### 38. What does this code print?

~~~php
<?php
$double = fn (int $number): int => $number * 2;

echo $double(6), "\n";
~~~

**Answer:**

~~~text
12
~~~

**Explanation:** The function receives **6**, multiplies it by **2**, and returns **12**.

### 39. What does this code print?

~~~php
<?php
$factor = 3;
$multiply = fn (int $number): int => $number * $factor;

$factor = 5;

echo $multiply(4), "\n";
~~~

**Answer:**

~~~text
12
~~~

**Explanation:** **$factor** was captured by value as **3** when the arrow function was created. The later assignment to **5** does not change that captured value.

### 40. What does this code print?

~~~php
<?php
$values = [2, 4, 6];

$squares = array_map(
    fn (int $value): int => $value * $value,
    $values,
);

echo implode(', ', $squares), "\n";
~~~

**Answer:**

~~~text
4, 16, 36
~~~

**Explanation:** **array_map()** squares each input and returns the three callback results.

### 41. What does this code print?

~~~php
<?php
$values = [4, 9, 12, 18];

$selected = array_filter(
    $values,
    fn (int $value): bool => $value >= 10,
);

echo implode(', ', $selected), "\n";
~~~

**Answer:**

~~~text
12, 18
~~~

**Explanation:** The callback is true for **12** and **18**, so those values remain in the filtered array.

### 42. What does this code print?

~~~php
<?php
$describe = fn (int $score): string =>
    $score >= 50 ? 'Pass' : 'Try again';

echo $describe(50), "\n";
echo $describe(49), "\n";
~~~

**Answer:**

~~~text
Pass
Try again
~~~

**Explanation:** The condition uses **>= 50**, so **50** passes and **49** does not.

## Write code

### 43. Write an arrow function that accepts an integer and returns its cube.

**Answer:**

~~~php
<?php
$cube = fn (int $number): int => $number * $number * $number;

echo $cube(3), "\n";
~~~

Output:

~~~text
27
~~~

**Explanation:** The one expression multiplies the input by itself three times.

### 44. Use **array_map()** and an arrow function to add five to every number in **[1, 2, 3]**.

**Answer:**

~~~php
<?php
$values = [1, 2, 3];

$increased = array_map(
    fn (int $value): int => $value + 5,
    $values,
);

echo implode(', ', $increased), "\n";
~~~

Output:

~~~text
6, 7, 8
~~~

**Explanation:** The callback returns one increased value for each input value.

### 45. Write a regular closure that increments an outer counter twice, and explain why it is not an arrow function.

**Answer:**

~~~php
<?php
$count = 0;

$next = function () use (&$count): int {
    $count++;
    return $count;
};

echo $next(), "\n";
echo $next(), "\n";
echo "Final count: {$count}\n";
~~~

Output:

~~~text
1
2
Final count: 2
~~~

**Explanation:** The closure needs both a statement block and a by-reference capture. A regular closure supports **use (&$count)**; an arrow function captures outer variables by value and has a single-expression body.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [041 - Anonymous Functions and Closures](../lessons/041-anonymous-functions-and-closures.md) | [Read lesson](../lessons/042-arrow-functions.md) | [Open example](../examples/042-arrow-functions.md) | [043 - Callable Values and First-Class Callable Syntax](../lessons/043-callable-values-and-first-class-callable-syntax.md) |
