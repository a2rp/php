# 033 - foreach Loops and Key-Value Iteration

PHP's `foreach` loop visits the elements of an array or an iterable object. It assigns each element to a value variable, and it can also assign the element's key. Unlike a counter loop, `foreach` advances through the iterable for you.

Use `foreach` when the task is to process every item in a collection. Use `for`, `while`, or `do-while` when the task is controlled by a counter or condition instead.

## What you will learn

- How to loop over values and over key-value pairs
- How numeric and associative array keys behave during iteration
- How to traverse nested arrays and iterable objects
- How to modify values safely without references
- How reference iteration changes the original array
- Why a by-reference loop variable should be unset afterward
- How to destructure simple array rows

## Loop over values

The simplest form assigns each value to a variable:

~~~php
<?php
$colors = ['red', 'green', 'blue'];

foreach ($colors as $color) {
    echo $color, "\n";
}
~~~

Output:

~~~text
red
green
blue
~~~

PHP takes the next element for every iteration. You do not need to initialize, increment, or test an index yourself.

## Loop over keys and values

Use `key => value` when each item's key is useful:

~~~php
<?php
$prices = [
    'notebook' => 4.50,
    'pen' => 1.25,
    'folder' => 2.00,
];

foreach ($prices as $product => $price) {
    echo $product, ' price=', $price, "\n";
}
~~~

Output:

~~~text
notebook price=4.5
pen price=1.25
folder price=2
~~~

The key is assigned to `$product` and the value to `$price` on each iteration. Use descriptive variable names so readers can tell which is which.

### Numeric keys are not always simple counters

PHP arrays can have integer keys with gaps, string keys, or a mixture of both. `foreach` visits the existing keys; it does not renumber them.

~~~php
<?php
$tasks = [
    2 => 'Review notes',
    5 => 'Practice examples',
];

foreach ($tasks as $taskId => $task) {
    echo "Task {$taskId}: {$task}\n";
}
~~~

Output:

~~~text
Task 2: Review notes
Task 5: Practice examples
~~~

The keys are `2` and `5`, not loop positions `0` and `1`. Use the key supplied by `foreach` when it represents an identifier or label.

## The iterable is not changed by a normal value variable

By default, the loop variable receives the current value. Reassigning that variable does not replace a scalar value in the original array.

~~~php
<?php
$numbers = [2, 4, 6];

foreach ($numbers as $number) {
    $number *= 10;
}

var_export($numbers);
~~~

Output:

~~~text
array (
  0 => 2,
  1 => 4,
  2 => 6,
)
~~~

If you want transformed values, write them to a new array:

~~~php
<?php
$prices = [
    'book' => 20,
    'bag' => 35,
];

$discountedPrices = [];

foreach ($prices as $product => $price) {
    $discountedPrices[$product] = $price * 0.9;
}

var_export($discountedPrices);
~~~

Output:

~~~text
array (
  'book' => 18.0,
  'bag' => 31.5,
)
~~~

For objects in an array, the loop variable contains the object handle. Changing a property on that object can affect the same object even though the loop did not use `&`. Reference iteration is primarily relevant when you need to replace array elements themselves.

## Modify array elements with a reference

Prefix the value variable with `&` to make it refer to each original array element:

~~~php
<?php
$scores = [10, 20, 30];

foreach ($scores as &$score) {
    $score += 5;
}
unset($score);

var_export($scores);
~~~

Output:

~~~text
array (
  0 => 15,
  1 => 25,
  2 => 35,
)
~~~

The `&` means assignments to `$score` update the corresponding element in `$scores`.

### Unset the reference variable after the loop

After a by-reference `foreach`, the value variable remains a reference to the final array element. Call `unset()` on that variable immediately after the loop to break the reference.

~~~php
<?php
foreach ($scores as &$score) {
    $score *= 2;
}
unset($score);

// $score is no longer connected to the final element of $scores.
~~~

Without `unset($score)`, reusing `$score` later can overwrite the last array element unexpectedly. `unset()` removes the variable's reference; it does not remove an element from the array.

Only use `&` when the original array must be modified in place. For ordinary reads or transformations into a new array, the non-reference form is easier to reason about.

## Iterate through a nested array

A `foreach` loop can be placed inside another `foreach` loop to visit nested collections:

~~~php
<?php
$inventory = [
    'fruit' => [
        'apples' => 4,
        'pears' => 2,
    ],
    'drinks' => [
        'water' => 6,
        'juice' => 3,
    ],
];

foreach ($inventory as $category => $products) {
    echo "{$category}:\n";

    foreach ($products as $product => $quantity) {
        echo "- {$product}: {$quantity}\n";
    }
}
~~~

Output:

~~~text
fruit:
- apples: 4
- pears: 2
drinks:
- water: 6
- juice: 3
~~~

Use different variable names for each level. In the inner loop, `$product` and `$quantity` describe the current nested entries while `$category` still refers to the outer iteration.

## Destructure list rows

When each array value has a known list shape, PHP can unpack its values directly into variables in the `foreach` declaration. This syntax is available from PHP 7.1.

~~~php
<?php
$users = [
    ['Asha', 24],
    ['Dev', 31],
];

foreach ($users as [$name, $age]) {
    echo "{$name} is {$age}\n";
}
~~~

Output:

~~~text
Asha is 24
Dev is 31
~~~

The order and shape of each row must match the variables you declare. For irregular or associative records, key-value iteration is often clearer.

## Iterate over objects and other iterables

`foreach` works with arrays and objects that are iterable. A plain object is traversed over the properties visible in the current scope. A class can define its own iteration behavior through iterable interfaces such as `Iterator` or `IteratorAggregate`.

~~~php
<?php
$profile = (object) [
    'name' => 'Asha',
    'role' => 'editor',
];

foreach ($profile as $property => $value) {
    echo "{$property}: {$value}\n";
}
~~~

Output:

~~~text
name: Asha
role: editor
~~~

An array or an iterable object can be used as the loop source. Passing an uninitialized variable or a value that is not iterable causes an error. When the source is uncertain, validate it before the loop, for example with `is_iterable()`.

~~~php
<?php
if (is_iterable($records)) {
    foreach ($records as $record) {
        // Process the record.
    }
}
~~~

The `iterable` type declaration is available from PHP 7.1 and describes arrays and objects implementing `Traversable`.

## Keys, ordering, and structure changes

A `foreach` iteration follows the iterable's iteration order. For arrays, that is the array's element order, including its existing keys. Do not infer a zero-based position from an array key unless the array is actually indexed that way.

Avoid adding or removing array entries while traversing the same array unless you have deliberately accounted for PHP's iteration behavior. If you are filtering a collection, build a new array or use a dedicated array operation; that makes the intended result easier to inspect.

## `foreach` compared with counter loops

| Choose | When it fits |
| --- | --- |
| `foreach` | Process each value or key-value pair in an array or iterable object |
| `for` | Count through a numeric range or use a controlled increment |
| `while` | Repeat while a condition remains true and the iteration count is not known in advance |
| `do-while` | Run the body once before deciding whether to repeat |

## Try it yourself

1. Print each value in a three-item array with `foreach`.
2. Print both the key and value from an associative array.
3. Iterate over an array with keys `2` and `5` and confirm those keys are preserved.
4. Multiply a scalar loop variable and check whether the original array changed.
5. Build a new array containing discounted values.
6. Double each value in place with `&` and then `unset()` the loop variable.
7. Add one more nested category to the inventory example.
8. Destructure a list of names and ages into variables.
9. Iterate over a simple object and print its property names and values.
10. Use `is_iterable()` to guard a value that may not be iterable.
11. Explain why reusing a by-reference loop variable after the loop can change the last array element.

## Key takeaways

- `foreach ($items as $value)` visits values; `foreach ($items as $key => $value)` visits keys and values.
- `foreach` preserves an array's existing keys and iterates in its element order.
- A normal value variable does not replace scalar values in the source array; use a separate result array for transformations.
- Use `&` only when you need to update original array elements, and `unset()` the reference variable immediately after the loop.
- Nested loops need distinct variable names so each iteration level remains clear.
- `foreach` handles arrays and iterable objects; it is usually preferable to a counter loop for collections.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [032 - Loops with while, do-while, and for](032-loops-with-while-do-while-and-for.md) | [Open Q&A](../qnas/033-foreach-loops-and-key-value-iteration.md) | [Open example](../examples/033-foreach-loops-and-key-value-iteration.md) | [034 - break, continue, goto, and declare](034-break-continue-goto-and-declare.md) |
