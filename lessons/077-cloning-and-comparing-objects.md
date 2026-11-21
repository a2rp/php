# 077 - Cloning and comparing objects

Cloning creates a separate object instance from an existing object. Comparing objects answers two different questions: do they have equivalent state, or are they the very same instance? Knowing the difference helps prevent accidental shared state and incorrect identity checks.

## Learning goals

By the end of this chapter, you will be able to:

- distinguish assigning an object from cloning it;
- explain why PHP's default clone is shallow;
- customize a copy with `__clone()`;
- decide when nested objects need their own copies;
- compare object state with `==` and object identity with `===`;
- recognize the newer clone features introduced in PHP 8.3 and PHP 8.5.

## Assigning an object does not copy it

Object variables hold an object handle. Assigning one object variable to another makes both variables refer to the same instance:

~~~php
$first = new stdClass();
$first->name = 'Asha';

$second = $first;
$second->name = 'Ravi';

echo $first->name; // Ravi
~~~

`$first` and `$second` refer to one object. Changing that object through either variable is visible through the other. This is an alias, not a copy.

## Create a separate instance with `clone`

Use the `clone` operator when you need a new instance initialized from an existing object's current state:

~~~php
$copy = clone $original;
~~~

The copied object has a different identity from the source. The clone operation does not call the constructor again. PHP first copies the object's properties, then calls `__clone()` on the new object if the class defines that method.

~~~php
$original = new stdClass();
$original->name = 'Asha';

$copy = clone $original;
$copy->name = 'Ravi';

echo $original->name; // Asha
echo $copy->name;     // Ravi
~~~

Cloning is a deliberate operation. A class may prevent or restrict cloning with its design, so do not assume every object should be copied.

## PHP's default clone is shallow

PHP copies each property value, but it does not recursively clone every object referenced by those properties. If a property contains another object, the original and clone refer to that same nested object unless you explicitly clone it.

~~~php
class TagList
{
    public function __construct(public array $values) {}
}

class Article
{
    public function __construct(
        public string $title,
        public TagList $tags,
    ) {}
}

$original = new Article('PHP guide', new TagList(['php']));
$copy = clone $original;

$copy->title = 'PHP guide, revised';
$copy->tags->values[] = 'objects';

// The title property belongs to each separate Article instance.
echo $original->title; // PHP guide

// The TagList object is still shared by both Article instances.
echo implode(', ', $original->tags->values); // php, objects
~~~

The clone has its own `Article` instance, but its `tags` property still points to the same `TagList` instance. Arrays are copied as values, but an object stored inside an array is still an object handle and remains shared unless cloned.

## Customize copies with `__clone()`

Define `__clone()` when a cloned object needs to detach mutable collaborators, reset derived state, or create a new per-instance identifier. PHP calls this method on the new instance after its properties have been shallow-copied.

~~~php
class Article
{
    public function __construct(
        public string $title,
        public TagList $tags,
    ) {}

    public function __clone(): void
    {
        $this->tags = clone $this->tags;
    }
}
~~~

Now a cloned `Article` receives its own `TagList` object. The values array in that `TagList` is copied as an array value as usual.

A complete `__clone()` method can also reset a cache or generate a new identifier. Keep the behavior intentional: constructors are not rerun, and PHP has already copied the properties before the hook starts. Make the magic method public and use its documented `void` return type.

### Clone a nested object graph deliberately

A shallow copy can contain several nested objects. Clone each mutable collaborator that must become independent. For object arrays, clone the objects in the array when the desired copy semantics require it:

~~~php
public function __clone(): void
{
    foreach ($this->items as $key => $item) {
        if (is_object($item)) {
            $this->items[$key] = clone $item;
        }
    }
}
~~~

This only handles the level shown. If each item itself contains mutable objects, those objects need their own copy strategy. Cyclic graphs and shared child objects need extra care: blindly cloning every edge can duplicate objects that were intentionally shared or fail to preserve the original graph relationships. Define what "independent copy" means for the class before implementing deep cloning.

Immutable value objects are often safe to share. Resource handles, sockets, database connections, and other external resources should not be duplicated with a generic clone unless the class has a safe, explicit way to create a new resource.

## Compare state with `==`

For ordinary user-defined objects, `==` reports equality when both objects are instances of the same class and their corresponding properties have equal values. Property values are compared using loose equality rules.

~~~php
final class Point
{
    public function __construct(
        public int $x,
        public int $y,
    ) {}
}

$first = new Point(3, 5);
$second = new Point(3, 5);

var_dump($first == $second); // true
~~~

These are separate objects, but their class and property values match, so `==` is true. For user-defined objects, the comparison follows PHP's object comparison rules; some internal classes or extensions may define their own comparison behavior.

## Compare identity with `===`

`===` checks whether both variables refer to the exact same object instance. Matching properties do not make two separate instances identical.

~~~php
$first = new Point(3, 5);
$second = new Point(3, 5);
$alias = $first;

var_dump($first === $second); // false
var_dump($first === $alias);  // true
~~~

A clone is a separate instance, so the original and clone are not identical. Their property values may still be equal:

~~~php
$copy = clone $first;

var_dump($first == $copy);  // true, if cloning preserved the same values
var_dump($first === $copy); // false, always a separate instance
~~~

Use `===` when identity matters, such as checking whether two references identify the same entity in memory. Use `==` only when PHP's property-based equality is the meaning you want. For domain-specific equality, an explicit method such as `hasSameCoordinatesAs()` or a value object design can make the rule clearer.

The related operators are:

- `==`: equal state according to PHP's object comparison rules;
- `!=` or `<>`: not equal according to `==`;
- `===`: same object instance;
- `!==`: not the same object instance.

## Readonly properties and cloning

As of PHP 8.3, a readonly property may be reinitialized inside `__clone()`. This is useful for creating a copy with a fresh readonly identifier or resetting a readonly field that belongs to the copy. It is still readonly everywhere else; the special reinitialization is limited to the clone hook.

~~~php
class Snapshot
{
    public function __construct(
        public readonly string $id,
        public readonly string $value,
    ) {}

    public function __clone(): void
    {
        $this->id = bin2hex(random_bytes(8));
    }
}
~~~

This feature requires PHP 8.3 or newer. If supporting an earlier version, use a factory or another explicit copy method to create an instance with the new identifier.

## Clone with selected property overrides in PHP 8.5

PHP 8.5 adds a `clone()` function with an optional second argument: an array of properties to override on the copy. Unlisted properties retain their copied values. Overrides happen after `__clone()` runs and follow property visibility rules. The function can also set readonly properties on the new copy.

~~~php
final class Point
{
    public function __construct(
        public readonly int $x,
        public readonly int $y,
    ) {}

    public function withX(int $x): static
    {
        return clone($this, ['x' => $x]);
    }
}

$point = new Point(3, 5);
$shifted = $point->withX(10);

// $point remains (3, 5); $shifted is (10, 5).
~~~

The `clone()` function requires PHP 8.5 or newer. On older PHP versions, create a new object through a named constructor or copy method instead.

## Choosing a copying strategy

Use the least surprising strategy that fits the object's meaning:

- **Assignment:** both variables should share one object instance.
- **`clone`:** a new instance should start with the current property values.
- **`__clone()`:** some nested mutable state must be detached or copy-specific state reset.
- **A named copy method or factory:** copying needs parameters, validation, or a business-specific transformation.
- **Immutable objects:** sharing is usually preferable because their observable state cannot change.

Do not clone every object automatically. Decide whether each nested value should be shared, copied, reset, or recreated.

## Common mistakes

- **Expecting assignment to copy an object:** it creates another reference to the same instance.
- **Assuming `clone` makes a deep copy:** the default operation is shallow.
- **Assuming the constructor runs during cloning:** it does not; use `__clone()` for clone-specific changes.
- **Cloning all nested objects without a model:** this can break intentional sharing or graph relationships.
- **Using `==` to check identity:** use `===` for the same-instance check.
- **Using `===` to compare values:** separate objects with the same state are not identical.
- **Cloning external resources casually:** a copied handle may not represent a valid independent resource.
- **Using PHP 8.5 clone syntax on older runtimes:** `clone($object, $overrides)` is a PHP 8.5 feature.

## Key takeaways

- Assignment aliases an object; `clone` creates a distinct instance.
- The default clone copies properties shallowly.
- `__clone()` runs on the new object after that shallow copy and can customize it.
- Deep copying must be designed for the object's nested state and graph.
- For ordinary user-defined objects, `==` compares class and property values; `===` checks instance identity.
- PHP 8.3 permits readonly property reinitialization inside `__clone()`.
- PHP 8.5 adds clone property overrides through the `clone()` function.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [076 - Magic methods](076-magic-methods.md) | [Q&A](../qnas/077-cloning-and-comparing-objects.md) | [Example](../examples/077-cloning-and-comparing-objects.md) | [078 - Late static binding](078-late-static-binding.md) |
