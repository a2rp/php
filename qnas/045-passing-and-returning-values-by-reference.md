# 045 - Passing and Returning Values by Reference: Q&A

Review PHP's default pass-by-value behavior, reference parameters, reference returns, and the difference between objects and variable references. Try each question before reading its answer. See the [chapter lesson](../lessons/045-passing-and-returning-values-by-reference.md).

## Multiple-choice questions

### 1. How are function arguments passed by default in PHP?

A. By reference
B. By value
C. As global variables
D. As constants

**Answer:** B. By value.

**Explanation:** A parameter normally receives the argument's value. Changing the parameter does not replace the caller's variable.

### 2. What must a function declaration use if a parameter should refer to the caller's variable?

A. **$&value**
B. **&$value**
C. **ref $value**
D. **global $value**

**Answer:** B. **&$value**.

**Explanation:** Put the ampersand before the parameter variable in the function declaration.

### 3. Where is **&** written for a by-reference parameter?

A. Only on the function parameter declaration
B. Only at the call site
C. Both at the declaration and call site
D. Nowhere; PHP infers it

**Answer:** A. Only on the function parameter declaration.

**Explanation:** A call such as **increment($count)** does not include an ampersand. The declaration specifies the reference behavior.

### 4. Which argument is appropriate for a parameter declared **int &$value**?

A. **7**
B. **$count**
C. **$count + 1**
D. **10 * 2**

**Answer:** B. **$count**.

**Explanation:** A by-reference parameter needs a variable that can be referenced and updated, not a literal or calculated expression.

### 5. Which declaration returns a reference?

A. **function getValue() &**
B. **function &getValue()**
C. **function getValue(&$)**
D. **function ref getValue()**

**Answer:** B. **function &getValue()**.

**Explanation:** Place **&** before the function name to declare a reference return.

### 6. How does the caller bind to a reference returned by a function?

A. **$value = &getValue()**
B. **$value =& getValue()**
C. **&$value = getValue()**
D. **$value == getValue()**

**Answer:** B. **$value =& getValue()**.

**Explanation:** Use **=&** to bind the caller's variable to the returned reference.

### 7. What does **$copy = getValue()** do when **getValue()** returns by reference?

A. It always binds **$copy** as a reference.
B. It assigns the returned value to **$copy**.
C. It causes a parse error.
D. It makes **$copy** global.

**Answer:** B. It assigns the returned value to **$copy**.

**Explanation:** To bind to the reference itself, write **$copy =& getValue()**.

### 8. Can a function that returns by reference return a calculated expression such as **$value + 1** as the reference?

A. Yes, every expression is a reference.
B. No, it must return a variable that can be referenced.
C. Only if the expression is an integer.
D. Only if the caller uses **var_dump()**.

**Answer:** B. No, it must return a variable that can be referenced.

**Explanation:** A calculated result is a value, not an alias to a variable.

### 9. What happens when a by-value object parameter changes a property on the object?

A. The property change can be visible to the caller.
B. The caller's object is always cloned first.
C. PHP converts the parameter to a reference automatically.
D. The property change is forbidden.

**Answer:** A. The property change can be visible to the caller.

**Explanation:** The parameter and caller refer to the same object, even though the variable itself was passed by value.

### 10. What does assigning a new object to a by-value object parameter do?

A. It replaces the caller's object variable.
B. It only reassigns the local parameter.
C. It changes the class definition.
D. It always throws a TypeError.

**Answer:** B. It only reassigns the local parameter.

**Explanation:** Use a by-reference parameter if the function should replace the caller's variable itself.

### 11. What should you do after a **foreach ($items as &$item)** loop?

A. Call **unset($item)**.
B. Call **global $item**.
C. Call **unset($items)**.
D. Nothing; the reference is automatically cleared.

**Answer:** A. Call **unset($item)**.

**Explanation:** The loop variable remains bound to the final array element after the loop. Unsetting that variable breaks the lingering binding.

### 12. What is **$alias =& $original**?

A. A normal value copy
B. A reference assignment that makes both names refer to the same value
C. A comparison
D. A declaration of a function

**Answer:** B. A reference assignment that makes both names refer to the same value.

**Explanation:** Changing the value through either variable affects the shared reference.

### 13. Which statement about references and performance is correct?

A. References should be used to make every function faster.
B. Returning by reference is required for large arrays.
C. References should be used for a real shared-modification requirement, not as a performance shortcut.
D. Every parameter should be passed by reference.

**Answer:** C. References should be used for a real shared-modification requirement, not as a performance shortcut.

**Explanation:** PHP handles normal value passing efficiently; use references for their semantics.

### 14. Which code modifies the caller's scalar variable?

A. **function set($n) { $n = 2; }**
B. **function set(&$n) { $n = 2; }**
C. **function set() { $n = 2; }**
D. **function &set($n) { return 2; }**

**Answer:** B. **function set(&$n) { $n = 2; }**

**Explanation:** The parameter must be declared by reference to update the caller's variable.

### 15. What is the main reason to return a reference?

A. To make every return faster
B. To let the caller bind to and modify a specific variable
C. To make a function return multiple values automatically
D. To turn a value into a class

**Answer:** B. To let the caller bind to and modify a specific variable.

**Explanation:** A reference return is useful when the caller needs an alias to an existing variable.

## Short-answer questions

### 16. What is the default behavior when a scalar argument is passed to a function?

**Answer:** It is passed by value.

**Explanation:** Reassigning the parameter changes the local parameter, not the caller's variable.

### 17. Write a function declaration that increments its caller's integer variable.

**Answer:**

~~~php
function increment(int &$number): void
{
    $number++;
}
~~~

**Explanation:** The ampersand belongs on the parameter in the declaration.

### 18. How do you call **increment()** if its parameter is by reference?

**Answer:**

~~~php
increment($count);
~~~

**Explanation:** Do not add **&** at the call site.

### 19. Why can a literal not be supplied to a by-reference parameter?

**Answer:** A literal has no variable storage for the function to refer to and change.

**Explanation:** Pass a variable or another supported variable target instead.

### 20. Can an array element be passed to a by-reference parameter?

**Answer:** Yes, an array element is a variable target that can be updated.

**Explanation:** For example, **increment($values[0])** changes that element.

### 21. How do you declare a function that returns a reference?

**Answer:**

~~~php
function &getValue()
{
    // Return a variable.
}
~~~

**Explanation:** Put **&** before the function name.

### 22. How do you bind a variable to the reference returned by **getValue()**?

**Answer:**

~~~php
$value =& getValue();
~~~

**Explanation:** The caller uses **=&** to receive a reference binding instead of just a value.

### 23. What is the difference between **$copy = $original** and **$alias =& $original**?

**Answer:** The first assigns a value. The second makes both variables refer to the same value.

**Explanation:** Changes to **$alias** affect the shared reference, while changes to an ordinary copy do not change **$original**.

### 24. Why is a static variable useful in a reference-return example?

**Answer:** It persists after the function call, so the returned reference continues to refer to existing storage.

**Explanation:** It makes the shared variable's lifetime clear and predictable.

### 25. What does **function &getScore()** promise?

**Answer:** The function returns a reference to a variable rather than only returning a value.

**Explanation:** The caller still needs **=&** if it wants to bind to that reference.

### 26. What changes when an object property is modified inside a by-value object parameter?

**Answer:** The object property can change for the caller because both variables refer to the same object.

**Explanation:** The parameter variable itself was passed by value; object property mutation and variable reassignment are different.

### 27. What happens if a new object is assigned to a by-value object parameter?

**Answer:** Only the local parameter is reassigned.

**Explanation:** The caller's variable still refers to its original object.

### 28. Why should a by-reference **foreach** loop call **unset($item)** after the loop?

**Answer:** The loop variable remains a reference to the last array element.

**Explanation:** Unsetting the loop variable prevents later assignments to it from unexpectedly modifying that element.

### 29. Should return-by-reference be used to improve performance?

**Answer:** No.

**Explanation:** Use reference returns only when the caller must alias or modify a specific variable.

## True or false

### 30. PHP function parameters are passed by reference by default.

**Answer:** False.

**Explanation:** The default is pass-by-value.

### 31. A parameter declared with **&** can modify the caller's variable.

**Answer:** True.

**Explanation:** Both names refer to the same variable during the call.

### 32. The caller writes **&** before the argument when calling a by-reference parameter.

**Answer:** False.

**Explanation:** The function declaration determines by-reference behavior; the call uses a normal variable argument.

### 33. A literal such as **5** is suitable for a parameter that must be passed by reference.

**Answer:** False.

**Explanation:** The function requires a variable that it can refer to and update.

### 34. A by-value object parameter can still be used to change the object's properties.

**Answer:** True.

**Explanation:** The parameter contains the object's handle, so property updates affect that object.

### 35. Assigning a new object to a by-value object parameter replaces the caller's variable.

**Answer:** False.

**Explanation:** It only changes the local parameter variable.

### 36. A caller must use **=&** to bind to a reference returned by a function.

**Answer:** True.

**Explanation:** Without **&** in the assignment, the caller receives the value.

### 37. A function returning by reference can safely return any arithmetic expression as an alias.

**Answer:** False.

**Explanation:** A reference return must refer to a variable, not a computed expression.

## Predict the output

### 38. What does this code print?

~~~php
<?php
function increment(int &$number): void
{
    $number++;
}

$count = 5;
increment($count);

echo $count, "\n";
~~~

**Answer:**

~~~text
6
~~~

**Explanation:** **$number** refers to **$count**, so incrementing it changes the caller's variable.

### 39. What does this code print?

~~~php
<?php
function swap(string &$first, string &$second): void
{
    $temporary = $first;
    $first = $second;
    $second = $temporary;
}

$left = 'A';
$right = 'B';

swap($left, $right);

echo $left, $right, "\n";
~~~

**Answer:**

~~~text
BA
~~~

**Explanation:** Both parameters refer to the caller's variables, so the swap updates them.

### 40. What does this code print?

~~~php
<?php
class User
{
    public function __construct(
        public string $name,
    ) {
    }
}

function renameUser(User $user): void
{
    $user->name = 'Mina';
}

$user = new User('Asha');
renameUser($user);

echo $user->name, "\n";
~~~

**Answer:**

~~~text
Mina
~~~

**Explanation:** The function modifies a property on the shared object. It does not need a by-reference parameter for this property change.

### 41. What does this code print?

~~~php
<?php
class User
{
    public function __construct(
        public string $name,
    ) {
    }
}

function replaceUser(User $user): void
{
    $user = new User('Ravi');
}

$user = new User('Asha');
replaceUser($user);

echo $user->name, "\n";
~~~

**Answer:**

~~~text
Asha
~~~

**Explanation:** Reassigning the by-value parameter changes only the local variable. The caller still holds the original object.

### 42. What does this code print?

~~~php
<?php
function &getCounter(): int
{
    static $counter = 1;

    return $counter;
}

$counterReference =& getCounter();
$counterReference = 9;

echo getCounter(), "\n";
~~~

**Answer:**

~~~text
9
~~~

**Explanation:** **$counterReference** is bound to the static variable returned by reference, so assigning **9** updates it.

## Write code

### 43. Write a function that adds one to a caller's integer variable.

**Answer:**

~~~php
<?php
function addOne(int &$value): void
{
    $value++;
}

$number = 10;
addOne($number);

echo $number, "\n";
~~~

Output:

~~~text
11
~~~

**Explanation:** The function parameter is declared by reference, so the caller's variable is updated.

### 44. Write a function that returns a persistent counter by reference and change it through an alias.

**Answer:**

~~~php
<?php
function &counter(): int
{
    static $value = 0;

    return $value;
}

$alias =& counter();
$alias += 3;

echo counter(), "\n";
~~~

Output:

~~~text
3
~~~

**Explanation:** The static variable persists between calls. The caller binds **$alias** to it with **=&**.

### 45. Use a by-reference **foreach** loop to add one to every number in **[2, 4, 6]**, then clean up the loop variable.

**Answer:**

~~~php
<?php
$numbers = [2, 4, 6];

foreach ($numbers as &$number) {
    $number++;
}

unset($number);

echo implode(', ', $numbers), "\n";
~~~

Output:

~~~text
3, 5, 7
~~~

**Explanation:** The loop variable refers to each array element, so each element is updated. **unset($number)** removes the lingering reference after the loop.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [044 - Partial Function Application](../lessons/044-partial-function-application.md) | [Read lesson](../lessons/045-passing-and-returning-values-by-reference.md) | [Open example](../examples/045-passing-and-returning-values-by-reference.md) | [046 - Generators and Yield](../lessons/046-generators-and-yield.md) |
