# 063 - Properties, Methods, and Object Creation: Q&A

Review typed properties, object state, methods, visibility, initialization, and dynamic properties. Try each question before reading its answer. See the [chapter lesson](../lessons/063-properties-methods-and-object-creation.md).

## Multiple-choice questions

### 1. What is a property in a PHP class?

A. A variable declared inside the class that can hold object state
B. A command-line option
C. A function call written outside the class
D. A Git branch

**Answer:** A. A variable declared inside the class that can hold object state.

**Explanation:** Properties represent data associated with an object or, when static, with the class.

### 2. What is a method?

A. A function declared in a class
B. A configuration file
C. A property value
D. An object instance

**Answer:** A. A function declared in a class.

**Explanation:** Methods define operations the object or class can perform.

### 3. Which operator accesses an instance property or calls an instance method?

A. `->`
B. `::`
C. `=>`
D. `??`

**Answer:** A. `->`.

**Explanation:** For example, `$book->title` accesses a property and `$book->summary()` calls a method.

### 4. What does `$this` refer to inside an instance method?

A. The object receiving the method call
B. The class filename
C. Any object in the program
D. A new instance

**Answer:** A. The object receiving the method call.

**Explanation:** Use `$this->property` to access the current object's instance property.

### 5. What happens to a typed property declared without a default value?

A. It contains an empty string automatically
B. It is uninitialized until assigned
C. It is automatically public static
D. It always contains null

**Answer:** B. It is uninitialized until assigned.

**Explanation:** Reading it before initialization throws an `Error`.

### 6. Which declaration allows either a string or null?

A. `public string? $name;`
B. `public ?string $name = null;`
C. `public null|string? $name;`
D. `public string $name = false;`

**Answer:** B. `public ?string $name = null;`.

**Explanation:** The nullable type permits a string or `null`, and the default initializes it to null.

### 7. Which return type indicates that a method does not return a value?

A. `none`
B. `void`
C. `null`
D. `empty`

**Answer:** B. `void`.

**Explanation:** A void method performs an operation without returning a value.

### 8. Which visibility allows access from any code that can access the object?

A. `private`
B. `protected`
C. `public`
D. `internal`

**Answer:** C. `public`.

**Explanation:** Public members form part of the object's accessible API.

### 9. Which visibility restricts a member to the class that declares it?

A. `public`
B. `private`
C. `protected`
D. `global`

**Answer:** B. `private`.

**Explanation:** Private members are implementation details of the declaring class.

### 10. Which visibility is available within the class and its child classes?

A. `protected`
B. `private`
C. `public`
D. `local`

**Answer:** A. `protected`.

**Explanation:** Protected members are available to the class and its descendants.

### 11. Which call creates a new object without passing constructor arguments?

A. `$book = Book();`
B. `$book = new Book();`
C. `$book = create Book;`
D. `$book = Book->new();`

**Answer:** B. `$book = new Book();`.

**Explanation:** The `new` keyword creates a class instance. This works when the class has no required constructor arguments.

### 12. When did PHP introduce typed properties?

A. PHP 5.0
B. PHP 7.4
C. PHP 8.2
D. PHP 8.4

**Answer:** B. PHP 7.4.

**Explanation:** Typed properties allow a class to declare the type of a property's value.

### 13. What does this method declaration mean: `public function setCount(int $count): void`?

A. It accepts an integer argument and is not intended to return a value
B. It returns an integer and takes no arguments
C. It is a private property
D. It creates a static method

**Answer:** A. It accepts an integer argument and is not intended to return a value.

**Explanation:** The parameter type is `int`; the return type is `void`.

### 14. What happens when code creates an undeclared dynamic property on an ordinary class in PHP 8.2 and later?

A. It is deprecated unless the class explicitly supports dynamic properties
B. It always creates a static property
C. It is silently converted into a method
D. It is always a parse error

**Answer:** A. It is deprecated unless the class explicitly supports dynamic properties.

**Explanation:** Declaring expected properties is clearer and avoids relying on deprecated behavior.

### 15. Which design helps prevent callers from bypassing a validation rule for a quantity?

A. Make the property private and expose a validating public method
B. Make every property public
C. Use a comment instead of a method
D. Remove the property's type

**Answer:** A. Make the property private and expose a validating public method.

**Explanation:** Encapsulation lets the class control how its state changes.

## Short-answer questions

### 16. What is the difference between an instance property and a static property?

**Answer:** An instance property belongs to an individual object; a static property belongs to the class itself.

**Explanation:** This chapter focuses on instance state; static members use class-level access rules.

### 17. Why should a typed property be initialized before reading it?

**Answer:** PHP cannot return a value that has not been assigned to that typed property.

**Explanation:** Reading an uninitialized typed property throws an `Error`.

### 18. How can an optional value be represented by a nullable property?

**Answer:** Declare a nullable type, such as `public ?string $subtitle = null;`.

**Explanation:** The type allows either a string or `null`, and the default initializes the property.

### 19. What is the purpose of a method parameter type?

**Answer:** It communicates and constrains the kind of value the method expects.

**Explanation:** Types make method contracts clearer and help catch invalid calls.

### 20. What is the purpose of a return type?

**Answer:** It states what kind of value a method returns, or that it returns no value when declared `void`.

**Explanation:** Return types document the method contract and let PHP enforce it.

### 21. Why can a public property make validation difficult?

**Answer:** Any caller can assign a value that bypasses rules implemented elsewhere.

**Explanation:** A private property with validating public methods keeps state changes under class control.

### 22. What does `$this->pageCount` mean inside an instance method?

**Answer:** It refers to the `pageCount` property on the current object.

**Explanation:** `$this` is the instance receiving the method call.

### 23. Why are parentheses used after a method name when calling it?

**Answer:** Parentheses perform the method call and can contain its arguments.

**Explanation:** `$book->summary()` calls the method; `$book->summary` is property-style access.

### 24. What does `public` visibility mean for a method?

**Answer:** Code outside the class can call the method through an object, subject to having access to that object.

**Explanation:** Public methods are part of the class's accessible API.

### 25. What is one appropriate use of `protected` visibility?

**Answer:** A member that should be accessible to the class and subclasses but not to unrelated outside code.

**Explanation:** Protected is useful when inheritance needs access to an implementation member.

### 26. Why is `private` useful for implementation details?

**Answer:** It prevents outside code from depending directly on internal state or helper operations.

**Explanation:** The class can change its implementation while preserving its public API.

### 27. Why should expected properties be declared explicitly?

**Answer:** It documents the object's shape and helps catch misspelled member names.

**Explanation:** It also avoids accidental dynamic-property creation.

### 28. What is a dynamic property?

**Answer:** A property assigned to an object even though it was not declared by the class.

**Explanation:** For ordinary classes, creating dynamic properties is deprecated as of PHP 8.2 unless the class explicitly supports them.

### 29. What should a method do if a requested state change violates a rule?

**Answer:** It should reject the change, for example by throwing an appropriate exception.

**Explanation:** A method can preserve the object's invariant instead of storing invalid state.

### 30. Why are constructors introduced in the next chapter relevant to object creation?

**Answer:** A constructor can initialize an object from values supplied when it is created.

**Explanation:** Without required constructor arguments, a class can still be instantiated with `new ClassName()` and its defaults.

## Scenarios and code review

### 31. A class declares `public int $count;` but the program reads it immediately after `new`. What can happen?

**Answer:** PHP throws an `Error` because the typed property has not been initialized.

**Explanation:** Assign it before reading or give it a suitable default.

### 32. A caller assigns `'many'` to a property declared as `int`. What is the design issue?

**Answer:** The value does not match the property's declared type and causes a type error.

**Explanation:** The caller should provide an integer; a class method can validate additional business constraints.

### 33. A developer declares a nullable property but forgets its default and reads it immediately. Is nullable enough to initialize it?

**Answer:** No. Nullable means the type allows null; the property is still uninitialized until it is assigned or given a default.

**Explanation:** Use `public ?string $name = null;` when null is the intended initial value.

### 34. A class has a private `$balance` and a public `withdraw()` method. Why is this useful?

**Answer:** The method can check the requested amount and available balance before changing private state.

**Explanation:** The class can enforce its rules consistently.

### 35. A child class needs to call a helper method from its parent, but unrelated code should not. Which visibility might fit?

**Answer:** `protected`.

**Explanation:** Protected members are accessible to the class and its child classes.

### 36. A developer writes `$object->save` when they meant to call the `save()` method. What is missing?

**Answer:** The method-call parentheses: `$object->save()`.

**Explanation:** Without parentheses, the expression is property-style access rather than a method call.

### 37. The code writes `$book->publiserName = 'Acme';` but the class declares `publisherName`. What might happen?

**Answer:** The typo may create an undeclared dynamic property on PHP versions that permit it, producing a deprecation on PHP 8.2+ for ordinary classes.

**Explanation:** Declare properties and use a clear public API; static analysis can also help catch misspellings.

### 38. A method returns a string but declares `: int`. What is wrong?

**Answer:** The returned value does not satisfy the declared return type.

**Explanation:** Make the implementation and return type agree, or change the method contract if a different value is intended.

### 39. An object property should never be changed directly by callers, but callers need to read it. What design could be used?

**Answer:** Keep the property private or protected and provide a public getter method.

**Explanation:** The method offers controlled read access without allowing arbitrary assignment.

### 40. A class stores a page count and lets callers set any integer, including negative values. What could improve the design?

**Answer:** Make the property private and validate the value in a public setter or another controlled operation.

**Explanation:** An integer type does not enforce the business rule that a page count must be non-negative.

## Coding questions

### 41. Declare a typed `Book` property with a default title and create an instance.

**Answer:**

~~~php
class Book
{
    public string $title = 'Untitled';
}

$book = new Book();
~~~

**Explanation:** The default initializes the typed property for each new instance.

### 42. Write a method that accepts an integer page count and returns no value.

**Answer:**

~~~php
public function setPageCount(int $pages): void
{
    $this->pageCount = $pages;
}
~~~

**Explanation:** The argument type is `int`, and `void` indicates there is no return value.

### 43. Declare an optional nullable string property initialized to null.

**Answer:**

~~~php
public ?string $subtitle = null;
~~~

**Explanation:** The property accepts a string or null and is initialized when the object is created.

### 44. Make a quantity property private and reject a negative value in its public setter.

**Answer:**

~~~php
class InventoryItem
{
    private int $quantity = 0;

    public function setQuantity(int $quantity): void
    {
        if ($quantity < 0) {
            throw new InvalidArgumentException(
                'Quantity cannot be negative.'
            );
        }

        $this->quantity = $quantity;
    }

    public function quantity(): int
    {
        return $this->quantity;
    }
}
~~~

**Explanation:** The property cannot be assigned directly from outside, and the method enforces the non-negative rule.

### 45. Create two objects from a class and update only the first object's public title.

**Answer:**

~~~php
class Book
{
    public string $title = 'Untitled';
}

$first = new Book();
$second = new Book();

$first->title = 'PHP Basics';

echo $first->title;  // PHP Basics
echo $second->title; // Untitled
~~~

**Explanation:** Each `new Book()` creates a distinct instance with its own property state.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [062 - Classes, objects, and object-oriented PHP](../lessons/062-classes-objects-and-object-oriented-php.md) | [Read lesson](../lessons/063-properties-methods-and-object-creation.md) | [Open example](../examples/063-properties-methods-and-object-creation.md) | [064 - Constructors and destructors](../lessons/064-constructors-and-destructors.md) |
