# 041 - Anonymous Functions and Closures: Q&A

Review anonymous function syntax, calling closure variables, callbacks, and capturing variables with **use**. Try each question before reading its answer. See the [chapter lesson](../lessons/041-anonymous-functions-and-closures.md).

## Multiple-choice questions

### 1. What is an anonymous function?

A. A function without a declared name
B. A function that cannot accept parameters
C. A function that always exits the script
D. A function with no body

**Answer:** A. A function without a declared name.

**Explanation:** It can still have parameters, a body, a return type, and a return value.

### 2. What PHP object represents an anonymous function value?

A. **Array**
B. **Closure**
C. **Generator**
D. **Iterator**

**Answer:** B. **Closure**.

**Explanation:** PHP represents anonymous function expressions as instances of the internal **Closure** class.

### 3. Which syntax assigns an anonymous function to **$greet**?

A. **$greet = function (...) { ... };**
B. **function $greet (...) { ... }**
C. **$greet := function (...)**
D. **closure $greet (...)**

**Answer:** A. **$greet = function (...) { ... };**

**Explanation:** It is a function expression assigned like another value.

### 4. How is an assigned closure called?

A. **run $greet**
B. **$greet()**
C. **call($greet)** only
D. **function($greet)**

**Answer:** B. **$greet()**.

**Explanation:** A closure variable can be invoked with the usual function-call parentheses.

### 5. What is a callback?

A. A function supplied to other code to be called at an appropriate time
B. A PHP comment
C. A class constant
D. A required return type

**Answer:** A. A function supplied to other code to be called at an appropriate time.

**Explanation:** Anonymous functions are commonly passed as callback values.

### 6. How does a regular closure access an ordinary variable from its surrounding scope?

A. It automatically sees every variable
B. It imports the variable with **use**
C. It declares it as a parameter without a call
D. It uses **global** by default

**Answer:** B. It imports the variable with **use**.

**Explanation:** Regular closures do not automatically capture surrounding ordinary variables.

### 7. What does **use ($message)** do?

A. Captures the variable by value
B. Captures it by reference
C. Turns it into a parameter
D. Prints it

**Answer:** A. It captures the variable by value.

**Explanation:** The closure gets the value that was present when the closure was created.

### 8. What does **use (&$count)** do?

A. Hides the variable from the outer scope
B. Shares the variable by reference
C. Declares a return type
D. Makes the closure static

**Answer:** B. It shares the variable by reference.

**Explanation:** Changes made through the closure affect the surrounding variable.

### 9. Where does a closure's return type appear if it has a **use** clause?

A. Before **function**
B. Between the parameters and **use**
C. After the **use** clause
D. After the body

**Answer:** C. After the **use** clause.

**Explanation:** For example: **function () use ($value): int { ... }**.

### 10. What is true of a closure's regular parameter?

A. It receives a value from the call
B. It is automatically imported by **use**
C. It can only be a global
D. It is the closure's return type

**Answer:** A. It receives a value from the call.

**Explanation:** Parameters receive call arguments; **use** captures values from the surrounding scope.

### 11. What does **array_map()** do with a callback in the lesson example?

A. Calls it for input values and collects the results
B. Turns it into a named function
C. Stops the script
D. Captures every outer variable

**Answer:** A. It calls the callback for input values and collects the results.

**Explanation:** The callback defines the transformation applied to each value.

### 12. What is required after assigning a closure to a variable?

A. A semicolon ending the assignment
B. A second function name
C. A **use** clause every time
D. An **exit** statement

**Answer:** A. A semicolon ending the assignment.

**Explanation:** The entire assignment statement ends with a semicolon after the closure body.

### 13. Does changing an outer scalar after **use ($value)** normally change the closure's captured value?

A. Yes, immediately
B. No, the value was captured when the closure was created
C. Only if the closure returns an integer
D. Only on PHP 5

**Answer:** B. No.

**Explanation:** By-value capture retains the value present at closure creation.

### 14. What is a consequence of **use (&$value)**?

A. The closure and outer scope share the variable
B. The closure receives a copy that can never change
C. The closure becomes an arrow function
D. The value becomes a class constant

**Answer:** A. The closure and outer scope share the variable.

**Explanation:** Updates on either side are visible through the shared reference.

### 15. What is the next chapter about?

A. Arrow functions
B. Database transactions
C. HTML forms
D. CSS selectors

**Answer:** A. Arrow functions.

**Explanation:** Arrow functions provide shorter syntax and have their own capture behavior.

## Short-answer questions

### 16. Why might you use an anonymous function instead of a named function?

**Answer:** It can keep a short piece of behavior next to the place that uses it, or provide a callback value.

**Explanation:** Give the behavior a reusable name when it has a clear purpose or is used in several places.

### 17. What is the basic keyword syntax for an anonymous function?

**Answer:** **function (parameters) { body }**

**Explanation:** It has no declared name, but otherwise supports parameters, a body, and a return type.

### 18. Why does this assignment end with a semicolon?

~~~php
$format = function (string $value): string {
    return strtoupper($value);
};
~~~

**Answer:** The closure is an expression assigned to **$format**, and the semicolon ends the assignment statement.

**Explanation:** The closing brace ends the function body; the semicolon ends the surrounding statement.

### 19. How do you call a closure stored in **$format**?

**Answer:** Call it as **$format('text')**.

**Explanation:** The variable contains a callable closure value.

### 20. What is a callback in simple terms?

**Answer:** A function value passed to other code so that code can call it.

**Explanation:** A closure passed to a collection function can define how each item is processed.

### 21. Can a closure have normal parameters as well as captured variables?

**Answer:** Yes.

**Explanation:** Parameters receive values each time it is called; captured variables come from its surrounding scope.

### 22. Why does a regular closure need a **use** clause for an outer local variable?

**Answer:** The closure has its own scope and does not automatically inherit ordinary local variables.

**Explanation:** **use** explicitly identifies the variables it needs from the surrounding scope.

### 23. When is a variable captured by value?

**Answer:** When the closure is created.

**Explanation:** Later reassignment of the outer variable does not replace the captured scalar value.

### 24. What changes when a variable is captured by reference?

**Answer:** The closure and surrounding scope share the same variable, so changes are visible to both.

**Explanation:** **use (&$value)** explicitly requests the shared reference.

### 25. Where should a return type be written on a closure with a **use** clause?

**Answer:** After the **use** clause and before the opening brace.

**Explanation:** Example: **function () use ($value): string { ... }**.

### 26. Why should capture by reference be used deliberately?

**Answer:** It allows the closure to mutate state outside itself.

**Explanation:** Shared mutable state can make behavior harder to follow if it is not intentional.

### 27. Can an anonymous function be passed directly to a callback-taking function without first assigning it to a variable?

**Answer:** Yes.

**Explanation:** It can be written inline when the callback is short and clear at that call site.

### 28. What does **array_map()** do with the returned value of each callback call?

**Answer:** It collects those values into the resulting array.

**Explanation:** Each callback result corresponds to an input element.

### 29. What does **static function** change when an anonymous function is declared in an object method?

**Answer:** It prevents the closure from automatically binding the current **$this** object.

**Explanation:** This is useful when the closure should not retain that object context.

## True or false

### 30. An anonymous function has no declared name.

**Answer:** True.

**Explanation:** It can still be assigned to a variable and called through that variable.

### 31. A regular closure automatically sees every variable in the surrounding scope.

**Answer:** False.

**Explanation:** Use **use** to capture ordinary surrounding variables.

### 32. **use ($value)** captures by reference.

**Answer:** False.

**Explanation:** It captures by value. Add **&** to capture by reference.

### 33. A closure assigned to a variable can be invoked with parentheses.

**Answer:** True.

**Explanation:** For example, **$greet('Asha')**.

### 34. A regular closure's return type appears before its **use** clause.

**Answer:** False.

**Explanation:** The return type follows **use**.

### 35. A closure may accept ordinary parameters even when it also captures variables.

**Answer:** True.

**Explanation:** Parameters and captures provide inputs in different ways.

### 36. Changing the outer scalar after by-value capture normally changes the value already captured.

**Answer:** False.

**Explanation:** The closure retains the value from its creation time.

### 37. A static anonymous function automatically receives **$this** from its object context.

**Answer:** False.

**Explanation:** Static anonymous functions do not receive that automatic **$this** binding.

## Predict the output

### 38. What is printed?

~~~php
<?php
$double = function (int $number): int {
    return $number * 2;
};

echo $double(6), "\n";
~~~

**Answer:**

~~~text
12
~~~

**Explanation:** The closure receives **6**, returns **12**, and **echo** prints it.

### 39. What is printed?

~~~php
<?php
$prefix = 'Hello';

$greet = function (string $name) use ($prefix): string {
    return "{$prefix}, {$name}";
};

$prefix = 'Welcome';

echo $greet('Asha'), "\n";
~~~

**Answer:**

~~~text
Hello, Asha
~~~

**Explanation:** **$prefix** was captured by value when the closure was created.

### 40. What is printed?

~~~php
<?php
$count = 0;

$next = function () use (&$count): int {
    $count++;
    return $count;
};

echo $next(), "\n";
echo $next(), "\n";
~~~

**Answer:**

~~~text
1
2
~~~

**Explanation:** The closure updates the shared **$count** variable on each call.

### 41. What is printed?

~~~php
<?php
$numbers = [1, 2, 3];

$squares = array_map(
    function (int $number): int {
        return $number * $number;
    },
    $numbers,
);

echo implode(', ', $squares), "\n";
~~~

**Answer:**

~~~text
1, 4, 9
~~~

**Explanation:** The callback squares each input value, and **array_map()** collects the results.

### 42. What is printed?

~~~php
<?php
$base = 10;

$addBase = function (int $value) use ($base): int {
    return $base + $value;
};

echo $addBase(5), "\n";
~~~

**Answer:**

~~~text
15
~~~

**Explanation:** The closure captures **$base** and adds it to the call's parameter.

## Write-code questions

### 43. Assign a closure to **$greet** that takes a name, returns **Hello, NAME!**, and call it with **Mina**.

**Answer:**

~~~php
<?php
$greet = function (string $name): string {
    return "Hello, {$name}!";
};

echo $greet('Mina'), "\n";
~~~

**Explanation:** The closure is stored in a variable and its returned string is printed by the caller.

### 44. Use a closure with **array_map()** to add five to each value in **[1, 2, 3]**.

**Answer:**

~~~php
<?php
$values = [1, 2, 3];

$increased = array_map(
    function (int $value): int {
        return $value + 5;
    },
    $values,
);

echo implode(', ', $increased), "\n";
~~~

Output:

~~~text
6, 7, 8
~~~

**Explanation:** The callback runs once per array value and returns the transformed value.

### 45. Capture a counter by reference, increment it each time the closure runs, and call the closure twice.

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

**Explanation:** Both the closure and outer scope share the same counter variable.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [040 - Function Return Types](../lessons/040-function-return-types.md) | [Read lesson](../lessons/041-anonymous-functions-and-closures.md) | [Open example](../examples/041-anonymous-functions-and-closures.md) | [042 - Arrow Functions](../lessons/042-arrow-functions.md) |
