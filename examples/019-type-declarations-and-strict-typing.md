# 019 - Type Declarations and Strict Typing: Complete Example

This command-line example uses typed parameters and return values in a strict PHP file. It demonstrates a successful call, the integer-to-float exception, a **void** function, a broad **mixed** parameter, and caught **TypeError** exceptions for an invalid argument and return value.

## Question

How do parameter types, return types, and strict scalar checking change what a PHP function accepts and returns?

## Goal

Create this project:

~~~text
strict-typing-practice/
|-- README.md
|-- types.php
~~~

The **types.php** script demonstrates:

- An integer parameter and integer return type
- A string parameter and string return type
- An integer passed to a float parameter in strict mode
- A **void** function that prints a message
- A **mixed** parameter that can receive different value types
- A **TypeError** caught when a string is passed to an integer parameter
- A **TypeError** caught when a function returns the wrong type
- An array parameter and integer return type

## Complete code: types.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

function add(int $left, int $right): int
{
    return $left + $right;
}

function greet(string $name): string
{
    return "Hello, " . $name;
}

function showRate(float $rate): void
{
    echo "Rate: " . $rate . PHP_EOL;
}

function showMessage(string $message): void
{
    echo $message . PHP_EOL;
}

function inspectValue(mixed $value): void
{
    var_dump($value);
}

function countEntries(array $values): int
{
    return count($values);
}

function getInvalidCount(): int
{
    return 2.5;
}

echo "Sum: " . add(4, 5) . PHP_EOL;
echo greet("Asha") . PHP_EOL;

showRate(5);
showMessage("Saved");

echo "Flexible values:" . PHP_EOL;
inspectValue(42);
inspectValue("demo");

try {
    add("4", 5);
} catch (TypeError $error) {
    echo "Rejected mismatched argument." . PHP_EOL;
}

try {
    getInvalidCount();
} catch (TypeError $error) {
    echo "Rejected mismatched return." . PHP_EOL;
}

echo "Count: " . countEntries(["first", "second"]) . PHP_EOL;
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Strict Typing Practice

A small PHP CLI script for practicing function parameter declarations, return types, and strict scalar checks.

## Requirements

- PHP 8.0 or later
- PHP CLI available as **php** in the terminal

## Check and run

~~~sh
php -l types.php
php types.php
~~~

The first command checks PHP syntax. The second runs the script.

## Expected output

~~~text
Sum: 9
Hello, Asha
Rate: 5
Saved
Flexible values:
int(42)
string(4) "demo"
Rejected mismatched argument.
Rejected mismatched return.
Count: 2
~~~

## What the script demonstrates

- **declare(strict_types=1);** enables strict scalar checks for calls made in this file.
- **add()** accepts integers and returns an integer.
- **greet()** accepts a string and returns a string.
- **showRate()** accepts an integer for its **float** parameter, the allowed int-to-float exception in strict mode.
- **showMessage()** returns no value because its return type is **void**.
- **inspectValue()** uses **mixed**, so it accepts different value types.
- Passing **"4"** to **add()** raises a **TypeError** in this strict call; the script catches it.
- **getInvalidCount()** returns a float despite declaring **int**. The strict return check raises a **TypeError**, which the script catches.
- **countEntries()** requires an array and returns an integer.

The argument check is controlled by the file making a function call. A return value's scalar check is controlled by the file where that function is defined.

## Experiment

1. Change **add(4, 5)** to **add("4", 5)**. The exception is caught and the script continues.
2. Change **showRate(5)** to **showRate(5.5)**. The float parameter accepts the float value.
3. Change **getInvalidCount()** so it returns an integer. The return exception no longer occurs.
4. Change **countEntries()** to accept a string instead of an array. Strict mode rejects that argument.
5. Remove **declare(strict_types=1);** and compare scalar argument coercion. Keep in mind that each file has its own typing mode.

## Important distinction

Strict typing enforces declared scalar types; it does not validate business meaning. For example, the **int** value **-5** satisfies an integer declaration, even if a particular application only allows positive counts. Add application-specific validation where needed.
~~~~

## Understand the output

### Matching arguments and return types succeed

**add()** receives two integers and returns an integer. **greet()** accepts and returns a string. Their calls satisfy their declarations.

### An integer can satisfy a float declaration

**showRate()** declares a **float** parameter but is called with integer **5**. PHP explicitly allows this int-to-float case in strict mode.

### void functions perform actions

**showMessage()** prints its argument and has no return value. Its **void** declaration describes that behavior.

### mixed accepts different types

**inspectValue()** can receive both integer **42** and string **"demo"**. The **mixed** declaration is broad; use a more specific type whenever the function's actual contract is narrower.

### The invalid argument is caught

The call **add("4", 5)** passes a string to an integer parameter from a strict file. PHP throws **TypeError**, and the **catch** block prints a clear message so the script continues.

### The invalid return value is caught

**getInvalidCount()** declares an integer result but returns **2.5**. Because the function is defined in this strict file, its scalar return check throws **TypeError**. The second **catch** block handles that error.

### An array declaration checks the container type

**countEntries()** requires an array. It returns the number of entries as an integer. The declaration does not check the meaning of each array item.

## Make a change

1. Add a **float** parameter to an arithmetic function and call it with both an integer and a float.
2. Add a function returning **bool** and return true or false based on a condition.
3. Change **showMessage()** so it returns a string, then update its return declaration and call site.
4. Call **countEntries()** with a string inside a **try/catch** and observe the **TypeError**.
5. Change **getInvalidCount()** to return a whole number and compare the output.
6. Pass a negative integer to **countEntries()** only after changing the function's meaning; note that type declarations do not enforce positive ranges.
7. Move a function into another file and reason about which file controls scalar argument and return checks.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Passing the wrong scalar type from a strict file | PHP throws **TypeError** instead of coercing the argument. | Validate or deliberately convert input before the call. |
| Returning a value different from the declared type | PHP checks the returned value and throws **TypeError** when it does not satisfy the declaration. | Return the declared type or change the declaration to match the contract. |
| Assuming strict typing is project-wide | The directive applies per file. | Add it consistently where strict scalar checks are desired. |
| Assuming the function's file controls argument checks | The calling file controls scalar argument coercion. | Check the mode at the call site. |
| Assuming the caller controls return checks | The function's definition file controls scalar return checks. | Keep the function body and its return declaration consistent. |
| Returning a value from a **void** function | A void function is not allowed to return a value. | Use **return;** for an early exit or declare the actual return type. |
| Using **mixed** when the value type is known | It weakens the function's contract. | Prefer a specific type. |
| Assuming an integer declaration means a positive integer | The declaration checks type, not range or meaning. | Add explicit validation for the allowed range. |

## Key takeaways

- Type declarations describe function arguments and return values.
- PHP throws **TypeError** when a value fails a declaration.
- **declare(strict_types=1);** enables strict scalar checks in that file.
- An integer can be supplied for a **float** parameter in strict mode.
- Argument strictness follows the calling file; return strictness follows the definition file.
- **void** means no returned value; **mixed** accepts any value.
- Type declarations enforce types, not application-specific rules.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [018 - Type juggling, casting, and conversion](../lessons/018-type-juggling-casting-and-conversion.md) | [Read this lesson](../lessons/019-type-declarations-and-strict-typing.md) | [Open Q&A](../qnas/019-type-declarations-and-strict-typing.md) | [020 - Nullable, union, and intersection types](../lessons/020-nullable-union-and-intersection-types.md) |
