# 013 - The PHP Type System and Value Model: Questions and Answers

Use this Q&A to review [013 - The PHP Type System and Value Model](../lessons/013-the-php-type-system-and-value-model.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does a value's type describe?

A. The name of the PHP file that contains it<br>
B. What kind of value it is and which operations may apply<br>
C. How many times it has been printed<br>
D. Whether a user has permission to change it

**Answer: B.** A type identifies the kind of value an expression produces. The type affects the operations PHP can perform with it.

### 2. What is the type of the literal **42**?

A. **string**<br>
B. **bool**<br>
C. **int**<br>
D. **null**

**Answer: C.** An unquoted whole-number literal such as **42** is an integer.

### 3. What is the type of **"42"**?

A. **int**<br>
B. **string**<br>
C. **float**<br>
D. **resource**

**Answer: B.** The quotes make **"42"** text. Similar-looking characters do not make it the same type as the integer **42**.

### 4. Which list contains PHP's scalar types?

A. **bool**, **int**, **float**, **string**<br>
B. **array**, **object**, **resource**, **null**<br>
C. **GET**, **POST**, **COOKIE**, **SESSION**<br>
D. **class**, **function**, **namespace**, **trait**

**Answer: A.** PHP's scalar types are boolean, integer, floating-point number, and string.

### 5. What type does **var_dump()** display in addition to the value?

A. The source file's location only<br>
B. The value's type<br>
C. The variable's access level<br>
D. The HTTP method

**Answer: B.** **var_dump()** displays structured information that includes the type and value.

### 6. What does it mean that PHP is dynamically typed?

A. PHP variables cannot change values<br>
B. In ordinary assignments, the current value determines the variable's type at runtime<br>
C. Every variable must be declared as a string<br>
D. PHP never checks types

**Answer: B.** Ordinary variables do not need a type declaration; their current assigned value determines their type. PHP still performs checks in operations and supports type declarations.

### 7. What happens after these assignments?

~~~php
$value = 42;
$value = "ready";
~~~

A. **$value** remains an integer<br>
B. **$value** becomes a string<br>
C. PHP creates a constant<br>
D. PHP rejects every reassignment to another type

**Answer: B.** The second assignment replaces the current value with a string, so **$value** now has type **string**.

### 8. Which statement about automatic type conversion is accurate?

A. It always happens in exactly the same way in every context<br>
B. It depends on the operation or context<br>
C. It makes every invalid value safe<br>
D. It means strings and integers are identical

**Answer: B.** PHP can perform type juggling in some contexts, but the rules depend on the operation. Do not rely on an assumed conversion.

### 9. What is the best use of **var_dump()**?

A. Display private variables to every website visitor<br>
B. Inspect types and values while debugging<br>
C. Validate every input automatically<br>
D. Convert any value to a safe SQL query

**Answer: B.** **var_dump()** is a diagnostic tool. Its output can reveal information, so it should not be left in user-facing pages.

### 10. What usually happens when an object is assigned to another variable?

A. The object is always deeply cloned<br>
B. Both variables can access the same object instance<br>
C. PHP converts it to a string<br>
D. The second variable is set to **null**

**Answer: B.** The second variable receives a copy of the object's identifier and can access the same instance. An explicit clone is a separate operation.

### 11. Does assigning an object to another variable automatically create a PHP reference alias using **&**?

A. Yes, every object assignment uses **&** internally as a PHP reference<br>
B. No, both variables can share an object instance without being reference aliases<br>
C. Yes, but only for **stdClass**<br>
D. No, because objects cannot be assigned

**Answer: B.** Object variables can contain identifiers for the same object instance. That is distinct from an explicit PHP reference alias created with **&**.

### 12. What does **null** represent?

A. A decimal number<br>
B. The absence of a value<br>
C. A function name<br>
D. An uploaded file

**Answer: B.** **null** is a special value representing no value. Its details are introduced in the next chapter.

## True or false

### 13. A variable assigned an integer must remain an integer for the rest of the script.

**Answer: False.** In ordinary untyped assignments, a later assignment can give the variable a value of another type.

### 14. A string containing digits is automatically an integer in every PHP context.

**Answer: False.** **"42"** is a string. PHP may convert values in some contexts, but conversion depends on the operation and should not be assumed.

### 15. A value with the correct type is always valid for the application's purpose.

**Answer: False.** An integer can still be out of range or otherwise invalid for the application. Type checks and application validation answer different questions.

### 16. An ordinary assignment of an integer to another variable gives the receiving variable a value that remains independent if the original variable is reassigned.

**Answer: True.** The copied integer value is unchanged when the original variable later receives a different value.

### 17. If two variables access the same object instance, they must be explicit PHP reference aliases.

**Answer: False.** Object assignment copies the object's identifier, so both variables can access the same instance without being reference aliases.

### 18. **var_dump()** output should normally be hidden from public users.

**Answer: True.** Diagnostic output may expose internal or private data and is not intended as a user-facing display format.

## Short-answer questions

### 19. Explain the difference between a type and a value.

**Answer:** The type describes what kind of data an expression produces; the value is the actual data. For example, **int** is a type and **42** is an integer value.

### 20. What is the difference between a type and the meaning your application gives a value?

**Answer:** The type describes the value's PHP representation. The meaning is the application's interpretation. The integer **-500** has type **int**, but it might not be an acceptable lesson count.

### 21. Why does a type declaration not contradict PHP's dynamic typing?

**Answer:** Dynamic typing describes ordinary values and variables at runtime. PHP also lets developers add type declarations to constrain parameters, return values, properties, and other contexts. The concepts work together.

### 22. What do **callable** and **iterable** roughly describe?

**Answer:** A callable is a value that can be invoked, and an iterable is a value that can be traversed. PHP's type system uses these concepts in declarations; their details come later.

### 23. What is a resource?

**Answer:** A resource is a special PHP value representing access to an external resource managed by PHP, such as an open resource handle. Resource types and their cleanup are covered later.

### 24. Why should code validate a value even after checking its type?

**Answer:** Type checking only confirms the kind of value. Application rules may impose further conditions, such as an allowed range, permitted format, or authorization requirement.

## Read and predict

### 25. What does this code print?

~~~php
$value = 42;
var_dump($value);

$value = "ready";
var_dump($value);
~~~

**Answer:**

~~~text
int(42)
string(5) "ready"
~~~

The variable has an integer value after the first assignment and a string value after the second.

### 26. What does this code print?

~~~php
$original = 13;
$copy = $original;
$original = "changed";

var_dump($copy);
~~~

**Answer:**

~~~text
int(13)
~~~

The receiving variable got the integer value **13**. Reassigning **$original** does not change **$copy**.

### 27. What does this code print?

~~~php
$first = new stdClass();
$first->status = "draft";

$second = $first;
$second->status = "published";

var_dump($first->status);
~~~

**Answer:**

~~~text
string(9) "published"
~~~

Both variables access the same object instance, so changing its **status** property through **$second** is visible through **$first**.

### 28. What does this code check, and what does it not check?

~~~php
if (is_int($lessonCount)) {
    echo "The value is an integer.";
}
~~~

**Answer:** It checks whether **$lessonCount** is currently an integer. It does not check whether that integer is a valid lesson count or within an allowed range.

## Find and correct the mistake

### 29. A learner says **$number = "42";** stores an integer because the text contains digits. What is the correction?

**Answer:** It stores a string because the value is quoted. If the application needs an integer, validate and convert it deliberately rather than assuming PHP will convert it in every context.

### 30. Why is this not a safe way to show diagnostic details on a public page?

~~~php
var_dump($userRecord);
~~~

**Answer:** **var_dump()** prints details about the value, which may include private information or internal structure. Use it only during debugging, and remove it or replace it with deliberate, safe output.

### 31. What is incorrect about saying “PHP is dynamically typed, so types never matter”?

**Answer:** Dynamic typing means ordinary variables get their current type from their values without requiring a declaration first. Types still affect operations, conversions, validation, and type declarations; incompatible use can still cause errors.

## Coding practice

### 32. Write code that inspects a boolean, an integer, a float, and a string.

**Answer:**

~~~php
<?php

$isPublished = false;
$lessonCount = 13;
$price = 19.95;
$courseTitle = "PHP Fundamentals";

var_dump($isPublished);
var_dump($lessonCount);
var_dump($price);
var_dump($courseTitle);
~~~

**var_dump()** displays the type and value for each expression.

### 33. Write code that assigns one integer to a second variable, updates the first variable, and then inspects the second.

**Answer:**

~~~php
$originalCount = 13;
$copiedCount = $originalCount;

$originalCount = 14;

var_dump($originalCount);
var_dump($copiedCount);
~~~

The output shows **14** for **$originalCount** and **13** for **$copiedCount**.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [012 - Superglobals and external input](../lessons/012-superglobals-and-external-input.md) | [Read this lesson](../lessons/013-the-php-type-system-and-value-model.md) | [Open example](../examples/013-the-php-type-system-and-value-model.md) | [014 - Null, booleans, integers, and floating-point numbers](../lessons/014-null-booleans-integers-and-floating-point-numbers.md) |
