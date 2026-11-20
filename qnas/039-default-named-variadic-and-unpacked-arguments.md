# 039 - Default, Named, Variadic, and Unpacked Arguments: Q&A

Review how PHP handles optional defaults, named arguments, variadic parameters, and unpacking. Try each question before reading its answer. See the [chapter lesson](../lessons/039-default-named-variadic-and-unpacked-arguments.md).

## Multiple-choice questions

### 1. When does PHP use a parameter's default value?

A. Whenever the supplied argument is null
B. When the caller omits that argument
C. Only when the function returns
D. Only inside a loop

**Answer:** B. When the caller omits that argument.

**Explanation:** An explicitly supplied value, including null when the type accepts it, is not omission.

### 2. What value does this call pass to **$mode**?

~~~php
function showMode(?string $mode = 'auto'): void
{
    // ...
}

showMode(null);
~~~

A. **'auto'**
B. **null**
C. An empty string in every case
D. No argument

**Answer:** B. **null**.

**Explanation:** The caller explicitly passed null, so PHP does not use the default.

### 3. How should required and defaulted parameters usually be ordered?

A. Defaulted first, required last
B. Required first, defaulted afterward
C. Alphabetically
D. The order never matters

**Answer:** B. Required first, defaulted afterward.

**Explanation:** A required parameter after a defaulted one is deprecated and does not work as an omittable optional parameter.

### 4. Which is a valid simple default value?

A. A local variable that changes at runtime
B. A call to a function
C. An array literal
D. The output of **echo**

**Answer:** C. An array literal.

**Explanation:** Defaults must be constant expressions. Scalar values, arrays, and null are common examples.

### 5. From which PHP version are named arguments available?

A. PHP 5.0
B. PHP 7.0
C. PHP 8.0
D. PHP 8.4 only

**Answer:** C. PHP 8.0.

**Explanation:** Named arguments were introduced in PHP 8.0.

### 6. What syntax supplies a named argument?

A. **name = value**
B. **name: value**
C. **$name => value**
D. **value as name**

**Answer:** B. **name: value**.

**Explanation:** The name must match a parameter name in the function declaration.

### 7. Can named arguments be given in a different order from the declaration?

A. Yes
B. No
C. Only when every parameter is a string
D. Only before PHP 8.0

**Answer:** A. Yes.

**Explanation:** Each named value is matched by its parameter name rather than its position.

### 8. How can a caller skip a defaulted parameter but provide a later option?

A. Leave an empty comma in its place
B. Use a named argument for the later parameter
C. Delete the parameter declaration
D. Put a required parameter after the default

**Answer:** B. Use a named argument for the later parameter.

**Explanation:** Named arguments can select a later defaulted option without supplying earlier defaults.

### 9. When positional and named arguments are combined, where do positional arguments go?

A. After named arguments
B. Before named arguments
C. Either order
D. Positional arguments cannot be used

**Answer:** B. Before named arguments.

**Explanation:** Once a named argument is used, later arguments should also be named.

### 10. What does a variadic parameter do?

A. Returns multiple values
B. Collects remaining arguments into an array
C. Makes every argument optional
D. Names the function dynamically

**Answer:** B. It collects remaining arguments into an array.

**Explanation:** The **...** token before the parameter name marks a variadic parameter.

### 11. Where must a variadic parameter appear?

A. First, and only first
B. Last in the parameter list
C. Outside the function declaration
D. Anywhere, including multiple times

**Answer:** B. Last in the parameter list.

**Explanation:** Ordinary parameters may come first, followed by one variadic parameter.

### 12. What does **...$values** do at a function call?

A. Collects extra values into a parameter
B. Unpacks an array or iterable into arguments
C. Declares a default value
D. Comments out the call

**Answer:** B. It unpacks an array or iterable into arguments.

**Explanation:** At a call site, **...** spreads the iterable's values into the argument list.

### 13. What is the most straightforward array shape for positional argument unpacking?

A. A list with numeric keys
B. An array with nested functions as keys
C. An object with private properties
D. A string

**Answer:** A. A list with numeric keys.

**Explanation:** Its values are supplied in order as positional arguments.

### 14. What can string keys in an unpacked array represent in PHP 8+?

A. Comments
B. Named arguments
C. Return types
D. Variable declarations

**Answer:** B. Named arguments.

**Explanation:** String keys are interpreted as parameter names when unpacked into a call.

### 15. What happens if an argument is supplied twice, once positionally and again by name?

A. PHP averages the values
B. PHP raises an **Error**
C. The function runs both times
D. The named value silently replaces all other values

**Answer:** B. PHP raises an **Error**.

**Explanation:** A parameter cannot receive two values in the same call.

## Short-answer questions

### 16. What is the difference between omitting a parameter and passing null to it?

**Answer:** Omitting it lets PHP use its default. Passing null supplies null as the argument.

**Explanation:** If the parameter is not nullable, the explicit null may fail its type check.

### 17. Write a function declaration with required **$name** and optional **$greeting** defaulting to **'Hi'**.

**Answer:**

~~~php
function greet(string $name, string $greeting = 'Hi'): void
{
    echo "{$greeting}, {$name}!\n";
}
~~~

**Explanation:** The required parameter comes before the defaulted one.

### 18. Why should required parameters come before defaulted parameters?

**Answer:** So callers can omit optional arguments without skipping a required one.

**Explanation:** A defaulted parameter before a required one is deprecated and does not provide the intended optional behavior in modern PHP.

### 19. What kind of expression can be used as a default parameter value?

**Answer:** A constant expression, such as a literal, array, or null.

**Explanation:** A default cannot depend on a runtime local variable or function call.

### 20. Since which PHP version can an object be created in a default parameter value using **new ClassName()**?

**Answer:** PHP 8.1.

**Explanation:** Earlier PHP versions did not support object creation in parameter defaults.

### 21. What is the named-argument syntax for passing **'Asha'** to a parameter named **$name**?

**Answer:** **name: 'Asha'**

**Explanation:** The variable marker **$** is not part of the name at the call site.

### 22. Does named-argument order have to match the declaration order?

**Answer:** No.

**Explanation:** PHP matches each named argument by parameter name.

### 23. Why can renaming a public function parameter be a compatibility change?

**Answer:** Callers using named arguments refer to the parameter by its name.

**Explanation:** If that name changes, their named calls may no longer match the function declaration.

### 24. What order should be used when combining positional and named arguments?

**Answer:** Put positional arguments first, then named arguments.

**Explanation:** Do not place positional arguments after a named argument.

### 25. How many variadic parameters can a function declare, and where must it appear?

**Answer:** One, and it must be last.

**Explanation:** It collects any remaining arguments after earlier parameters.

### 26. What does a variadic parameter contain?

**Answer:** An array of the arguments collected by the variadic parameter.

**Explanation:** A typed variadic parameter checks each collected argument against its declared type.

### 27. Can a variadic parameter receive zero arguments?

**Answer:** Yes.

**Explanation:** It collects zero or more trailing arguments, so its array may be empty.

### 28. Explain the difference between **function collect(string ...$items)** and **collect(...$items)**.

**Answer:** The declaration syntax collects arguments into **$items**. The call syntax expands an array or iterable into separate arguments.

**Explanation:** The same token has different roles depending on whether it appears in a declaration or a call.

### 29. What is a simple use case for unpacking?

**Answer:** Supplying the values in a list to a function whose parameters correspond to those values.

**Explanation:** A numeric-keyed list spreads values positionally and in order.

## True or false

### 30. A default value is used when the caller explicitly passes null.

**Answer:** False.

**Explanation:** Defaults apply when an argument is omitted, not when null is supplied.

### 31. A required parameter should be placed after parameters with defaults.

**Answer:** False.

**Explanation:** Required parameters should come first.

### 32. Named arguments have been supported since PHP 8.0.

**Answer:** True.

**Explanation:** This is the minimum PHP version for named argument syntax.

### 33. Named arguments must be written in the same order as the function declaration.

**Answer:** False.

**Explanation:** Names identify the parameters, so named arguments can be reordered.

### 34. A caller may put a positional argument after a named argument.

**Answer:** False.

**Explanation:** Use positional arguments first and named arguments afterward.

### 35. A variadic parameter stores the extra arguments in an array.

**Answer:** True.

**Explanation:** The values are collected into the variadic parameter's array.

### 36. A function may declare multiple variadic parameters.

**Answer:** False.

**Explanation:** Only one variadic parameter is allowed, and it must be last.

### 37. The **...** syntax means the same thing in a parameter declaration and a function call.

**Answer:** False.

**Explanation:** In a declaration it collects; at a call it unpacks.

## Predict the output

### 38. What is printed?

~~~php
<?php
function showMode(?string $mode = 'auto'): void
{
    var_dump($mode);
}

showMode();
showMode(null);
~~~

**Answer:**

~~~text
string(4) "auto"
NULL
~~~

**Explanation:** The first call omits the value and uses the default. The second passes null explicitly.

### 39. What is printed?

~~~php
<?php
function describe(string $name, string $role = 'Learner'): void
{
    echo "{$name}: {$role}\n";
}

describe(role: 'Teacher', name: 'Mina');
~~~

**Answer:**

~~~text
Mina: Teacher
~~~

**Explanation:** Named arguments are matched by parameter names, not their order in the call.

### 40. What is printed?

~~~php
<?php
function describe(string $name, string $role = 'Learner', bool $active = true): void
{
    echo "{$name}: {$role}; active=" . ($active ? 'yes' : 'no') . "\n";
}

describe('Asha', active: false);
~~~

**Answer:**

~~~text
Asha: Learner; active=no
~~~

**Explanation:** The positional argument sets **$name**. **$role** is omitted and uses its default; the named argument sets **$active**.

### 41. What is printed?

~~~php
<?php
function showItems(string ...$items): void
{
    echo count($items), "\n";
    foreach ($items as $item) {
        echo $item, "\n";
    }
}

showItems('Book', 'Pen');
~~~

**Answer:**

~~~text
2
Book
Pen
~~~

**Explanation:** The variadic parameter collects both trailing arguments into an array.

### 42. What is printed?

~~~php
<?php
function showPoint(int $x, int $y): void
{
    echo "{$x},{$y}\n";
}

$point = [4, 9];
showPoint(...$point);
~~~

**Answer:**

~~~text
4,9
~~~

**Explanation:** The array is unpacked into two positional arguments.

## Write-code questions

### 43. Define **greet()** with required **$name** and a default **$greeting** of **'Hello'**. Call it once with the default and once with **'Welcome'**.

**Answer:**

~~~php
<?php
function greet(string $name, string $greeting = 'Hello'): void
{
    echo "{$greeting}, {$name}!\n";
}

greet('Asha');
greet('Mina', 'Welcome');
~~~

**Explanation:** The first call omits **$greeting**; the second overrides its default.

### 44. Define a variadic **printTags()** function that accepts a required title followed by any number of string tags. Call it with two tags.

**Answer:**

~~~php
<?php
function printTags(string $title, string ...$tags): void
{
    echo "{$title}\n";

    foreach ($tags as $tag) {
        echo "- {$tag}\n";
    }
}

printTags('Topics', 'PHP', 'Functions');
~~~

**Explanation:** The first argument maps to **$title**; remaining arguments collect into **$tags**.

### 45. Define a function with two integer parameters, then call it by unpacking **[12, 7]**.

**Answer:**

~~~php
<?php
function printDimensions(int $width, int $height): void
{
    echo "{$width} x {$height}\n";
}

$dimensions = [12, 7];
printDimensions(...$dimensions);
~~~

**Explanation:** The array values are unpacked from left to right and assigned to the corresponding parameters.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [038 - Function Parameters and Arguments](../lessons/038-function-parameters-and-arguments.md) | [Read lesson](../lessons/039-default-named-variadic-and-unpacked-arguments.md) | [Open example](../examples/039-default-named-variadic-and-unpacked-arguments.md) | [040 - Function Return Types](../lessons/040-function-return-types.md) |
