# 077 - Cloning and comparing objects: Example

## Question

What changes when a PHP object is assigned, cloned shallowly, or cloned with a custom `__clone()` method? How can you tell whether two objects have matching state or are the exact same instance?

## Complete PHP example

```php
<?php

declare(strict_types=1);

final class MutableTags
{
    /** @param list<string> $values */
    public function __construct(public array $values)
    {
    }
}

final class ShallowArticle
{
    public function __construct(
        public string $title,
        public MutableTags $tags,
    ) {
    }
}

final class IndependentArticle
{
    public function __construct(
        public string $title,
        public MutableTags $tags,
    ) {
    }

    public function __clone(): void
    {
        // The outer article has already been copied.
        // Give the new article its own mutable tag collection too.
        $this->tags = clone $this->tags;
    }
}

final class Point
{
    public function __construct(
        public int $x,
        public int $y,
    ) {
    }
}

function yesNo(bool $value): string
{
    return $value ? 'yes' : 'no';
}

// Assignment creates an alias to the same object.
$assignedArticle = new ShallowArticle(
    'PHP basics',
    new MutableTags(['php']),
);
$articleAlias = $assignedArticle;
$articleAlias->title = 'Updated through the alias';

echo "Assignment changes the same object: ";
echo yesNo($assignedArticle === $articleAlias) . "\n";
echo "Title visible through the original variable: {$assignedArticle->title}\n\n";

// A default clone is a new outer object, but its nested MutableTags stays shared.
$shallowSource = new ShallowArticle(
    'Shallow copy demo',
    new MutableTags(['php']),
);
$shallowCopy = clone $shallowSource;
$shallowCopy->title = 'Changed clone title';
$shallowCopy->tags->values[] = 'objects';

echo "Shallow clone has a separate outer object: ";
echo yesNo($shallowSource !== $shallowCopy) . "\n";
echo "Source title after changing clone: {$shallowSource->title}\n";
echo 'Source tags after changing clone: ';
echo implode(', ', $shallowSource->tags->values) . "\n\n";

// __clone() detaches the nested mutable object.
$independentSource = new IndependentArticle(
    'Independent copy demo',
    new MutableTags(['php']),
);
$independentCopy = clone $independentSource;
$independentCopy->tags->values[] = 'objects';

echo 'Deep-copy source tags after changing clone: ';
echo implode(', ', $independentSource->tags->values) . "\n";
echo 'Deep-copy clone tags: ';
echo implode(', ', $independentCopy->tags->values) . "\n\n";

// == compares ordinary object state; === checks whether it is one instance.
$point = new Point(3, 5);
$matchingPoint = new Point(3, 5);
$pointClone = clone $point;
$pointAlias = $point;

echo 'Matching values with ==: ';
echo yesNo($point == $matchingPoint) . "\n";
echo 'Matching values with ===: ';
echo yesNo($point === $matchingPoint) . "\n";
echo 'Clone has matching values with ==: ';
echo yesNo($point == $pointClone) . "\n";
echo 'Clone is the same instance with ===: ';
echo yesNo($point === $pointClone) . "\n";
echo 'Alias is the same instance with ===: ';
echo yesNo($point === $pointAlias) . "\n";
```

The example uses constructor property promotion and named classes, so it requires PHP 8.0 or newer. It does not use PHP 8.5's clone property override syntax.

## Expected output

```text
Assignment changes the same object: yes
Title visible through the original variable: Updated through the alias

Shallow clone has a separate outer object: yes
Source title after changing clone: Shallow copy demo
Source tags after changing clone: php, objects

Deep-copy source tags after changing clone: php
Deep-copy clone tags: php, objects

Matching values with ==: yes
Matching values with ===: no
Clone has matching values with ==: yes
Clone is the same instance with ===: no
Alias is the same instance with ===: yes
```

## Explanation

### Assignment keeps one object instance

`$articleAlias = $assignedArticle` does not copy the article. Both variables identify one object. Changing the title through the alias changes the title seen through the original variable.

### A default clone separates only the outer object

`$shallowCopy = clone $shallowSource` creates a new `ShallowArticle`. Its `title` property can change independently. The `tags` property still refers to the original `MutableTags`, though, so appending a tag through the clone also changes the source's list.

### `__clone()` detaches the nested object

`IndependentArticle::__clone()` runs on the new article after PHP copies its properties. The method then clones the `MutableTags` object. The source and copy now have separate tag collections, so adding a tag to one does not alter the other.

This is a targeted deep-copy policy: it clones the one mutable collaborator that needs independence. If nested values contain more objects, decide explicitly whether each one should be shared, cloned, reset, or recreated.

### Equality and identity are different checks

`$point` and `$matchingPoint` are two separate `Point` objects with the same property values. For these ordinary user-defined objects, `==` returns `true` and `===` returns `false`.

`$pointClone` is also a separate object. Since the clone kept the same values, it compares equal with `==`, but it is not identical with `===`. `$pointAlias`, in contrast, points to the original instance and is identical to it.

## Try these changes

1. Change `$shallowCopy->title`. Does the source title change?
2. Add a second mutable object property to `IndependentArticle`. What should `__clone()` do with it?
3. Change one coordinate on `$pointClone`. What do the `==` and `===` comparisons return afterward?
4. Create a `Point` with different coordinates. How does `==` change?
5. Add a readonly ID to `IndependentArticle`. Which PHP version allows reinitializing it inside `__clone()`?
6. On PHP 8.5 or newer, add a `withX()` method to `Point` that returns `clone($this, ['x' => $x])`.

## Key takeaways

- Assignment makes an alias; `clone` creates a different outer object.
- The default clone is shallow, so nested object references remain shared.
- Use `__clone()` to give the copy its own mutable collaborators or copy-specific state.
- `==` compares ordinary object state; `===` checks whether both variables identify one instance.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [076 - Magic methods](../lessons/076-magic-methods.md) | [Lesson](../lessons/077-cloning-and-comparing-objects.md) | [Q&A](../qnas/077-cloning-and-comparing-objects.md) | [078 - Late static binding](../lessons/078-late-static-binding.md) |
