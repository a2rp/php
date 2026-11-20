# 047 - Iterators, Traversable, and iterable Values: Example

## Question

How can one function use **foreach** to read values from an array, a custom **Iterator**, an **IteratorAggregate** collection, and a generator?

## Goal

Build a small catalog program that:

- Accepts all supported inputs through an **iterable** parameter
- Implements a custom **Iterator** with its five required methods
- Implements **IteratorAggregate** by returning an **ArrayIterator**
- Supplies a generator as another iterable source
- Checks a dynamic value with **is_iterable()**

## Project structure

~~~text
iterable-catalog-practice/
└── index.php
~~~

## Complete code

Save this as **index.php**:

~~~php
<?php

class NumberIterator implements Iterator
{
    private int $position = 0;

    /** @var list<string> */
    private array $items;

    public function __construct(array $items)
    {
        $this->items = array_values($items);
    }

    public function current(): mixed
    {
        return $this->items[$this->position];
    }

    public function key(): mixed
    {
        return $this->position;
    }

    public function next(): void
    {
        $this->position++;
    }

    public function rewind(): void
    {
        $this->position = 0;
    }

    public function valid(): bool
    {
        return array_key_exists($this->position, $this->items);
    }
}

class BookCatalog implements IteratorAggregate
{
    /** @param array<string, string> $books */
    public function __construct(
        private array $books,
    ) {
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->books);
    }
}

function printEntries(string $heading, iterable $entries): void
{
    echo $heading, ":\n";

    foreach ($entries as $key => $value) {
        echo "{$key}: {$value}\n";
    }
}

function featuredBooks(): Generator
{
    yield 'featured-1' => 'Generators in Practice';
    yield 'featured-2' => 'PHP Collections';
}

$arrayEntries = [
    'first' => 'PHP Basics',
    'second' => 'Web Development',
];

$customIterator = new NumberIterator([
    'One',
    'Two',
]);

$catalog = new BookCatalog([
    'book-1' => 'Clean Code',
    'book-2' => 'Practical PHP',
]);

printEntries('Array', $arrayEntries);
printEntries('Custom Iterator', $customIterator);
printEntries('IteratorAggregate collection', $catalog);
printEntries('Generator', featuredBooks());

echo 'Array is iterable: ';
echo is_iterable($arrayEntries) ? "yes\n" : "no\n";

echo 'Plain object is iterable: ';
echo is_iterable(new stdClass()) ? "yes\n" : "no\n";
~~~

## Expected output

~~~text
Array:
first: PHP Basics
second: Web Development
Custom Iterator:
0: One
1: Two
IteratorAggregate collection:
book-1: Clean Code
book-2: Practical PHP
Generator:
featured-1: Generators in Practice
featured-2: PHP Collections
Array is iterable: yes
Plain object is iterable: no
~~~

## Explanation

### One function accepts different iterable values

~~~php
function printEntries(string $heading, iterable $entries): void
{
    echo $heading, ":\n";

    foreach ($entries as $key => $value) {
        echo "{$key}: {$value}\n";
    }
}
~~~

The **iterable** parameter accepts an array or an object implementing **Traversable**. The function can process each input the same way without checking its specific class.

### The custom Iterator controls its position

**NumberIterator** implements the five methods required by **Iterator**:

- **rewind()** resets the cursor to position zero.
- **valid()** reports whether that position has an item.
- **current()** returns the item.
- **key()** returns its numeric position.
- **next()** advances the position.

The **foreach** loop calls these methods as it moves through the object. This is useful when an iterator needs to manage its own cursor or calculate values as the caller advances it.

### IteratorAggregate returns an iterator

**BookCatalog** implements **IteratorAggregate**. It does not implement cursor methods; **getIterator()** returns an **ArrayIterator** over its stored books.

When **printEntries()** receives the catalog, **foreach** asks it for the iterator and traverses the returned object. This approach is convenient when a collection already has its data and can return an iterator over it.

### The generator is iterable too

**featuredBooks()** contains **yield**, so calling it returns a **Generator** object. A generator implements **Traversable**, so **printEntries()** accepts it as an **iterable**.

The generator supplies each book only as the loop requests it. It also preserves the explicit keys used in its **yield** statements.

### Check dynamic input

~~~php
is_iterable($arrayEntries)
is_iterable(new stdClass())
~~~

An array is iterable. A plain **stdClass** object is not, because it does not implement **Traversable**.

## Make a change

1. Add another entry to **$arrayEntries** and confirm the consumer needs no changes.
2. Add another item to **NumberIterator** and check the generated numeric key.
3. Add a third book to **BookCatalog**.
4. Add another keyed yield to **featuredBooks()**.
5. Change **printEntries()** to print only values and omit keys.
6. Pass an **ArrayIterator** directly to **printEntries()**.

## Common mistakes

- Typing **printEntries()** with **array** instead of **iterable** when it should accept iterator objects too.
- Assuming a plain object is iterable.
- Trying to implement **Traversable** directly in a userland class.
- Forgetting a required method in **NumberIterator**.
- Returning an array from **BookCatalog::getIterator()** instead of a **Traversable** object.
- Expecting a generator to behave like an array with random access.
- Reusing one stateful iterator in nested loops when separate cursor states are needed.

## Key takeaways

- A function with an **iterable** parameter can accept arrays, generators, and traversable objects.
- A custom **Iterator** implements its own position, key, current value, reset, and validity.
- **IteratorAggregate** returns another traversable iterator for a collection.
- A generator is an iterable source that can provide values on demand.
- **is_iterable()** checks whether the outer value can be used with **foreach**.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [046 - Generators and Yield](../lessons/046-generators-and-yield.md) | [Read lesson](../lessons/047-iterators-traversable-and-iterable-values.md) | [Open Q&A](../qnas/047-iterators-traversable-and-iterable-values.md) | [048 - Working with Strings](../lessons/048-working-with-strings.md) |
