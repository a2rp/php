# 021 - Mixed, Void, Never, and Literal Types

PHP type declarations can express different kinds of contracts. **mixed** accepts any value, **void** says a function has no meaningful return value, **never** says it does not finish normally, and literal types such as **true**, **false**, and **null** describe one exact value.

## What you will learn

- What values **mixed** accepts and when to narrow them
- Why **void** is different from **null**
- How **never** describes a function that cannot return normally
- How **true**, **false**, and **null** can be used as literal types
- How literal types work with **bool** and union types
- Which PHP versions support these declarations
- How to choose a useful return type instead of making a contract vague

## The mixed type

The **mixed** type means that a value may have any PHP type. It is a built-in top type: it covers values such as integers, strings, arrays, objects, resources, and **null**.

~~~php
<?php
declare(strict_types=1);

function describeValue(mixed $value): string
{
    if (is_string($value)) {
        return "string: " . $value;
    }

    if (is_int($value)) {
        return "integer: " . $value;
    }

    if ($value === null) {
        return "null";
    }

    return get_debug_type($value);
}

echo describeValue("ready") . PHP_EOL;
echo describeValue(42) . PHP_EOL;
echo describeValue(null) . PHP_EOL;
~~~

Output:

~~~text
string: ready
integer: 42
null
~~~

A **mixed** parameter is useful at a boundary where the input is genuinely not known yet, such as a decoded value or a generic inspection function. Check the value before using operations that only make sense for a particular type.

### Narrow a mixed value before using it

A declaration of **mixed** does not tell PHP or the reader what operations are safe. Use checks such as **is_string()**, **is_int()**, **is_array()**, **instanceof**, or a strict comparison with **null** to establish the value's type.

~~~php
<?php
function normalizeLabel(mixed $value): string
{
    if (is_string($value)) {
        return trim($value);
    }

    if (is_int($value)) {
        return (string) $value;
    }

    return "Unknown";
}
~~~

Do not assume that a **mixed** value is a string just because most current callers pass strings. The next caller may pass **null**, an array, or an object.

### mixed versus a specific declaration

A specific type communicates more than **mixed** because it rules out unrelated values before the function body runs.

~~~php
<?php
function formatPrice(int|float $amount): string
{
    return "$" . number_format($amount, 2);
}
~~~

This function only accepts an integer or a float. Use **mixed** only when the function really must accept any PHP value.

### mixed is already the broadest type

Do not add types to **mixed** in a union. A declaration such as **mixed|string** is redundant and invalid. **mixed** already includes strings. Likewise, **mixed** cannot be used as a member of an intersection.

## The void return type

Use **void** when a function performs an action but does not promise a useful value to its caller. **void** can only be used as a return type.

~~~php
<?php
declare(strict_types=1);

function writeStatus(string $message): void
{
    echo "Status: " . $message . PHP_EOL;
}

writeStatus("Saved");
~~~

Output:

~~~text
Status: Saved
~~~

A **void** function may finish normally. PHP reaches the closing brace, or the function may use a bare **return;** to leave early.

~~~php
<?php
function writeOptionalStatus(?string $message): void
{
    if ($message === null) {
        return;
    }

    echo $message . PHP_EOL;
}
~~~

A **void** function must not return an expression. Both **return "done";** and **return null;** violate the declared contract. A **void** type also cannot appear inside a union.

### void is not null

**void** describes what a function promises about its return. **null** is a value that a function can return.

~~~php
<?php
function lookupLabel(): ?string
{
    return null;
}

function printLabel(string $label): void
{
    echo $label . PHP_EOL;
}
~~~

The first function returns either a string or the value **null**. The second function is for its side effect and declares no useful return value. If callers need a value, declare that value's type instead of **void**.

## The never return type

Use **never** when a function cannot finish normally and give control back to its caller. A **never** function must terminate by throwing an exception, calling **exit()**, or continuing forever in an infinite loop.

~~~php
<?php
declare(strict_types=1);

function failRequest(string $message): never
{
    throw new RuntimeException($message);
}
~~~

The function has no normal return path. Code that calls it can rely on execution not continuing past that call.

~~~php
<?php
function requirePositive(int $value): int
{
    if ($value <= 0) {
        failRequest("Value must be positive.");
    }

    return $value;
}
~~~

Here **failRequest()** throws when the value is invalid. For a valid value, execution continues to **return $value**.

**never** is a return-only type. It cannot be a parameter type or a member of a union. It is the bottom type: it represents no normally returned value and is compatible with any return type when overriding a method.

### never is not void

A **void** function may complete and return control to its caller, just without a useful result. A **never** function does not complete normally at all.

- Use **void** for actions such as printing, writing a log, or updating state.
- Use **never** for operations such as throwing an exception or terminating the process.
- Use **null** when the function returns the actual **null** value.

Do not mark a function **never** if there is a path that reaches its closing brace or executes a bare **return;**.

## Literal types: true, false, and null

A literal type accepts exactly one value. PHP supports **true**, **false**, and **null** as standalone types from PHP 8.2.

~~~php
<?php
declare(strict_types=1);

function featureIsEnabled(): true
{
    return true;
}

function featureIsDisabled(): false
{
    return false;
}

function noConfiguredValue(): null
{
    return null;
}
~~~

These return declarations make a narrow promise: each function can return only the one literal value named by its type.

Literal types are useful when a result is intentionally fixed, or when a function's result has a meaningful exact state. They do not replace **bool** when either boolean value is valid.

### Boolean literals and unions

If a function may return either boolean value, use **bool**, not **true|false**. PHP rejects a union containing both **true** and **false** because **bool** already expresses that complete set.

~~~php
<?php
function isFeatureEnabled(string $name): bool
{
    return $name === "search";
}
~~~

A union such as **User|false** can be useful for an API that returns a user or the exact failure value **false**. The **false** type has been permitted in unions since PHP 8.0. PHP 8.2 added **false** as a standalone type. A union such as **bool|false** is redundant.

### null literal and nullable declarations

PHP 8.2 added **null** as a standalone type. In earlier PHP versions, **null** could still appear as part of a union, such as **string|null**, or through nullable shorthand **?string**.

Use **?T** or **T|null** when a function may return either a value of type **T** or **null**. Use the standalone **null** type only when **null** is the one permitted result.

## Choosing among the types

| Intent | Suitable declaration |
| --- | --- |
| Accept any PHP value at a generic boundary | **mixed** |
| Perform an action without returning a useful result | **void** |
| Stop normally returning by throwing, exiting, or looping | **never** |
| Return either boolean value | **bool** |
| Return only the value **true** | **true** |
| Return only the value **false** | **false** |
| Return only the value **null** | **null** |
| Return a string or **null** | **?string** or **string|null** |

Make the type as specific as the function's real contract allows. A narrower contract catches mistakes sooner and helps callers understand how to use the function.

## Version support

| Feature | First supported version |
| --- | --- |
| **void** return type | PHP 7.1 |
| **mixed** type | PHP 8.0 |
| **false** as a union member | PHP 8.0 |
| **never** return type | PHP 8.1 |
| Standalone **true**, **false**, and **null** types | PHP 8.2 |

A PHP version must support every syntax used in a declaration. For example, a function returning **true** as a standalone type requires PHP 8.2 or later.

## Common mistakes

- Treating **mixed** as if it were already a string, array, or object
- Using **mixed|string** even though **mixed** already includes strings
- Declaring a function **void** when its caller needs a returned value
- Returning an expression from a **void** function
- Using **void** to mean the value **null**
- Declaring **never** for a function that can finish normally
- Using **never** or **void** as a parameter type
- Writing **true|false** instead of **bool**
- Writing **bool|false** even though the **false** case is already included
- Using standalone **true**, **false**, or **null** on PHP versions older than 8.2

## Try it yourself

1. Write a function that accepts **mixed**, then returns the value's debug type without assuming it is a string.
2. Write a **void** function that prints a short message and has an early bare **return;** for an empty message.
3. Write a **never** function that throws a **RuntimeException** with a supplied message.
4. Write separate functions returning the standalone literal types **true**, **false**, and **null**.
5. Explain why a function that can return normally must not declare **never**.
6. Choose a type for a function that returns either a string or **null**.
7. Identify the minimum PHP version for a standalone **true** return type.

## Key takeaways

- **mixed** explicitly accepts any PHP value; inspect or narrow it before type-specific operations.
- **void** is for a function that may finish but has no useful return value.
- **never** is for a function that cannot return normally.
- **true**, **false**, and **null** are standalone literal types from PHP 8.2.
- Use **bool** for either boolean value, and avoid redundant combinations.
- Prefer the narrowest type that accurately describes the function's contract.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [020 - Nullable, union, and intersection types](020-nullable-union-and-intersection-types.md) | [Open Q&A](../qnas/021-mixed-void-never-and-literal-types.md) | [Open example](../examples/021-mixed-void-never-and-literal-types.md) | [022 - Constants, predefined constants, and magic constants](022-constants-predefined-constants-and-magic-constants.md) |