# 009 - Identifiers, Keywords, Reserved Words, and Naming

Programs use names to refer to things such as variables, functions, classes, and constants. In PHP, these names are called **identifiers**. Some words have a special role in the language, so choosing clear names also means recognizing keywords and names that PHP already reserves.

## What you will learn

- What an identifier is
- How PHP variable names are formed
- Why variable names are case-sensitive
- What keywords and reserved names are
- How to choose names that are readable and maintainable
- Why a name that looks valid can still be a poor choice

## What is an identifier?

An identifier is a name used by code to refer to a program element. Examples include names for variables, functions, classes, methods, and constants. Different kinds of identifiers can have different syntax and rules; this chapter focuses mainly on variable names and beginner naming choices.

A PHP variable is written with a dollar sign followed by its name:

~~~php
<?php
$firstName = "Asha";
$userCount = 3;
~~~

The **$** is the variable marker, and **firstName** is the variable name. Keep the marker attached to the name whenever you write that variable in PHP code.

## Variable-name rules

For ordinary PHP variable names, follow these rules:

1. Begin the name with a letter or underscore.
2. After the first character, use letters, numbers, or underscores.
3. Put **$** before the name to refer to the variable.
4. Do not put spaces or hyphens in a variable name.

These are valid examples:

~~~php
<?php
$name = "Asha";
$firstName = "Asha";
$first_name = "Asha";
$user2 = "Learner";
$_draftName = "Draft";
~~~

For beginner projects, use plain English names made from the usual A-Z letters, digits, and underscores. PHP accepts additional byte values in identifiers, but sticking to simple ASCII names is easier to type, review, and share across tools.

These are not valid ordinary variable names:

| Example | Why |
| --- | --- |
| **$2ndUser** | The name starts with a number after the **$**. |
| **$first name** | Spaces do not belong inside a variable name. |
| **$first-name** | A hyphen is not allowed in a variable name; PHP reads it as an operator between expressions. |
| **firstName** when referring to a variable | The **$** marker is missing. |

A variable name can start with an underscore, but meaningful names are easier to understand than names such as **$_**, **$a**, or **$x** when the purpose is not obvious.

## Variable names are case-sensitive

PHP treats variable names with different letter case as different variables:

~~~php
<?php
$name = "Asha";
$Name = "Ravi";

echo $name, PHP_EOL;
echo $Name, PHP_EOL;
~~~

Here, **$name** and **$Name** are two separate names. This is legal, but names that differ only by capitalization are easy to confuse. Choose one spelling and use it consistently.

## Keywords and reserved names

A **keyword** is a word with a special meaning in PHP syntax. Examples include **if**, **function**, **class**, and **return**. PHP recognizes these words as part of the language, so they cannot be freely reused wherever a user-defined name might appear.

PHP also provides predefined identifiers, such as built-in classes, constants, variables, and other names. Reserved-word lists include several categories of predefined names and may change as PHP evolves. Avoid using language keywords or PHP-provided names for your own functions, classes, constants, and other code symbols.

For example, do not try to declare a function named **if** or a class named **class**. Choose a descriptive name such as **showSummary** or **CoursePage** instead. A variable has a different syntax because it begins with **$**, but it is still clearer to avoid keyword-looking names in your own code.

The special variable **$this** is managed by PHP in object methods and cannot be assigned like an ordinary variable. Classes and methods are introduced later in the course.

PHP may add new keywords or predefined names in future versions. A name that does not conflict today could cause a compatibility problem later. Prefer names that describe your application and avoid names that look like core language or built-in names.

## Choose names that explain their purpose

A variable name is a small piece of documentation. Prefer names that tell a reader what the value represents:

| Vague name | Clearer name |
| --- | --- |
| **$x** | **$userCount** |
| **$d** | **$dueDate** |
| **$temp** | **$pageTitle** when it contains the page title |
| **$data** | A specific name such as **$orderDetails** when that is what it stores |

Names do not have to be long. A short name can be appropriate for a very small, obvious local use, but avoid abbreviations that make readers guess.

## Naming style and consistency

PHP's variable-name syntax does not require one capitalization style. For example, both **$firstName** and **$first_name** follow the basic variable-name rules. A project should choose a consistent style so names are predictable.

This course uses **camelCase** for its example variables:

~~~php
<?php
$firstName = "Asha";
$courseTitle = "Learning PHP";
$userCount = 3;
~~~

The first word starts lowercase, and each following word begins with an uppercase letter. This is a style choice for readability, not a PHP language requirement. If you join an existing project, follow its established naming style.

Use singular names for one item, plural names for a collection, and names that match the value's meaning. Later chapters introduce naming styles for functions, classes, and constants in more detail.

## Avoid names that can cause confusion

- Do not change capitalization when referring to the same variable: **$userName** and **$username** are different names.
- Do not use spaces, hyphens, or punctuation as if they were part of one ordinary variable name.
- Avoid names that differ only by case because they are easy to mistype.
- Avoid names that are too vague to explain what the value represents.
- Avoid names that conflict with PHP keywords or predefined identifiers.
- Do not create your own symbols beginning with **__**; PHP reserves this pattern for special internal or magic functionality.
- Do not rely on clever tricks or obscure spellings when a descriptive name is available.

## Common mistakes

| Mistake | What happens | Better approach |
| --- | --- | --- |
| Writing **$2users** | The variable name begins with a digit and is invalid. | Use **$users2** or another name that starts with a letter. |
| Writing **$first-name** | PHP does not treat the hyphenated text as one variable name. | Use **$firstName** or **$first_name**. |
| Writing **name** instead of **$name** | PHP does not interpret the bare name as that variable. | Include the **$** marker when referring to a variable. |
| Using **$userName** and **$username** interchangeably | PHP treats them as different variables. | Pick one exact spelling and capitalization. |
| Naming application code after a keyword or built-in symbol | It may be rejected, conflict, or confuse readers. | Choose a distinctive name that describes the application concept. |
| Using **$x** for an important value | Readers have to search to understand its meaning. | Choose a descriptive name such as **$itemCount**. |
| Starting a custom method name with **__** | It can collide with PHP's reserved magic-name conventions. | Use a normal descriptive name unless implementing documented magic behavior. |

## Try it yourself

1. Mark the variable name and the **$** marker separately in **$firstName**.
2. Identify which of these are valid ordinary variable names: **$user**, **$_draft**, **$user2**, **$2user**, **$user-name**, and **$user name**.
3. Explain why **$name** and **$Name** refer to different variables.
4. Replace **$x**, **$d**, and **$temp** with names that describe the sample values they hold.
5. Choose either **$firstName** or **$first_name** for a small practice project and use that form consistently.
6. Name two PHP keywords and explain why they have a special role.
7. Explain why a class or function name should not reuse a PHP-provided name.
8. Look at a project you use and identify the naming style it follows before adding new variables.

## Key points

- An identifier is a name used to refer to a program element.
- An ordinary PHP variable begins with **$**, followed by a name that starts with a letter or underscore.
- After its first character, a variable name can contain letters, digits, and underscores.
- Variable names are case-sensitive; **$name** and **$Name** are different.
- Keywords and predefined PHP identifiers have special meaning; avoid reusing them for your own symbols.
- Descriptive, consistent names make code easier to read and change.
- Naming style is a convention; this course uses camelCase for example variables.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [008 - Statements, instruction separation, and comments](008-statements-instruction-separation-and-comments.md) | [Open Q&A](../qnas/009-identifiers-keywords-reserved-words-and-naming.md) | [Open example](../examples/009-identifiers-keywords-reserved-words-and-naming.md) | [010 - Variables and assignment](010-variables-and-assignment.md) |
