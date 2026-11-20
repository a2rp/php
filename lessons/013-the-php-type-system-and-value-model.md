# 013 - The PHP Type System and Value Model

Every PHP expression has a type, and every value has a value model that describes how it behaves when assigned, passed, or used. Learning these ideas makes later topics such as conversions, arrays, objects, and type declarations easier to understand.

## What you will learn

- What a type tells you about a value
- How a variable's current value determines its type
- The difference between values that look similar but have different types
- Common PHP value families
- How **var_dump()** can show a value and its type
- Why assignment behavior is simple for scalar values but needs care for objects
- What PHP's dynamic typing does and does not mean

## A value has a type

A type describes what kind of value an expression produces and which operations make sense for it. For example, the number **13** is an integer, while the text **"13"** is a string:

~~~php
<?php
$lessonCount = 13;
$lessonCountText = "13";

var_dump($lessonCount);
var_dump($lessonCountText);
~~~

The output shows different types even though the values contain similar characters:

~~~text
int(13)
string(2) "13"
~~~

The quotes matter: a quoted number is text. The type can affect which operations are valid and how PHP treats a value in a particular context.

## PHP is dynamically typed

In an ordinary assignment, PHP determines a variable's type from the value assigned to it. You do not have to declare a variable's type first:

~~~php
<?php
$value = 13;
var_dump($value);

$value = "ready";
var_dump($value);
~~~

The variable first refers to an integer and then to a string:

~~~text
int(13)
string(5) "ready"
~~~

This is dynamic typing. It does not mean that PHP has no type checks or that every conversion is safe. Functions, properties, and other parts of a program can use type declarations to constrain accepted values; those declarations are covered later.

PHP can also convert values automatically in some contexts. That behavior is called **type juggling**. It depends on the operation, and a conversion that happens in one context might not happen in another. Learn the actual types first; automatic conversions and explicit casts are covered in a later chapter.

## Common value families

PHP's built-in types can be introduced in a few practical groups. This is an overview; dedicated chapters explain each family.

| Family | Types | Beginner description |
| --- | --- | --- |
| Scalar values | **bool**, **int**, **float**, **string** | One simple value such as true/false, a whole number, a decimal number, or text. |
| Collections and instances | **array**, **object** | A value that groups data, or an instance created from a class. |
| Special values | **null**, **resource** | **null** represents no value; a resource represents access to an external resource managed by PHP. |
| Value roles used by PHP | **callable**, **iterable** | A callable can be invoked; an iterable can be traversed. More detailed rules appear later. |

The next chapter introduces **null**, booleans, integers, and floating-point numbers. Strings, arrays, objects, resources, callables, and iterables each receive their own lessons later.

## Inspect a value with **var_dump()**

During development, **var_dump()** displays structured information about an expression, including its type and value:

~~~php
<?php
$isPublished = false;
$price = 19.95;
$title = "PHP types";

var_dump($isPublished);
var_dump($price);
var_dump($title);
~~~

Example output:

~~~text
bool(false)
float(19.95)
string(9) "PHP types"
~~~

Use **var_dump()** as a debugging tool. It writes diagnostic output directly, so do not leave it in a page intended for users or use it to reveal private data.

PHP also provides functions such as **is_int()**, **is_string()**, and **is_bool()** to check for specific types. A type check tells you what a value is; it does not, by itself, prove that the value is valid for your application.

## Values and assignment

For a simple scalar value, ordinary assignment gives another variable its own value:

~~~php
<?php
$originalCount = 13;
$copiedCount = $originalCount;

$originalCount = "updated";

var_dump($originalCount);
var_dump($copiedCount);
~~~

The output is:

~~~text
string(7) "updated"
int(13)
~~~

The later assignment changes **$originalCount**. It does not change the copied integer in **$copiedCount**. Arrays also use value semantics for ordinary assignment, although PHP can optimize the underlying storage. Array contents and edge cases are explained in the array chapter.

Objects behave differently from simple scalar values. Assigning an object to another variable gives the second variable access to the same object instance; it does not automatically clone the object:

~~~php
<?php
$first = new stdClass();
$first->status = "draft";

$second = $first;
$second->status = "published";

var_dump($first->status);
~~~

The output is:

~~~text
string(9) "published"
~~~

Both variables can access the same object instance. This does not mean **$first** and **$second** are PHP references created with **&**. Object identity, cloning, and explicit references are separate topics covered later.

## Type declarations add constraints

PHP permits dynamic values in ordinary untyped variables, but it also supports type declarations. A declaration can specify the type a function expects or returns, or constrain a property:

~~~php
function formatLessonCount(int $count): string
{
    return "Lessons: " . $count;
}
~~~

This example previews a parameter type and return type. The details of declarations, strict typing, and conversion behavior are taught in later chapters. The key idea here is that dynamic typing and type declarations coexist in PHP.

## Type, value, and meaning are different

A type says what kind of value PHP currently has. The value is the data itself. The meaning is what your program decides the data represents.

For example:

- **13** is an integer value.
- **"13"** is a string value.
- Either could represent a lesson number if the application interprets it that way, but the string must be validated or converted before numeric operations are appropriate.
- A valid type does not prove that a value is allowed; an integer such as **-500** may still be an invalid lesson count.

Keep these questions separate:

1. What type is this value?
2. What value does it contain?
3. Is that value valid for this application and operation?

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Treating **13** and **"13"** as the same type | One is an integer and one is text. | Check the type and convert deliberately when a numeric value is required. |
| Assuming a variable keeps one type forever | An ordinary variable can be assigned a value of another type. | Track the current value and use type declarations where constraints are needed. |
| Assuming dynamic typing means PHP never checks types | Operations and declarations can produce type checks or errors. | Learn each operation's rules and use declared types deliberately. |
| Depending on an automatic conversion without checking its context | Type juggling depends on the operation and can be surprising. | Validate and convert values explicitly when the application needs a specific type. |
| Assuming every assigned object becomes an independent copy | Another variable can refer to the same object instance. | Learn object assignment and use cloning when an independent object copy is required. |
| Treating a valid type as proof that a value is acceptable | A value can have the right type but still be out of range or otherwise invalid. | Validate the value against the application's rules as well as checking its type. |
| Leaving **var_dump()** in a public page | Debug output can expose data and disrupt page output. | Remove diagnostic output or use an appropriate development-only logger. |

## Try it yourself

1. Assign **13** to one variable and **"13"** to another. Use **var_dump()** to compare their types.
2. Assign an integer to a variable, inspect it, then assign a string to the same variable and inspect it again.
3. Check a value with **is_int()** and another with **is_string()**. Explain what each check tells you.
4. Assign a simple integer to a second variable, change the original, and inspect both values.
5. Create a **stdClass** object, assign it to a second variable, change a property through the second variable, and inspect that property through the first.
6. Explain why changing a variable's type and changing the value's meaning are not the same thing.
7. Give an example of an integer that has the correct type but is invalid for a lesson count.
8. Explain why **var_dump()** is useful during development but should not expose private values to users.

## Key points

- Every PHP expression has a runtime type.
- Ordinary variables do not need type declarations; their current value determines their type.
- A quoted number is a string, not an integer.
- PHP is dynamically typed, and type declarations can still add constraints in selected contexts.
- Automatic type juggling is context-dependent and is covered later.
- Simple scalar assignment gives a separate value; object assignment can give two variables access to the same instance.
- **var_dump()** shows types and values for debugging.
- A correct type does not guarantee that a value is valid for the application.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [012 - Superglobals and external input](012-superglobals-and-external-input.md) | [Open Q&A](../qnas/013-the-php-type-system-and-value-model.md) | [Open example](../examples/013-the-php-type-system-and-value-model.md) | [014 - Null, booleans, integers, and floating-point numbers](014-null-booleans-integers-and-floating-point-numbers.md) |
