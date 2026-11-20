# 041 - Anonymous Functions and Closures: Example

## Question

How can a small callback use a configuration value from its surrounding scope while also updating a counter there?

## Goal

Create a closure that formats prices with tax. Capture the tax rate by value, capture a processed-item counter by reference, and pass the closure to **array_map()**.

## Project structure

~~~text
closure-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

$taxRate = 0.08;
$processedCount = 0;

$formatWithTax = function (float $price) use ($taxRate, &$processedCount): string {
    $processedCount++;

    $priceWithTax = $price * (1 + $taxRate);

    return '$' . number_format($priceWithTax, 2);
};

// The closure keeps the value captured above.
$taxRate = 0.10;

$prices = [10, 25, 5];
$formattedPrices = array_map($formatWithTax, $prices);

echo "Prices including tax:\n";

foreach ($formattedPrices as $formattedPrice) {
    echo $formattedPrice, "\n";
}

echo 'Current outer tax rate: ', number_format($taxRate * 100, 0), "%\n";
echo "Items processed by closure: {$processedCount}\n";
~~~

## Expected output

Run the file from a terminal with PHP installed:

~~~sh
php index.php
~~~

The output is:

~~~text
Prices including tax:
$10.80
$27.00
$5.40
Current outer tax rate: 10%
Items processed by closure: 3
~~~

## How it works

### The closure is stored in a variable

~~~php
$formatWithTax = function (float $price) use ($taxRate, &$processedCount): string {
    // ...
};
~~~

This anonymous function is assigned to **$formatWithTax**. Its parameter **$price** receives one value each time it is called. Its return type **string** promises a formatted string.

The semicolon after the closing brace ends the assignment statement.

### The use list captures surrounding values

**use ($taxRate, &$processedCount)** imports two outer variables:

- **$taxRate** is captured by value. The closure keeps **0.08**, the value present when it was created.
- **&$processedCount** is captured by reference. The closure increments the same counter that the outer scope can read.

The outer **$taxRate** changes to **0.10** after the closure is created, but the closure still calculates prices with its captured eight-percent rate. The final output prints the current outer rate as ten percent, while the formatted prices reflect eight percent.

### array_map() calls the closure

~~~php
$formattedPrices = array_map($formatWithTax, $prices);
~~~

**array_map()** calls the closure once for each price. Each call returns a formatted string, and **array_map()** collects those strings in **$formattedPrices**.

Because the closure increments **$processedCount** every time it runs, that counter is **3** after the three prices have been processed.

### Captured and returned values play different roles

The closure reads its captured tax rate and updates the captured counter. Its normal parameter supplies the current price, and its return value supplies the formatted string to **array_map()**.

This keeps each role clear:

- **$price** is the input for one call.
- **$taxRate** is configuration captured when the closure is created.
- **$processedCount** is shared mutable state captured by reference.
- The returned string becomes an item in **$formattedPrices**.

## Make a change

1. Change the initial **$taxRate** to **0.05** and predict the formatted values.
2. Add another number to **$prices** and observe the counter.
3. Remove **&** before **$processedCount** in the **use** clause and observe why a by-value capture cannot update the outer counter.
4. Change **$formatWithTax** to return the unformatted numeric total, then decide what output formatting should happen outside the closure.
5. Pass an inline anonymous function directly to **array_map()** instead of assigning it to **$formatWithTax**.

## Common mistakes

- Forgetting **use** and expecting **$taxRate** to be automatically visible inside a regular closure.
- Capturing **$taxRate** by value and expecting a later outer assignment to replace the captured value.
- Forgetting **&** when the closure is intended to update the outer counter.
- Putting the return type before the **use** clause.
- Forgetting the semicolon after the closure assignment.
- Expecting **array_map()** to keep only side effects; it returns an array of callback results.

## Key takeaways

- A closure can be stored and passed as a callback value.
- Closure parameters receive per-call inputs.
- **use ($value)** captures by value; **use (&$value)** shares the outer variable by reference.
- A closure can combine captured values with ordinary parameters and a declared return type.
- **array_map()** calls the callback for each input and collects the results.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [040 - Function Return Types](../lessons/040-function-return-types.md) | [Read lesson](../lessons/041-anonymous-functions-and-closures.md) | [Open Q&A](../qnas/041-anonymous-functions-and-closures.md) | [042 - Arrow Functions](../lessons/042-arrow-functions.md) |
