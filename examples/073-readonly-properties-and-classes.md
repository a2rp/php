# Readonly properties and classes - Example

## Goal

Build a small product snapshot using readonly value objects. The example shows how constructor initialization works, how to return a changed copy without mutating the original, and why a readonly property does not deeply freeze a referenced object.

## Requirements

This complete example uses readonly classes and requires PHP 8.2 or later. It does not rely on PHP 8.3 clone behavior, PHP 8.4 setter visibility, or PHP 8.6 property defaults.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

readonly class Money
{
    public function __construct(
        public int $minorUnits,
        public string $currency,
    ) {
        if ($minorUnits < 0) {
            throw new InvalidArgumentException(
                'The amount cannot be negative.'
            );
        }

        if (preg_match('/^[A-Z]{3}$/D', $currency) !== 1) {
            throw new InvalidArgumentException(
                'Currency must be a three-letter uppercase code.'
            );
        }
    }

    public function format(): string
    {
        $amount = number_format(
            $this->minorUnits / 100,
            2,
            '.',
            ','
        );

        return $this->currency . ' ' . $amount;
    }

    public function withMinorUnits(int $minorUnits): self
    {
        return new self($minorUnits, $this->currency);
    }
}

readonly class ProductSnapshot
{
    public function __construct(
        public int $id,
        public string $name,
        public Money $price,
        public DateTimeImmutable $updatedAt,
    ) {
        if ($id < 1) {
            throw new InvalidArgumentException(
                'Product ID must be positive.'
            );
        }

        if (trim($name) === '') {
            throw new InvalidArgumentException(
                'Product name cannot be empty.'
            );
        }
    }

    public function withName(string $name): self
    {
        return new self(
            $this->id,
            $name,
            $this->price,
            $this->updatedAt,
        );
    }
}

final class MetadataHolder
{
    public function __construct(
        public readonly object $metadata,
    ) {
    }
}

$price = new Money(3999, 'USD');

$product = new ProductSnapshot(
    id: 42,
    name: 'Desk lamp',
    price: $price,
    updatedAt: new DateTimeImmutable(
        '2026-04-12T10:30:00+00:00'
    ),
);

$renamedProduct = $product->withName('Reading lamp');

echo 'Product: ' . $product->name . PHP_EOL;
echo 'Price: ' . $product->price->format() . PHP_EOL;
echo 'Original name: ' . $product->name . PHP_EOL;
echo 'Copy name: ' . $renamedProduct->name . PHP_EOL;
echo 'Updated: ' . $product->updatedAt->format('Y-m-d H:i') . PHP_EOL;

$metadata = (object) ['views' => 1];
$holder = new MetadataHolder($metadata);

$holder->metadata->views++;

echo 'Metadata views: ' . $holder->metadata->views . PHP_EOL;

// These writes are invalid because the properties are already initialized:
// $product->name = 'Floor lamp';
// $product->price->minorUnits = 4500;
// $holder->metadata = new stdClass();
~~~

## Expected output

~~~text
Product: Desk lamp
Price: USD 39.99
Original name: Desk lamp
Copy name: Reading lamp
Updated: 2026-04-12 10:30
Metadata views: 2
~~~

## Explanation

### The constructor establishes valid values

The <code>Money</code> constructor receives an amount in minor units and a currency code. It validates both inputs before the object can be used. The properties are promoted and readonly, so callers can read them but cannot replace them after construction.

The <code>ProductSnapshot</code> constructor similarly validates its identifier and name. Its price is another readonly value object, and its timestamp uses <code>DateTimeImmutable</code> so the nested date value cannot be changed through a mutating date method.

### A changed value is represented by a new object

<code>withName()</code> returns a new <code>ProductSnapshot</code>. It passes the existing ID, price, and timestamp into the new instance and supplies a replacement name.

The original product still contains <code>Desk lamp</code>; the returned snapshot contains <code>Reading lamp</code>. This copy-style approach keeps the original stable while making the requested change clear.

### Readonly does not deep-freeze an object

<code>MetadataHolder</code> has a readonly property that points to a regular mutable object. The code changes the referenced object’s <code>views</code> field, which is allowed. Replacing the <code>metadata</code> property itself is not allowed after initialization.

This is why the example uses immutable nested values for the product snapshot, but deliberately uses a mutable object in the metadata demonstration.

### Invalid writes

The commented assignments at the bottom would raise errors:

- <code>$product->name</code> is already initialized.
- <code>$product->price</code> is readonly, so its value cannot be replaced.
- <code>$holder->metadata</code> is readonly, even though the object stored in it can mutate internally.

## Practice

1. Add a <code>withCurrency()</code> method to <code>Money</code> that returns a new value.
2. Add a <code>withPrice()</code> method to <code>ProductSnapshot</code> that returns a new snapshot.
3. Try constructing <code>Money</code> with a lowercase currency code. Which validation rejects it?
4. Replace <code>DateTimeImmutable</code> with <code>DateTime</code>. What kind of mutation would then be possible through the readonly property?
5. Change <code>MetadataHolder</code> to store an immutable value object. How does that affect the depth of immutability?

## Key takeaways

- Readonly properties are initialized once, commonly by a constructor.
- A readonly class applies readonly behavior to its declared instance properties.
- Validation in a constructor helps ensure every constructed value is valid.
- Return a new object when a value object needs a changed value.
- Readonly prevents replacing a property; it does not freeze mutable objects stored inside it.
- This example requires PHP 8.2 or later because it uses readonly classes.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [072 - Class constants and constant expressions](../lessons/072-class-constants-and-constant-expressions.md) | [Lesson](../lessons/073-readonly-properties-and-classes.md) | [Q&A](../qnas/073-readonly-properties-and-classes.md) | [074 - Constructor property promotion](../lessons/074-constructor-property-promotion.md) |
