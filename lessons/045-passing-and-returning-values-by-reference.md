# 045 - Passing and Returning Values by Reference

PHP passes function arguments by value by default. This means a function normally receives the value for its parameter, and changing that parameter does not replace the caller's variable.

Use a reference when you intentionally want two variable names to refer to the same value. A function parameter can be passed by reference so the function can change the caller's variable. A function can also return a reference to a variable so the caller can bind to it.

References are a tool for specific behavior, not a performance shortcut. Use them when shared modification is part of the function's design.

## What you will learn

- Understand the default pass-by-value behavior
- Declare a parameter that is passed by reference
- Call a by-reference parameter without adding **&** at the call site
- Recognize which arguments can be passed by reference
- Return a variable by reference
- Bind the caller's variable to a returned reference
- Distinguish reference assignment from ordinary assignment
- Avoid unnecessary or confusing references

## Parameters are passed by value by default

A parameter receives the argument's value. Reassigning the parameter changes that local parameter, not the caller's variable:

~~~php
<?php
function setToZero(int $number): void
{
    $number = 0;
}

$score = 12;
setToZero($score);

echo $score, "\n";
~~~

Output:

~~~text
12
~~~

The function's **$number** and the caller's **$score** are separate variables. The function does not change **$score**.

For arrays, PHP uses copy-on-write internally, so a normal function parameter behaves as a value. You can modify the local array without changing the caller's array.

## Pass a variable by reference

Put **&** before the parameter name in the function declaration:

~~~php
<?php
function addBonus(int &$score): void
{
    $score += 5;
}

$score = 12;
addBonus($score);

echo $score, "\n";
~~~

Output:

~~~text
17
~~~

The parameter **$score** refers to the same variable as the argument passed by the caller. Updating it changes that caller variable.

The call is **addBonus($score)**. There is no **&** at the call site; the function declaration determines that the parameter expects a reference.

## Use references when multiple inputs should be updated

A swap function changes two caller variables:

~~~php
<?php
function swap(string &$first, string &$second): void
{
    $temporary = $first;
    $first = $second;
    $second = $temporary;
}

$left = 'north';
$right = 'south';

swap($left, $right);

echo $left, "\n";
echo $right, "\n";
~~~

Output:

~~~text
south
north
~~~

Both parameters are declared with **&**, so assignments to **$first** and **$second** update the variables supplied by the caller.

For many APIs, returning a new value is easier to read than changing several caller variables. Use by-reference parameters only when the mutation is clear and useful.

## Which arguments can be passed by reference?

Pass a variable to a by-reference parameter:

~~~php
$amount = 10;
addBonus($amount);
~~~

A supported variable target, such as an array element or object property, can also be passed by reference:

~~~php
$points = [10];
addBonus($points[0]);

echo $points[0], "\n";
~~~

Output:

~~~text
15
~~~

A literal or arbitrary expression is not a variable that can be updated:

~~~php
addBonus(10);          // Invalid: 10 is a constant value.
addBonus($score + 1);   // Invalid: this is an expression.
~~~

The function needs a variable to refer to and modify.

## Objects are a useful distinction

PHP variables that contain objects hold object identifiers. Passing an object to a function by value still lets the function call methods and change that object's state:

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

Output:

~~~text
Mina
~~~

The function changed the same object's property. This does not mean the **$user** variable itself was passed by reference.

Reassigning the local parameter is different:

~~~php
function replaceUser(User $user): void
{
    $user = new User('Ravi');
}
~~~

Without **&**, assigning a new object to the parameter does not replace the caller's **$user** variable. A by-reference parameter is needed if the function must replace the caller's variable itself.

## Return a value by reference

A normal function return supplies a value. To let the caller bind to a variable inside the function, place **&** before the function name:

~~~php
<?php
function &getBestScore(): int
{
    static $score = 10;

    return $score;
}

$scoreReference =& getBestScore();

$scoreReference = 25;

echo getBestScore(), "\n";
~~~

Output:

~~~text
25
~~~

There are two reference markers for two separate choices:

- **function &getBestScore()** declares that the function returns a reference.
- **$scoreReference =& getBestScore()** asks the caller to bind to that reference.

The static variable persists between calls, so it is suitable for this demonstration.

## Assignment by value and assignment by reference

Without **&**, assignment copies the current value:

~~~php
$copy = getBestScore();
$copy = 30;
~~~

Changing **$copy** does not change the stored score.

With **&**, both variable names refer to the same value:

~~~php
$alias =& getBestScore();
$alias = 30;
~~~

Now the stored score is **30**. The caller has intentionally made **$alias** an alias for the function's variable.

This difference is important even when the function is declared to return by reference. The caller must use **=&** when it wants a reference binding.

## Return an actual variable

A function that returns by reference must return a variable that can be referenced. It cannot return a calculated expression as a reference:

~~~php
function &getScore(): int
{
    static $score = 10;

    return $score;
}
~~~

Returning **$score + 1** would return the result of an expression, not the variable itself. If you only need a calculated result, return it normally:

~~~php
function getScoreWithBonus(int $score): int
{
    return $score + 1;
}
~~~

Most functions should return values normally. Return by reference only when callers need to alias a specific variable.

## References in loops

A **foreach** loop can also bind its value variable to each array element by reference:

~~~php
<?php
$prices = [10, 20, 30];

foreach ($prices as &$price) {
    $price += 1;
}

unset($price);

echo implode(', ', $prices), "\n";
~~~

Output:

~~~text
11, 21, 31
~~~

The **&** in **as &$price** means each iteration variable refers to the current array element. After the loop, **$price** remains a reference to the final element. Calling **unset($price)** breaks that lingering reference and helps prevent accidental changes later.

Use by-reference iteration only when you intend to update the array elements in place.

## Common mistakes

- Expecting a normal by-value parameter assignment to change the caller's scalar variable.
- Putting **&** at the call site instead of declaring it on the parameter.
- Passing a literal or expression to a parameter that requires a reference.
- Assuming that changing an object's property proves the object variable was passed by reference.
- Returning by reference when a normal value return is enough.
- Declaring **function &getValue()** but using **$value = getValue()** when an alias is intended.
- Using **=&** with a function that does not return by reference.
- Returning a calculated expression from a function declared to return by reference.
- Forgetting to **unset()** a **foreach** reference variable after a by-reference loop.
- Using references as a supposed performance optimization.

## Try it yourself

1. Write a function that accepts **int &$count** and increments the caller's variable.
2. Pass an array element to that function and observe which element changes.
3. Rewrite the increment function to return a new integer instead of modifying its input.
4. Create a function that returns a static variable by reference, then compare **=** with **=&**.
5. Modify an object property from a by-value object parameter, then reassign the local object parameter and observe the difference.
6. Use a by-reference **foreach** loop to add tax to each array value, then **unset()** the loop variable.
7. Explain why returning **$value + 1** by reference is different from returning **$value** by reference.

## Key takeaways

- Function parameters are passed by value by default.
- Declare **&$parameter** when a function should modify the caller's variable.
- The call site does not add **&** for a by-reference parameter.
- A function declared **function &name()** can return a reference to a variable.
- The caller uses **=&** to bind to that returned reference instead of receiving its value.
- Objects can have their properties changed through a by-value object parameter; replacing the caller's object variable is a different operation.
- Use references sparingly and only when shared modification is part of the API.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [044 - Partial Function Application](044-partial-function-application.md) | [Open Q&A](../qnas/045-passing-and-returning-values-by-reference.md) | [Open example](../examples/045-passing-and-returning-values-by-reference.md) | [046 - Generators and Yield](046-generators-and-yield.md) |
