# 019 - Type Declarations and Strict Typing: Questions and Answers

Use this Q&A to review [019 - Type Declarations and Strict Typing](../lessons/019-type-declarations-and-strict-typing.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does a parameter type declaration describe?

A. The file where the function must be saved<br>
B. The type of value a function parameter expects<br>
C. The number of times a function can run<br>
D. The type of every variable in the file

**Answer: B.** A parameter declaration states the expected type at a function boundary.

### 2. Where is a parameter type written?

A. After the parameter name and a semicolon<br>
B. Before the parameter name<br>
C. After the function body<br>
D. Before the opening PHP tag

**Answer: B.** In **function greet(string $name)**, **string** is written before the parameter name.

### 3. Where is a return type written?

A. After the closing parenthesis and a colon<br>
B. Before the function name<br>
C. After the final brace<br>
D. Inside the argument list

**Answer: A.** A signature such as **function add(int $a): int** declares an integer return type after a colon.

### 4. What happens when an argument cannot satisfy its declared type?

A. PHP always changes it to null<br>
B. PHP throws a **TypeError**<br>
C. The function is skipped without an error<br>
D. PHP removes the declaration

**Answer: B.** A value that cannot satisfy a declaration causes a runtime **TypeError**.

### 5. What does a return type declaration specify?

A. The type of value the function is expected to return<br>
B. The type of the function's file<br>
C. The type of every argument<br>
D. The visibility of the function

**Answer: A.** PHP checks the function's returned value against its return declaration.

### 6. What does **void** mean as a return type?

A. The function returns an empty string<br>
B. The function does not return a value<br>
C. The function can return any type<br>
D. The function must throw an exception

**Answer: B.** A **void** function may perform an action, but it must not return a value.

### 7. Which statement is allowed in a **void** function when it needs to finish early?

A. **return;**<br>
B. **return 5;**<br>
C. **return "done";**<br>
D. **return [];**

**Answer: A.** A void function can use **return;** without a value to end early.

### 8. What does **never** mean as a return type?

A. The function always returns null<br>
B. The function never accepts arguments<br>
C. The function does not complete normally<br>
D. The function can return any value

**Answer: C.** A **never** function must not return to its caller or reach the end normally. It can throw or terminate execution.

### 9. From which PHP version is **never** available as a return type?

A. PHP 5.6<br>
B. PHP 7.0<br>
C. PHP 8.0<br>
D. PHP 8.1

**Answer: D.** The **never** return type was added in PHP 8.1.

### 10. What does **mixed** accept?

A. Only strings and integers<br>
B. Any value type<br>
C. Only null<br>
D. Only objects

**Answer: B.** **mixed** describes a parameter or return value that may be any type. It is available from PHP 8.0.

### 11. Which statement about coercive scalar typing is correct?

A. It is disabled by default<br>
B. PHP may convert a scalar argument to the declared scalar type when possible<br>
C. It converts every array to a string<br>
D. It validates the application's business rules

**Answer: B.** By default, PHP may coerce scalar arguments when they can be converted to the declared type.

### 12. In a file without strict typing, what happens in this call?

~~~php
function add(int $left, int $right): int
{
    return $left + $right;
}

add("4", 5);
~~~

A. The numeric string can be coerced to an integer<br>
B. The string is always rejected<br>
C. The function returns a string<br>
D. The call is ignored

**Answer: A.** In the default coercive mode, PHP can convert the numeric string **"4"** to an integer for this scalar parameter.

### 13. What does **declare(strict_types=1);** enable?

A. Strict checks for scalar type declarations in that file<br>
B. Strict checks for all variables in every PHP file<br>
C. Automatic validation of all user input<br>
D. A ban on returning values

**Answer: A.** The directive enables strict scalar type checking for code in that file.

### 14. Where should **declare(strict_types=1);** be placed?

A. After the first function call<br>
B. As the first statement in the file, near the opening PHP tag<br>
C. Inside every function body<br>
D. At the end of the file

**Answer: B.** It must be the first statement in the file, before executable code.

### 15. What happens if strict code calls a function requiring **int** with the string **"4"**?

A. The string is always converted to an integer<br>
B. The call throws a **TypeError**<br>
C. The string becomes a float<br>
D. The function returns null

**Answer: B.** Strict scalar checks reject a string argument for an integer parameter, even when the string contains digits.

### 16. Which value can satisfy a **float** parameter even in strict typing mode?

A. An integer<br>
B. Any numeric string<br>
C. A boolean<br>
D. An array

**Answer: A.** Strict mode still permits an integer value for a float declaration.

### 17. Which file's typing mode controls scalar argument checks for a function call?

A. The file where the function is declared<br>
B. The file making the call<br>
C. The file containing the main HTML document<br>
D. The file with the most type declarations

**Answer: B.** Scalar argument checking follows the mode at the call site.

### 18. Which file's typing mode controls a function's scalar return-value check?

A. The file where the function body is defined<br>
B. The file that last called the function<br>
C. The repository's README<br>
D. The file containing the function's first argument

**Answer: A.** Return checks follow the typing mode where the function's return statement is defined.

### 19. What does a file's **strict_types** declaration do to a different included PHP file?

A. It automatically sets that file to strict mode<br>
B. It changes only the mode for its own file<br>
C. It disables all declarations in the included file<br>
D. It converts every value in the included file

**Answer: B.** Strict typing is per file. One file's declaration does not set another file's mode.

### 20. Which type cannot be used as a user-defined parameter or return declaration?

A. **array**<br>
B. **callable**<br>
C. **iterable**<br>
D. **resource**

**Answer: D.** PHP does not support **resource** as a user-defined type declaration.

### 21. Which is the correct scalar type name for a boolean declaration?

A. **boolean**<br>
B. **bool**<br>
C. **logical**<br>
D. **bit**

**Answer: B.** Use **bool**. Aliases such as **boolean**, **integer**, and **double** are not supported as scalar declaration names.

### 22. Does strict typing validate that a string is non-empty and allowed by the application?

A. Yes, strict typing checks every business rule<br>
B. No, a type declaration checks type, not the meaning or allowed contents<br>
C. Yes, but only for a string parameter<br>
D. No, because PHP does not check type declarations

**Answer: B.** A declared **string** can still be empty or invalid for the application. Validate its contents separately.

## True or false

### 23. Type declarations can be added to function parameters and return values.

**Answer: True.** They describe the expected argument types and return values.

### 24. A type declaration guarantees that a value is valid for the application's business rules.

**Answer: False.** It checks the declared type, not constraints such as length, range, format, or authorization.

### 25. Strict typing rejects an integer passed to a **float** parameter.

**Answer: False.** An integer is explicitly accepted for a float declaration, even in strict mode.

### 26. Putting **declare(strict_types=1);** in one file makes the entire project strict.

**Answer: False.** The directive is per file.

### 27. A function declared in a strict file always receives strict argument checks, no matter which file calls it.

**Answer: False.** The calling file determines the scalar argument mode.

### 28. A **void** function may return the string **"done"**.

**Answer: False.** A void function cannot return a value. It can use **return;** without a value.

### 29. A function declared with **never** may reach the closing brace and return normally.

**Answer: False.** A never function must not complete normally.

### 30. **mixed** is a very broad type declaration.

**Answer: True.** It accepts any value type, so use a narrower declaration when the function has a known contract.

### 31. PHP supports **integer** and **boolean** as aliases for scalar type declarations.

**Answer: False.** Use **int** and **bool** as type declaration names.

### 32. A function signature can declare a **resource** parameter directly.

**Answer: False.** Resource values cannot be expressed as a user-defined parameter type declaration.

## Read the code

### 33. What does this code print?

~~~php
function greet(string $name): string
{
    return "Hello, " . $name;
}

echo greet("Asha");
~~~

**Answer:**

~~~text
Hello, Asha
~~~

**Explanation:** The argument and returned value both satisfy their string declarations.

### 34. What does this code print in a file without strict typing?

~~~php
function add(int $left, int $right): int
{
    return $left + $right;
}

var_dump(add("4", 5));
~~~

**Answer:**

~~~text
int(9)
~~~

**Explanation:** The call is coercive by default, so the numeric string is converted to an integer.

### 35. What does this code print in PHP 8 or later?

~~~php
declare(strict_types=1);

function add(int $left, int $right): int
{
    return $left + $right;
}

try {
    add("4", 5);
} catch (TypeError $error) {
    echo "Rejected";
}
~~~

**Answer:**

~~~text
Rejected
~~~

**Explanation:** The strict call passes a string to an integer parameter. PHP throws **TypeError**, which the example catches.

### 36. Does this strict call succeed, and what does it print?

~~~php
declare(strict_types=1);

function showRate(float $rate): void
{
    echo $rate;
}

showRate(5);
~~~

**Answer:** Yes. It prints **5**.

**Explanation:** Strict typing permits an integer for a float parameter.

### 37. What does this code print?

~~~php
function printStatus(): void
{
    echo "Ready";
}

printStatus();
~~~

**Answer:**

~~~text
Ready
~~~

**Explanation:** The function performs an output action and returns no value, which matches **void**.

### 38. Is this a valid implementation of a **never** function?

~~~php
function stopNow(): never
{
    exit();
}
~~~

**Answer: Yes.** The function terminates execution and does not return normally.

**Explanation:** A **never** function must throw, terminate, or otherwise not complete normally.

### 39. If a strict file includes a separate file that has no **strict_types** declaration, does the included file inherit strict mode?

**Answer: No.** The included file retains its own per-file typing mode.

**Explanation:** A file's **strict_types** declaration does not automatically change the included file.

### 40. If a function is defined in a strict file but called from a file without strict typing, which file controls scalar argument coercion?

**Answer:** The calling file.

**Explanation:** The scalar argument mode is determined at the call site, not by the file where the function was declared.

### 41. Which file controls strictness for this function's scalar return value?

~~~php
// functions.php
declare(strict_types=1);

function getCount(): int
{
    return 3.5;
}
~~~

**Answer:** The file containing the function definition and return statement, in this case **functions.php**.

**Explanation:** Scalar return checks use the typing mode where the function is defined.

## Write code

### 42. Write a function that accepts two integers and returns their sum as an integer.

**Answer:**

~~~php
function add(int $left, int $right): int
{
    return $left + $right;
}

echo add(4, 5) . PHP_EOL;
~~~

**Output:**

~~~text
9
~~~

### 43. Write a function that accepts a string and returns a greeting string.

**Answer:**

~~~php
function greet(string $name): string
{
    return "Hello, " . $name;
}

echo greet("Asha") . PHP_EOL;
~~~

**Output:**

~~~text
Hello, Asha
~~~

### 44. Write a **void** function that prints a message.

**Answer:**

~~~php
function showMessage(string $message): void
{
    echo $message . PHP_EOL;
}

showMessage("Saved");
~~~

**Output:**

~~~text
Saved
~~~

### 45. Add strict scalar checking and handle a string passed to an integer parameter.

**Answer:**

~~~php
<?php
declare(strict_types=1);

function printCount(int $count): void
{
    echo $count . PHP_EOL;
}

try {
    printCount("4");
} catch (TypeError $error) {
    echo "Count must be an integer." . PHP_EOL;
}
~~~

**Output:**

~~~text
Count must be an integer.
~~~

**Explanation:** The strict call rejects the string. The caller can convert and validate the input before invoking the function if that is appropriate for the application.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [018 - Type juggling, casting, and conversion](../lessons/018-type-juggling-casting-and-conversion.md) | [Read this lesson](../lessons/019-type-declarations-and-strict-typing.md) | [Open example](../examples/019-type-declarations-and-strict-typing.md) | [020 - Nullable, union, and intersection types](../lessons/020-nullable-union-and-intersection-types.md) |
