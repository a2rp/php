# 021 - Mixed, Void, Never, and Literal Types: Questions and Answers

Use this Q&A to review [021 - Mixed, Void, Never, and Literal Types](../lessons/021-mixed-void-never-and-literal-types.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does the **mixed** type accept?

A. Only scalar values<br>
B. Any PHP value<br>
C. Only objects<br>
D. Any value except **null**

**Answer: B.** **mixed** is PHP's broad top type and includes values of every PHP type, including **null**.

### 2. Where may **void** be used?

A. As a parameter type only<br>
B. As a return type only<br>
C. As a property type only<br>
D. As a union member

**Answer: B.** **void** is a return-only declaration.

### 3. What does **void** promise?

A. The function throws an exception<br>
B. The function performs no operations<br>
C. The function has no useful return value<br>
D. The function returns the value **false**

**Answer: C.** A **void** function can perform work and finish normally; it simply does not promise a meaningful result.

### 4. Which statement is allowed inside a **void** function for an early exit?

A. **return null;**<br>
B. **return "done";**<br>
C. **return;**<br>
D. **return false;**

**Answer: C.** A bare **return;** is allowed. A return expression violates the **void** contract.

### 5. What does **never** mean as a return type?

A. The function returns **null**<br>
B. The function cannot return normally<br>
C. The function returns either **true** or **false**<br>
D. The function is optional

**Answer: B.** A **never** function throws, exits, or loops forever, so it never reaches a normal return to its caller.

### 6. Which is a valid way for a **never** function to terminate?

A. Reach the closing brace normally<br>
B. Execute **return;**<br>
C. Throw an exception<br>
D. Return **null**

**Answer: C.** Throwing an exception ends the function without a normal return. Calling **exit()** or looping forever also satisfies the contract.

### 7. Which type should describe a function that prints a message and finishes?

A. **never**<br>
B. **void**<br>
C. **null**<br>
D. **false**

**Answer: B.** The function completes normally but has no useful return value, which is the purpose of **void**.

### 8. Which type should describe a function that returns only the exact value **true**?

A. **bool**<br>
B. **mixed**<br>
C. **true**<br>
D. **void**

**Answer: C.** The literal type **true** restricts the result to that one value.

### 9. Since which PHP version can **mixed** be used?

A. PHP 7.0<br>
B. PHP 7.4<br>
C. PHP 8.0<br>
D. PHP 8.2

**Answer: C.** PHP added the **mixed** type in PHP 8.0.

### 10. Since which PHP version is **never** available as a return type?

A. PHP 7.1<br>
B. PHP 8.0<br>
C. PHP 8.1<br>
D. PHP 8.2

**Answer: C.** PHP 8.1 introduced the **never** return type.

### 11. Which declaration is redundant and invalid?

A. **int|string**<br>
B. **bool|null**<br>
C. **mixed|string**<br>
D. **User|false**

**Answer: C.** **mixed** already includes strings, so adding **string** does not make the type more precise.

### 12. If a function may return either boolean value, which type should it use?

A. **true|false**<br>
B. **bool**<br>
C. **void**<br>
D. **never**

**Answer: B.** **bool** already represents both boolean values. PHP does not allow **true|false** as a union.

### 13. Which declaration can accept a **User** object or the exact value **false**?

A. **User&false**<br>
B. **User|false**<br>
C. **User|void**<br>
D. **User|never**

**Answer: B.** The union means a value may match either member. **false** has been supported as a union member since PHP 8.0.

### 14. What is the minimum PHP version for a standalone **null** return type?

A. PHP 7.1<br>
B. PHP 8.0<br>
C. PHP 8.1<br>
D. PHP 8.2

**Answer: D.** PHP 8.2 added **null** as a standalone type. It could appear in unions before that.

### 15. What is the minimum PHP version for a standalone **true** return type?

A. PHP 7.4<br>
B. PHP 8.0<br>
C. PHP 8.1<br>
D. PHP 8.2

**Answer: D.** Standalone **true** is available from PHP 8.2.

## Short-answer questions

### 16. Why is **mixed** useful at a generic boundary?

**Answer:** It explicitly says the function may receive any PHP value.

**Explanation:** This can be useful for generic inspection or untrusted decoded data. The function should narrow the value before type-specific operations.

### 17. Does **mixed** mean the function may skip validation?

**Answer:** No.

**Explanation:** The declaration accepts any type, but the code still needs to handle each value safely and validate application-specific requirements.

### 18. Name three checks that can narrow a **mixed** value.

**Answer:** **is_string()**, **is_array()**, and **instanceof** are examples.

**Explanation:** Checks such as **is_int()**, **is_bool()**, and strict comparison with **null** are also useful.

### 19. Is **void** the same as the value **null**?

**Answer:** No.

**Explanation:** **void** is a return contract saying there is no useful result. **null** is a value that can be returned and declared as a type.

### 20. May a **void** function contain a bare **return;**?

**Answer:** Yes.

**Explanation:** A bare return exits the function without returning an expression.

### 21. May a **void** function use **return null;**?

**Answer:** No.

**Explanation:** That is a return expression. A **void** function must not return an expression, even if the expression is **null**.

### 22. May a **void** function finish by reaching its closing brace?

**Answer:** Yes.

**Explanation:** **void** permits normal completion. It only says the caller must not expect a useful value.

### 23. What three behaviors can satisfy a **never** return type?

**Answer:** Throwing an exception, calling **exit()**, or looping forever.

**Explanation:** Each path prevents a normal return to the caller.

### 24. May a **never** function execute a bare **return;**?

**Answer:** No.

**Explanation:** A bare return completes the function normally, contradicting the **never** contract.

### 25. Can **never** be used as a parameter type?

**Answer:** No.

**Explanation:** **never** is a return-only type; it describes a function's control flow.

### 26. What does the literal type **false** permit?

**Answer:** Only the exact boolean value **false**.

**Explanation:** It does not accept **true**, **0**, an empty string, or **null**.

### 27. When should a function use **bool** rather than **true**?

**Answer:** When it may return either **true** or **false**.

**Explanation:** **bool** expresses both values and is the correct alternative to an invalid **true|false** union.

### 28. Why is **bool|false** redundant?

**Answer:** **bool** already includes **false**.

**Explanation:** Adding a value already covered by a broader union member does not add a new accepted value.

### 29. Since when may **false** be used in a union such as **User|false**?

**Answer:** PHP 8.0.

**Explanation:** PHP 8.2 later added **false** as a standalone type too.

### 30. What does the standalone type **null** allow?

**Answer:** Only the value **null**.

**Explanation:** A function returning either a string or **null** should instead declare **string|null** or **?string**.

## True or false

### 31. **mixed** includes **null**.

**Answer: True.** It accepts values of every PHP type.

### 32. **void** means a function never reaches its caller again.

**Answer: False.** A **void** function may finish normally. **never** describes a function that cannot return normally.

### 33. A **void** function may return a string if the caller ignores it.

**Answer: False.** Ignoring a result does not make a return expression valid in a **void** function.

### 34. A **never** function may throw an exception.

**Answer: True.** Throwing prevents normal completion.

### 35. A function that reaches its closing brace on a normal path can correctly declare **never**.

**Answer: False.** Reaching the end is normal completion and violates the contract.

### 36. PHP permits **true|false** as a union type.

**Answer: False.** Use **bool** when both boolean values are allowed.

### 37. Standalone **true**, **false**, and **null** types are available from PHP 8.2.

**Answer: True.** Earlier PHP versions only allowed some of these values in unions.

### 38. **mixed|string** is more precise than **mixed**.

**Answer: False.** The **string** member is already included in **mixed**, so this combination is redundant.

## Read the code

### 39. What does each call print?

~~~php
<?php
function identify(mixed $value): string
{
    if (is_int($value)) {
        return "integer";
    }

    if ($value === null) {
        return "null";
    }

    return get_debug_type($value);
}

echo identify(12) . PHP_EOL;
echo identify(null) . PHP_EOL;
~~~

**Answer:**

~~~text
integer
null
~~~

**Explanation:** The first call passes the **is_int()** check. The second matches the strict **null** check.

### 40. Is this function compatible with its declared return type?

~~~php
<?php
function stopNow(): never
{
    return;
}
~~~

**Answer:** No.

**Explanation:** A bare return completes normally. A **never** function must throw, exit, or continue forever.

### 41. Is this a valid **void** function?

~~~php
<?php
function saveMessage(): void
{
    return "saved";
}
~~~

**Answer:** No.

**Explanation:** A **void** function cannot return an expression. Use a string return type if the caller needs the text.

### 42. Which values may this function return?

~~~php
<?php
function featureFlag(): true
{
    return true;
}
~~~

**Answer:** Only **true**.

**Explanation:** This standalone literal return type requires PHP 8.2 or later. Returning **false** or **null** would violate the declaration.

## Write code

### 43. Declare a function that accepts any PHP value and returns its debug type.

**Answer:**

~~~php
<?php
function debugTypeOf(mixed $value): string
{
    return get_debug_type($value);
}
~~~

**Explanation:** The parameter accepts every type; **get_debug_type()** provides a string description.

### 44. Write a **void** function that prints a message and exits early for an empty string.

**Answer:**

~~~php
<?php
function printMessage(string $message): void
{
    if ($message === "") {
        return;
    }

    echo $message . PHP_EOL;
}
~~~

**Explanation:** The early return is bare, and the function communicates its output through a side effect.

### 45. Write a **never** function that throws a runtime exception.

**Answer:**

~~~php
<?php
function fail(string $message): never
{
    throw new RuntimeException($message);
}
~~~

**Explanation:** The function cannot return normally. This return type is available from PHP 8.1.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [020 - Nullable, union, and intersection types](../lessons/020-nullable-union-and-intersection-types.md) | [Read this lesson](../lessons/021-mixed-void-never-and-literal-types.md) | [Open example](../examples/021-mixed-void-never-and-literal-types.md) | [022 - Constants, predefined constants, and magic constants](../lessons/022-constants-predefined-constants-and-magic-constants.md) |