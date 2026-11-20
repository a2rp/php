# 047 - Iterators, Traversable, and iterable Values

PHP's **foreach** loop can work with arrays and with objects that provide iteration behavior. The **iterable** type accepts either an array or an object implementing **Traversable**.

Use the **Iterator** interface when an object needs to manage its own position, current value, and key. Use **IteratorAggregate** when an object should expose another iterator for its contents. For many custom collections, **IteratorAggregate** is the simpler option.

## What you will learn

- Understand the **iterable** type
- Know how arrays and **Traversable** objects work with **foreach**
- Distinguish **Traversable**, **Iterator**, and **IteratorAggregate**
- Implement the methods required by **Iterator**
- Expose a collection with **IteratorAggregate**
- Accept iterable values as parameters and return them from functions
- Use **is_iterable()** for a runtime check

## What does iterable mean?

In PHP, **iterable** means an array or an object that implements **Traversable**. Both can be consumed with **foreach**:

~~~php
<?php
$fromArray = ['PHP', 'JavaScript'];

foreach ($fromArray as $language) {
    echo $language, "\n";
}
~~~

An iterator object can also be used by **foreach**:

~~~php
<?php
$fromIterator = new ArrayIterator(['PHP', 'JavaScript']);

foreach ($fromIterator as $language) {
    echo $language, "\n";
}
~~~

Output:

~~~text
PHP
JavaScript
~~~

The loop does not need to know whether its input is an array or an iterator object.

## Use iterable as a parameter or return type

Use **iterable** when a function accepts either arrays or traversable objects:

~~~php
<?php
function printLanguages(iterable $languages): void
{
    foreach ($languages as $language) {
        echo $language, "\n";
    }
}

printLanguages(['PHP', 'JavaScript']);
printLanguages(new ArrayIterator(['Python', 'Ruby']));
~~~

A function may also return an iterable. A generator is one way to do that:

~~~php
function activeNames(array $names): iterable
{
    foreach ($names as $name) {
        if ($name !== '') {
            yield $name;
        }
    }
}
~~~

The function's return type is **iterable**, while the call returns a **Generator** object. A generator implements **Traversable**, so it satisfies the declared type.

## Check a value with is_iterable()

Use **is_iterable()** when the input's type is dynamic:

~~~php
$items = new ArrayIterator(['a', 'b']);

if (is_iterable($items)) {
    foreach ($items as $item) {
        echo $item, "\n";
    }
}
~~~

**is_iterable()** returns **true** for arrays and objects that implement **Traversable**. It does not mean that arbitrary objects can be traversed.

The **iterable** type and **is_iterable()** are available starting in PHP 7.1.

## Understand Traversable

**Traversable** is the base interface recognized by PHP's iteration system. It allows objects to be used in **foreach**.

Do not implement **Traversable** directly in userland. Implement **Iterator** or **IteratorAggregate** instead; both extend **Traversable** and define how iteration works.

A generator object also implements **Traversable**, so it can be accepted anywhere an **iterable** is expected.

## Implement the Iterator interface

A custom **Iterator** object manages its own current position. It must implement five methods:

- **current()** returns the current value.
- **key()** returns the current key.
- **next()** advances to the next position.
- **rewind()** returns to the first position.
- **valid()** reports whether the current position contains an item.

~~~php
<?php
class NumberIterator implements Iterator
{
    private int $position = 0;

    /** @var list<int> */
    private array $numbers;

    public function __construct(array $numbers)
    {
        $this->numbers = array_values($numbers);
    }

    public function current(): mixed
    {
        return $this->numbers[$this->position];
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
        return array_key_exists($this->position, $this->numbers);
    }
}

$numbers = new NumberIterator([10, 20, 30]);

foreach ($numbers as $key => $number) {
    echo "{$key}: {$number}\n";
}
~~~

Output:

~~~text
0: 10
1: 20
2: 30
~~~

When **foreach** starts, the iterator is rewound. The loop checks **valid()** before reading a position, gets its current key and value, runs the loop body, then calls **next()**. It repeats until **valid()** returns **false**.

The interface lets an object control how values are produced, but implementing it requires careful state management.

## Use IteratorAggregate for a collection

A collection often already stores its items. **IteratorAggregate** lets the collection return an iterator over those items rather than implementing position tracking itself:

~~~php
<?php
class Playlist implements IteratorAggregate
{
    /** @param array<string, string> $tracks */
    public function __construct(
        private array $tracks,
    ) {
    }

    public function getIterator(): Traversable
    {
        return new ArrayIterator($this->tracks);
    }
}

$playlist = new Playlist([
    'track-1' => 'Opening',
    'track-2' => 'Main theme',
]);

foreach ($playlist as $key => $title) {
    echo "{$key}: {$title}\n";
}
~~~

Output:

~~~text
track-1: Opening
track-2: Main theme
~~~

The collection provides **getIterator()**, which returns a **Traversable** object. Here, **ArrayIterator** wraps the array. **foreach** automatically asks the collection for its iterator.

### Return a generator from getIterator()

A generator can also provide the iteration behavior:

~~~php
<?php
class NumberCollection implements IteratorAggregate
{
    public function __construct(
        private array $numbers,
    ) {
    }

    public function getIterator(): Traversable
    {
        foreach ($this->numbers as $number) {
            yield $number * 10;
        }
    }
}

foreach (new NumberCollection([1, 2, 3]) as $number) {
    echo $number, "\n";
}
~~~

Output:

~~~text
10
20
30
~~~

Because **getIterator()** contains **yield**, it returns a generator, which implements **Traversable**.

## Iterator or IteratorAggregate?

Choose **Iterator** when the object itself needs to control a cursor or calculate each step through the five iterator methods.

Choose **IteratorAggregate** when the object can return an existing iterator, such as **ArrayIterator**, or a generator. It keeps the collection class focused on its data and lets the returned iterator handle traversal.

A custom **Iterator** has internal state. Reusing the same iterator in nested loops or expecting independent simultaneous traversals can be surprising. An **IteratorAggregate** can return a fresh iterator each time **getIterator()** is called.

## Iterate multiple sources with yield from

A generator that returns an iterable can delegate to another iterable with **yield from**:

~~~php
<?php
function allLanguages(iterable $first, iterable $second): iterable
{
    yield from $first;
    yield from $second;
}

$languages = allLanguages(
    ['PHP', 'JavaScript'],
    new ArrayIterator(['Python', 'Ruby']),
);

foreach ($languages as $language) {
    echo $language, "\n";
}
~~~

Output:

~~~text
PHP
JavaScript
Python
Ruby
~~~

Both arrays and traversable objects can be used as inputs. The generator exposes them as one iterable sequence.

## Common mistakes

- Assuming **iterable** means any object.
- Passing an object that does not implement **Traversable** to a function expecting **iterable**.
- Trying to implement **Traversable** directly instead of implementing **Iterator** or **IteratorAggregate**.
- Returning an array from **IteratorAggregate::getIterator()** instead of a **Traversable** object.
- Forgetting one of the five required **Iterator** methods.
- Returning an invalid current value when **valid()** is true.
- Forgetting to reset the iterator's state in **rewind()**.
- Sharing one stateful iterator across nested loops when independent traversal is expected.
- Treating **iterable** as a class that can be instantiated.
- Assuming **is_iterable()** validates the values inside a collection.

## Try it yourself

1. Call **printLanguages()** once with an array and once with an **ArrayIterator**.
2. Add a **count()** method to **Playlist** and compare it with counting items during iteration.
3. Update **NumberIterator** to return string keys instead of numeric positions.
4. Implement **IteratorAggregate** with a generator in **getIterator()**.
5. Use **is_iterable()** to check an array, a generator, an **ArrayIterator**, and a plain **stdClass** object.
6. Make **allLanguages()** accept two different iterable sources and combine them.
7. Explain why a collection returning a fresh iterator can be easier to reuse than a single stateful **Iterator** object.

## Key takeaways

- **iterable** is the type for arrays and objects implementing **Traversable**.
- **Traversable** is the base interface recognized by PHP's iteration system.
- Implement **Iterator** to manage position, keys, values, and advancement yourself.
- Implement **IteratorAggregate** to return another **Traversable** iterator for the collection.
- Generators, **ArrayIterator**, and custom iterator objects can all be iterable.
- Use **is_iterable()** to check a dynamic value at runtime.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [046 - Generators and Yield](046-generators-and-yield.md) | [Open Q&A](../qnas/047-iterators-traversable-and-iterable-values.md) | [Open example](../examples/047-iterators-traversable-and-iterable-values.md) | [048 - Working with Strings](048-working-with-strings.md) |
