# 062 - Classes, Objects, and Object-Oriented PHP: Example

## Question

How can one class create independent objects, and how can you tell whether two variables share the same object or merely have objects with equal values?

## Complete example

This standalone example works with PHP 7.4 and later. Save it as `classes-and-objects.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

class Counter
{
    public int $value = 0;

    public function increment(): void
    {
        $this->value++;
    }
}

// Each new expression creates a separate instance.
$firstCounter = new Counter();
$secondCounter = new Counter();

$firstCounter->increment();
$secondCounter->increment();

echo 'First counter: ' . $firstCounter->value . PHP_EOL;
echo 'Second counter: ' . $secondCounter->value . PHP_EOL;

// Assignment creates another variable that identifies the same object.
$alias = $firstCounter;
$alias->increment();

echo 'First counter after alias increment: '
    . $firstCounter->value
    . PHP_EOL;
echo 'Alias is the same instance: '
    . ($firstCounter === $alias ? 'yes' : 'no')
    . PHP_EOL;

// The separate objects have equal state, but are not identical.
echo 'Counters have equal values: '
    . ($firstCounter == $secondCounter ? 'yes' : 'no')
    . PHP_EOL;
echo 'Counters are the same instance: '
    . ($firstCounter === $secondCounter ? 'yes' : 'no')
    . PHP_EOL;

// Reassigning the alias makes it identify a different object.
// It does not replace the object still identified by $firstCounter.
$alias = new Counter();
$alias->increment();

echo 'First counter after alias reassignment: '
    . $firstCounter->value
    . PHP_EOL;
echo 'New alias counter: ' . $alias->value . PHP_EOL;
echo 'First counter is the alias instance: '
    . ($firstCounter === $alias ? 'yes' : 'no')
    . PHP_EOL;

// clone creates a distinct, shallow object copy.
$counterCopy = clone $firstCounter;
$counterCopy->increment();

echo 'Original after clone increment: '
    . $firstCounter->value
    . PHP_EOL;
echo 'Clone after increment: '
    . $counterCopy->value
    . PHP_EOL;
echo 'Clone is the original instance: '
    . ($counterCopy === $firstCounter ? 'yes' : 'no')
    . PHP_EOL;
~~~

## Expected output

~~~text
First counter: 1
Second counter: 1
First counter after alias increment: 2
Alias is the same instance: yes
Counters have equal values: no
Counters are the same instance: no
First counter after alias reassignment: 2
New alias counter: 1
First counter is the alias instance: no
Original after clone increment: 2
Clone after increment: 3
Clone is the original instance: no
~~~

Notice that after the alias increments the first counter, the first counter has value `2` while the separate second counter still has value `1`. Therefore they are separate instances with different values and compare false with both `==` and `===`.

## Explanation

### 1. A class defines the object shape

`Counter` declares a property and a method. The property stores state; the method changes that state. The class itself is not a counter object.

### 2. Each new expression creates an instance

`new Counter()` creates a counter object. Calling it twice creates two separate objects with independent property values. Calling `increment()` on one object does not change the other.

The method uses `$this->value` to update the property on the object receiving the method call.

### 3. Assignment can create an alias to the same object

After `$alias = $firstCounter`, both variables identify the same object. Incrementing through `$alias` changes the state visible through `$firstCounter` too.

The `===` comparison checks object identity. It returns `yes` for the alias and original.

### 4. Reassignment does not mutate the old object

The later statement `$alias = new Counter()` points `$alias` at a new object. The original variable remains connected to its original counter.

The distinction is:

- Calling a method may mutate an object.
- Assigning a new object to a variable changes which object that variable identifies.
- Neither operation automatically changes a different variable's assignment.

### 5. Value equality differs from identity

The `==` operator compares two objects of the same class by their property values. The `===` operator checks whether the variables identify the same instance.

At the point of the comparison, the first counter is `2` and the second is `1`, so `==` is false. If both were incremented to the same value, `==` would be true while `===` would remain false because they are separate objects.

### 6. Cloning creates a shallow copy

`clone $firstCounter` creates a distinct object with the current property values. Incrementing the clone does not change the original counter.

Cloning is shallow: if an object contains another object as a property, the nested object is not automatically deep-copied. For nested mutable state, define the copy behavior deliberately.

## Try it yourself

1. Increment `$secondCounter` before the equality checks. Observe the difference between `==` and `===`.
2. Create `$secondAlias = $secondCounter` and check whether those two variables are identical.
3. Reassign `$firstCounter` to a new `Counter` and observe that the previous object can still be reached through any remaining alias.
4. Add a `reset()` method that sets the current object's value to zero.
5. Add an object property to `Counter`, clone it, and investigate which nested changes are shared.

## Key takeaways

- A class describes a kind of object; `new` creates an instance.
- Each instance holds its own state.
- Object assignment can make two variables identify the same instance.
- Reassigning one variable does not reassign other variables.
- Use `===` for object identity and `==` for value comparison.
- `clone` makes a shallow copy, not a guaranteed deep copy.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [061 - Debugging, diagnostics, and application logging](../lessons/061-debugging-diagnostics-and-application-logging.md) | [Q&A](../qnas/062-classes-objects-and-object-oriented-php.md) | [Lesson](../lessons/062-classes-objects-and-object-oriented-php.md) | [063 - Properties, methods, and object creation](../lessons/063-properties-methods-and-object-creation.md) |
