# 010 - Variables and Assignment: Complete Example

This example is a small command-line program that assigns values to variables, reads them, updates one, and keeps a separate copy of a simple integer value.

## Question

How can a PHP program store a value, use it, update it, and keep the earlier simple value available?

## Goal

Create this project:

~~~text
variable-practice/
|-- README.md
|-- variables.php
~~~

The **variables.php** script prints a learner's details, calculates the next lesson number, and demonstrates that ordinary assignment of an integer gives the receiving variable its own value.

## Complete code: variables.php

Save this file at the project root:

~~~php
<?php

$learnerName = "Asha";
$courseTitle = "PHP Fundamentals";
$completedLessons = 9;

echo "Learner: ", $learnerName, PHP_EOL;
echo "Course: ", $courseTitle, PHP_EOL;
echo "Completed lessons: ", $completedLessons, PHP_EOL;

// Calculate a value from the current lesson count.
$nextLessonNumber = $completedLessons + 1;
echo "Next lesson number: ", $nextLessonNumber, PHP_EOL;

// Keep a copy before updating the original variable.
$previousLessonCount = $completedLessons;
$completedLessons = $nextLessonNumber;

echo "Updated completed lessons: ", $completedLessons, PHP_EOL;
echo "Previous lesson count: ", $previousLessonCount, PHP_EOL;
~~~

The assignment statements read from left to right as follows:

- **$learnerName = "Asha";** assigns a string to **$learnerName**.
- **$nextLessonNumber = $completedLessons + 1;** calculates the right-hand expression, then assigns its result.
- **$previousLessonCount = $completedLessons;** assigns the current integer value to a second variable.
- **$completedLessons = $nextLessonNumber;** updates the current value of **$completedLessons**.

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Variable Practice

A small PHP CLI script demonstrating variable assignment and updates.

## Check and run

~~~sh
php -l variables.php
php variables.php
~~~

## What it demonstrates

- Assigning values before using variables
- Reading a variable in an output statement
- Calculating a value on the right side of an assignment
- Updating a variable with a later assignment
- Keeping an earlier simple integer value in another variable
~~~~

This Markdown page is the README text to create in the project. The PHP code remains in **variables.php**.

## Check and run the program

Open a terminal in **variable-practice/** and check the PHP syntax:

~~~sh
php -l variables.php
~~~

For a valid file, PHP reports that no syntax errors were detected. This command checks the syntax; it does not print the program's output.

Run the script:

~~~sh
php variables.php
~~~

Expected terminal output:

~~~text
Learner: Asha
Course: PHP Fundamentals
Completed lessons: 9
Next lesson number: 10
Updated completed lessons: 10
Previous lesson count: 9
~~~

The original count is copied before **$completedLessons** is updated. That is why the final two lines contain different values.

## Follow the values

| Statement | Value after the statement |
| --- | --- |
| **$completedLessons = 9;** | **$completedLessons** contains **9**. |
| **$nextLessonNumber = $completedLessons + 1;** | PHP calculates **9 + 1**; **$nextLessonNumber** contains **10**. |
| **$previousLessonCount = $completedLessons;** | **$previousLessonCount** contains **9**. |
| **$completedLessons = $nextLessonNumber;** | **$completedLessons** now contains **10**; **$previousLessonCount** stays **9**. |

The single equals sign (**=**) assigns the right-hand value to the variable on the left. It does not perform an equality comparison.

## Make a change

1. Change **$learnerName** to another name and run the program again.
2. Change the initial **$completedLessons** value from **9** to **4**. Predict the next lesson number and both final counts before running it.
3. Change the output label **"Course:"** without changing the value in **$courseTitle**. Notice that the label is just text and the variable holds the course title.
4. Move the assignment to **$previousLessonCount** below the update to **$completedLessons**. Predict how this changes the final previous count, then run the program.
5. Remove the initial assignment to **$learnerName** in a scratch copy. Read the PHP diagnostic and then restore the initialization. Do not rely on an unset variable in application code.
6. Rename **$previousLessonCount** to another descriptive name. Update both the assignment and its use consistently.

## Common mistakes in this example

| Mistake | What happens | Correction |
| --- | --- | --- |
| Writing **courseTitle** instead of **$courseTitle** | The code does not refer to the variable. | Include the **$** marker when using a variable. |
| Writing **$completedLessons = $nextLessonNumber** before assigning **$nextLessonNumber** | The right-hand variable has not been initialized yet. | Assign a value to **$nextLessonNumber** first. |
| Expecting **$previousLessonCount** to change after **$completedLessons** is updated | Ordinary assignment of this integer copied its value at that point in the program. | Assign to **$previousLessonCount** again if a new value is intended. |
| Treating **=** as a check for equality | It assigns rather than compares. | Use the comparison operators covered in the later comparison lesson when checking values. |
| Omitting a semicolon after an assignment | PHP cannot parse the simple statements as written. | End each assignment statement with **;**. |

## Key takeaways

- Assign a deliberate value before reading a variable.
- **=** assigns the evaluated right-hand value to the variable on the left.
- Reassigning a variable changes the value later statements read from it.
- Assigning a simple integer to another variable keeps that earlier value available if the original is updated.
- Descriptive variable names make the program easier to follow.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [009 - Identifiers, keywords, reserved words, and naming](../lessons/009-identifiers-keywords-reserved-words-and-naming.md) | [Read this lesson](../lessons/010-variables-and-assignment.md) | [Open Q&A](../qnas/010-variables-and-assignment.md) | [011 - Variable scope, global variables, and static locals](../lessons/011-variable-scope-global-variables-and-static-locals.md) |
