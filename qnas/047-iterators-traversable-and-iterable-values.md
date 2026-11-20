# 047 - Iterators, Traversable, and iterable Values: Q&A

Review PHP's **iterable** type, the **Traversable** interface, custom iterators, and **IteratorAggregate** collections. Try each question before reading its answer. See the [chapter lesson](../lessons/047-iterators-traversable-and-iterable-values.md).

## Multiple-choice questions

### 1. Which values are accepted by PHP's **iterable** type?

A. Arrays and objects implementing **Traversable**
B. Strings and integers
C. Every object
D. Arrays only

**Answer:** A. Arrays and objects implementing **Traversable**.

**Explanation:** **iterable** is the type alias for **array|Traversable**.

### 2. Which interface is the base recognized by PHP's iteration system?

A. **Countable**
B. **Traversable**
C. **ArrayAccess**
D. **IteratorAggregate**

**Answer:** B. **Traversable**.

**Explanation:** **Iterator** and **IteratorAggregate** extend **Traversable**.

### 3. Which interfaces should userland classes implement to provide custom traversal?

A. **Traversable** directly
B. **Iterator** or **IteratorAggregate**
C. **Countable** only
D. **ArrayAccess** only

**Answer:** B. **Iterator** or **IteratorAggregate**.

**Explanation:** Userland classes should implement one of these interfaces rather than implementing **Traversable** directly.

### 4. How many methods does **Iterator** require?

A. Three
B. Four
C. Five
D. Seven

**Answer:** C. Five.

**Explanation:** The methods are **current()**, **key()**, **next()**, **rewind()**, and **valid()**.

### 5. What does **Iterator::current()** return?

A. The current value
B. The current key
C. The number of items
D. The next iterator object

**Answer:** A. The current value.

**Explanation:** The method returns the value at the iterator's current position.

### 6. What does **Iterator::key()** return?

A. The current value
B. The current key
C. The next key only
D. A boolean

**Answer:** B. The current key.

**Explanation:** The key is the value associated with the current position.

### 7. What does **Iterator::next()** do?

A. Returns the current value
B. Advances the iterator to its next position
C. Checks whether the iterator is finished
D. Resets the iterator

**Answer:** B. Advances the iterator to its next position.

**Explanation:** **foreach** calls the iterator's advancement method after processing a value.

### 8. What does **Iterator::rewind()** do?

A. Moves to the first position
B. Deletes the current element
C. Returns the iterator's key
D. Closes the iterator

**Answer:** A. Moves to the first position.

**Explanation:** It resets the iterator for traversal from the beginning, if the iterator supports that behavior.

### 9. What does **Iterator::valid()** communicate?

A. Whether the current position contains an item
B. Whether the current value is a string
C. Whether the iterator is an array
D. The current key

**Answer:** A. Whether the current position contains an item.

**Explanation:** When **valid()** returns **false**, iteration is finished.

### 10. What method does **IteratorAggregate** require?

A. **getIterator()**
B. **current()**
C. **getCurrent()**
D. **rewind()**

**Answer:** A. **getIterator()**.

**Explanation:** It returns a **Traversable** object to iterate over the collection's contents.

### 11. Which is a valid return value from **IteratorAggregate::getIterator()**?

A. A plain array
B. An **ArrayIterator**
C. An integer
D. A string

**Answer:** B. An **ArrayIterator**.

**Explanation:** The method must return an object implementing **Traversable**, such as **ArrayIterator** or a generator.

### 12. Which PHP value is iterable?

A. A plain **stdClass** with no traversal interface
B. An **ArrayIterator**
C. A closure
D. An integer

**Answer:** B. An **ArrayIterator**.

**Explanation:** **ArrayIterator** implements iterator behavior and can be used with **foreach**.

### 13. What can a function parameter typed as **iterable** receive?

A. Only an array
B. Only an **Iterator**
C. An array or any **Traversable** object
D. Only a generator

**Answer:** C. An array or any **Traversable** object.

**Explanation:** The type accepts both array data and objects implementing the traversal interfaces.

### 14. What does **is_iterable($value)** check?

A. Whether a value is an array or implements **Traversable**
B. Whether all elements have the same type
C. Whether a value is countable
D. Whether an object implements **Countable**

**Answer:** A. Whether a value is an array or implements **Traversable**.

**Explanation:** It checks the outer value's iterability, not the types of its contents.

### 15. Which is usually simpler when a collection can return an existing iterator?

A. Implementing **IteratorAggregate**
B. Implementing **Iterator** and duplicating the collection
C. Implementing **Traversable** directly
D. Converting the collection to a string

**Answer:** A. Implementing **IteratorAggregate**.

**Explanation:** The collection returns an iterator and does not need to implement its own cursor methods.

## Short-answer questions

### 16. What does **iterable** represent in PHP?

**Answer:** It represents an array or an object implementing **Traversable**.

**Explanation:** It can be used as a parameter or return type.

### 17. Can a plain object be passed to a function typed as **iterable**?

**Answer:** Not unless it implements **Traversable** through **Iterator** or **IteratorAggregate**.

**Explanation:** Being an object alone does not make a value iterable.

### 18. Name the five methods required by **Iterator**.

**Answer:** **current()**, **key()**, **next()**, **rewind()**, and **valid()**.

**Explanation:** Together they provide the current value, key, advancement, reset, and completion check.

### 19. What is the role of **IteratorAggregate::getIterator()**?

**Answer:** It returns a **Traversable** iterator for the collection's contents.

**Explanation:** **foreach** uses that iterator to traverse the aggregate.

### 20. Why might a collection choose **IteratorAggregate** over **Iterator**?

**Answer:** It can return an existing iterator or generator instead of managing cursor state itself.

**Explanation:** This often keeps collection code simpler and can provide a fresh iterator for each traversal.

### 21. Can an array be returned directly from **getIterator()** when its return type is **Traversable**?

**Answer:** No.

**Explanation:** The method must return a traversable object, such as **ArrayIterator** or a generator.

### 22. What does a generator have to do with **Traversable**?

**Answer:** A generator object implements **Traversable**, so it can be used where an iterable is accepted.

**Explanation:** A function with **yield** returns a generator object.

### 23. What does **yield from $items** do inside a generator?

**Answer:** It delegates the values from **$items** to the current generator.

**Explanation:** **$items** may be an array or another traversable value.

### 24. What happens when **Iterator::valid()** returns **false** during **foreach**?

**Answer:** The loop ends.

**Explanation:** There is no current item to process.

### 25. Does **is_iterable()** validate every element inside an iterable?

**Answer:** No.

**Explanation:** It checks whether the outer value can be iterated, not whether its elements satisfy a particular type.

### 26. Which PHP version introduced the **iterable** type?

**Answer:** PHP 7.1.

**Explanation:** The type and **is_iterable()** were introduced in PHP 7.1.

### 27. What does **Iterator::rewind()** typically do before a traversal?

**Answer:** It resets the iterator's position to the beginning.

**Explanation:** Custom iterators define exactly how this reset works.

### 28. Why can nested loops over one stateful iterator be surprising?

**Answer:** Both loops may operate on the same internal cursor.

**Explanation:** Use independent iterators when each loop needs its own traversal state.

### 29. Is **Traversable** a class that you instantiate?

**Answer:** No. It is an interface recognized by PHP's iteration system.

**Explanation:** Userland objects provide traversal by implementing **Iterator** or **IteratorAggregate**.

## True or false

### 30. An array is iterable in PHP.

**Answer:** True.

**Explanation:** Arrays are one of the two kinds of value represented by the **iterable** type.

### 31. Every object is automatically iterable.

**Answer:** False.

**Explanation:** An object must implement **Traversable** through an appropriate interface.

### 32. **IteratorAggregate** extends **Traversable**.

**Answer:** True.

**Explanation:** It defines the method for returning a traversable iterator.

### 33. Userland classes should implement **Traversable** directly.

**Answer:** False.

**Explanation:** Implement **Iterator** or **IteratorAggregate** instead.

### 34. **Iterator::current()** returns the current key.

**Answer:** False.

**Explanation:** It returns the current value; **key()** returns the key.

### 35. **IteratorAggregate::getIterator()** must return a traversable object.

**Answer:** True.

**Explanation:** It can return an **ArrayIterator**, a generator, or another **Traversable**.

### 36. A generator object can be passed to a function that expects **iterable**.

**Answer:** True.

**Explanation:** Generators implement **Traversable**.

### 37. **is_iterable([])** returns **false** because an empty array has no items.

**Answer:** False.

**Explanation:** An empty array is still an array and therefore is iterable.

## Predict the output

### 38. What does this code print?

~~~php
<?php
$items = new ArrayIterator(['PHP', 'Ruby']);

foreach ($items as $key => $value) {
    echo "{$key}: {$value}\n";
}
~~~

**Answer:**

~~~text
0: PHP
1: Ruby
~~~

**Explanation:** **ArrayIterator** exposes the array's keys and values to **foreach**.

### 39. What does this code print?

~~~php
<?php
function printAll(iterable $items): void
{
    foreach ($items as $item) {
        echo $item, "\n";
    }
}

printAll(['red', 'blue']);
printAll(new ArrayIterator(['green']));
~~~

**Answer:**

~~~text
red
blue
green
~~~

**Explanation:** The **iterable** parameter accepts both the array and the **ArrayIterator**.

### 40. What does this code print?

~~~php
<?php
class Shelf implements IteratorAggregate
{
    public function getIterator(): Traversable
    {
        return new ArrayIterator([
            'top' => 'PHP',
            'bottom' => 'SQL',
        ]);
    }
}

foreach (new Shelf() as $position => $book) {
    echo "{$position}: {$book}\n";
}
~~~

**Answer:**

~~~text
top: PHP
bottom: SQL
~~~

**Explanation:** **foreach** asks **Shelf** for its iterator and traverses the returned **ArrayIterator**.

### 41. What does this code print?

~~~php
<?php
$values = [1, 2];
$iterator = new ArrayIterator([3, 4]);

var_dump(is_iterable($values));
var_dump(is_iterable($iterator));
~~~

**Answer:**

~~~text
bool(true)
bool(true)
~~~

**Explanation:** Arrays and **Traversable** objects both satisfy the iterable type.

### 42. What does this code print?

~~~php
<?php
var_dump(is_iterable(new stdClass()));
~~~

**Answer:**

~~~text
bool(false)
~~~

**Explanation:** A plain **stdClass** object does not implement **Traversable**.

## Write code

### 43. Create an **IteratorAggregate** collection that exposes **['a', 'b']** with an **ArrayIterator**.

**Answer:**

~~~php
<?php
class Letters implements IteratorAggregate
{
    public function getIterator(): Traversable
    {
        return new ArrayIterator(['a', 'b']);
    }
}

foreach (new Letters() as $letter) {
    echo $letter, "\n";
}
~~~

Output:

~~~text
a
b
~~~

**Explanation:** The collection returns a traversable object from **getIterator()**.

### 44. Write a function with an **iterable** parameter that prints all values.

**Answer:**

~~~php
<?php
function printValues(iterable $values): void
{
    foreach ($values as $value) {
        echo $value, "\n";
    }
}

printValues([5, 8]);
printValues(new ArrayIterator([13]));
~~~

Output:

~~~text
5
8
13
~~~

**Explanation:** The same function accepts an array and a traversable object.

### 45. Write an **iterable** generator that delegates to an array and an **ArrayIterator**.

**Answer:**

~~~php
<?php
function allValues(iterable $first, iterable $second): iterable
{
    yield from $first;
    yield from $second;
}

foreach (allValues([1, 2], new ArrayIterator([3, 4])) as $value) {
    echo $value, "\n";
}
~~~

Output:

~~~text
1
2
3
4
~~~

**Explanation:** The generator's **yield from** statements delegate values from both iterable inputs.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [046 - Generators and Yield](../lessons/046-generators-and-yield.md) | [Read lesson](../lessons/047-iterators-traversable-and-iterable-values.md) | [Open example](../examples/047-iterators-traversable-and-iterable-values.md) | [048 - Working with Strings](../lessons/048-working-with-strings.md) |
