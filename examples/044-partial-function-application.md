# 044 - Partial Function Application: Example

## Question

How can you make reusable callbacks by fixing some arguments now and leaving the rest to be supplied later?

## Goal

Use PHP 8.6 partial function application to:

- Fix a tax rate and pass the remaining price to **array_map()**
- Fix formatting options and leave the amount as a placeholder
- Use a named placeholder with **str_replace()**
- Forward an optional argument with **...**
- Delay a fully configured calculation until the returned closure is invoked
- Observe that fixed argument expressions are evaluated when the closure is created

## Project structure

~~~text
partial-application-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**. Run it with PHP 8.6 or later:

~~~php
<?php

function addTax(float $rate, float $price): float
{
    return $price * (1 + $rate);
}

function formatMoney(
    string $currency,
    float $amount,
    int $decimals = 2,
): string {
    return $currency . number_format($amount, $decimals);
}

function greet(
    string $name,
    string $greeting = 'Hello',
): string {
    return "{$greeting}, {$name}!";
}

function selectTaxRate(): float
{
    echo "Tax rate selected.\n";
    return 0.08;
}

// The rate expression is evaluated now; the price is supplied later.
$withTax = addTax(selectTaxRate(), ?);

echo "Tax callback created.\n";

$prices = [10, 25, 5];
$totals = array_map($withTax, $prices);

// Fix the currency and decimal count; leave the amount for later.
$formatDollars = formatMoney('$', ?, 2);
$formattedTotals = array_map($formatDollars, $totals);

echo "Prices including tax:\n";

foreach ($formattedTotals as $formattedTotal) {
    echo $formattedTotal, "\n";
}

// Leave the subject string for later and remove every hyphen.
$removeDashes = str_replace(
    subject: ?,
    search: '-',
    replace: '',
);

echo 'Text without hyphens: ';
echo $removeDashes('a-b-c'), "\n";

// Forward the remaining arguments, including the optional greeting.
$greet = greet(?, ...);

echo $greet('Mina', 'Welcome'), "\n";

// Supply every target argument, but delay the function call.
$calculateTax = addTax(0.08, 100, ...);

echo "Deferred calculation created.\n";
echo number_format($calculateTax(), 2), "\n";
~~~

## Expected output

~~~text
Tax rate selected.
Tax callback created.
Prices including tax:
$10.80
$27.00
$5.40
Text without hyphens: abc
Welcome, Mina!
Deferred calculation created.
108.00
~~~

## Explanation

### Fix the tax rate and leave the price

~~~php
$withTax = addTax(selectTaxRate(), ?);
~~~

The **?** placeholder becomes a required parameter of the returned closure. The closure stores the result of **selectTaxRate()** as the fixed tax rate, and **array_map()** supplies each price when it invokes the callback.

Notice the output order: **Tax rate selected.** appears before **Tax callback created.** The function used to produce the fixed rate runs once while the closure is created. It does not run again for every price.

### Use the callback with array_map()

~~~php
$totals = array_map($withTax, $prices);
~~~

**array_map()** calls **$withTax** once for each price. Because the callback was created with one placeholder, each call supplies one price to **addTax()**.

The rate is fixed at **0.08**, while the price changes on every callback invocation.

### Fix formatting options

~~~php
$formatDollars = formatMoney('$', ?, 2);
~~~

This closure stores the currency symbol and number of decimal places. Its one required parameter is the amount. Passing the closure to **array_map()** formats each taxed total consistently.

### Use a named placeholder

~~~php
$removeDashes = str_replace(
    subject: ?,
    search: '-',
    replace: '',
);
~~~

The **subject** placeholder is supplied when the closure is called. The search string and replacement are fixed. The closure removes hyphens from a later-provided string.

### Forward the optional greeting

~~~php
$greet = greet(?, ...);
~~~

The question mark leaves the name for later, and the final **...** forwards remaining arguments to **greet()**. That allows **'Welcome'** to replace the function's default greeting.

Without **...**, an extra argument passed to this closure would be ignored, and **greet()** would use **'Hello'**.

### Delay a fully configured calculation

~~~php
$calculateTax = addTax(0.08, 100, ...);
~~~

Both parameters are fixed, but the final **...** delays the call. The result is a closure that takes no arguments. **addTax()** calculates the total only when **$calculateTax()** is invoked.

## Make a change

1. Change the tax rate returned by **selectTaxRate()** to **0.05** and predict all three totals.
2. Add **40** to **$prices** and check that it is included in the formatted output.
3. Change **$formatDollars** to use zero decimal places.
4. Change **$removeDashes** so it removes commas instead of hyphens.
5. Call **$greet('Mina')** without a second argument and observe the default greeting.
6. Remove the final **...** from **greet(?, ...)** and compare what happens to an extra greeting argument.
7. Call **$calculateTax()** twice and confirm that the deferred target call runs on each invocation, while the fixed arguments are not re-evaluated.

## Common mistakes

- Running this file on PHP earlier than 8.6.
- Expecting **addTax(selectTaxRate(), ?)** to run **addTax()** while the closure is created.
- Expecting **selectTaxRate()** to run again for each mapped price.
- Forgetting that **?** creates a required parameter in the returned closure.
- Omitting **...** when extra arguments must be forwarded to optional parameters.
- Using **...** before another argument or using it more than once.
- Confusing the **...** placeholder with argument unpacking such as **...$values**.
- Treating named placeholders as optional values.
- Expecting a fully configured closure with **...** to run before it is invoked.

## Key takeaways

- Partial application creates a closure from a callable plus fixed arguments and placeholders.
- **?** leaves exactly one required argument for later.
- **...** forwards remaining arguments and can delay a fully configured call.
- Fixed argument expressions run once when the closure is created.
- **array_map()** can call the resulting closures for each input value.
- Named placeholders can identify and order arguments for the resulting closure.
- Partial function application requires PHP 8.6 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [043 - Callable Values and First-Class Callable Syntax](../lessons/043-callable-values-and-first-class-callable-syntax.md) | [Read lesson](../lessons/044-partial-function-application.md) | [Open Q&A](../qnas/044-partial-function-application.md) | [045 - Passing and Returning Values by Reference](../lessons/045-passing-and-returning-values-by-reference.md) |
