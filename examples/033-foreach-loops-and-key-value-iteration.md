# 033 - foreach Loops and Key-Value Iteration: Complete Example

This command-line program visits array values and keys, builds a transformed array, updates values in place by reference, traverses nested collections, destructures list rows, and iterates over a plain object's visible properties.

## Question

How can PHP process an entire collection without manually managing a counter, preserve keys when creating transformed data, and safely update array elements in place when needed?

## Goal

Create this command-line project:

~~~text
foreach-iteration-example/
└── index.php
~~~

The program will:

- Visit values with the simple `foreach` form
- Print keys and values from an associative array
- Show that changing a normal loop variable does not replace a scalar in the source array
- Build a new array with transformed values and preserved keys
- Update an array in place using a reference and then unset that reference
- Iterate over a nested array
- Destructure fixed-shape rows
- Iterate over a plain object's properties

## Complete code

Create `index.php` with this code:

~~~php
<?php
echo "=== 1. Values ===\n";

$colors = ['red', 'green', 'blue'];

foreach ($colors as $color) {
    echo $color, "\n";
}

echo "\n=== 2. Keys and values ===\n";

$prices = [
    'notebook' => 4.50,
    'pen' => 1.25,
    'folder' => 2.00,
];

foreach ($prices as $product => $price) {
    echo $product, ': ', $price, "\n";
}

echo "\n=== 3. A normal value variable ===\n";

$numbers = [2, 4, 6];

foreach ($numbers as $number) {
    $number *= 10;
}

echo 'Stored values: ', implode(', ', $numbers), "\n";

echo "\n=== 4. Build a transformed array ===\n";

$originalPrices = [
    'book' => 20,
    'bag' => 35,
];

$discountedPrices = [];

foreach ($originalPrices as $product => $price) {
    $discountedPrices[$product] = $price * 0.9;
}

var_export($discountedPrices);
echo "\n";

echo "\n=== 5. Update elements by reference ===\n";

$scores = [10, 20, 30];

foreach ($scores as &$score) {
    $score += 5;
}
unset($score);

var_export($scores);
echo "\n";

echo "\n=== 6. Nested key-value iteration ===\n";

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

echo "\n=== 7. Destructure list rows ===\n";

$users = [
    ['Asha', 24],
    ['Dev', 31],
];

foreach ($users as [$name, $age]) {
    echo "{$name} is {$age}\n";
}

echo "\n=== 8. Iterate over object properties ===\n";

$profile = (object) [
    'name' => 'Asha',
    'role' => 'editor',
];

foreach ($profile as $property => $value) {
    echo "{$property}: {$value}\n";
}
~~~

## Expected output

~~~text
=== 1. Values ===
red
green
blue

=== 2. Keys and values ===
notebook: 4.5
pen: 1.25
folder: 2

=== 3. A normal value variable ===
Stored values: 2, 4, 6

=== 4. Build a transformed array ===
array (
  'book' => 18.0,
  'bag' => 31.5,
)

=== 5. Update elements by reference ===
array (
  0 => 15,
  1 => 25,
  2 => 35,
)

=== 6. Nested key-value iteration ===
fruit:
- apples: 4
- pears: 2
drinks:
- water: 6
- juice: 3

=== 7. Destructure list rows ===
Asha is 24
Dev is 31

=== 8. Iterate over object properties ===
name: Asha
role: editor
~~~

## How it works

### 1. Visit each value

The first loop assigns each color to `$color`. PHP advances to the next array element automatically; no numeric index or counter update is required.

### 2. Read keys and values

The `$product => $price` form exposes both parts of each array entry. Associative keys such as `notebook` and `pen` are useful labels, not positions in a numeric sequence.

### 3. Understand the ordinary value variable

The loop multiplies `$number`, but that variable is the current value rather than a reference to the source slot. The original `$numbers` array stays `2, 4, 6`.

### 4. Create a new transformed array

`$discountedPrices` is a separate result. The source key is copied to the new array and its price is multiplied by `0.9`. The original prices remain unchanged.

### 5. Update source values intentionally

The ampersand makes `$score` refer to each element in `$scores`, so adding five changes the array itself. `unset($score)` breaks its reference to the last element immediately after the loop.

### 6. Traverse nested arrays

The outer loop visits categories; the inner loop visits products in the current category. Distinct names keep the two levels clear.

### 7. Destructure rows

Each user row has a name followed by an age, so `[$name, $age]` assigns those values directly. Every row needs that same shape and order. This short destructuring syntax requires PHP 7.1 or later.

### 8. Traverse object properties

The plain object exposes its `name` and `role` properties to `foreach` in this scope. Classes can define custom iteration behavior with iterable interfaces.

## Run it

Use PHP 7.1 or later because this example uses short array destructuring in a `foreach` declaration.

~~~sh
php --version
php index.php
~~~

The code is displayed here as a learning example. Run it locally if you want to compare your output with the expected output above.

## Make a change

1. Add another color and print it with the value-only loop.
2. Add an item to `$prices` and print its key and price.
3. Change the multiplication in the normal value loop and confirm `$numbers` stays unchanged.
4. Add another product to `$originalPrices` and check that its key appears in `$discountedPrices`.
5. Remove `unset($score)`, reuse `$score` in a later loop, and inspect why that is unsafe; then restore `unset()`.
6. Add another category and nested product to `$inventory`.
7. Add a third user row with the same name-and-age shape.
8. Add another public property to `$profile` and display it.
9. Replace a by-reference update with a new result array and compare which approach is easier to follow.

## Common mistakes

- Treating an array key as if it were always a zero-based position
- Expecting reassignment of a normal loop variable to update a scalar source element
- Using `&` when a separate transformed array would be clearer
- Forgetting to `unset()` a by-reference loop variable afterward
- Reusing the same key or value variable for both outer and inner loops
- Destructuring rows whose order or shape does not match the variables
- Passing a scalar or uninitialized value to `foreach`
- Changing collection structure during iteration without a deliberate plan

## Key takeaways

- Use `foreach ($items as $value)` for values and `foreach ($items as $key => $value)` for key-value pairs.
- The source array's keys and iteration order are preserved.
- Build a new array for ordinary transformations; use reference iteration only when in-place updates are intended.
- Always unset a by-reference loop variable after the loop.
- Nested arrays, iterable objects, and fixed-shape rows can all be traversed with `foreach`.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [032 - Loops with while, do-while, and for](../lessons/032-loops-with-while-do-while-and-for.md) | [Read this lesson](../lessons/033-foreach-loops-and-key-value-iteration.md) | [Open Q&A](../qnas/033-foreach-loops-and-key-value-iteration.md) | [034 - break, continue, goto, and declare](../lessons/034-break-continue-goto-and-declare.md) |
