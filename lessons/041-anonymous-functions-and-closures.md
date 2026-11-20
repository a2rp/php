# 041 - Anonymous Functions and Closures

A named function is useful when behavior has a stable name and is reused across a program. Sometimes a small function is needed only as a value: for example, to store in a variable or pass to another function as a callback. PHP supports functions without a declared name, called anonymous functions. These function values are represented by **Closure** objects and can capture selected variables from their surrounding scope.

## What you will learn

- Define an anonymous function with the **function** keyword
- Store a function value in a variable and invoke it
- Pass an anonymous function inline as a callback
- Capture surrounding variables with **use**
- Understand capture by value and capture by reference
- Declare parameters and a return type on a closure
- Distinguish anonymous functions from the shorter arrow functions covered next

## Define an anonymous function

An anonymous function has no declared name:

~~~php
function (string $name): string {
    return "Hello, {$name}!";
}
~~~

It still has a parameter list, a body, and can have a return type. Because it has no name to call directly, programs usually assign it to a variable or pass it to code that accepts a callback.

## Store and call a closure

Assign an anonymous function to a variable, then call the variable with parentheses:

~~~php
<?php
$greet = function (string $name): string {
    return "Hello, {$name}!";
};

echo $greet('Asha'), "\n";
echo $greet('Mina'), "\n";
~~~

Output:

~~~text
Hello, Asha!
Hello, Mina!
~~~

The semicolon after the closing brace belongs to the assignment statement. The variable **$greet** now holds a callable closure. Calling **$greet('Asha')** runs the function body and returns its string.

Anonymous functions can also be invoked immediately:

~~~php
$result = (function (int $number): int {
    return $number * 2;
})(5);

echo $result, "\n";
~~~

Output:

~~~text
10
~~~

The parentheses around the function expression let PHP treat it as a value that can be called immediately.

## Pass a closure as a callback

A callback is a function supplied to another function so it can perform an operation for each value or at a particular step. An anonymous function is useful when the callback is short and only needed at that call site:

~~~php
<?php
$numbers = [1, 2, 3];

$doubled = array_map(
    function (int $number): int {
        return $number * 2;
    },
    $numbers,
);

echo implode(', ', $doubled), "\n";
~~~

Output:

~~~text
2, 4, 6
~~~

The closure describes the transformation. **array_map()** calls it for each input value and collects the returned values.

When a callback has a clear purpose or is used in several places, give it a name or assign it to a variable. Inline callbacks work best when their behavior is short and clear in context.

## Capture a variable with use

A normal function has its own local scope. An anonymous function does not automatically see ordinary variables from the scope where it was created. Import a variable with **use**:

~~~php
<?php
$message = 'Hello';

$greet = function (string $name) use ($message): string {
    return "{$message}, {$name}!";
};

$message = 'Welcome';

echo $greet('Asha'), "\n";
~~~

Output:

~~~text
Hello, Asha!
~~~

The closure captured the value of **$message** when the function expression was created. Changing the outer **$message** afterward does not replace the closure's captured value.

The **use** clause appears after the parameter list and before the return type. If the closure declares a return type, put that declaration after **use**:

~~~php
function (string $name) use ($message): string {
    return "{$message}, {$name}!";
}
~~~

A closure can capture more than one variable by listing them in **use**, separated by commas.

## Capture a variable by reference

Use **&** in the **use** list when the closure must share and modify the surrounding variable:

~~~php
<?php
$count = 0;

$nextCount = function () use (&$count): int {
    $count++;
    return $count;
};

echo $nextCount(), "\n";
echo $nextCount(), "\n";
echo "Outside: {$count}\n";
~~~

Output:

~~~text
1
2
Outside: 2
~~~

The closure and the outer scope refer to the same **$count** variable. Each call updates it, and the changed value is visible outside the closure too.

Capture by reference deliberately shares mutable state. Use ordinary capture by value when the closure only needs the value it had when it was created.

## Parameters and captured variables are different

A closure can have normal parameters and separately capture values with **use**:

~~~php
<?php
$prefix = 'Item';

$label = function (string $name) use ($prefix): string {
    return "{$prefix}: {$name}";
};

echo $label('Notebook'), "\n";
~~~

Output:

~~~text
Item: Notebook
~~~

The argument **'Notebook'** is passed to the **$name** parameter when the closure is called. The surrounding **$prefix** value is imported when the closure is created.

Do not list a parameter name again in the **use** clause. The parameter already receives its value from the call and cannot also be imported as a captured variable.

## Closures and object context

When an anonymous function is created inside an object method, PHP normally binds the current **$this** object to it. A **static function** prevents that automatic binding. This is an object-oriented detail; the key beginner rule is that ordinary variables still need to be captured explicitly with **use**.

Arrow functions, introduced in the next chapter, have a shorter syntax and automatically capture variables by value. Use a regular anonymous function when you need a statement body, capture by reference, or explicit control over the **use** list.

## Common mistakes

- Forgetting the semicolon after assigning an anonymous function to a variable.
- Trying to call a closure variable without parentheses.
- Expecting an ordinary outer variable to be visible without **use**.
- Putting the return type before the **use** clause instead of after it.
- Capturing a variable by value and then expecting later outer assignments to update the captured value.
- Using **use (&$value)** without realizing the closure can modify surrounding state.
- Listing the same variable as both a parameter and a captured variable.
- Writing a long inline callback that would be clearer as a named function.
- Treating a regular closure and an arrow function as identical; arrow functions have different capture syntax.

## Try it yourself

1. Assign a closure that accepts a name and returns a greeting to **$greet**.
2. Pass an anonymous function to **array_map()** to add one to every number.
3. Capture a message with **use ($message)**, then change the outer variable and observe which value the closure uses.
4. Capture a counter with **use (&$counter)** and increment it on each call.
5. Add a return type to a closure and make sure every path returns that type.
6. Rewrite a short closure as an arrow function in the next lesson and compare the syntax.

## Key takeaways

- An anonymous function has no declared name and can be assigned or passed as a value.
- A closure variable is called like a function, with parentheses.
- Ordinary outer variables must be captured explicitly with **use**.
- **use ($value)** captures by value; **use (&$value)** shares the surrounding variable by reference.
- A closure can also receive normal call arguments through parameters.
- In a closure declaration, the return type follows the **use** clause.
- Arrow functions provide shorter syntax and automatic by-value capture in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [040 - Function Return Types](040-function-return-types.md) | [Open Q&A](../qnas/041-anonymous-functions-and-closures.md) | [Open example](../examples/041-anonymous-functions-and-closures.md) | [042 - Arrow Functions](042-arrow-functions.md) |
