# 009 - Identifiers, Keywords, Reserved Words, and Naming: Complete Example

This example creates a small CLI program with descriptive variable names. It demonstrates the **$** marker, valid name shapes, case sensitivity, and the difference between a syntax rule and a style convention.

## Question

How can you choose valid, readable PHP variable names and check that the names are used consistently?

## Goal

Create this project:

~~~text
identifier-practice/
├── README.md
└── names.php
~~~

The **names.php** script prints a learner name, course title, lesson count, and a small demonstration that PHP variable names are case-sensitive.

## Complete code: names.php

Save this file at the project root:

~~~php
<?php

// Descriptive names make each value's purpose easier to understand.
$learnerName = "Asha";
$courseTitle = "PHP Fundamentals";
$completedLessonCount = 9;

echo "Learner: ", $learnerName, PHP_EOL;
echo "Course: ", $courseTitle, PHP_EOL;
echo "Lessons completed: ", $completedLessonCount, PHP_EOL;

// These are two distinct variables because PHP variable names are case-sensitive.
$caseSensitiveName = "lowercase spelling";
$CaseSensitiveName = "uppercase spelling";

echo "First variable: ", $caseSensitiveName, PHP_EOL;
echo "Second variable: ", $CaseSensitiveName, PHP_EOL;
~~~

The final two variables intentionally differ only in capitalization to demonstrate the language rule. In everyday code, avoid names that differ only by case because they are easy to confuse.

## Complete code: README.md

Save this as **README.md** at the project root:

~~~markdown
# Identifier Practice

A small CLI script for learning clear PHP variable names.

## Check and run

```sh
php -l names.php
php names.php
```

## Naming notes

- Variable references start with `$`.
- Names start with a letter or underscore, then may contain letters, digits, and underscores.
- Variable names are case-sensitive.
- This example uses camelCase as a consistent style choice.
~~~

This Markdown page shows the README text as an example. Create the file in your project to keep the run instructions with the source.

## Check and run the program

Open a terminal in **identifier-practice/** and check the syntax:

~~~sh
php -l names.php
~~~

For a valid file, PHP reports that no syntax errors were detected. This checks syntax without running the script.

Run it with:

~~~sh
php names.php
~~~

Expected terminal output:

~~~text
Learner: Asha
Course: PHP Fundamentals
Lessons completed: 9
First variable: lowercase spelling
Second variable: uppercase spelling
~~~

The output confirms that **$caseSensitiveName** and **$CaseSensitiveName** keep separate values. The code follows the variable-name syntax, but using these similar names together is only for this demonstration.

## Sort the name examples

| Example | Valid ordinary variable name? | Reason |
| --- | --- | --- |
| **$learnerName** | Yes | Starts with a letter and uses letters. |
| **$_draftName** | Yes | Starts with an underscore. |
| **$user2** | Yes | The digit appears after the first character. |
| **$2user** | No | Starts with a digit after **$**. |
| **$learner-name** | No | A hyphen is not part of one ordinary variable name. |
| **$learner name** | No | A space cannot appear inside the name. |
| **learnerName** when referring to a variable | No | The variable marker **$** is missing. |

Both **$learnerName** and **$learner_name** are syntactically valid. This project uses camelCase, but PHP does not require that capitalization style.

## Make the names clearer

1. Change **$learnerName** to a different descriptive name and update the use in the output.
2. Rename **$completedLessonCount** to **$count**. Decide which name would make the purpose easier for another reader to understand, then restore the descriptive form.
3. Change **$courseTitle** to **$CourseTitle** in only the assignment or only the output line. Run the file and observe that the differently cased variable is a separate name. Restore consistent capitalization afterward.
4. Change the project to use **$learner_name** consistently instead of **$learnerName**. Both are valid; consistency is the point.
5. In a scratch copy, try names that start with a digit or contain a hyphen, run **php -l**, read the parser message, and then correct the names. Do not leave the invalid versions in the completed script.
6. Pick two PHP keywords, such as **if** and **return**, and explain why they should not be used to name your own functions or classes.

## Why some names are avoided

- **$x** is valid syntax, but it gives little context when used for an important value.
- **$completedLessonCount** tells a reader what the value represents.
- **$2user** starts with a digit and is not a valid ordinary variable name.
- **$user-name** does not name one variable; the hyphen is not allowed in the name.
- **$userName** and **$username** are both valid, but they are distinct because capitalization differs.
- Names such as **if** and **class** already have language meaning. Choose descriptive application names instead of reusing keywords.
- Names beginning with **__** are reserved for PHP's special or magic functionality; use a normal application-specific name.

## Key takeaways

- A variable uses **$** followed by a valid name.
- Start the name with a letter or underscore; add digits only after the first character.
- PHP variable names are case-sensitive.
- Pick descriptive names and use their spelling consistently.
- camelCase is the style chosen for this example, not a PHP syntax requirement.
- Avoid language keywords, PHP-provided names, and the reserved **__** prefix for your own symbols.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [008 - Statements, instruction separation, and comments](../lessons/008-statements-instruction-separation-and-comments.md) | [Read this lesson](../lessons/009-identifiers-keywords-reserved-words-and-naming.md) | [Open Q&A](../qnas/009-identifiers-keywords-reserved-words-and-naming.md) | [010 - Variables and assignment](../lessons/010-variables-and-assignment.md) |
