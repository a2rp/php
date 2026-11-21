# Property hooks - Q&A

These questions review PHP property hooks, backed and virtual properties, validation, defaults, and hook restrictions. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. Which PHP version introduced property hooks?

- A. PHP 8.1
- B. PHP 8.2
- C. PHP 8.3
- D. PHP 8.4

**Answer: D.** Property hooks were introduced in PHP 8.4.

### 2. What does a <code>get</code> hook control?

- A. How a property is read
- B. How a class is constructed
- C. How a method is called
- D. How a class constant is declared

**Answer: A.** A get hook runs when code reads the property.

### 3. What does a <code>set</code> hook control?

- A. How a property is written
- B. How a property is named
- C. How an object is cloned
- D. How a constructor returns

**Answer: A.** A set hook runs when code assigns a value to the property.

### 4. What is a backed property?

- A. A property that stores a value on the object
- B. A property declared only in an interface
- C. A property that must be private
- D. A static property

**Answer: A.** A backed property has storage for its value. Its hooks can access that backing value.

### 5. What is a virtual property?

- A. A property that is always static
- B. A property whose hooks do not use its own backing value
- C. A property that cannot have a type
- D. A property that exists only in PHP 8.0

**Answer: B.** A virtual property has hooks but no backing value of its own. It can calculate a value or redirect writes to other state.

### 6. How does a backed property hook access its own backing value?

- A. <code>$this::property</code>
- B. <code>$this->property</code> inside that property's hook
- C. <code>parent-&gt;property</code>
- D. <code>self::$property</code>

**Answer: B.** Inside the hook, direct access to <code>$this->property</code> refers to its backing value and does not recursively call the same hook.

### 7. Which hook is needed to read a virtual property?

- A. <code>get</code>
- B. <code>set</code>
- C. Both are always required
- D. Neither; virtual properties are always readable

**Answer: A.** A virtual property needs a get hook to be read.

### 8. Which hook is needed to write to a virtual property?

- A. <code>get</code>
- B. <code>set</code>
- C. Both are always required
- D. Neither; virtual properties are always writable

**Answer: B.** A virtual property needs a set hook to accept writes.

### 9. What happens if a backed property defines a getter but no setter?

- A. The property can never be written
- B. Its ordinary write behavior remains available
- C. The property becomes virtual
- D. PHP rejects the property declaration

**Answer: B.** On a backed property, omitting one hook leaves that operation with its normal property behavior.

### 10. What does an expression-bodied set hook do with its expression result?

~~~php
public string $name {
    set => trim($value);
}
~~~

- A. Discards the result
- B. Stores the result in the backing value
- C. Returns the result to the caller
- D. Converts the property to virtual

**Answer: B.** For an expression-bodied set hook, the expression result is assigned to the property's backing value.

### 11. What must a block-bodied set hook do to update a backed property?

- A. Return the property name
- B. Explicitly assign to the backing value
- C. Declare a second property
- D. Call <code>parent::__set()</code>

**Answer: B.** A block body performs arbitrary code; it must explicitly write <code>$this->property</code> if the backing value should change.

### 12. Does a backed property's declared default pass through its set hook?

- A. Yes, every time the object is created
- B. No, the default is assigned directly
- C. Only if the getter is present
- D. Only when the property is public

**Answer: B.** The initial default value is assigned directly. Later assignments use the set hook.

### 13. Can a virtual property declare a default backing value?

- A. Yes, it stores the value in a hidden field
- B. No, it has no backing storage
- C. Only if its type is nullable
- D. Only on PHP 8.6

**Answer: B.** A virtual property has no backing value to hold a default.

### 14. Which property kind can use hooks?

- A. Only static properties
- B. Only object instance properties
- C. Both static and instance properties
- D. Only constants

**Answer: B.** Property hooks apply to instance properties, not static properties.

### 15. Can a readonly property also declare property hooks?

- A. Yes, on PHP 8.4
- B. Yes, but only with a get hook
- C. No, hooks and readonly properties are incompatible
- D. Only when declared private

**Answer: C.** Property hooks are incompatible with readonly properties. Use property visibility, including asymmetric visibility, when access control is needed.

### 16. What does <code>isset($object-&gt;name)</code> do when the property has a getter?

- A. It always reads the backing value directly
- B. It invokes the getter and checks the returned value
- C. It invokes the setter with null
- D. It removes the hook

**Answer: B.** With a get hook, <code>isset()</code> checks the value returned by that hook.

### 17. What happens when code calls <code>unset()</code> on a hooked property?

- A. The property is removed
- B. The backing value is set to null
- C. PHP throws an <code>Error</code>
- D. The getter runs once

**Answer: C.** Properties with hooks cannot be unset.

### 18. Why can modifying an array offset through a hooked property be risky?

- A. PHP arrays cannot be property values
- B. Indirect modification may bypass the set-hook path
- C. The array becomes static
- D. The property's type is removed

**Answer: B.** Array-offset modification uses reference-like behavior, so it may not follow the expected setter validation path.

### 19. What does a property with this hook calculate?

~~~php
public int $area {
    get => $this->width * $this->height;
}
~~~

- A. A stored class constant
- B. A virtual value derived from width and height
- C. A static property
- D. A constructor argument

**Answer: B.** The hook reads other properties and does not access a backing <code>area</code> value, so the property is virtual.

### 20. What should you use if a property operation has many steps or unclear side effects?

- A. A method
- B. A class constant
- C. A virtual property in every case
- D. A reference getter

**Answer: A.** Methods make complex operations and side effects more explicit than property syntax.

## Short-answer questions

### 21. What is the basic purpose of a property hook?

**Answer:** To add targeted behavior to property reads or writes while keeping ordinary property syntax for callers.

**Explanation:** A getter can calculate or transform a read value, and a setter can validate, normalize, or redirect a write.

### 22. What is the difference between a backed and a virtual property?

**Answer:** A backed property stores its own value; a virtual property derives a value or forwards operations without its own backing value.

**Explanation:** A backed hook can access its own value through <code>$this->property</code>. A virtual hook typically uses other properties or methods.

### 23. How do you avoid infinite recursion in a backed hook?

**Answer:** Access the property directly as <code>$this->property</code> from inside its own hook to use its backing value.

**Explanation:** PHP treats that direct self-reference inside the hook specially. A virtual getter must instead read other state, not itself.

### 24. When does a property become backed?

**Answer:** When a hook directly accesses the property itself as <code>$this->propertyName</code>.

**Explanation:** That exact access establishes that the property has a backing value for the hook to read or write.

### 25. What happens if a block-bodied set hook validates input but never assigns to the backing value?

**Answer:** It does not update the backing value.

**Explanation:** Once a set hook is defined, its block controls the write. For a backed property, the hook needs to assign the accepted or transformed value.

### 26. When should a set hook use expression syntax?

**Answer:** When one expression can produce the transformed value to store.

**Explanation:** Use a block body when validation needs multiple statements, errors, or other operations.

### 27. What is the purpose of a virtual set hook?

**Answer:** To translate an assignment into changes to other state or behavior without storing the virtual property's own value.

**Explanation:** For example, setting a rectangle's area could calculate and update its height.

### 28. Why does a virtual property's getter need to avoid reading the same property?

**Answer:** Doing so would call the getter recursively.

**Explanation:** A virtual property has no backing value. Its getter should derive the result from other properties or methods.

### 29. Are property hooks a replacement for every getter and setter method?

**Answer:** No.

**Explanation:** Hooks suit property-shaped reads and writes with simple, predictable behavior. Methods are clearer for operations with substantial work or side effects.

### 30. Why is a setter hook not called for the default value of a backed property?

**Answer:** PHP assigns the declared default directly during object initialization.

**Explanation:** The setter hook handles subsequent writes. Do not rely on it to validate or normalize a declared default.

## Code reading and application

### 31. What does this print?

~~~php
class Name
{
    public string $value {
        set => $this->value = trim($value);
    }
}

$name = new Name();
$name->value = '  Ada  ';
echo $name->value;
~~~

**Answer: <code>Ada</code>.**

**Explanation:** The assignment calls the set hook, which trims the input and writes the result to the backing value. The omitted get hook uses ordinary read behavior.

### 32. What does this print?

~~~php
class Rectangle
{
    public function __construct(
        public int $width,
        public int $height,
    ) {
    }

    public int $area {
        get => $this->width * $this->height;
    }
}

$rectangle = new Rectangle(4, 5);
echo $rectangle->area;
~~~

**Answer: <code>20</code>.**

**Explanation:** The virtual getter multiplies width by height each time the area property is read.

### 33. Is this a virtual or a backed property?

~~~php
class Example
{
    public string $label {
        get => strtoupper($this->label);
    }
}
~~~

**Answer: It is backed.**

**Explanation:** The getter directly accesses <code>$this->label</code>, which refers to the backing value inside the hook. The getter itself will fail if that backing value has not been initialized.

### 34. Why is this getter incorrect for a virtual property?

~~~php
public string $fullName {
    get => $this->fullName;
}
~~~

**Answer:** It recursively invokes itself.

**Explanation:** A virtual getter has no backing value to read. Derive the name from other properties such as <code>firstName</code> and <code>lastName</code>.

### 35. What is missing from this backed setter?

~~~php
public string $name {
    set {
        if (trim($value) === '') {
            throw new InvalidArgumentException();
        }
    }
}
~~~

**Answer:** It never stores the accepted value.

**Explanation:** After validation, add an assignment such as <code>$this->name = trim($value);</code>.

### 36. What does this print?

~~~php
class Role
{
    public string $name = ' reader ' {
        set => trim($value);
    }
}

$role = new Role();
echo $role->name;
~~~

**Answer: <code> reader </code>, including the spaces.**

**Explanation:** The default is assigned directly and does not pass through the set hook. A later assignment would be trimmed.

### 37. What happens when this code writes to <code>area</code>?

~~~php
class Rectangle
{
    public function __construct(public int $width, public int $height)
    {
    }

    public int $area {
        get => $this->width * $this->height;
    }
}

$rectangle = new Rectangle(4, 5);
$rectangle->area = 30;
~~~

**Answer:** PHP throws an error because the virtual property has no set hook.

**Explanation:** A virtual property's read and write operations exist only when their corresponding hooks are defined.

### 38. Complete this virtual property so that assigning an area updates the rectangle's height.

~~~php
class Rectangle
{
    public function __construct(public int $width, public int $height)
    {
    }

    public int $area {
        get => $this->width * $this->height;

        set {
            // Add validation and update height.
        }
    }
}
~~~

**Answer:**

~~~php
public int $area {
    get => $this->width * $this->height;

    set {
        if ($this->width <= 0 || $value < 0 || $value % $this->width !== 0) {
            throw new InvalidArgumentException('Area is not valid.');
        }

        $this->height = intdiv($value, $this->width);
    }
}
~~~

**Explanation:** The setter changes another stored property. It never reads or writes <code>$this->area</code>, so area remains virtual.

### 39. Is this combination valid?

~~~php
public readonly string $name {
    get => $this->name;
}
~~~

**Answer: No.**

**Explanation:** Property hooks and readonly properties are incompatible. Use a normal readonly property without hooks, or use hooks with suitable property visibility.

### 40. What does <code>empty($object-&gt;label)</code> check when the property has a get hook?

**Answer:** It invokes the getter and checks the returned value.

**Explanation:** If the getter reads an uninitialized backing value, <code>empty()</code> can throw an <code>Error</code> just like a normal read.

## Key takeaways

- Property hooks were introduced in PHP 8.4.
- Get hooks intercept reads; set hooks intercept writes.
- Backed properties store values; virtual properties derive or redirect operations.
- A block-bodied setter must explicitly write its backing value when appropriate.
- Defaults bypass the set hook, and virtual properties cannot have defaults.
- Hooked properties cannot be readonly or unset.
- Use methods when property syntax would hide complex behavior.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [074 - Constructor property promotion](../lessons/074-constructor-property-promotion.md) | [Lesson](../lessons/075-property-hooks.md) | [Example](../examples/075-property-hooks.md) | [076 - Magic methods](../lessons/076-magic-methods.md) |
