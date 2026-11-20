# 045 - Passing and Returning Values by Reference: Example

## Question

When should a function receive a variable by reference, and how does a caller bind to a reference returned by a function?

## Goal

Create one runnable PHP program that demonstrates:

- The difference between a by-value parameter and a by-reference parameter
- Updating a caller's scalar variable
- Changing an object property through a by-value object parameter
- Replacing the caller's object variable through a by-reference parameter
- Returning a persistent variable by reference
- Updating array elements in a by-reference **foreach** loop and cleaning up the loop variable

## Project structure

~~~text
references-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function setLocalToZero(int $number): void
{
    $number = 0;
}

function addBonus(int &$score): void
{
    $score += 5;
}

class User
{
    public function __construct(
        public string $name,
    ) {
    }
}

function renameUser(User $user, string $name): void
{
    $user->name = $name;
}

function replaceUser(User &$user, string $name): void
{
    $user = new User($name);
}

function &getCounter(): int
{
    static $counter = 1;

    return $counter;
}

// A normal parameter receives a value.
$number = 7;
setLocalToZero($number);

echo 'After by-value call: ', $number, "\n";

// A by-reference parameter can update the caller's variable.
$score = 10;
addBonus($score);

echo 'Score after bonus: ', $score, "\n";

// A by-value object parameter can still modify the object's property.
$user = new User('Asha');
renameUser($user, 'Mina');

echo 'User after property change: ', $user->name, "\n";

// A by-reference object parameter can replace the caller's variable.
replaceUser($user, 'Ravi');

echo 'User after object replacement: ', $user->name, "\n";

// Bind to the persistent counter returned by reference.
$counterReference =& getCounter();
$counterReference += 4;

echo 'Shared counter through reference: ', getCounter(), "\n";

// An ordinary assignment receives a value copy.
$counterCopy = getCounter();
$counterCopy = 100;

echo 'Shared counter after copy changes: ', getCounter(), "\n";

// A by-reference foreach loop updates each array element.
$numbers = [2, 4, 6];

foreach ($numbers as &$value) {
    $value++;
}

// Break the lingering reference to the final array element.
unset($value);

echo 'Numbers after the loop: ';
echo implode(', ', $numbers), "\n";
~~~

## Expected output

~~~text
After by-value call: 7
Score after bonus: 15
User after property change: Mina
User after object replacement: Ravi
Shared counter through reference: 5
Shared counter after copy changes: 5
Numbers after the loop: 3, 5, 7
~~~

## Explanation

### A by-value parameter does not replace the caller's variable

~~~php
function setLocalToZero(int $number): void
{
    $number = 0;
}
~~~

The parameter is passed by value because it has no **&**. Changing **$number** only changes the local parameter, so the caller's **$number** remains **7**.

### A by-reference parameter updates the caller's variable

~~~php
function addBonus(int &$score): void
{
    $score += 5;
}
~~~

The ampersand in the declaration means **$score** refers to the variable supplied by the caller. The call uses ordinary syntax:

~~~php
addBonus($score);
~~~

No ampersand is written at the call site. The declaration defines the parameter's reference behavior, and the value becomes **15**.

### Changing an object property is different from replacing the object variable

**renameUser()** receives its object parameter by value, but both the local parameter and the caller's variable refer to the same object. Changing **$user->name** is therefore visible to the caller.

**replaceUser()** assigns a new object to its local parameter. It declares **User &$user**, so this reassignment replaces the caller's **$user** variable too.

### A returned reference can be bound by the caller

~~~php
function &getCounter(): int
{
    static $counter = 1;

    return $counter;
}
~~~

The ampersand before the function name declares a reference return. The caller uses **=&** to bind **$counterReference** to the persistent static variable:

~~~php
$counterReference =& getCounter();
$counterReference += 4;
~~~

The shared counter is now **5**. By contrast, **$counterCopy = getCounter()** receives the value. Reassigning **$counterCopy** to **100** does not change the stored counter.

### Update array elements with a reference loop

~~~php
foreach ($numbers as &$value) {
    $value++;
}

unset($value);
~~~

The loop variable refers to each element as the loop visits it, so all three values increase. After the loop, **$value** remains a reference to the last element. **unset($value)** breaks that link before the variable is used again.

## Make a change

1. Change the bonus from **5** to **10** and predict the caller's score.
2. Remove **&** from **addBonus()** and observe that the caller's score no longer changes.
3. Change **renameUser()** to assign a new **User** object to its parameter. Predict whether the caller's variable is replaced.
4. Remove **&** from **replaceUser()** and observe which name is printed afterward.
5. Change **$counterReference += 4** to **$counterReference = 12** and read the counter again.
6. Change **$counterCopy = 100** and verify that the shared counter stays the same.
7. Remove **unset($value)** and add code that assigns a new value to **$value** after the loop. Observe which array element it changes.

## Common mistakes

- Expecting **setLocalToZero()** to change the caller's value even though its parameter is passed by value.
- Adding **&** at the call site instead of declaring it on the parameter.
- Passing a literal to a parameter that needs a reference.
- Assuming an object property update means the object variable itself was passed by reference.
- Forgetting **&** on **replaceUser()** when the caller's object variable must be replaced.
- Using **=** when the caller intends to bind to a reference returned by a function.
- Expecting an ordinary assignment from a reference-returning function to remain an alias.
- Forgetting to **unset()** a by-reference loop variable.
- Using references only because they seem faster.

## Key takeaways

- **setLocalToZero()** changes only its local parameter because it is passed by value.
- **addBonus()** declares **int &$score** to update the caller's variable.
- Object properties can be changed through a by-value object parameter; replacing the caller's object variable requires a by-reference parameter.
- **function &getCounter()** returns a reference, and **$counterReference =& getCounter()** binds to it.
- A normal **=** assignment receives a value rather than preserving a reference binding.
- A by-reference loop changes array elements in place; unset its loop variable afterward.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [044 - Partial Function Application](../lessons/044-partial-function-application.md) | [Read lesson](../lessons/045-passing-and-returning-values-by-reference.md) | [Open Q&A](../qnas/045-passing-and-returning-values-by-reference.md) | [046 - Generators and Yield](../lessons/046-generators-and-yield.md) |
