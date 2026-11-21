# Readonly properties and classes - Q&A

These questions review readonly property initialization, readonly classes, shallow immutability, and PHP version behavior. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. Which PHP version introduced readonly properties?

- A. PHP 7.4
- B. PHP 8.0
- C. PHP 8.1
- D. PHP 8.2

**Answer: C.** PHP 8.1 introduced readonly typed properties.

### 2. Which declaration is valid for a readonly property?

- A. <code>public readonly $name;</code>
- B. <code>public readonly string $name;</code>
- C. <code>readonly static string $name;</code>
- D. <code>public readonly string name;</code>

**Answer: B.** A readonly property requires a type and a variable name. Readonly static properties are not supported.

### 3. When is a readonly property normally initialized?

- A. Only in a global function
- B. In the scope allowed to write it, commonly the declaring class constructor
- C. Only in a child class
- D. Every time the property is read

**Answer: B.** The class that declares a readonly property can initialize it in an allowed scope, commonly during construction.

### 4. What happens when code assigns to an initialized readonly property again?

- A. The new value replaces the old value
- B. PHP ignores the assignment
- C. PHP throws an error
- D. PHP converts it to a mutable property

**Answer: C.** A readonly property can only be initialized once. A later assignment is rejected even if the value is identical.

### 5. Does a nullable readonly property start as <code>null</code> automatically?

- A. Yes, every nullable property does
- B. No, it remains uninitialized until assigned
- C. Only inside a readonly class
- D. Only when the property is public

**Answer: B.** A nullable type allows a value of <code>null</code>; it does not initialize the property. Assign <code>null</code> explicitly if that is the intended value.

### 6. Which type may be used when a readonly property intentionally accepts any value?

- A. No type is allowed
- B. <code>mixed</code>
- C. <code>static</code>
- D. <code>readonly</code>

**Answer: B.** Readonly properties require a type declaration. <code>mixed</code> is the broad type when a value is intentionally unrestricted.

### 7. Which PHP version introduced readonly classes?

- A. PHP 8.1
- B. PHP 8.2
- C. PHP 8.3
- D. PHP 8.4

**Answer: B.** Readonly classes became available in PHP 8.2.

### 8. What does <code>readonly class Record</code> do?

- A. Makes every method static
- B. Makes every declared instance property readonly
- C. Makes every class constant private
- D. Prevents the class from being instantiated

**Answer: B.** The class modifier applies readonly behavior to its declared instance properties.

### 9. Which property declaration is forbidden in a readonly class?

- A. Typed instance property
- B. Public instance property
- C. Static property
- D. Property initialized in the constructor

**Answer: C.** Readonly classes cannot declare static properties.

### 10. Can a readonly class declare an untyped instance property?

- A. Yes, if the property is private
- B. Yes, if it is initialized in the constructor
- C. No, all instance properties need types
- D. Only on PHP 8.6

**Answer: C.** Readonly properties must be typed, so every declared instance property in a readonly class must have a type.

### 11. What happens if code adds an undeclared property to a readonly class instance?

- A. PHP creates a dynamic property
- B. PHP stores it in a static property
- C. The operation is rejected
- D. The class automatically becomes mutable

**Answer: C.** Readonly classes prevent dynamic properties.

### 12. Can a child class extend a readonly class if the child is not readonly?

- A. Yes, always
- B. Yes, only if it has no properties
- C. No, the child must also be readonly
- D. Only in PHP 8.6

**Answer: C.** A non-readonly class cannot extend a readonly class. The child must also be readonly.

### 13. Can a readonly class extend a non-readonly parent class?

- A. Yes
- B. No
- C. Only if the parent has no methods
- D. Only when all parent properties are private

**Answer: B.** A readonly class cannot extend a non-readonly class.

### 14. What is the effect of modifying an object stored in a readonly property?

- A. Always forbidden
- B. The property is replaced
- C. The object's internal state may still change
- D. The object is copied automatically

**Answer: C.** Readonly prevents replacing the property value. It does not deeply freeze the object referenced by that value.

### 15. Which operation on a readonly array property is rejected after initialization?

- A. Reading its length with <code>count()</code>
- B. Reading an element
- C. Appending an element through the property
- D. Passing a copy to a function

**Answer: C.** A readonly array cannot be modified indirectly through an offset or append operation.

### 16. Which PHP version changed the implicit setter visibility of readonly properties to <code>protected(set)</code>?

- A. PHP 8.1
- B. PHP 8.2
- C. PHP 8.3
- D. PHP 8.4

**Answer: D.** Starting in PHP 8.4, readonly properties implicitly use <code>protected(set)</code>; before that they implicitly used private setter visibility.

## Short-answer questions

### 17. What is the main purpose of a readonly property?

**Answer:** To let code read a property while preventing it from being reassigned after initialization.

**Explanation:** This communicates and enforces an invariant, such as “this object’s identifier stays the same after it is created.”

### 18. Why should a readonly property usually be initialized in the constructor?

**Answer:** The constructor provides one clear place where a valid object receives all required values.

**Explanation:** Initializing required readonly state during construction helps avoid objects that exist but fail when their properties are first read.

### 19. What happens if an uninitialized readonly property is read?

**Answer:** PHP throws an <code>Error</code> because the typed property has no value yet.

**Explanation:** Declaring a type does not assign a default. Every valid construction path should initialize properties that will be read.

### 20. Why does <code>?string</code> not initialize a readonly property to <code>null</code>?

**Answer:** The question mark changes the set of allowed values; it does not assign a value.

**Explanation:** A nullable property can contain a string or <code>null</code>, but it is still uninitialized until code assigns one of those values.

### 21. Name three writes that PHP rejects after a readonly property is initialized.

**Answer:** Reassignment, incrementing it, and unsetting it are rejected.

**Explanation:** Compound assignment and indirect writes, such as modifying an array offset, are also rejected because they change the property's value.

### 22. Why can a mutable object still change when it is stored in a readonly property?

**Answer:** The readonly property holds a fixed object reference, but the referenced object may expose its own mutable state.

**Explanation:** Readonly is shallow. Use immutable nested values or avoid exposing mutable objects when deep immutability is required.

### 23. How are readonly arrays different from readonly object properties?

**Answer:** A readonly array cannot have its elements changed through the property, while an object held by a readonly property may still have its own fields changed.

**Explanation:** Arrays are values subject to the property’s readonly restriction. An object is a reference to a separate object whose internal behavior is governed by its own class.

### 24. What is one good alternative to mutating a value object?

**Answer:** Provide a method that returns a new object containing the updated value.

**Explanation:** Copy-style methods such as <code>withName()</code> preserve the original and make the change explicit.

### 25. Can a readonly class contain class constants?

**Answer:** Yes.

**Explanation:** Class constants are not properties, so the readonly property restrictions do not prohibit them.

### 26. Can a readonly class have methods that calculate and return values?

**Answer:** Yes.

**Explanation:** Readonly constrains property writes. Methods can read properties, perform calculations, and return results or new objects.

### 27. What is the <code>#[AllowDynamicProperties]</code> attribute’s effect on a readonly class?

**Answer:** It cannot enable dynamic properties on a readonly class.

**Explanation:** Readonly classes explicitly forbid dynamic properties; applying that attribute is not a workaround.

### 28. What changed for readonly properties during cloning in PHP 8.3?

**Answer:** A class may reinitialize readonly properties while its <code>__clone()</code> method runs.

**Explanation:** This supports custom clone behavior. Ordinary assignments after construction remain forbidden, and code should account for its minimum PHP version.

## Code reading and application

### 29. What does this print?

~~~php
class AccountId
{
    public function __construct(public readonly int $value)
    {
    }
}

$id = new AccountId(17);
echo $id->value;
~~~

**Answer: <code>17</code>.**

**Explanation:** The constructor initializes the promoted readonly property, and reading it afterward is allowed.

### 30. Is the following assignment valid?

~~~php
class AccountId
{
    public function __construct(public readonly int $value)
    {
    }
}

$id = new AccountId(17);
$id->value = 17;
~~~

**Answer: No.**

**Explanation:** The constructor already initialized <code>value</code>. Assigning the same number again is still a second write and PHP rejects it.

### 31. Is this property initialized when a new object is constructed?

~~~php
class OptionalLabel
{
    public function __construct(public readonly ?string $label)
    {
    }
}

$label = new OptionalLabel(null);
~~~

**Answer: Yes, it is initialized to <code>null</code>.**

**Explanation:** The constructor explicitly passes and assigns <code>null</code>. A nullable property declared without a constructor assignment would remain uninitialized.

### 32. What does this print?

~~~php
class Box
{
    public function __construct(public readonly object $item)
    {
    }
}

$item = (object) ['count' => 1];
$box = new Box($item);
$box->item->count++;
echo $box->item->count;
~~~

**Answer: <code>2</code>.**

**Explanation:** The property still points to the same object, but that object's public field can change. Replacing <code>$box->item</code> would be rejected.

### 33. Is this valid after construction?

~~~php
class Results
{
    public function __construct(public readonly array $items)
    {
    }
}

$results = new Results(['a']);
$results->items[] = 'b';
~~~

**Answer: No.**

**Explanation:** Appending changes the array held by the readonly property. Readonly arrays cannot be indirectly modified through that property.

### 34. What does this print?

~~~php
readonly class Coordinate
{
    public function __construct(public int $x, public int $y)
    {
    }
}

$point = new Coordinate(3, 8);
echo $point->x + $point->y;
~~~

**Answer: <code>11</code>.**

**Explanation:** The constructor initializes both typed instance properties. The class modifier does not prevent reading them or calculating with them.

### 35. Is this legal in PHP 8.2?

~~~php
readonly class Cache
{
    public static int $hits = 0;
}
~~~

**Answer: No.**

**Explanation:** Readonly classes cannot declare static properties. The readonly modifier applies to instance properties.

### 36. Is this legal in PHP 8.2?

~~~php
readonly class Item
{
    public $name;
}
~~~

**Answer: No.**

**Explanation:** A readonly property must have a type. The property could be declared as <code>public string $name;</code> and assigned in the constructor.

### 37. Why does this fail when <code>getName()</code> is called?

~~~php
class ProductName
{
    public readonly string $name;

    public function getName(): string
    {
        return $this->name;
    }
}
~~~

**Answer:** The property is never initialized.

**Explanation:** Reading a typed readonly property before its first assignment throws an <code>Error</code>. Add a constructor that assigns <code>$this->name</code>.

### 38. Make this class return an updated object without changing the original.

~~~php
readonly class Point
{
    public function __construct(public int $x, public int $y)
    {
    }
}
~~~

**Answer:**

~~~php
readonly class Point
{
    public function __construct(public int $x, public int $y)
    {
    }

    public function withX(int $x): self
    {
        return new self($x, $this->y);
    }
}
~~~

**Explanation:** The method constructs a separate object. The original point remains unchanged.

### 39. When may a separately declared readonly property have a default value?

**Answer:** As of PHP 8.6.

**Explanation:** In earlier PHP versions, initialize it in the constructor. In PHP 8.6, the declared default initializes the property, so it cannot be assigned again in the constructor.

### 40. A PHP 8.3 child class initializes an inherited readonly property. Why might the same code behave differently on PHP 8.4?

**Answer:** PHP 8.4 changed the implicit setter visibility from private-set to protected-set.

**Explanation:** The change affects which class scopes may initialize the property. Keep initialization in the declaring class or explicitly design and document the supported version behavior.

## Key takeaways

- Readonly properties require types, are initialized once, and cannot be reassigned.
- Nullable does not mean initialized; assign <code>null</code> explicitly when needed.
- Readonly arrays cannot be modified through their property; referenced objects can still mutate internally.
- Readonly classes were added in PHP 8.2 and disallow static and dynamic properties.
- PHP 8.3 allows readonly property reinitialization during cloning; PHP 8.4 changes implicit setter visibility; PHP 8.6 allows readonly property defaults.
- Return a new value object when callers need a changed value.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [072 - Class constants and constant expressions](../lessons/072-class-constants-and-constant-expressions.md) | [Lesson](../lessons/073-readonly-properties-and-classes.md) | [Example](../examples/073-readonly-properties-and-classes.md) | [074 - Constructor property promotion](../lessons/074-constructor-property-promotion.md) |
