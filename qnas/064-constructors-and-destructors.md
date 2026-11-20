# 064 - Constructors and Destructors: Q&A

Review object initialization, constructor arguments, property promotion, inheritance, and destructor timing. Try each question before reading its answer. See the [chapter lesson](../lessons/064-constructors-and-destructors.md).

## Multiple-choice questions

### 1. Which method is PHP's modern constructor?

A. A method named after the class
B. `__construct()`
C. `__start()`
D. `__new()`

**Answer:** B. `__construct()`.

**Explanation:** Use the magic method name with two leading underscores.

### 2. When is `__construct()` called?

A. When the class file is deleted
B. When an object is instantiated with `new`
C. Whenever any method is called
D. At the end of every PHP file

**Answer:** B. When an object is instantiated with `new`.

**Explanation:** The constructor initializes each newly created instance.

### 3. How are required constructor arguments supplied?

A. After the class name in the `new` expression
B. In curly braces after the object is returned
C. As a comment above the class
D. Through `error_reporting()`

**Answer:** A. After the class name in the `new` expression.

**Explanation:** For example, `new Product('Notebook', 1299)`.

### 4. What is the purpose of a default parameter in a constructor?

A. Make that argument optional when creating the object
B. Make the constructor run twice
C. Convert the argument to a static property
D. Call the parent's constructor automatically

**Answer:** A. Make that argument optional when creating the object.

**Explanation:** The default is used when the caller omits that argument.

### 5. What is the recommended constructor name in modern PHP?

A. The class name
B. `__construct()`
C. `constructor()`
D. `initialize()` is always called automatically

**Answer:** B. `__construct()`.

**Explanation:** A method named after the class is not a constructor in PHP 8 and later.

### 6. Can one PHP class declare multiple constructors with different signatures?

A. Yes, every overload runs
B. No; a class has one constructor method
C. Yes, if they have different return types
D. Only when the class has no properties

**Answer:** B. No; a class has one constructor method.

**Explanation:** Use optional arguments or named static factory methods for distinct creation paths.

### 7. In which PHP version was constructor property promotion introduced?

A. PHP 5.0
B. PHP 7.4
C. PHP 8.0
D. PHP 8.4

**Answer:** C. PHP 8.0.

**Explanation:** Promotion lets a constructor parameter declare and initialize a property in one step.

### 8. What makes a constructor parameter a promoted property?

A. A visibility modifier or another applicable property modifier is declared with its type
B. Its name is written in uppercase
C. It is passed by reference
D. It has no type

**Answer:** A. A visibility modifier or another applicable property modifier is declared with its type.

**Explanation:** For example, `public function __construct(private string $name)`.

### 9. When are promoted property values assigned relative to the constructor body?

A. After the destructor runs
B. Before the constructor body executes
C. Only when a getter is called
D. At script shutdown

**Answer:** B. Before the constructor body executes.

**Explanation:** The constructor body can validate or further process the promoted property.

### 10. What happens if a constructor throws an exception?

A. The caller receives a successfully completed object anyway
B. The `new` expression does not produce a usable completed object
C. The destructor always saves the failed object
D. The class declaration is removed

**Answer:** B. The `new` expression does not produce a usable completed object.

**Explanation:** The caller can catch the exception and handle the failed construction.

### 11. If a child class defines its own constructor, does PHP automatically call the parent constructor?

A. Yes, always
B. No; call `parent::__construct()` when parent initialization is needed
C. Only if the parent has private properties
D. Only during shutdown

**Answer:** B. No; call `parent::__construct()` when parent initialization is needed.

**Explanation:** The child constructor is responsible for invoking parent setup.

### 12. What if a child class does not define a constructor?

A. It may inherit an accessible parent constructor
B. It cannot be instantiated
C. It receives a destructor as its constructor
D. PHP creates one from the class name

**Answer:** A. It may inherit an accessible parent constructor.

**Explanation:** The inherited constructor is used when the child does not replace it.

### 13. What does a static factory method provide?

A. A named alternative way to create an object
B. Automatic destructor ordering
C. A second `__construct()` declaration
D. A global variable for every instance

**Answer:** A. A named alternative way to create an object.

**Explanation:** Methods such as `fromJson()` can parse or validate input and then construct an instance.

### 14. When can PHP call an object's destructor?

A. When no references remain or during script shutdown
B. Every time a property is read
C. Before every constructor
D. Only when `exit()` is called

**Answer:** A. When no references remain or during script shutdown.

**Explanation:** Exact shutdown timing and order should not be relied on for essential work.

### 15. Where should predictable resource cleanup usually happen?

A. Only in `__destruct()`
B. In an explicit cleanup path, such as a `finally` block
C. In a random public method
D. In a class comment

**Answer:** B. In an explicit cleanup path, such as a `finally` block.

**Explanation:** A `finally` block makes the cleanup point predictable even when an exception occurs.

## Short-answer questions

### 16. What is the main responsibility of a constructor?

**Answer:** Establish the initial state the object needs to be valid and usable.

**Explanation:** It can accept arguments, validate them, and assign properties.

### 17. Why validate constructor arguments?

**Answer:** To prevent an object from being created in an invalid state.

**Explanation:** Rejecting invalid input during construction protects the class's invariants from the start.

### 18. Can a constructor accept typed and optional parameters?

**Answer:** Yes. It can use parameter types and default values like other methods.

**Explanation:** Required arguments must be supplied; optional arguments use their defaults when omitted.

### 19. What feature was added to constructor arguments in PHP 8.0?

**Answer:** Named arguments for calls and constructor property promotion.

**Explanation:** Named arguments address parameters by name; promotion declares and initializes properties in the constructor signature.

### 20. Do all constructor parameters need to be promoted?

**Answer:** No. Promoted and ordinary parameters can be mixed.

**Explanation:** Use promotion for simple assignment cases and explicit properties when extra logic is clearer.

### 21. Why can public parameter names matter to callers?

**Answer:** Callers can use PHP 8 named arguments and refer to parameters by name.

**Explanation:** Renaming a parameter can therefore affect source compatibility.

### 22. How can a class support two named input formats if PHP has one constructor?

**Answer:** Use separate named static factory methods that convert each format and then call the constructor.

**Explanation:** Method names such as `fromJson()` and `fromArray()` communicate the intended path.

### 23. What does `parent::__construct()` do?

**Answer:** It explicitly runs the parent's constructor from a child constructor.

**Explanation:** PHP does not automatically invoke the parent constructor when the child defines its own.

### 24. What happens when a child does not define a constructor?

**Answer:** It can use an accessible inherited parent constructor.

**Explanation:** The child object still needs to satisfy the arguments required by that constructor.

### 25. What is a destructor?

**Answer:** The `__destruct()` method PHP may call when an object is being destroyed.

**Explanation:** It is generally used for optional or fallback cleanup.

### 26. Why is destructor timing not predictable enough for essential work?

**Answer:** The object may remain referenced, and shutdown order among objects is not guaranteed.

**Explanation:** Use an explicit method or `finally` when timing matters.

### 27. What does a `finally` block provide for cleanup?

**Answer:** It runs as execution leaves the associated `try` block, including when an exception is thrown.

**Explanation:** This gives a clear place for closing a file or releasing another resource.

### 28. Why should a destructor not throw an exception during script shutdown?

**Answer:** Throwing during shutdown can cause a fatal error and obscure the original failure.

**Explanation:** Report cleanup failures through an explicit method while normal exception handling is available.

### 29. If a child class defines its own destructor, how can required parent cleanup run?

**Answer:** The child destructor can call `parent::__destruct()` explicitly.

**Explanation:** The parent destructor is not automatically invoked when a child provides its own destructor.

### 30. Why should new PHP code avoid a method named exactly like its class as a constructor?

**Answer:** Use `__construct()`; the old-style behavior was deprecated before PHP 8 and is not treated as a constructor in PHP 8 and later.

**Explanation:** The magic method name is clear and portable across modern versions.

## Scenarios and code review

### 31. A child constructor assigns its own fields but never calls `parent::__construct()`. The parent expects an ID to be initialized. What is the bug?

**Answer:** The parent initialization is skipped, so the object may violate the parent's invariant.

**Explanation:** Call `parent::__construct($id)` from the child constructor when that setup is required.

### 32. A constructor accepts a negative price and stores it without validation. What could improve it?

**Answer:** Validate the price in the constructor and throw an appropriate exception if it is negative.

**Explanation:** The object should not become available to callers in an invalid state.

### 33. A developer wants `Product($id)` and `Product($json)` as two overloaded constructors. What should they use instead?

**Answer:** Use one constructor with a clear contract and named factory methods such as `fromId()` and `fromJson()`.

**Explanation:** PHP does not overload constructors by signature.

### 34. A constructor throws while validating input. Should the caller expect to use the partially initialized object?

**Answer:** No. The `new` expression fails and does not return a usable completed object.

**Explanation:** Catch the construction exception and handle the invalid input.

### 35. A file must be closed whether processing succeeds or throws. Where should the close happen?

**Answer:** In a `finally` block around the processing operation.

**Explanation:** The close runs on both normal and exceptional exits from the block.

### 36. A database commit is only performed in `__destruct()`. Why is this unsafe?

**Answer:** Destructor timing and shutdown order are not reliable, and the operation can fail after normal exception handling is unavailable.

**Explanation:** Commit or roll back explicitly in the transaction flow.

### 37. A class defines a child destructor but omits the parent's required cleanup. What is missing?

**Answer:** An explicit `parent::__destruct()` call if the parent's cleanup is required.

**Explanation:** Parent destructors are not automatically called when a child defines its own.

### 38. A class has three different ways to parse input into a valid object. What design may make the call sites clearer?

**Answer:** Add named static factory methods for each input format.

**Explanation:** The factory can parse and validate the input before calling the single constructor.

### 39. A class supports PHP 7.4, but a developer uses constructor property promotion. What is the issue?

**Answer:** Property promotion requires PHP 8.0 or later.

**Explanation:** Use explicit property declarations and assignments to support PHP 7.4.

### 40. An object has no constructor. Can it still be instantiated?

**Answer:** Yes. Use `new ClassName()` with no arguments.

**Explanation:** If the class inherits a constructor with required arguments, those requirements still apply.

## Coding questions

### 41. Define a constructor that requires a non-empty name and stores it in a private property.

**Answer:**

~~~php
class Member
{
    private string $name;

    public function __construct(string $name)
    {
        if (trim($name) === '') {
            throw new InvalidArgumentException(
                'Name cannot be empty.'
            );
        }

        $this->name = trim($name);
    }
}
~~~

**Explanation:** The constructor validates the required value before storing it.

### 42. Create a member with an optional role that defaults to `reader`.

**Answer:**

~~~php
class Member
{
    public function __construct(
        private string $name,
        private string $role = 'reader',
    ) {
    }
}

$member = new Member('Mina');
~~~

**Explanation:** The promoted property `$role` receives its default when the caller omits it. This syntax requires PHP 8.0 or later.

### 43. Show a child constructor calling a parent constructor.

**Answer:**

~~~php
class Record
{
    public function __construct(
        protected string $id,
    ) {
    }
}

class UserRecord extends Record
{
    public function __construct(string $id, private string $name)
    {
        parent::__construct($id);
    }
}
~~~

**Explanation:** The explicit parent call initializes the base record's state.

### 44. Write a named factory that validates an integer ID before creating an object.

**Answer:**

~~~php
class UserId
{
    private function __construct(
        private int $value,
    ) {
    }

    public static function fromInt(int $value): self
    {
        if ($value < 1) {
            throw new InvalidArgumentException(
                'ID must be positive.'
            );
        }

        return new self($value);
    }
}

$id = UserId::fromInt(42);
~~~

**Explanation:** The private constructor forces callers to use the validating factory.

### 45. Close a file after processing even if processing throws.

**Answer:**

~~~php
$handle = fopen('report.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the report.');
}

try {
    processReport($handle);
} finally {
    fclose($handle);
}
~~~

**Explanation:** The `finally` block performs predictable cleanup on normal and exceptional paths.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [063 - Properties, methods, and object creation](../lessons/063-properties-methods-and-object-creation.md) | [Read lesson](../lessons/064-constructors-and-destructors.md) | [Open example](../examples/064-constructors-and-destructors.md) | [065 - Visibility and encapsulation](../lessons/065-visibility-and-encapsulation.md) |
