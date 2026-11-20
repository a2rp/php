# 010 - Variables and Assignment

A variable lets a PHP script keep a value under a name so the script can use it later. Assignment puts a value into a variable or replaces its current value. These two ideas appear in nearly every useful PHP program.

## What you will learn

- How PHP variables are written and initialized
- How the assignment operator **=** works
- How to update a variable and use its value
- What happens when one simple value is assigned to another variable
- How to avoid common undefined-variable and assignment mistakes

## A variable stores a value under a name

In PHP, write a dollar sign (**$**) followed by a variable name. The name follows the rules from the previous chapter.

~~~php
<?php
$learnerName = "Asha";
$completedLessons = 9;
~~~

Here, **$learnerName** refers to the string value **"Asha"**, and **$completedLessons** refers to the integer value **9**. A variable name is case-sensitive, and the **$** marker is part of how the variable is written in PHP.

PHP does not require a separate declaration before an ordinary variable can be assigned. The assignment creates or updates it. For clarity, give a variable a meaningful value before trying to use it.

## Assign a value with **=**

The assignment operator is a single equals sign:

~~~php
<?php
$courseTitle = "PHP Fundamentals";
~~~

Read this as “assign the value on the right to the variable on the left.” The left side must be something that can receive an assigned value. The right side can be a literal value or an expression that produces a value.

**=** assigns a value; it does not ask whether two values are equal. Comparisons are covered in a later chapter.

A simple assignment statement ends with a semicolon:

~~~php
<?php
$lessonNumber = 10;
$nextLessonNumber = $lessonNumber + 1;
~~~

After the first statement, **$lessonNumber** contains **10**. The second statement calculates **$lessonNumber + 1** and assigns the resulting value to **$nextLessonNumber**.

## Read and update a variable

Use the variable name, including **$**, whenever you want to read its current value:

~~~php
<?php
$completedLessons = 9;

echo "Completed: ", $completedLessons, PHP_EOL;

$completedLessons = 10;

echo "Completed now: ", $completedLessons, PHP_EOL;
~~~

The first output uses the value **9**. The later assignment replaces that variable's current value, so the second output uses **10**. A variable can be assigned more than once; the value used by later statements is the value most recently assigned.

PHP determines a variable's value type from the value assigned to it. This course will explain types and conversions in detail in later chapters. For now, give each variable values that make sense for its purpose, and avoid changing a variable's meaning halfway through a program.

## Assign a value to another variable

With ordinary assignment, the value is assigned to the variable on the left. For simple values such as integers and strings, the receiving variable gets its own value:

~~~php
<?php
$originalCount = 9;
$copiedCount = $originalCount;

$originalCount = 10;

echo "Original: ", $originalCount, PHP_EOL;
echo "Copy: ", $copiedCount, PHP_EOL;
~~~

The output is:

~~~text
Original: 10
Copy: 9
~~~

Changing **$originalCount** after the assignment does not change **$copiedCount**. This is the expected beginner model for ordinary assignment of simple values. PHP also has references, which deliberately make two variable names refer to the same variable; that is a separate feature and is not needed for these examples.

## Initialize variables before using them

An assignment such as **$pageTitle = "Welcome";** gives **$pageTitle** a known value before later code reads it. If code reads a variable that has not been set, PHP reports an undefined-variable diagnostic. Current PHP versions report a warning; older PHP versions reported a notice. The exact diagnostic depends on the PHP version and error-reporting configuration, so do not rely on it being hidden.

For example, this is incorrect because the variable is read before it has been assigned:

~~~php
<?php
echo $pageTitle;
~~~

Initialize it first:

~~~php
<?php
$pageTitle = "Welcome";
echo $pageTitle;
~~~

If a value may genuinely be absent, handle that situation explicitly. Do not use an undefined variable as a shortcut for an empty string, zero, or **null**.

## Assignment does not mean comparison

A common source of confusion is the single equals sign:

~~~php
<?php
$attempts = 3;
~~~

This puts **3** into **$attempts**. It is not a question about whether **$attempts** already equals **3**. PHP uses comparison operators for questions about equality; they are introduced later in this course.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Reading a variable before assigning it | PHP reports that the variable is undefined, and the result is not a reliable application value. | Assign a deliberate initial value before using it. |
| Leaving off the **$** when referring to a variable | The code no longer refers to that variable. | Write its complete variable name, such as **$courseTitle**. |
| Reassigning a value and expecting the old value to remain | A variable refers to its current assigned value. | Copy a simple value to another variable first if both values are needed. |
| Assuming **=** checks equality | **=** performs assignment. | Use the comparison operators introduced in the later comparison chapter when checking values. |
| Using unclear names such as **$x** for important data | Readers cannot tell what the value represents. | Use a name such as **$completedLessons**. |
| Depending on an undefined variable behaving like an empty value | Undefined-variable diagnostics and error handling vary by PHP version and configuration. | Initialize it or handle the absent-value case explicitly. |

## Try it yourself

1. Create **$learnerName** and assign it a name. Print a sentence that includes the variable.
2. Assign **9** to **$completedLessons**, then assign **10** to it. Print the value after each assignment.
3. Assign **$completedLessons** to **$previousCount**, update **$completedLessons**, and print both values. Explain why they differ.
4. Change the right-hand value in a simple assignment and predict what the next **echo** will display.
5. Read this statement aloud and identify its left and right sides: **$nextLesson = $currentLesson + 1;**
6. Explain why the assignment **$pageTitle = "Welcome";** does not test whether the title is already **"Welcome"**.
7. Write a short example that reads a variable before assigning it. Do not rely on its output; explain why the variable should be initialized first.
8. Choose a variable name that clearly describes a count, a title, and a learner's name.

## Key points

- A PHP variable is written with **$** followed by its case-sensitive name.
- Assignment uses **=** and places the right-hand value in the variable on the left.
- Assigning to a variable for the first time gives it a value; assigning again replaces its current value.
- Read variables only after giving them a deliberate value.
- Ordinary assignment of simple values lets each variable keep its own value when the other is later reassigned.
- **=** assigns; comparison operators are used to compare values.
- Prefer descriptive names and values that match each variable's purpose.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [009 - Identifiers, keywords, reserved words, and naming](009-identifiers-keywords-reserved-words-and-naming.md) | [Open Q&A](../qnas/010-variables-and-assignment.md) | [Open example](../examples/010-variables-and-assignment.md) | [011 - Variable scope, global variables, and static locals](011-variable-scope-global-variables-and-static-locals.md) |
