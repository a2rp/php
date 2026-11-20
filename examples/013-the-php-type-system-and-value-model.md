# 013 - The PHP Type System and Value Model: Complete Example

This command-line example inspects common PHP values, changes a variable from one type to another, and compares scalar assignment with object assignment.

## Question

How can you inspect a value's type, and why can assigning an object behave differently from assigning an integer?

## Goal

Create this project:

~~~text
type-practice/
|-- README.md
|-- types.php
~~~

The **types.php** script demonstrates:

- Several common PHP value types
- How **var_dump()** displays type and value
- How an ordinary variable's current type can change
- How a copied integer stays the same when the original is reassigned
- How two variables can access the same object instance

## Complete code: types.php

Save this file at the project root:

~~~php
<?php

$isPublished = true;
$lessonNumber = 13;
$price = 19.95;
$courseTitle = "PHP type demo";
$lessonNumberText = "13";
$noValue = null;
$topics = ["PHP", "Types"];

var_dump($isPublished);
var_dump($lessonNumber);
var_dump($price);
var_dump($courseTitle);
var_dump($lessonNumberText);
var_dump($noValue);
var_dump($topics);

$changingValue = 13;
var_dump($changingValue);

$changingValue = "thirteen";
var_dump($changingValue);

$originalCount = 13;
$countCopy = $originalCount;
$originalCount = "updated";

var_dump($countCopy);

$first = new stdClass();
$first->status = "draft";

$second = $first;
$second->status = "published";

var_dump($first->status);
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Type Practice

A small PHP CLI script for inspecting values and comparing assignment behavior.

## Check and run

~~~sh
php -l types.php
php types.php
~~~

## What it demonstrates

- Common PHP types and values
- Using var_dump() while debugging
- Assigning different value types to one ordinary variable
- Copying a simple integer value
- Sharing an object instance between two variables
~~~~

This Markdown page is the README text to create in the project. The PHP source stays in **types.php**.

## Check and run the program

Open a terminal in **type-practice/** and check the syntax:

~~~sh
php -l types.php
~~~

For a valid file, PHP reports that no syntax errors were detected. Then run the script:

~~~sh
php types.php
~~~

Expected output:

~~~text
bool(true)
int(13)
float(19.95)
string(13) "PHP type demo"
string(2) "13"
NULL
array(2) {
  [0]=>
  string(3) "PHP"
  [1]=>
  string(5) "Types"
}
int(13)
string(8) "thirteen"
int(13)
string(9) "published"
~~~

## Understand the output

### Similar-looking values can have different types

**$lessonNumber** is an integer. **$lessonNumberText** is a string containing digit characters. Their values may look related, but they are not the same type.

### The variable's current type can change

**$changingValue** first contains the integer **13** and later contains the string **"thirteen"**. The later assignment replaces its value and changes its current type.

### A scalar copy keeps its value

**$countCopy** receives the integer **13** from **$originalCount**. Later assigning the string **"updated"** to **$originalCount** does not change **$countCopy**.

### Two variables can access one object instance

**$second = $first;** gives **$second** access to the same object instance. Changing the object's **status** through **$second** is visible through **$first**. This is object sharing, not an explicit PHP reference alias using **&**.

## Make a change

1. Change **$lessonNumber** from **13** to another integer and compare the output.
2. Change **$lessonNumberText** to **"thirteen"**. Use **var_dump()** to see its new string length.
3. Change **$changingValue** to a boolean, then inspect it. Explain how the variable's current type follows its assigned value.
4. Change **$originalCount** after **$countCopy** is assigned. Confirm that **$countCopy** retains its integer value.
5. Change **$second->status** to another string. Check the value through **$first->status**.
6. Create a second **stdClass** object instead of assigning **$first** to **$second**. Give it a different status and compare the two objects.
7. Remove one **var_dump()** call and predict which output line disappears.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Saying **"13"** is an integer | Quotes make it a string value. | Inspect it with **var_dump()** and convert deliberately if an integer is needed. |
| Assuming **$changingValue** must keep its first type | An ordinary variable can receive another kind of value later. | Track the current assignment or use type declarations where constraints are needed. |
| Expecting **$countCopy** to follow later assignments to **$originalCount** | The integer value was copied when the second assignment occurred. | Assign a new value to **$countCopy** explicitly if it should change. |
| Expecting **$second = $first** to create an independent object | Both variables access the same object instance. | Create a new object or use the object-copying feature taught later if an independent copy is required. |
| Calling object sharing a PHP reference alias | Shared object identity and a variable reference created with **&** are distinct concepts. | Keep object assignment and explicit references separate. |
| Leaving diagnostic output in a public page | **var_dump()** can reveal internal values and structure. | Use it during development and remove or replace it before user-facing output. |

## Key takeaways

- **var_dump()** shows both type and value.
- **13** is an integer; **"13"** is a string.
- In ordinary assignments, a variable's type follows its current value.
- Copying a simple integer gives the receiving variable its own value.
- Assigning an object lets another variable access the same instance; it does not clone the object.
- A type check does not establish whether the value is valid for the application's purpose.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [012 - Superglobals and external input](../lessons/012-superglobals-and-external-input.md) | [Read this lesson](../lessons/013-the-php-type-system-and-value-model.md) | [Open Q&A](../qnas/013-the-php-type-system-and-value-model.md) | [014 - Null, booleans, integers, and floating-point numbers](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) |
