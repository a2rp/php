# 077 - Cloning and comparing objects: Q&A

Use these questions to review object assignment, shallow and deep copying, `__clone()`, and the difference between comparing object state and identity. Each answer appears immediately after its question.

## Multiple-choice questions

### 1. What happens when an object variable is assigned to another variable?

- A. PHP creates a deep copy.
- B. Both variables refer to the same object instance.
- C. PHP calls the constructor again.
- D. PHP converts the object to an array.

**Answer:** B. Both variables refer to the same object instance.

**Explanation:** Assignment copies the object handle. Mutating the instance through either variable is visible through the other.

### 2. What does `clone $object` produce?

- A. A new object instance initialized from the object's properties.
- B. A second variable for the same instance.
- C. A serialized string.
- D. A new object with only default property values.

**Answer:** A. A new object instance initialized from the object's properties.

**Explanation:** The cloned object has a different identity. By default, its properties are copied shallowly.

### 3. If a property refers to another mutable object, what does a default clone do with it?

- A. Recursively clone it.
- B. Set the property to `null`.
- C. Keep the original nested object reference.
- D. Rerun the nested object's constructor.

**Answer:** C. Keep the original nested object reference.

**Explanation:** The outer object is new, but a nested object property remains shared until clone logic copies it too.

### 4. When is `__clone()` called?

- A. Before properties are copied.
- B. On the original object before cloning.
- C. On the new object after PHP's shallow property copy.
- D. Every time any property is assigned.

**Answer:** C. On the new object after PHP's shallow property copy.

**Explanation:** The hook customizes the already-created clone, for example by replacing a nested collaborator with its own clone.

### 5. For ordinary user-defined objects, what does `==` compare?

- A. Whether the variables refer to the same instance only.
- B. Whether they are instances of the same class and their properties have equal values.
- C. Only public property names.
- D. Whether the objects were created at the same time.

**Answer:** B. Whether they are instances of the same class and their properties have equal values.

**Explanation:** PHP compares the class and property values according to its object comparison rules. Some internal classes or extensions can define their own behavior.

### 6. What does `===` check for two objects?

- A. Equal property values.
- B. The same class, regardless of instance.
- C. The exact same object instance.
- D. Whether either object is a clone.

**Answer:** C. The exact same object instance.

**Explanation:** Two objects with matching properties are not identical unless both variables refer to the same instance.

### 7. Does `clone` call the object's constructor again?

- A. Yes, always.
- B. Only when `__clone()` is missing.
- C. No.
- D. Only for readonly classes.

**Answer:** C. No.

**Explanation:** PHP copies the current properties, then calls `__clone()` on the new instance if the method exists. It does not rerun `__construct()`.

### 8. An array property contains object instances. What happens to those objects under a default shallow clone?

- A. The array entries are all recursively cloned.
- B. The array is copied, but its object entries still refer to the same objects.
- C. The array is shared as one object.
- D. The array becomes empty.

**Answer:** B. The array is copied, but its object entries still refer to the same objects.

**Explanation:** The array value is copied, while object handles stored inside it still identify the original nested instances.

### 9. Since which PHP version can `__clone()` reinitialize a readonly property on the clone?

- A. PHP 7.4
- B. PHP 8.1
- C. PHP 8.3
- D. PHP 8.5

**Answer:** C. PHP 8.3.

**Explanation:** The special reinitialization is allowed inside `__clone()` on the new object. The property remains readonly in other contexts.

### 10. Which PHP version introduced `clone($object, $propertyOverrides)`?

- A. PHP 8.0
- B. PHP 8.2
- C. PHP 8.3
- D. PHP 8.5

**Answer:** D. PHP 8.5.

**Explanation:** PHP 8.5 added the `clone()` function form with a second argument for selected property overrides.

## Short-answer questions

### 11. What is the difference between assigning an object and cloning it?

**Answer:** Assignment creates another variable for the same instance; cloning creates a distinct object instance.

**Explanation:** Mutations through an alias affect the shared object. A clone has its own top-level properties, although nested objects may still be shared.

### 12. What is the basic syntax for cloning an object?

**Answer:** `$copy = clone $original;` (without the backslashes when writing PHP code).

**Explanation:** The `clone` operator takes an object expression and returns the new instance.

### 13. What is the normal order of work when PHP clones an object with a `__clone()` method?

**Answer:** PHP creates the new instance, shallow-copies its properties, and then calls `__clone()` on that new instance.

**Explanation:** `__clone()` can detach nested mutable state or reset fields after the initial copy.

### 14. How can a class make one nested object independent in a clone?

**Answer:** Clone that property from inside `__clone()`, for example: `$this->settings = clone $this->settings;`.

**Explanation:** This replaces the shared nested reference on the new object with a second instance.

### 15. Why can a shallow clone still share mutable state?

**Answer:** Properties containing object handles still point to the same nested object.

**Explanation:** Changing that nested instance through the clone is therefore also visible through the source object.

### 16. What does cloning do to the constructor?

**Answer:** It does not call the constructor again.

**Explanation:** If a copy needs a fresh cache, identifier, or collaborator, implement that clone-specific behavior explicitly.

### 17. Give two useful tasks for `__clone()`.

**Answer:** Detach mutable nested objects and reset or regenerate state that belongs uniquely to the copy.

**Explanation:** The method is also useful for refreshing a cache or copy-specific identifier. It should preserve clear class invariants.

### 18. Explain the difference between `==` and `===` for objects.

**Answer:** `==` checks equivalent state under PHP's object comparison rules; `===` checks whether both variables refer to the same instance.

**Explanation:** A clone can compare equal with `==` while always being a different instance from the source.

### 19. Does `==` consider two ordinary objects of different user-defined classes equal just because their visible values match?

**Answer:** No.

**Explanation:** Ordinary object equality requires the same class as well as equal property values.

### 20. When might a domain-specific equality method be clearer than `==`?

**Answer:** When the domain defines equality differently from comparing every property, such as two users being equal when their stable IDs match.

**Explanation:** A named method like `hasSameIdAs()` makes that business rule explicit and avoids accidental dependence on unrelated properties.

## Scenario questions

### 21. `$copy = clone $article;` and both articles have a `TagList` property. How do you make the cloned article's list independent?

**Answer:** Add `__clone()` to the article and assign `$this->tags = clone $this->tags;`.

**Explanation:** The default clone shares the nested `TagList` instance. Cloning that property in the hook gives the copy its own list object.

### 22. A clone's scalar title changes, but changing a nested object's value also changes the source. Why?

**Answer:** The title is a property of the separate outer instances, while both outer objects still refer to the same nested object.

**Explanation:** This is the normal result of a shallow copy. Clone mutable nested state if the copy must be independent.

### 23. A report object has a cached total that should not carry into a clone. Where should it be reset?

**Answer:** In the report's `__clone()` method.

**Explanation:** The cache was copied with the other properties. The hook can invalidate it or set it to its initial cache state on the new object.

### 24. Should a class blindly clone a database connection property?

**Answer:** No.

**Explanation:** Connections and other external resources have lifecycle and ownership rules. A cloned wrapper may still point to the same connection or may become invalid; define an explicit resource strategy instead.

### 25. A property contains an array of mutable objects. Is cloning the parent enough to isolate every item?

**Answer:** No.

**Explanation:** The array value is copied, but object entries still refer to their original instances. Clone the entries in `__clone()` if they must be independent.

### 26. What should you consider before deep-cloning a cyclic object graph?

**Answer:** Decide which nodes should remain shared, how cycles should be preserved, and whether the graph needs a memoized copy process.

**Explanation:** Naively cloning every reference can duplicate intentionally shared nodes or recurse indefinitely. Deep copy needs a graph-aware design when cycles or shared references exist.

### 27. A readonly identifier needs a fresh value on a clone. What PHP feature can support that?

**Answer:** In PHP 8.3 or later, assign the new identifier inside `__clone()`.

**Explanation:** PHP allows readonly property reinitialization on the cloned object from its clone hook. Older runtimes need a factory or explicit copy method.

### 28. In PHP 8.5, if `clone($object, ['name' => 'Copy'])` is used and the object defines `__clone()`, when is `name` overridden?

**Answer:** After `__clone()` has run.

**Explanation:** The override list is applied after the hook. Only selected properties change; other properties retain their copied or hook-adjusted values.

### 29. Two ordinary objects have matching properties but belong to different classes. What do `==` and `===` return?

**Answer:** Both return `false`.

**Explanation:** `==` requires the same class for ordinary user-defined objects. `===` also returns false because they are not the same instance.

### 30. Two separately created `Point(3, 5)` instances are compared. What are `$a == $b` and `$a === $b`?

**Answer:** `$a == $b` is `true`; `$a === $b` is `false`.

**Explanation:** Their class and property values match, but they are distinct instances.

## Code-reading questions

For questions 31–35, assume this class and code:

~~~php
final class Counter
{
    public function __construct(public int $value) {}
}

$first = new Counter(4);
$alias = $first;
$copy = clone $first;
~~~

### 31. What does `$first === $alias` return?

**Answer:** `true`.

**Explanation:** Assignment makes `$alias` refer to the same `Counter` instance as `$first`.

### 32. What does `$first === $copy` return immediately after cloning?

**Answer:** `false`.

**Explanation:** Cloning creates a distinct object instance.

### 33. What does `$first == $copy` return immediately after cloning?

**Answer:** `true`.

**Explanation:** They are instances of the same class and their `value` properties are equal.

### 34. What happens after `$alias->value = 9`? What is `$copy->value`?

**Answer:** `$first->value` is `9`, and `$copy->value` remains `4`.

**Explanation:** `$alias` and `$first` are aliases for one object. `$copy` is a different object with its own scalar property.

### 35. Does `new Counter(4)` run again when `$copy` is cloned?

**Answer:** No.

**Explanation:** `clone` copies the current object state and does not invoke `__construct()`.

## Deeper understanding

### 36. Is a deep clone always better than a shallow clone?

**Answer:** No. The copy policy depends on the meaning of each property.

**Explanation:** Immutable values can safely be shared, while some mutable collaborators should be detached. Resources may require recreation rather than cloning. Unnecessary deep copies can waste work and break intended sharing.

### 37. Why might an explicit `copyWith()` method be preferable to `clone`?

**Answer:** It can expose required changes and validation in the method's arguments.

**Explanation:** A named method is often clearer when copying applies business rules, needs parameters, or must preserve invariants. PHP 8.5 also offers property overrides, but a domain method can still provide a safer public API.

### 38. Can `==` prove that two objects represent the same real-world entity?

**Answer:** No.

**Explanation:** It checks the objects' class and property values according to PHP's comparison rules, not the application's identity policy. Compare stable domain identifiers when that is the intended rule.

### 39. What does `!==` mean when comparing two object variables?

**Answer:** They do not refer to the same object instance.

**Explanation:** It is the inverse of `===`. The two objects may still have equal property values under `==`.

### 40. What decisions should a class make before it supports cloning?

**Answer:** It should define which state is copied, shared, reset, or recreated and whether callers are allowed to clone it.

**Explanation:** This includes nested mutable objects, readonly values, identifiers, caches, resource ownership, and graph relationships. A factory or explicit copy method may express the policy more safely.

## Key takeaways

- Assignment aliases an object; `clone` makes a distinct instance.
- PHP's default cloning is shallow, and it does not rerun the constructor.
- `__clone()` customizes the new object after its properties are copied.
- Ordinary `==` compares class and property values; `===` checks identity.
- PHP 8.3 allows readonly reinitialization in `__clone()`; PHP 8.5 adds property overrides to `clone()`.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [076 - Magic methods](../lessons/076-magic-methods.md) | [Lesson](../lessons/077-cloning-and-comparing-objects.md) | [Example](../examples/077-cloning-and-comparing-objects.md) | [078 - Late static binding](../lessons/078-late-static-binding.md) |
