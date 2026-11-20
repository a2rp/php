# 021 - Mixed, Void, Never, and Literal Types: Complete Example

This PHP CLI example demonstrates a generic **mixed** value, a side-effect-only **void** function, a **never** function that throws, and standalone **true**, **false**, and **null** return types.

## Question

How can a PHP program accept an unknown value safely, distinguish a function that performs an action from one that cannot return, and express an exact boolean or null result?

## Goal

Create this project:

~~~text
php-type-contracts/
|-- README.md
|-- type-contracts.php
~~~

The **type-contracts.php** script demonstrates:

- Inspecting several values accepted by a **mixed** parameter
- Narrowing a **mixed** value before formatting it
- Printing a message from a **void** function
- Throwing from a **never** function and catching that exception at the call site
- Returning exact **true**, **false**, and **null** literal values
- The PHP 8.2 minimum version required for the standalone literal declarations

## Complete code: type-contracts.php

Save this file at the project root:

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

function writeAuditMessage(string $message): void
{
    if ($message === "") {
        return;
    }

    echo "Saved: " . $message . PHP_EOL;
}

function stopWithError(string $message): never
{
    throw new RuntimeException($message);
}

function featureIsEnabled(): true
{
    return true;
}

function featureIsDisabled(): false
{
    return false;
}

function missingSetting(): null
{
    return null;
}

echo describeValue("ready") . PHP_EOL;
echo describeValue(42) . PHP_EOL;
echo describeValue(null) . PHP_EOL;

writeAuditMessage("profile updated");
writeAuditMessage("");

try {
    stopWithError("Could not connect to the service.");
    echo "This line is unreachable." . PHP_EOL;
} catch (RuntimeException $exception) {
    echo "Caught: " . $exception->getMessage() . PHP_EOL;
}

var_dump(featureIsEnabled());
var_dump(featureIsDisabled());
var_dump(missingSetting());
~~~

## Complete code: README.md

~~~~markdown
# PHP Type Contracts

This small command-line project demonstrates PHP's **mixed**, **void**, and **never** return contracts, along with standalone **true**, **false**, and **null** literal types.

## Requirements

- PHP 8.2 or later
- A terminal

PHP 8.2 is required because the script uses standalone literal return types. The **mixed** type requires PHP 8.0, and **never** requires PHP 8.1.

## Run

~~~sh
php type-contracts.php
~~~

## Expected output

~~~text
string: ready
integer: 42
null
Saved: profile updated
Caught: Could not connect to the service.
bool(true)
bool(false)
NULL
~~~
~~~~

## Expected output

~~~text
string: ready
integer: 42
null
Saved: profile updated
Caught: Could not connect to the service.
bool(true)
bool(false)
NULL
~~~

## Understand the code

### The mixed value

**describeValue()** accepts any PHP value. It checks for a string, integer, and **null** before using a generic debug type. Those checks ensure the function only performs operations that suit the value it received.

Try passing **true**, an array, or an object to the function. Each is accepted by **mixed**, and **get_debug_type()** describes values not handled by the earlier branches.

### The void function

**writeAuditMessage()** communicates through output. It has no useful result for its caller, so **void** is the appropriate return type. An empty message exits with a bare **return;**. The function never returns an expression.

### The never function

**stopWithError()** throws a **RuntimeException**. It cannot finish normally, so its return type is **never**. The exception is caught at the call site, after which the program continues in the **catch** block. The echo placed after the call in the **try** block is unreachable.

Do not replace **never** with **void** here. A function declared **void** is allowed to complete normally; this function has no normal completion path.

### Literal return types

Each literal function can return only one exact value:

- **featureIsEnabled(): true** returns only **true**.
- **featureIsDisabled(): false** returns only **false**.
- **missingSetting(): null** returns only **null**.

The final **var_dump()** calls display these values. Standalone literal declarations require PHP 8.2 or later. If a function can return either boolean value, declare **bool**.

## Make a change

1. Pass **true** to **describeValue()** and observe the debug type.
2. Add an **is_bool()** branch that returns a readable description.
3. Call **writeAuditMessage()** with another message and with an empty string.
4. Change **stopWithError()** to throw a different **RuntimeException** message and observe what the catch block prints.
5. Try returning **false** from **featureIsEnabled()**. PHP reports a **TypeError** because the declared literal type permits only **true**.
6. On a PHP version earlier than 8.2, replace the literal return declarations with compatible broader types before running the file.

## Common mistakes

- Assuming **mixed** means a value is safe to use as a string
- Returning a value from **writeAuditMessage()**
- Letting a **never** function reach its closing brace normally
- Expecting the echo after **stopWithError()** to run
- Returning **false** from a function declared to return **true**
- Using **true|false** where **bool** is the appropriate type
- Running this exact script on PHP earlier than 8.2

## Key takeaways

- **mixed** permits any input, so inspect values before type-specific work.
- **void** describes an action with no useful return value.
- **never** describes a function that cannot return normally.
- Literal types can make exact result contracts explicit.
- The complete script requires PHP 8.2 because it uses standalone literal types.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [020 - Nullable, union, and intersection types](../lessons/020-nullable-union-and-intersection-types.md) | [Read this lesson](../lessons/021-mixed-void-never-and-literal-types.md) | [Open Q&A](../qnas/021-mixed-void-never-and-literal-types.md) | [022 - Constants, predefined constants, and magic constants](../lessons/022-constants-predefined-constants-and-magic-constants.md) |