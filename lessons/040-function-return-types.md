# 040 - Function Return Types

A function can send a value back to the code that called it with **return**. A return type in the function signature states what kind of value the function promises to return. Together, **return** and a return type make a function's output clear and checkable.

## What you will learn

- Return a value from a function and use it at the call site
- Declare a function return type after the parameter list
- Make every normal execution path return a compatible value
- Distinguish returned data from printed output
- Use nullable and union return types
- Choose **void** when a function produces no return value
- Recognize **never** for functions that do not complete normally
- Understand return-type errors and scalar strict-typing behavior

## Return a value

The **return** statement sends a value back to the caller and immediately ends the current function:

~~~php
<?php
function calculateTotal(int $unitPrice, int $quantity): int
{
    return $unitPrice * $quantity;
}

$total = calculateTotal(12, 3);
echo "Total: {$total}\n";
~~~

Output:

~~~text
Total: 36
~~~

The expression **$unitPrice * $quantity** is evaluated, and its value is returned. The caller stores that value in **$total** and prints it.

A returned value can be stored, passed to another function, or used in a larger expression:

~~~php
$finalTotal = calculateTotal(12, 3) + 5;
~~~

**return** does not print the value. It gives the value to the caller. Use **echo** when the goal is to display text.

## Declare a return type

A return type appears after the closing parenthesis of the parameter list and a colon:

~~~php
function calculateTotal(int $unitPrice, int $quantity): int
{
    return $unitPrice * $quantity;
}
~~~

The **: int** part is the return type declaration. It promises that the function returns an integer. PHP checks the returned value against that declaration.

A function can return many kinds of values, including scalar values, arrays, and objects. A return type can name a built-in type or a class, interface, or enum type. The available type forms include nullable and union types; those type-system forms were introduced earlier in the course.

## Return ends the current function

When PHP executes **return**, it stops the function immediately. Statements after the reached return are skipped:

~~~php
<?php
function shippingCost(int $subtotal): int
{
    if ($subtotal >= 50) {
        return 0;
    }

    return 5;
}

echo shippingCost(75), "\n";
echo shippingCost(20), "\n";
~~~

Output:

~~~text
0
5
~~~

For the first call, the condition is true, so the function returns **0** and ends. For the second call, execution reaches and returns **5**.

A function can contain multiple return statements for different cases. Each execution path must return a compatible value whenever the function promises a non-void value.

## Return a value that matches the declared type

The declared return type is a contract:

~~~php
<?php
function getDisplayName(bool $hasName): string
{
    if ($hasName) {
        return 'Asha';
    }

    return 'Guest';
}

echo getDisplayName(true), "\n";
echo getDisplayName(false), "\n";
~~~

Output:

~~~text
Asha
Guest
~~~

Both paths return strings, matching **: string**. If the function returns a value that PHP cannot use as the declared type, PHP raises a **TypeError**.

If a typed, non-void function reaches its end without returning a value, PHP also raises a **TypeError**. Do not rely on an accidental missing return. Check every branch and make the function's contract clear.

Scalar return types such as **int**, **float**, **bool**, and **string** can be affected by scalar coercion. Whether PHP coerces a compatible scalar or throws a **TypeError** depends on strict typing. Review [chapter 019 on type declarations and strict typing](019-type-declarations-and-strict-typing.md) for those rules.

## Return null or one of several types

Use a nullable type when a function may return either a value of one type or **null**:

~~~php
<?php
function findNickname(bool $found): ?string
{
    if (!$found) {
        return null;
    }

    return 'Mina';
}

var_dump(findNickname(true));
var_dump(findNickname(false));
~~~

Output:

~~~text
string(4) "Mina"
NULL
~~~

**?string** means **string|null**. A union type can list other allowed alternatives, for example **int|string**:

~~~php
function getIdentifier(bool $numeric): int|string
{
    return $numeric ? 42 : 'user-42';
}
~~~

Use the narrowest useful return type. A precise type tells callers what they can safely expect.

## Return multiple related pieces in one value

PHP functions return one value. When a result has several related parts, return an array or an object:

~~~php
<?php
function makeSummary(string $name, int $score): array
{
    return [
        'name' => $name,
        'score' => $score,
        'passed' => $score >= 50,
    ];
}

$summary = makeSummary('Asha', 82);
echo "{$summary['name']}: {$summary['score']}\n";
echo $summary['passed'] ? "Passed\n" : "Try again\n";
~~~

Output:

~~~text
Asha: 82
Passed
~~~

The function returned one array containing multiple related fields. It did not return three separate values.

## Use void when there is no return value

A function that performs an action but does not return data can declare **void**:

~~~php
<?php
function showNotice(string $message): void
{
    echo "Notice: {$message}\n";
}

showNotice('Saved');
~~~

Output:

~~~text
Notice: Saved
~~~

A **void** function may finish without a **return** statement or use **return;** to stop early. It must not return a value with **return $value;**. **void** is different from **null**: a function typed **?string** may return null as a value, while **void** promises no result for the caller to use.

Use **void** instead of inventing a dummy value when the function's purpose is to produce an effect, such as printing or recording a log.

## Use never when a function cannot finish normally

The **never** return type describes a function that does not return control normally to its caller. Such a function must always terminate execution by throwing an exception, calling **exit**, or not reaching the end (for example, an intentional infinite loop). **never** is available since PHP 8.1.

~~~php
<?php
function failRequest(string $reason): never
{
    throw new RuntimeException($reason);
}

failRequest('Request could not be processed');
~~~

This function cannot complete normally. It does not return a value, so there must be no **return** statement and no normal path that reaches the closing brace.

Use **never** only when the function truly cannot return. For the difference between **void** and **never**, see [chapter 021 on mixed, void, never, and literal types](021-mixed-void-never-and-literal-types.md).

## A function without a declared return type

PHP allows a function without a return type declaration. If it reaches the end without executing **return**, its call produces **null**:

~~~php
<?php
function printStatus(): void
{
    echo "Ready\n";
}

printStatus();
~~~

Here the function explicitly declares **void**. Omitting a type declaration is different: it leaves the function's return contract unspecified. For new code, declare a useful return type when the function returns data, or **void** when it only performs an action.

The earlier [chapter 036 on returning values](036-returning-values-and-ending-execution.md) explains the **return** statement and its behavior in more detail.

## Common mistakes

- Using **echo** when the caller needs a value for a calculation.
- Expecting **return** to display a value automatically.
- Forgetting to return a value on one branch of a typed, non-void function.
- Returning a value that does not match the declared return type.
- Writing **return $value;** in a function declared **void**.
- Using **void** for a function that must never complete normally; that case may require **never**.
- Treating **?string** and **void** as interchangeable.
- Assuming a function can return several independent values instead of grouping them in one array or object.
- Ignoring strict-typing behavior when declaring scalar return types.

## Try it yourself

1. Define **double(int $number): int** and return twice the input.
2. Define **findLabel(bool $found): ?string** and return either a label or null.
3. Add a second branch to a typed function and make sure it returns the declared type.
4. Change a return type to an incompatible type and observe the **TypeError**.
5. Convert an output-only function to **void**.
6. Write a function that throws an exception and decide whether **never** accurately describes it.

## Key takeaways

- **return** passes a value to the caller and ends the current function.
- A return type is declared after the parameter list with a colon.
- PHP checks returned values against the declared return type.
- Every normal path in a non-void function must return a compatible value.
- **void** is for functions that return no value; **never** is for functions that do not complete normally.
- Use nullable or union types only when the function can genuinely return those alternatives.
- Return one array or object when a result needs to group related pieces.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [039 - Default, Named, Variadic, and Unpacked Arguments](039-default-named-variadic-and-unpacked-arguments.md) | [Open Q&A](../qnas/040-function-return-types.md) | [Open example](../examples/040-function-return-types.md) | [041 - Anonymous Functions and Closures](041-anonymous-functions-and-closures.md) |
