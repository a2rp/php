# 019 - Type Declarations and Strict Typing

PHP type declarations describe the values a function expects and the values it promises to return. PHP checks those declarations when code runs. By default, scalar values may be coerced when possible; **strict typing** changes how scalar declarations handle mismatched values.

## What you will learn

- How to add parameter and return type declarations to a function
- What happens when a value does not satisfy a declaration
- Which built-in and user-defined types can appear in declarations
- How PHP's default coercive mode handles scalar arguments
- How **declare(strict_types=1)** changes scalar type checks
- Why strict typing is per file and how caller and function files affect checks
- What **void**, **never**, and **mixed** mean
- What type declarations do not validate

## Parameter type declarations

A parameter declaration documents and enforces what kind of value a function accepts. Put the type before the parameter name:

~~~php
<?php
function greet(string $name): string
{
    return "Hello, " . $name;
}

echo greet("Asha") . PHP_EOL;
~~~

Output:

~~~text
Hello, Asha
~~~

The function expects a string argument and declares that it returns a string. If a value cannot satisfy a declaration, PHP throws a **TypeError**.

~~~php
<?php
function greet(string $name): string
{
    return "Hello, " . $name;
}

greet([]);
~~~

This call throws a **TypeError** because an array is not a string and cannot be coerced to the declared scalar type.

Type declarations are useful contracts at function boundaries. They do not replace input validation: a string can still be empty, too long, or outside the allowed set for your application.

## Return type declarations

Write the return type after the closing parenthesis and a colon:

~~~php
<?php
function add(int $left, int $right): int
{
    return $left + $right;
}

var_dump(add(4, 5));
~~~

Output:

~~~text
int(9)
~~~

PHP checks the value returned by the function. If it does not satisfy the declared return type under the active typing rules, PHP throws a **TypeError**.

### The void return type

Use **void** when a function performs an action and should not return a value:

~~~php
<?php
function showMessage(string $message): void
{
    echo $message . PHP_EOL;
}

showMessage("Saved");
~~~

Output:

~~~text
Saved
~~~

A **void** function must not return a value. It may finish without a **return** statement or use **return;** to finish early.

### The never return type

**never** describes a function that does not finish normally. It must not return to its caller; it can throw an exception or terminate execution. **never** is available as a return type in PHP 8.1 and later.

~~~php
<?php
function stopRequest(): never
{
    exit("Request stopped." . PHP_EOL);
}

stopRequest();
~~~

The script ends with the message. A function declared **never** cannot return a value or reach the end of its body normally.

### The mixed type

**mixed** accepts a value of any type. It can be useful at a boundary where the value is intentionally unrestricted, but it provides less information to callers than a specific type. The **mixed** type is available in PHP 8.0 and later.

~~~php
<?php
function inspectValue(mixed $value): void
{
    var_dump($value);
}

inspectValue(42);
inspectValue("forty-two");
~~~

Output:

~~~text
int(42)
string(9) "forty-two"
~~~

Prefer a more specific declaration when the function has a clear expectation.

## Common declaration types

PHP supports scalar and compound types in user-defined function declarations. Common types include:

| Type | What it accepts |
| --- | --- |
| **int** | An integer |
| **float** | A floating-point number; an integer is also accepted for a float declaration |
| **string** | A string |
| **bool** | A boolean |
| **array** | An array |
| **object** | An object |
| **callable** | A value PHP can invoke |
| **iterable** | An array or an object implementing **Traversable** |
| A class or interface name | An object of that class or an implementing class |

Some return-only types and special cases have their own rules:

- **void** means no value is returned.
- **never** means the function does not complete normally; it is available from PHP 8.1.
- **mixed** accepts any value; it is available from PHP 8.0.
- **resource** cannot be used as a user-defined parameter or return type declaration.

Use the actual scalar type names **int**, **float**, **string**, and **bool**. Aliases such as **integer**, **double**, and **boolean** are not supported as scalar type declarations.

Nullable and union types let a declaration accept more than one possibility. They are covered in the next chapter.

## Coercive typing by default

Without strict mode, PHP uses **coercive typing** for scalar declarations when it can convert an argument to the declared scalar type:

~~~php
<?php
function add(int $left, int $right): int
{
    return $left + $right;
}

var_dump(add("4", 5));
~~~

Output:

~~~text
int(9)
~~~

The string **"4"** is converted to an integer for the call. If PHP cannot coerce an argument to a declared type, it throws a **TypeError**.

Coercion is not the same as validation. For example, accepting **"4"** as an integer does not prove that four is a valid quantity for your application.

## Enable strict typing

Add **declare(strict_types=1);** near the beginning of a PHP file to enable strict scalar type checking for that file:

~~~php
<?php
declare(strict_types=1);

function add(int $left, int $right): int
{
    return $left + $right;
}

var_dump(add(4, 5));

try {
    add("4", 5);
} catch (TypeError $error) {
    echo "The string argument was rejected." . PHP_EOL;
}
~~~

Output:

~~~text
int(9)
The string argument was rejected.
~~~

The declaration must be the first statement in the file. Put it immediately after the opening PHP tag, before executable code. Comments may appear around it, but no other statement should precede it.

Strict mode rejects a string where an **int** is required, even when the string contains digits. One exception is that an integer may be passed to a **float** parameter:

~~~php
<?php
declare(strict_types=1);

function showRate(float $rate): void
{
    echo "Rate: " . $rate . PHP_EOL;
}

showRate(5);
~~~

Output:

~~~text
Rate: 5
~~~

## Strict typing is per file

Strict mode is not a project-wide switch. Each PHP file has its own **strict_types** declaration.

- For a scalar parameter, the mode in effect in the file making the function call determines whether an argument is coerced or strictly checked.
- For a scalar return type, the mode in the file where the function is defined determines how its returned value is checked.
- A declaration in one file does not set the mode for other files that include it.
- Calls made from inside PHP's internal functions are not affected by the caller file's **strict_types** declaration.

This means a function can be declared in one file and called from another with a different scalar argument mode. Keep strictness and declarations consistent across your application to make behavior easier to understand.

Strict typing only changes checks for scalar type declarations. It does not turn off type declarations for arrays or objects, and it does not change ordinary variable assignment, casts, loose comparisons, or application-level validation.

## Function type declarations are not variable declarations

PHP does not use this syntax to declare a local variable's type:

~~~php
<?php
// This is not valid PHP syntax:
// int $count = 3;
~~~

Instead, assign values to variables normally. Function parameter and return declarations describe function boundaries. Typed properties are a separate feature covered with classes later in the course.

## Type declarations and errors

A declaration violation produces a **TypeError**. It is an error object, not a false return value. If your program can reasonably recover from invalid input, catch the error at an appropriate boundary. Prefer checking and validating ordinary user input before it reaches a function that requires a specific type.

This chapter introduces parameter and return declarations. The next chapter covers nullable, union, and intersection types, which let signatures express more combinations.

## Try it yourself

1. Add an **int** parameter and an **int** return type to an addition function.
2. Call the function with an integer and with the numeric string **"4"** in a file without strict mode. Compare the results.
3. Add **declare(strict_types=1);** to the file and catch the **TypeError** from passing **"4"**.
4. Write a function that accepts a **string** and returns a greeting as a **string**.
5. Write a **void** function that prints a message without returning a value.
6. Write a function with a **float** parameter and call it with an integer in strict mode.
7. Explain why a strict declaration in a function's definition file does not set the mode for every file that calls it.
8. Describe when **never** is appropriate and why it is not interchangeable with **void**.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming declarations silently convert every invalid value | A value that cannot satisfy a declaration causes a **TypeError**. | Match the type or deliberately validate and convert input before the call. |
| Assuming a numeric string is always accepted for an **int** parameter | Strict scalar mode rejects the string, even if it contains digits. | Pass an integer or explicitly parse and validate the input. |
| Thinking **strict_types** applies to the whole project | The setting is per file. | Add it consistently to files where strict scalar checks are desired. |
| Thinking the function's file controls scalar argument coercion | The calling file controls argument checks. | Check the mode at the call site. |
| Thinking the caller controls a function's return check | Return checking follows the function's definition file. | Keep return values consistent with the function's declared contract. |
| Returning a value from a **void** function | A void function is not allowed to return a value. | Use **return;** to exit early, or declare the actual return type. |
| Declaring **never** for a function that returns normally | A **never** function must not complete normally. | Use **never** only for code that always throws or terminates. |
| Using **mixed** when a narrower type is known | It hides useful information from callers and tools. | Declare the most specific useful type. |
| Treating a type declaration as input validation | A correct type does not prove that a value is meaningful or allowed. | Validate format, range, and business rules separately. |
| Using **integer** or **boolean** as scalar type aliases | Scalar declaration aliases are not supported. | Use **int** and **bool**. |

## Key takeaways

- Parameter declarations state what a function accepts; return declarations state what it returns.
- PHP checks declarations at runtime and throws **TypeError** when a value does not satisfy them.
- Scalar declarations are coercive by default.
- **declare(strict_types=1);** enables strict scalar checks per file; an integer can still satisfy a float declaration.
- Scalar argument checks depend on the calling file; scalar return checks depend on the function's definition file.
- **void**, **never**, and **mixed** describe different return or value contracts.
- Type declarations do not validate the meaning or allowed range of application input.
- Nullable, union, and intersection types are covered in the next chapter.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [018 - Type juggling, casting, and conversion](018-type-juggling-casting-and-conversion.md) | [Open Q&A](../qnas/019-type-declarations-and-strict-typing.md) | [Open example](../examples/019-type-declarations-and-strict-typing.md) | [020 - Nullable, union, and intersection types](020-nullable-union-and-intersection-types.md) |
