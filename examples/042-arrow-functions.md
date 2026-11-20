# 042 - Arrow Functions: Example

## Question

How can an arrow function use a tax rate from its surrounding scope to transform each price, then filter the resulting totals?

## Goal

Create a reusable arrow-function callback for **array_map()**, capture a tax rate automatically, and use an inline arrow function with **array_filter()**. Observe that a later change to the outer tax-rate variable does not change the value already captured.

## Project structure

~~~text
arrow-functions-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

$taxRate = 0.08;

$withTax = fn (float $price): float =>
    $price * (1 + $taxRate);

// The arrow function already captured 0.08 by value.
$taxRate = 0.10;

$prices = [10, 25, 5];

$totals = array_map($withTax, $prices);

echo "Prices with the captured tax rate:\n";

foreach ($totals as $total) {
    echo '$' . number_format($total, 2), "\n";
}

$largeTotals = array_filter(
    $totals,
    fn (float $total): bool => $total >= 20,
);

echo "Totals of at least $20:\n";

foreach ($largeTotals as $total) {
    echo '$' . number_format($total, 2), "\n";
}

echo 'Current outer tax rate: ';
echo number_format($taxRate * 100, 0), "%\n";
~~~

## Expected output

Run the file from a terminal with PHP 7.4 or later installed:

~~~sh
php index.php
~~~

The output is:

~~~text
Prices with the captured tax rate:
$10.80
$27.00
$5.40
Totals of at least $20:
$27.00
Current outer tax rate: 10%
~~~

## Explanation

### Define an arrow function

~~~php
$withTax = fn (float $price): float =>
    $price * (1 + $taxRate);
~~~

The **fn** keyword begins the arrow function. It accepts one **float** parameter and declares a **float** return type. The expression after **=>** calculates the price including tax, and PHP returns that expression's value automatically.

### Capture the tax rate

The arrow function references **$taxRate**, so PHP captures it automatically by value when the function is created. At that point, **$taxRate** is **0.08**.

The next line assigns **0.10** to the outer variable. That changes the outer **$taxRate**, but does not replace the value already captured by **$withTax**. The mapped prices therefore use an 8% rate. The final line reads the outer variable and displays 10%.

### Transform the prices

**array_map($withTax, $prices)** calls the arrow function once for each price. It returns a new array containing the calculated totals:

- **10** becomes **10.80**
- **25** becomes **27.00**
- **5** becomes **5.40**

The first **foreach** loop formats each numeric total to two decimal places and prints it.

### Filter the totals

~~~php
fn (float $total): bool => $total >= 20
~~~

This inline arrow function returns **true** for totals of at least 20. **array_filter()** keeps values for which its callback returns a truthy result, so only **27.00** appears in the second list.

The condition is one expression, so it fits an arrow function. A multi-step calculation would be clearer in a regular closure.

### Read the output carefully

The example intentionally prints two different tax values:

- The calculations use the captured rate **0.08**.
- The outer variable has since been changed to **0.10**.

Arrow functions capture outer variables by value. To share and modify an outer variable by reference, use a regular closure with **use (&$variable)**.

## Make a change

1. Change the initial **$taxRate** to **0.05** and calculate the new totals.
2. Add **40** to **$prices** and check which totals pass the filter.
3. Change the filter condition from **>= 20** to **> 20**.
4. Remove the reassignment to **$taxRate** and compare the displayed outer rate.
5. Rewrite **$withTax** as a regular closure using **use ($taxRate)**. Compare the two forms.

## Common mistakes

- Adding a **use** clause to the arrow function. Referenced outer variables are captured automatically.
- Expecting the later outer reassignment to change the captured tax rate.
- Writing a brace-delimited body or an explicit **return** in the arrow function.
- Forgetting that **array_map()** returns a new array of callback results.
- Treating the callback's return values from **array_filter()** as transformed values. They are conditions used to keep or discard input values.
- Running the example on a PHP version earlier than 7.4.

## Key takeaways

- An arrow function uses **fn (parameters) => expression**.
- The expression's result is returned automatically.
- Referenced outer variables are captured by value when the arrow function is created.
- **array_map()** uses the callback result to create a transformed array.
- **array_filter()** uses the callback result as a condition for keeping values.
- Use a regular closure for multiple statements or by-reference capture.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [041 - Anonymous Functions and Closures](../lessons/041-anonymous-functions-and-closures.md) | [Read lesson](../lessons/042-arrow-functions.md) | [Open Q&A](../qnas/042-arrow-functions.md) | [043 - Callable Values and First-Class Callable Syntax](../lessons/043-callable-values-and-first-class-callable-syntax.md) |
