# 063 - Properties, Methods, and Object Creation: Example

## Question

How can a class expose useful operations while keeping important object state private and validating changes?

## Complete example

This standalone example works with PHP 7.4 and later. Save it as `properties-methods.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

class InventoryItem
{
    public string $name = 'Unnamed';

    private int $stock = 0;

    private ?string $sku = null;

    public function setStock(int $stock): void
    {
        if ($stock < 0) {
            throw new InvalidArgumentException(
                'Stock cannot be negative.'
            );
        }

        $this->stock = $stock;
    }

    public function stock(): int
    {
        return $this->stock;
    }

    public function setSku(?string $sku): void
    {
        if ($sku !== null && trim($sku) === '') {
            throw new InvalidArgumentException(
                'SKU must be a non-empty string or null.'
            );
        }

        $this->sku = $sku === null ? null : trim($sku);
    }

    public function summary(): string
    {
        $skuLabel = $this->sku ?? 'none';

        return sprintf(
            '%s (SKU: %s, stock: %d)',
            $this->name,
            $skuLabel,
            $this->stock
        );
    }
}

// The class has no constructor yet, so create it without arguments.
$notebook = new InventoryItem();
$pen = new InventoryItem();

// The public name can be assigned directly.
$notebook->name = 'Notebook';
$pen->name = 'Pen';

// Private state is updated through methods that enforce the class rules.
$notebook->setSku(' NOTE-01 ');
$notebook->setStock(12);
$pen->setStock(40);

echo 'First item: ' . $notebook->summary() . PHP_EOL;
echo 'Second item: ' . $pen->summary() . PHP_EOL;

try {
    $notebook->setStock(-3);
} catch (InvalidArgumentException $exception) {
    echo 'Rejected invalid stock: '
        . $exception->getMessage()
        . PHP_EOL;
}

echo 'Stock after rejected update: '
    . $notebook->stock()
    . PHP_EOL;
~~~

## Expected output

~~~text
First item: Notebook (SKU: NOTE-01, stock: 12)
Second item: Pen (SKU: none, stock: 40)
Rejected invalid stock: Stock cannot be negative.
Stock after rejected update: 12
~~~

## Explanation

### 1. Object creation gives each item its own state

Each call to `new InventoryItem()` creates a separate object. The declared defaults initialize the public `name`, private `stock`, and nullable private `sku` for every new instance.

Changing `$notebook` does not change `$pen`.

### 2. Typed properties define the allowed values

The declarations specify:

- `name` must be a string.
- `stock` must be an integer.
- `sku` can be either a string or null.

The `sku` property is initialized with null, so it can be read without first assigning a string. A typed property without a default would need to be assigned before the first read.

### 3. Visibility protects internal state

The `stock` and `sku` properties are private. Code outside the class cannot assign to them directly. The public `setStock()` and `setSku()` methods provide controlled ways to change them.

The stock method rejects a negative value before changing state. After the exception is caught, the previous stock remains 12.

### 4. Methods use $this for the current item

Inside `setStock()`, `$this->stock` refers to the stock property of the object on which the method was called. `summary()` reads the current instance's name, SKU, and stock to build a string.

### 5. Public properties are convenient but open

The example leaves `name` public so you can see direct property access:

~~~php
$notebook->name = 'Notebook';
~~~

Any caller can change a public property directly, so it cannot enforce additional rules by itself. For important state, prefer private properties with methods that validate changes.

### 6. Constructor setup comes next

This class has no constructor, so it can be created with `new InventoryItem()` and then configured through properties and methods. The next chapter explains constructors, which let an object receive required initialization values as part of creation.

## Try it yourself

1. Try to set stock to `-1` and confirm that the previous value remains unchanged.
2. Try setting the SKU to an empty string and read the validation message.
3. Create a third item without changing any properties and inspect its defaults.
4. Add a private integer property for a reorder threshold and a method that checks whether the current stock is below it.
5. Change the public `name` property to private and add a validating `setName()` method.

## Key takeaways

- Typed properties describe the kind of value each property stores.
- A nullable property should be initialized to null if it can be read before a string is assigned.
- Private properties keep callers from bypassing important rules.
- Methods can validate input and update the current object through `$this`.
- Each `new InventoryItem()` call creates a separate object with its own state.
- Constructors provide creation-time initialization and are covered next.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [062 - Classes, objects, and object-oriented PHP](../lessons/062-classes-objects-and-object-oriented-php.md) | [Q&A](../qnas/063-properties-methods-and-object-creation.md) | [Lesson](../lessons/063-properties-methods-and-object-creation.md) | [064 - Constructors and destructors](../lessons/064-constructors-and-destructors.md) |
