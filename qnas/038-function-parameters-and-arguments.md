# 038 - Function Parameters and Arguments: Q&A

Use these questions to review PHP parameters, arguments, positional matching, local scope, and the default by-value behavior. Try each question before reading its answer. See the [chapter lesson](../lessons/038-function-parameters-and-arguments.md).

## Multiple-choice questions

### 1. Where is a function parameter declared?

A. In the function signature
B. Only in the function call
C. In a comment
D. In the return statement

**Answer:** A. In the function signature.

**Explanation:** The parameter is the named variable listed between the declaration's parentheses.

### 2. What is an argument?

A. The variable declared by the function
B. The value or expression supplied in a function call
C. The braces around a function
D. The function's return type

**Answer:** B. The value or expression supplied in a function call.

**Explanation:** The caller provides an argument, and PHP assigns it to the corresponding parameter.

### 3. In this call, what is **'Asha'**?

~~~php
greet('Asha');
~~~

A. A parameter
B. An argument
C. A return type
D. A function declaration

**Answer:** B. An argument.

**Explanation:** It is the value provided at the call site.

### 4. In a positional call, how does PHP match arguments to parameters?

A. Alphabetically by variable name
B. From left to right by position
C. In reverse order
D. By the length of each value

**Answer:** B. From left to right by position.

**Explanation:** The first argument maps to the first parameter, the second to the second, and so on.

### 5. Where is a parameter variable available?

A. Automatically throughout every PHP file
B. Inside the function body for that call
C. Only in the function declaration
D. Only after the script exits

**Answer:** B. Inside the function body for that call.

**Explanation:** Parameters are local variables of the function.

### 6. What is PHP's default argument-passing behavior?

A. By reference
B. By value
C. By global variable
D. By return type

**Answer:** B. By value.

**Explanation:** A parameter receives a value. Reassigning a scalar parameter does not reassign the caller's variable.

### 7. If a function requires two arguments and the caller omits one, what does PHP raise?

A. **ArgumentCountError**
B. **ParseWarning**
C. **UndefinedParameterNotice**
D. Nothing; PHP invents a value

**Answer:** A. **ArgumentCountError**.

**Explanation:** A required parameter must be supplied. Defaults and optional parameters are covered separately.

### 8. What can happen if an argument is not accepted by a declared parameter type?

A. PHP can raise a **TypeError**
B. PHP always renames the parameter
C. The function is silently deleted
D. The argument becomes a comment

**Answer:** A. PHP can raise a **TypeError**.

**Explanation:** Scalar coercion rules depend on strict typing in the calling file, but incompatible values can produce a type error.

### 9. Which call maps **'Mug'** to **$name** and **2** to **$quantity**?

~~~php
function printItem(string $name, int $quantity): void
{
    // ...
}
~~~

A. **printItem(2, 'Mug');**
B. **printItem('Mug', 2);**
C. **printItem(name, quantity);**
D. **printItem();**

**Answer:** B. **printItem('Mug', 2);**

**Explanation:** Positional values follow the declaration's order.

### 10. Can an argument be an expression instead of a literal?

A. Yes
B. No, only strings are allowed
C. Only in a function declaration
D. Only if the function returns **void**

**Answer:** A. Yes.

**Explanation:** A caller may pass a variable, literal, or expression whose value is accepted by the parameter.

### 11. In **function addTax(float $price): void**, what is **float**?

A. An argument
B. A parameter type declaration
C. A function call
D. A return value

**Answer:** B. A parameter type declaration.

**Explanation:** It declares the kind of value the parameter is intended to receive.

### 12. What does a parameter type declaration do?

A. Displays the value
B. Documents and checks the accepted parameter type
C. Makes the parameter global
D. Automatically returns the parameter

**Answer:** B. It documents and checks the accepted parameter type.

**Explanation:** PHP checks arguments against declared types. Scalar coercion depends on strict typing.

### 13. If a function reassigns a scalar parameter, what normally happens to the caller's variable?

A. It is automatically reassigned too
B. It keeps its original value
C. It becomes a function
D. It is deleted

**Answer:** B. It keeps its original value.

**Explanation:** By-value passing is the default; assignment to the local parameter does not reassign the caller's variable.

### 14. When are argument expressions evaluated?

A. From right to left after the function body
B. From left to right before the function body runs
C. Only when printed
D. Never; PHP ignores expressions

**Answer:** B. From left to right before the function body runs.

**Explanation:** PHP evaluates the expressions in the call and assigns their resulting values to the parameters before entering the function.

### 15. Which feature is the subject of the next chapter?

A. Optional parameters, named arguments, variadic arguments, and unpacking
B. HTML headings
C. Database indexes
D. CSS selectors

**Answer:** A. Optional parameters, named arguments, variadic arguments, and unpacking.

**Explanation:** This chapter focuses on basic required positional parameters and arguments.

## Short-answer questions

### 16. Explain the difference between a parameter and an argument.

**Answer:** A parameter is the receiving variable declared in a function signature. An argument is the value or expression supplied by a caller.

**Explanation:** For **function greet(string $name)**, **$name** is the parameter. In **greet('Asha')**, **'Asha'** is the argument.

### 17. In this call, which argument maps to each parameter?

~~~php
function describe(string $title, int $year): void
{
    // ...
}

describe('My Book', 2025);
~~~

**Answer:** **'My Book'** maps to **$title**; **2025** maps to **$year**.

**Explanation:** Positional arguments are matched left to right.

### 18. What does it mean that a parameter is local to a function?

**Answer:** The parameter variable is available inside the function body, not automatically in the caller's scope.

**Explanation:** The caller passes a value; this does not merge the caller's variable scope with the function's local scope.

### 19. If the caller's variable and a parameter use the same variable name, are they the same local variable?

**Answer:** No. They belong to different scopes.

**Explanation:** The argument value initializes the function's parameter; the local parameter is not the caller's variable.

### 20. What is the default way PHP passes an argument?

**Answer:** By value.

**Explanation:** Reassigning a scalar parameter changes the local parameter, not the caller's scalar variable.

### 21. What happens when a required argument is omitted?

**Answer:** PHP raises an **ArgumentCountError**.

**Explanation:** A parameter without a default requires the caller to supply an argument.

### 22. What error can occur if an argument is incompatible with the declared parameter type?

**Answer:** A **TypeError** can occur.

**Explanation:** PHP may coerce compatible scalar values in weak typing mode. Strict typing changes scalar coercion behavior.

### 23. Can a function call use a calculation as an argument?

**Answer:** Yes. For example, **showScore(4 + 5)** passes the result of the expression.

**Explanation:** Argument expressions are evaluated before the function body runs.

### 24. If a function takes **$first** and **$second**, what risk comes from reversing positional arguments?

**Answer:** Each value is assigned to the wrong parameter, possibly causing incorrect results or a type error.

**Explanation:** Position, not the caller's variable name, determines the mapping.

### 25. What information does a parameter type declaration provide?

**Answer:** It documents the expected kind of value and lets PHP check the argument.

**Explanation:** For example, **int $quantity** declares an integer parameter. Scalar coercion still depends on strict typing.

### 26. Does passing an argument to a parameter automatically return a result to the caller?

**Answer:** No.

**Explanation:** Parameters pass input into a function. Returning output is a separate operation.

### 27. Why is it often clearer to pass a function the values it needs as parameters?

**Answer:** Its dependencies are visible at the call site and it does not need hidden caller state.

**Explanation:** Explicit inputs make a function easier to understand and reuse.

### 28. What should you do if a function needs an optional argument?

**Answer:** Learn and use a default parameter value, covered in chapter 039.

**Explanation:** Do not leave a required argument out and expect PHP to invent a value.

### 29. What is the difference between a parameter's type and a function's return type?

**Answer:** A parameter type describes an input value. A return type describes the function's output value.

**Explanation:** They appear in different parts of a signature and serve different purposes.

## True or false

### 30. A parameter appears in the function declaration.

**Answer:** True.

**Explanation:** It is listed between the declaration's parentheses.

### 31. An argument is supplied in the function call.

**Answer:** True.

**Explanation:** The caller provides values or expressions for the function's parameters.

### 32. Positional arguments are matched from right to left.

**Answer:** False.

**Explanation:** PHP matches them from left to right.

### 33. A parameter is automatically available in the caller's scope.

**Answer:** False.

**Explanation:** The parameter is local to the function call.

### 34. PHP passes arguments by reference by default.

**Answer:** False.

**Explanation:** The default is by value. Reference parameters must be explicitly declared.

### 35. Reassigning an integer parameter normally reassigns the caller's integer variable.

**Answer:** False.

**Explanation:** The parameter is a local variable initialized with the argument's value.

### 36. Argument expressions are evaluated before the function body begins.

**Answer:** True.

**Explanation:** PHP evaluates argument expressions left to right, then assigns the results to parameters.

### 37. A missing required argument causes PHP to invent a default value.

**Answer:** False.

**Explanation:** Without a declared default, the argument is required; omitting it raises **ArgumentCountError**.

## Predict the output

### 38. What is printed?

~~~php
<?php
function greet(string $name): void
{
    echo "Hello, {$name}\n";
}

greet('Asha');
~~~

**Answer:**

~~~text
Hello, Asha
~~~

**Explanation:** The string argument is assigned to **$name**, then the function prints it.

### 39. What is printed?

~~~php
<?php
function showPair(string $first, string $second): void
{
    echo "{$first}-{$second}\n";
}

showPair('left', 'right');
~~~

**Answer:**

~~~text
left-right
~~~

**Explanation:** The first and second arguments map to **$first** and **$second** in order.

### 40. What is printed?

~~~php
<?php
function showNext(int $score): void
{
    echo $score + 1, "\n";
}

$current = 8;
showNext($current);
~~~

**Answer:**

~~~text
9
~~~

**Explanation:** The argument is the value of **$current**, which is assigned to the local parameter **$score**.

### 41. What is printed?

~~~php
<?php
function changeLocal(int $value): void
{
    $value = 99;
    echo "Inside: {$value}\n";
}

$number = 4;
changeLocal($number);
echo "Caller: {$number}\n";
~~~

**Answer:**

~~~text
Inside: 99
Caller: 4
~~~

**Explanation:** Reassigning the local parameter does not reassign the caller's scalar variable.

### 42. What happens when this code runs?

~~~php
<?php
function printPair(string $first, string $second): void
{
    echo "{$first} {$second}\n";
}

printPair('only one');
~~~

**Answer:** PHP raises an **ArgumentCountError** because the required **$second** argument was omitted.

**Explanation:** Both parameters are required because neither has a default value.

## Write-code questions

### 43. Define **greet(string $name)** to print **Hello, NAME!**, then call it with **Mina**.

**Answer:**

~~~php
<?php
function greet(string $name): void
{
    echo "Hello, {$name}!\n";
}

greet('Mina');
~~~

**Explanation:** **$name** is the parameter, and **'Mina'** is the argument.

### 44. Define **printProduct(string $name, int $quantity)** to print both values, then call it for three notebooks.

**Answer:**

~~~php
<?php
function printProduct(string $name, int $quantity): void
{
    echo "{$name}: {$quantity}\n";
}

printProduct('Notebook', 3);
~~~

**Explanation:** The first positional argument maps to **$name** and the second to **$quantity**.

### 45. Write a function that accepts an integer score, prints the score plus one, and show that the caller's variable remains unchanged.

**Answer:**

~~~php
<?php
function printNextScore(int $score): void
{
    $score = $score + 1;
    echo "Inside: {$score}\n";
}

$currentScore = 10;
printNextScore($currentScore);
echo "Caller: {$currentScore}\n";
~~~

Output:

~~~text
Inside: 11
Caller: 10
~~~

**Explanation:** The function receives the value by default. Updating the local parameter does not reassign the caller's variable.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [037 - Defining and Calling Functions](../lessons/037-defining-and-calling-functions.md) | [Read lesson](../lessons/038-function-parameters-and-arguments.md) | [Open example](../examples/038-function-parameters-and-arguments.md) | [039 - Default, Named, Variadic, and Unpacked Arguments](../lessons/039-default-named-variadic-and-unpacked-arguments.md) |
