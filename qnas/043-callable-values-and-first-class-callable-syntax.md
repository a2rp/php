# 043 - Callable Values and First-Class Callable Syntax: Q&A

Review callable values, callback type declarations, and PHP 8.1 first-class callable syntax. Try each question before reading its answer. See the [chapter lesson](../lessons/043-callable-values-and-first-class-callable-syntax.md).

## Multiple-choice questions

### 1. What is a callable in PHP?

A. Any value that is an array
B. A value PHP can invoke as a function
C. A string that contains HTML
D. A class that has a constructor

**Answer:** B. A value PHP can invoke as a function.

**Explanation:** Function names, method references, closures, and invokable objects can all represent callable behavior.

### 2. What is a callback?

A. A callable supplied to other code so that code can invoke it
B. A value returned only by **array_map()**
C. A function that always calls itself
D. A PHP syntax error

**Answer:** A. A callable supplied to other code so that code can invoke it.

**Explanation:** A callback lets a function or method delegate a step of its work to the caller.

### 3. Which value can be a callable?

A. **'trim'**
B. **[$object, 'method']**, when that method is accessible
C. An object with **__invoke()**
D. All of the above

**Answer:** D. All of the above.

**Explanation:** PHP accepts several callable representations, provided the target can be called in the relevant context.

### 4. What does the **callable** parameter type communicate?

A. The parameter must be an integer.
B. The parameter must be invokable as a callback.
C. The parameter must be a string.
D. The parameter must be a **Closure** and nothing else.

**Answer:** B. The parameter must be invokable as a callback.

**Explanation:** The **callable** type can accept callable forms beyond closure objects, including function names and method references.

### 5. Which is a **Closure** object?

A. **'trim'**
B. **[$object, 'method']**
C. **fn ($value) => $value**
D. **'ClassName::method'**

**Answer:** C. **fn ($value) => $value**.

**Explanation:** Anonymous functions, arrow functions, and first-class callable syntax create **Closure** objects. A function-name string or callable array is not itself a closure object.

### 6. When was PHP first-class callable syntax introduced?

A. PHP 5.6
B. PHP 7.0
C. PHP 7.4
D. PHP 8.1

**Answer:** D. PHP 8.1.

**Explanation:** The **callable** concept and older callback forms existed before PHP 8.1; the new **functionName(...)** syntax arrived in PHP 8.1.

### 7. Which expression creates a first-class callable for **strlen()**?

A. **strlen()**
B. **strlen(...)**
C. **new strlen()**
D. **callable strlen**

**Answer:** B. **strlen(...)**.

**Explanation:** The ellipsis is part of the syntax for acquiring a closure for the existing function.

### 8. What does **strlen(...)** do?

A. Calls **strlen()** immediately with an empty string
B. Creates a closure that can call **strlen()** later
C. Creates an array of string lengths
D. Passes the literal text **...** to **strlen()**

**Answer:** B. Creates a closure that can call **strlen()** later.

**Explanation:** The callable is invoked only when the resulting closure is called with an argument.

### 9. Which expression creates a first-class callable for an instance method?

A. **$object->method(...)**
B. **$object->method()**
C. **$object::method(...)**
D. **new $object->method(...)**

**Answer:** A. **$object->method(...)**.

**Explanation:** The expression creates a closure that refers to the object's method.

### 10. Which expression creates a first-class callable for a static method?

A. **ClassName->method(...)**
B. **ClassName::method(...)**
C. **ClassName::method()**
D. **new ClassName::method**

**Answer:** B. **ClassName::method(...)**.

**Explanation:** Use the scope-resolution operator and the callable ellipsis.

### 11. Which older callable form refers to an object method?

A. **[$object, 'method']**
B. **['method', $object]**
C. **$object::['method']**
D. **method($object)**

**Answer:** A. **[$object, 'method']**.

**Explanation:** The array contains the target object followed by the method name.

### 12. What does **Closure::fromCallable($value)** do?

A. Calls the target immediately and returns its result
B. Creates a **Closure** from a value that is callable
C. Checks whether a value is a string
D. Turns a closure into an array

**Answer:** B. Creates a **Closure** from a value that is callable.

**Explanation:** It is useful for converting an existing callable representation to a closure.

### 13. What does **is_callable($value)** tell you?

A. Whether the value can be called in the current context
B. Whether the value is a valid integer
C. Whether calling it with any possible arguments will succeed
D. Whether the value is specifically a closure

**Answer:** A. Whether the value can be called in the current context.

**Explanation:** It does not verify that every future set of arguments will be accepted.

### 14. Which declaration is suitable for a property that must specifically hold a closure object?

A. **public callable $callback;**
B. **public Closure $callback;**
C. **public function $callback;**
D. **public function callable $callback;**

**Answer:** B. **public Closure $callback;**

**Explanation:** The **callable** type is not supported as a class property type. Use **Closure** when the property must store a closure object.

### 15. Which is a language construct rather than an ordinary function that can be referenced as a normal callable?

A. **strlen**
B. **trim**
C. **echo**
D. **strtolower**

**Answer:** C. **echo**.

**Explanation:** Language constructs such as **echo** are not ordinary functions, so they cannot be acquired as **echo(...)**.

## Short-answer questions

### 16. Name four common forms of callable values in PHP.

**Answer:** A function-name string, a method reference array, a **Closure** object, and an object implementing **__invoke()**.

**Explanation:** The value must be callable in the context where it is used.

### 17. What is the difference between a callable and a closure?

**Answer:** A callable is any value PHP can invoke. A **Closure** is one particular object type that represents function behavior.

**Explanation:** Every closure is callable, but not every callable is a closure.

### 18. What does a **callable** type declaration do on a function parameter?

**Answer:** It requires the argument to be callable.

**Explanation:** It does not declare the callback's complete parameter list or return type.

### 19. How do you make a string containing **trim** into a callable variable?

**Answer:**

~~~php
$callback = 'trim';
$result = $callback('  PHP  ');
~~~

**Explanation:** The variable function call invokes the function named by the string.

### 20. How do you represent an object's method with the older array form?

**Answer:**

~~~php
$callback = [$object, 'methodName'];
~~~

**Explanation:** The first item is the object and the second is the method name.

### 21. Write first-class callable syntax for the existing **strtolower()** function.

**Answer:**

~~~php
$lowercase = strtolower(...);
~~~

**Explanation:** The ellipsis creates a closure for later invocation.

### 22. Write first-class callable syntax for an object's **format()** method.

**Answer:**

~~~php
$formatter = $object->format(...);
~~~

**Explanation:** The object method is acquired as a closure. It is not called at this point.

### 23. Write first-class callable syntax for a static **format()** method on **Formatter**.

**Answer:**

~~~php
$formatter = Formatter::format(...);
~~~

**Explanation:** Use **ClassName::method(...)** for a static method.

### 24. Does **strlen(...)** return the length of an empty string?

**Answer:** No. It creates a closure for **strlen()**.

**Explanation:** To get a length immediately, pass a string as an argument: **strlen('PHP')**.

### 25. What does **Closure::fromCallable()** return?

**Answer:** A **Closure** object representing the supplied callable.

**Explanation:** The callable is not invoked just by converting it.

### 26. What limitation should you remember about **is_callable()**?

**Answer:** A true result does not guarantee that every invocation will succeed with every argument list.

**Explanation:** The function can still reject the arguments or raise an error when called.

### 27. Why does a first-class callable created inside a class matter for private method access?

**Answer:** The closure uses the scope where the callable is acquired, so code inside a class can create a closure for a method it can access.

**Explanation:** Returning that closure can intentionally expose the behavior outside the class.

### 28. Does first-class callable syntax pre-fill any function arguments?

**Answer:** No.

**Explanation:** It creates a callable reference. Binding arguments for a later call is a separate feature.

### 29. Which PHP version is required for first-class callable syntax?

**Answer:** PHP 8.1 or later.

**Explanation:** Older callback representations, such as function-name strings and method arrays, remain available on older PHP versions.

## True or false

### 30. A string containing the name of an existing function can be callable.

**Answer:** True.

**Explanation:** For example, **'trim'** can be invoked as a variable function when the function is available.

### 31. Every callable value is an instance of **Closure**.

**Answer:** False.

**Explanation:** A callable may be a string, an array, a closure, or an invokable object.

### 32. A **Closure** object can be passed where a **callable** parameter is expected.

**Answer:** True.

**Explanation:** A closure can be invoked, so it is a callable.

### 33. The ellipsis in **strlen(...)** is optional punctuation.

**Answer:** False.

**Explanation:** It is required syntax for first-class callable creation.

### 34. **strlen(...)** calculates a string's length immediately.

**Answer:** False.

**Explanation:** It creates a callable closure. The closure computes a length when later invoked with a string.

### 35. First-class callable syntax was available in PHP 7.4 along with arrow functions.

**Answer:** False.

**Explanation:** Arrow functions arrived in PHP 7.4; first-class callable syntax arrived in PHP 8.1.

### 36. **is_callable()** proves that a callback accepts a particular set of arguments.

**Answer:** False.

**Explanation:** It checks callability in the current context, not all future invocation details.

### 37. An object with an **__invoke()** method can be called like a function.

**Answer:** True.

**Explanation:** Its **__invoke()** method defines the behavior used when the object is called.

## Predict the output

### 38. What does this code print?

~~~php
<?php
$lengthOf = strlen(...);

echo $lengthOf('elephant'), "\n";
~~~

**Answer:**

~~~text
8
~~~

**Explanation:** **strlen(...)** creates a closure. Calling it with **'elephant'** returns that string's length.

### 39. What does this code print?

~~~php
<?php
$callback = 'trim';

echo $callback('  ready  '), "\n";
~~~

**Answer:**

~~~text
ready
~~~

**Explanation:** The string names the **trim()** function, which is invoked through the variable.

### 40. What does this code print?

~~~php
<?php
$words = [' ONE ', ' Two '];

$clean = array_map(trim(...), $words);

echo implode('|', $clean), "\n";
~~~

**Answer:**

~~~text
ONE|Two
~~~

**Explanation:** **trim(...)** creates a closure, and **array_map()** calls it for each word.

### 41. What does this code print?

~~~php
<?php
class Counter
{
    public static function addOne(int $number): int
    {
        return $number + 1;
    }
}

$increment = Counter::addOne(...);

echo $increment(4), "\n";
~~~

**Answer:**

~~~text
5
~~~

**Explanation:** The static method is acquired as a closure, then called with **4**.

### 42. What does this code print?

~~~php
<?php
class Greeter
{
    public function __invoke(string $name): string
    {
        return "Hello, {$name}!";
    }
}

$greeter = new Greeter();

echo $greeter('Mina'), "\n";
~~~

**Answer:**

~~~text
Hello, Mina!
~~~

**Explanation:** Calling the object runs its **__invoke()** method.

## Write code

### 43. Use a first-class callable for **strtoupper()** with **array_map()** to uppercase each word in **['php', 'notes']**.

**Answer:**

~~~php
<?php
$words = ['php', 'notes'];

$uppercaseWords = array_map(strtoupper(...), $words);

echo implode(', ', $uppercaseWords), "\n";
~~~

Output:

~~~text
PHP, NOTES
~~~

**Explanation:** The callable is created once as an argument; **array_map()** invokes it for each item.

### 44. Write a function that accepts a **callable** and a string, then returns the callback result.

**Answer:**

~~~php
<?php
function transform(callable $callback, string $value): string
{
    return $callback($value);
}

echo transform('trim', '  hello  '), "\n";
~~~

Output:

~~~text
hello
~~~

**Explanation:** The type declaration ensures that the argument can be called. The function invokes it with the string.

### 45. Create a first-class callable for an object's public **upper()** method and call it with **'php'**.

**Answer:**

~~~php
<?php
class Formatter
{
    public function upper(string $text): string
    {
        return strtoupper($text);
    }
}

$formatter = new Formatter();
$uppercase = $formatter->upper(...);

echo $uppercase('php'), "\n";
~~~

Output:

~~~text
PHP
~~~

**Explanation:** **$formatter->upper(...)** acquires the method as a closure. The method runs when the returned closure is called.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [042 - Arrow Functions](../lessons/042-arrow-functions.md) | [Read lesson](../lessons/043-callable-values-and-first-class-callable-syntax.md) | [Open example](../examples/043-callable-values-and-first-class-callable-syntax.md) | [044 - Partial Function Application](../lessons/044-partial-function-application.md) |
