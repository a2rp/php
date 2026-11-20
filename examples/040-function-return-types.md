# 040 - Function Return Types: Example

## Question

How can a PHP program calculate values, describe different possible results, and keep output-only behavior separate from returned data?

## Goal

Build a small order summary that uses scalar, nullable, union, array, **void**, and **never** return types. Use returned values in the caller and observe that a **never** function does not continue normally.

## Project structure

~~~text
typed-results/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function calculateTotal(int $unitPrice, int $quantity): int
{
    return $unitPrice * $quantity;
}

function findDiscountCode(bool $isMember): ?string
{
    if (!$isMember) {
        return null;
    }

    return 'SAVE5';
}

function createReference(bool $shortForm): int|string
{
    if ($shortForm) {
        return 42;
    }

    return 'ORD-42';
}

function buildSummary(string $item, int $quantity, int $total): array
{
    return [
        'item' => $item,
        'quantity' => $quantity,
        'total' => $total,
    ];
}

function printSummary(array $summary): void
{
    echo $summary['item']
        . ' x '
        . $summary['quantity']
        . ': $'
        . number_format($summary['total'], 2)
        . PHP_EOL;
}

function stopAfterReport(): never
{
    exit;
}

$total = calculateTotal(12, 3);
$summary = buildSummary('Notebook', 3, $total);

printSummary($summary);

echo 'Member code: ', findDiscountCode(true), PHP_EOL;
echo 'Guest code: ', findDiscountCode(false) ?? 'none', PHP_EOL;

echo 'Short reference: ', createReference(true), PHP_EOL;
echo 'Full reference: ', createReference(false), PHP_EOL;

echo 'Another calculation: ', calculateTotal(5, 2), PHP_EOL;
echo "Report complete\n";

stopAfterReport();

echo "This line is never reached\n";
~~~

## Expected output

Run the file from a terminal with PHP installed:

~~~sh
php index.php
~~~

The output is:

~~~text
Notebook x 3: $36.00
Member code: SAVE5
Guest code: none
Short reference: 42
Full reference: ORD-42
Another calculation: 10
Report complete
~~~

The last line in the code is not printed because **stopAfterReport()** exits the script.

## How it works

### A scalar return type returns one value

**calculateTotal()** declares **: int** and returns the product of its two integer inputs:

~~~php
function calculateTotal(int $unitPrice, int $quantity): int
{
    return $unitPrice * $quantity;
}
~~~

The caller assigns that returned integer to **$total** and passes it to **buildSummary()**. The function does not print the total itself.

### A nullable type can return null

**findDiscountCode()** declares **: ?string**, which permits a string or null. A member receives a string code; a non-member receives explicit null. The caller uses the null coalescing operator to display **none** when there is no code.

### A union type describes alternatives

**createReference()** declares **: int|string**. It returns an integer for the short form or a string for the full form. Each value matches one of the declared alternatives.

### An array groups related values

**buildSummary()** returns one array containing the item, quantity, and total. **printSummary()** accepts that array and formats its fields. The return type **array** checks that the result is an array; PHP does not infer a fixed shape for its keys and inner values from that declaration alone.

### A void function performs an action

**printSummary()** writes a formatted line with **echo** and declares **: void** because it is not intended to return a value. The caller invokes it as a statement.

### A never function cannot complete normally

**stopAfterReport()** declares **: never** and calls **exit**. It cannot return normally to the caller, so the following **echo** is unreachable. **never** is available in PHP 8.1 and later.

Do not use **never** just because a function usually exits early. Its every path must be unable to complete normally.

### Type mismatches

PHP checks the returned value against the declared type. Returning an incompatible value or falling off the end of a non-void typed function raises a **TypeError**. Scalar coercion and strict typing affect some scalar return checks; review [chapter 019](../lessons/019-type-declarations-and-strict-typing.md) for those rules.

## Make a change

1. Change the unit price or quantity and predict the new total.
2. Make **findDiscountCode()** return a different code for members.
3. Call **createReference(true)** and **createReference(false)** again, then compare the return types.
4. Add a **'passed'** field to the summary array and print it.
5. Temporarily change **calculateTotal()** so a path does not return a value and observe the type error.
6. Remove the call to **stopAfterReport()** and observe the final line that was previously unreachable.

## Common mistakes

- Using **echo** when the caller needs a value for later work.
- Expecting **return** to display the returned value automatically.
- Returning a value outside the declared type.
- Forgetting a return in one branch of a non-void function.
- Treating **?string** as permission to omit a return; return null explicitly.
- Declaring an output-only function **void** and then returning a value.
- Declaring a function **never** when a normal path can reach the end.
- Assuming a return type of **array** validates every key and element type.

## Key takeaways

- A function's return type states what value its caller can expect.
- **return** passes the value back and ends the current function.
- Nullable and union types describe multiple valid return alternatives.
- Arrays and objects can group related data into one return value.
- **void** describes an action-only function; **never** describes a function that does not complete normally.
- PHP checks a declared return contract and can raise **TypeError** when it is violated.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [039 - Default, Named, Variadic, and Unpacked Arguments](../lessons/039-default-named-variadic-and-unpacked-arguments.md) | [Read lesson](../lessons/040-function-return-types.md) | [Open Q&A](../qnas/040-function-return-types.md) | [041 - Anonymous Functions and Closures](../lessons/041-anonymous-functions-and-closures.md) |
