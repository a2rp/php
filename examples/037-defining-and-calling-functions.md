# 037 - Defining and Calling Functions: Example

## Question

How can a PHP program print the same useful output more than once without copying the same statements everywhere?

## Goal

Create a small course welcome report with named functions. Declare each function once, call it from the main program flow, and observe how execution enters and returns from each function.

## Project structure

~~~text
function-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function printDivider(): void
{
    echo "====================\n";
}

function printHeading(): void
{
    echo "PHP Function Practice\n";
}

function printWelcome(): void
{
    echo "Welcome, learner!\n";
}

function printReminder(): void
{
    echo "Practice one small change at a time.\n";
}

printDivider();
printHeading();
printDivider();

printWelcome();
printReminder();
printWelcome();

printDivider();
~~~

## Expected output

Run the file from a terminal with PHP installed:

~~~sh
php index.php
~~~

The output is:

~~~text
====================
PHP Function Practice
====================
Welcome, learner!
Practice one small change at a time.
Welcome, learner!
====================
~~~

## How it works

### 1. The declarations define behavior

The four blocks beginning with **function** define **printDivider()**, **printHeading()**, **printWelcome()**, and **printReminder()**. Their statements do not print anything just because PHP reads these declarations.

Each function has an empty parameter list, written as **()**, because this example does not pass values into the functions. The **void** return type says they perform an action without returning a value.

### 2. The calls run the functions

After the declarations, the main program flow calls the functions:

~~~php
printDivider();
printHeading();
printDivider();
~~~

At the first call, PHP runs **printDivider()**, prints the row of equals signs, and then continues at **printHeading()**. It later runs **printDivider()** again.

### 3. Reuse avoids copying behavior

The call to **printWelcome()** appears twice. Both calls run the same function body, so the welcome message appears twice. If that message needs to change, edit its declaration once; both calls use the new text.

### 4. Output is not a return value

The functions use **echo** to display text. They do not return that text to the caller. **void** makes this action-only intent explicit.

## Trace the execution

Follow the calls in order:

1. **printDivider()** prints the first divider.
2. **printHeading()** prints the heading.
3. **printDivider()** prints the second divider.
4. **printWelcome()** prints the first welcome.
5. **printReminder()** prints the reminder.
6. **printWelcome()** runs again and prints the second welcome.
7. **printDivider()** prints the final divider.
8. The script reaches its end.

The declarations are written before the calls here to keep the example easy to scan. Ordinary unconditional named functions can also be called before their declaration appears in the same file, but placing declarations first is a clear convention for this beginner example.

## Make a change

Try these changes one at a time:

1. Change the heading text inside **printHeading()**.
2. Add another call to **printWelcome()** and predict where its message will appear.
3. Change the divider in **printDivider()** from equals signs to hyphens. Every divider should update.
4. Add a **printClosing()** function that prints **See you next time!**, then call it before the final divider.
5. Temporarily remove one call to **printWelcome()**. The function remains defined, but its body no longer runs at that location.

## Common mistakes

- Removing the parentheses from a call, such as writing **printWelcome;**.
- Expecting a declaration to print output before the first call.
- Defining the same function name a second time instead of editing the existing body.
- Expecting **echo** to provide a returned value.
- Placing a function declaration inside a condition and then calling it on a path where that condition was false.

## Key takeaways

- A declaration defines a reusable function; a call executes its body.
- A no-parameter call still uses parentheses.
- Each reached call runs the body and then returns execution to the caller.
- Reusing a function keeps repeated behavior in one place.
- **echo** displays output, while **void** communicates that the function does not return a value.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [036 - Returning Values and Ending Execution](../lessons/036-returning-values-and-ending-execution.md) | [Read lesson](../lessons/037-defining-and-calling-functions.md) | [Open Q&A](../qnas/037-defining-and-calling-functions.md) | [038 - Function Parameters and Arguments](../lessons/038-function-parameters-and-arguments.md) |
