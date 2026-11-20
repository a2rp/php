# 038 - Function Parameters and Arguments: Example

## Question

How can one function describe different order items using values supplied by each caller?

## Goal

Write a typed function with three required parameters. Call it with variables and with literal or calculated arguments, then see how PHP maps each argument to its parameter.

## Project structure

~~~text
order-items/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

function printLineItem(string $name, int $quantity, float $unitPrice): void
{
    $lineTotal = $quantity * $unitPrice;

    echo $name
        . ': '
        . $quantity
        . ' x $'
        . number_format($unitPrice, 2)
        . ' = $'
        . number_format($lineTotal, 2)
        . PHP_EOL;
}

echo "Order items\n";

$itemName = 'Notebook';
$itemQuantity = 2;
$notebookUnitPrice = 3.50;

printLineItem($itemName, $itemQuantity, $notebookUnitPrice);
printLineItem('Pen', 3, 1.25);
printLineItem('Folder', 1 + 1, 2.00);
~~~

## Expected output

Run the file from a terminal with PHP installed:

~~~sh
php index.php
~~~

The output is:

~~~text
Order items
Notebook: 2 x $3.50 = $7.00
Pen: 3 x $1.25 = $3.75
Folder: 2 x $2.00 = $4.00
~~~

## How it works

### The signature declares three parameters

~~~php
function printLineItem(string $name, int $quantity, float $unitPrice): void
~~~

The function declares **$name**, **$quantity**, and **$unitPrice** as parameters. Their type declarations describe the expected string, integer, and floating-point inputs. The **void** return type tells the caller that this function prints a line rather than returning a value.

### Each call supplies arguments

In this call:

~~~php
printLineItem($itemName, $itemQuantity, $notebookUnitPrice);
~~~

The first argument **$itemName** is assigned to **$name**. The second, **$itemQuantity**, is assigned to **$quantity**. The third, **$notebookUnitPrice**, is assigned to **$unitPrice**.

The other calls pass literals. The last call uses the expression **1 + 1** as its second argument. PHP evaluates that expression before entering the function, then assigns the resulting value, **2**, to **$quantity**.

### The function calculates a line total

Inside the function, **$lineTotal** is a local variable. It multiplies the quantity by the unit price. **number_format()** displays both prices with two digits after the decimal point.

Every call uses the same body with different argument values. Changing the function once changes how all the item lines are printed.

### Parameters are local variables

The function uses local parameters **$name**, **$quantity**, and **$unitPrice**. The caller's variables are inputs; they are not automatically available in the function under the caller's names.

This function does not reassign the caller's variables. PHP passes arguments by value by default.

### Type behavior

The declared types document the inputs and allow PHP to check them. Scalar coercion depends on strict typing in the calling file. Review [chapter 019 on type declarations and strict typing](../lessons/019-type-declarations-and-strict-typing.md) for the complete rules.

If the call omits one of these required arguments, PHP raises an **ArgumentCountError**. If an argument cannot be accepted for its declared parameter type, PHP can raise a **TypeError**.

## Make a change

1. Add **'Eraser'** with quantity **4** and unit price **0.50**.
2. Change the notebook quantity to **5** and predict the new line total.
3. Pass a variable containing a new item name instead of a literal.
4. Change the output to include the unit price and line total on separate lines.
5. Temporarily swap the first two arguments and observe how PHP checks them against the parameter types.

## Common mistakes

- Confusing the parameter names in the declaration with argument values in a call.
- Passing arguments in a different order from the parameter list.
- Omitting a required argument.
- Assuming a parameter variable is also defined in the caller.
- Expecting a function to return the text it prints.
- Assuming every scalar type mismatch behaves identically in strict and weak typing modes.

## Key takeaways

- Parameters are declared in the function signature; arguments are supplied by each call.
- Positional arguments map left to right.
- Arguments can be variables, literals, or expressions.
- Parameters are local variables, and values are passed by value by default.
- Type declarations document and check inputs; scalar coercion depends on strict typing.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [037 - Defining and Calling Functions](../lessons/037-defining-and-calling-functions.md) | [Read lesson](../lessons/038-function-parameters-and-arguments.md) | [Open Q&A](../qnas/038-function-parameters-and-arguments.md) | [039 - Default, Named, Variadic, and Unpacked Arguments](../lessons/039-default-named-variadic-and-unpacked-arguments.md) |
