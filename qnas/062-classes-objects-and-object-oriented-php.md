# 062 - Classes, Objects, and Object-Oriented PHP: Q&A

Review the difference between a class and an object, instance behavior, object identity, and the basic ideas behind OOP. Try each question before reading its answer. See the [chapter lesson](../lessons/062-classes-objects-and-object-oriented-php.md).

## Multiple-choice questions

### 1. What is a class?

A. One particular object in memory
B. A definition describing a kind of object
C. A PHP configuration file
D. A variable that always contains a string

**Answer:** B. A definition describing a kind of object.

**Explanation:** A class describes the members that its instances can have.

### 2. What is an object?

A. An instance created from a class
B. A comment inside a class
C. A class name written as a string
D. The same thing as a PHP function

**Answer:** A. An instance created from a class.

**Explanation:** A class can be used to create many distinct objects.

### 3. Which keyword creates a new instance of a class?

A. `instance`
B. `create`
C. `new`
D. `object`

**Answer:** C. `new`.

**Explanation:** For example, `$lamp = new Lamp();` creates an instance of `Lamp`.

### 4. What does the `->` operator do in `$lamp->turnOn()`?

A. Accesses or calls a member on an object instance
B. Declares a new class
C. Combines two classes
D. Compares two object values

**Answer:** A. It accesses or calls a member on an object instance.

**Explanation:** Here, the member is the `turnOn()` method.

### 5. Inside an instance method, what does `$this` refer to?

A. The current object instance
B. The parent class name
C. The most recently declared variable
D. A new object created automatically

**Answer:** A. The current object instance.

**Explanation:** It lets an instance method access the object on which the method was called.

### 6. If `new Lamp()` is called twice, what is created?

A. Two separate object instances
B. One object with two names
C. One class and no objects
D. A syntax error

**Answer:** A. Two separate object instances.

**Explanation:** Each `new` expression creates its own object.

### 7. What happens when an object variable is assigned to another variable?

A. A new object is always cloned
B. Both variables can refer to the same object instance
C. The original class is deleted
D. The object's methods become static

**Answer:** B. Both variables can refer to the same object instance.

**Explanation:** Assignment does not automatically clone an object.

### 8. What does `$a === $b` check for two objects?

A. Whether they are the same instance of the same class
B. Whether both variables have the same variable name
C. Whether the objects were created on the same line
D. Whether the class has a constructor

**Answer:** A. Whether they are the same instance of the same class.

**Explanation:** The identity operator checks whether both variables identify the same object.

### 9. In general, what does `$a == $b` compare for objects?

A. Whether both variables refer to the same variable
B. Whether they are instances of the same class with equal properties
C. Whether their class names contain the same letters
D. Whether one object is a clone

**Answer:** B. Whether they are instances of the same class with equal properties.

**Explanation:** Two distinct objects can compare equal with `==` while not being identical with `===`.

### 10. What is object state?

A. The information held by an object
B. The file where the class was saved
C. The number of methods in PHP
D. The current Git branch

**Answer:** A. The information held by an object.

**Explanation:** State is commonly stored in properties.

### 11. What is object behavior?

A. The work an object can perform
B. The class filename
C. A PHP extension
D. A comment above a property

**Answer:** A. The work an object can perform.

**Explanation:** Behavior is commonly implemented by methods.

### 12. Which OOP idea means keeping related data and behavior together?

A. Encapsulation
B. Logging
C. Parsing
D. Serialization

**Answer:** A. Encapsulation.

**Explanation:** Encapsulation also includes controlling how other code interacts with that data and behavior.

### 13. What does abstraction aim to do?

A. Show callers the useful operations while hiding unnecessary details
B. Remove every method from a class
C. Force every function into an object
D. Make all object properties public

**Answer:** A. Show callers the useful operations while hiding unnecessary details.

**Explanation:** A clear abstraction exposes what callers need and keeps implementation details behind the boundary.

### 14. Which statement about OOP in PHP is true?

A. Every PHP program must use classes
B. OOP is one way to organize a program, not a requirement for every task
C. Functions cannot be used with classes
D. Every class must inherit from another class

**Answer:** B. OOP is one way to organize a program, not a requirement for every task.

**Explanation:** Functions can be clearer for simple operations without meaningful object state.

### 15. What is a common convention for PHP class names?

A. PascalCase, such as `ShoppingCart`
B. All lowercase with spaces
C. Uppercase with hyphens
D. Start every name with a number

**Answer:** A. PascalCase, such as `ShoppingCart`.

**Explanation:** This is a style convention; follow the conventions used by the project.

## Short-answer questions

### 16. Explain the difference between a class and an object.

**Answer:** A class is a definition for a kind of object; an object is one particular instance created from that definition.

**Explanation:** Many objects can be created from one class and each can hold independent state.

### 17. Give one example of object state and one example of object behavior.

**Answer:** A shopping cart's list of items is state; adding an item is behavior.

**Explanation:** Properties typically represent state, while methods typically implement behavior.

### 18. Why might a class be useful when several values belong together?

**Answer:** The class can keep the related state and operations in one meaningful unit.

**Explanation:** This can make rules easier to enforce consistently and give the concept a clear boundary.

### 19. What does `$this->value` mean inside an instance method?

**Answer:** It accesses the `value` member on the object that the method call is currently using.

**Explanation:** `$this` is the current object context.

### 20. Does assigning an object variable automatically create a clone?

**Answer:** No. The new variable can refer to the same object instance.

**Explanation:** Use `clone` when a shallow copy is intended; cloning behavior has additional rules.

### 21. What happens to one variable when another variable pointing to the same object mutates that object?

**Answer:** The change is visible through both variables because they refer to the same instance.

**Explanation:** Mutating the object is different from reassigning one variable.

### 22. What is the difference between mutating an object and reassigning a variable?

**Answer:** Mutating changes the object instance; reassigning makes that variable refer to a different value or object.

**Explanation:** Reassigning one variable does not automatically replace the object another variable refers to.

### 23. When can two distinct objects compare equal with `==`?

**Answer:** When they are instances of the same class and have equal properties according to PHP's object comparison rules.

**Explanation:** They can still be distinct instances, so `===` can return false.

### 24. When should `===` be used for objects?

**Answer:** Use it when the question is whether both variables refer to the same instance.

**Explanation:** `==` compares object values; `===` checks object identity.

### 25. What is inheritance?

**Answer:** Inheritance defines a specialized class based on another class.

**Explanation:** The child class can build on behavior from the parent, subject to PHP's inheritance rules.

### 26. What is polymorphism?

**Answer:** Polymorphism lets different object types respond to a shared operation in their own way.

**Explanation:** It helps callers work with a common contract without needing to know every concrete implementation.

### 27. Why is composition often a useful alternative to a deep inheritance tree?

**Answer:** It lets an object use other objects to share responsibilities without building many rigid parent-child relationships.

**Explanation:** Composition can make dependencies and behavior easier to replace independently.

### 28. Is a class always better than a function for a small helper?

**Answer:** No. A function is often clearer when the operation has no meaningful state or lifecycle.

**Explanation:** Use a class when it clarifies the concept or responsibility rather than adding structure without benefit.

### 29. What does the `clone` keyword do at a high level?

**Answer:** It creates a shallow copy of an object and may invoke the object's `__clone()` method.

**Explanation:** Nested objects are not automatically deep-copied.

### 30. Why should a class have a clear responsibility?

**Answer:** A focused class is easier to understand, change, and test than one that combines unrelated work.

**Explanation:** A class with unrelated responsibilities becomes harder to reuse and maintain.

## Scenarios and code review

### 31. Two variables are assigned from the same object. One calls a method that changes a property. What should you expect?

**Answer:** Reading that property through either variable shows the changed value.

**Explanation:** Both variables can refer to the same object instance.

### 32. Two variables are created with separate `new Badge()` calls, and the objects have the same property values. What do `==` and `===` return?

**Answer:** Typically `==` is true and `===` is false.

**Explanation:** They have matching class and property values but are not the same instance.

### 33. A developer creates a class called `Utility` containing many unrelated static functions and no state. What design question should they ask?

**Answer:** Whether these operations would be clearer as regular functions or as separate focused services.

**Explanation:** A class should represent a useful concept or responsibility, not exist just to hold unrelated code.

### 34. A method uses `$this->name`, but the developer calls it as `Person::greet()`. What is wrong?

**Answer:** The method needs an object context, but the call is static.

**Explanation:** Create an instance and call it with `->`, unless the method was specifically declared static and does not rely on `$this`.

### 35. A developer writes `$copy = $original;` and expects an independent object. What should they do?

**Answer:** Use `clone $original` if a shallow object copy is suitable.

**Explanation:** Assignment alone makes both variables refer to the same object. Deep copying nested objects requires additional design.

### 36. A function receives an object and changes one of its properties. Can the caller observe that change?

**Answer:** Yes, because the function parameter identifies the same object instance.

**Explanation:** Reassigning the local parameter to a different object is different and does not by itself reassign the caller's variable.

### 37. A developer checks `$first == $second` to determine whether two variables refer to the same in-memory object. Is this the right check?

**Answer:** No. Use `$first === $second` for object identity.

**Explanation:** `==` can be true for two separate objects with equal values.

### 38. An application has a one-line function that trims a string and stores no state. Must it become a class?

**Answer:** No. A regular function is likely simpler.

**Explanation:** OOP should clarify a design, not add a class automatically.

### 39. A `Report` class handles report data, sends email, connects to a database, and renders HTML. What is a potential design problem?

**Answer:** It may have too many unrelated responsibilities.

**Explanation:** Separate responsibilities can make each part easier to maintain and change.

### 40. A developer says that assigning an object variable is the same as using the `&` reference operator. Is that a good explanation?

**Answer:** No. Object assignment makes variables identify the same object instance, but the variable assignment semantics are not the same as using PHP's explicit reference operator.

**Explanation:** Avoid the shorthand claim that PHP objects are simply "passed by reference." Mutation of an object and reassignment of a variable behave differently.

## Coding questions

### 41. Define an empty class called `Lamp` and create one instance.

**Answer:**

~~~php
class Lamp
{
}

$deskLamp = new Lamp();
~~~

**Explanation:** The class defines the type; `new` creates the object instance.

### 42. Add a method to `Lamp` that prints a message and call it on the instance.

**Answer:**

~~~php
class Lamp
{
    public function turnOn(): void
    {
        echo "Lamp is on." . PHP_EOL;
    }
}

$deskLamp = new Lamp();
$deskLamp->turnOn();
~~~

**Explanation:** The `->` operator calls an instance method.

### 43. Create two `Counter` objects and increment only the first one's state.

**Answer:**

~~~php
class Counter
{
    public int $value = 0;

    public function increment(): void
    {
        $this->value++;
    }
}

$first = new Counter();
$second = new Counter();

$first->increment();

echo $first->value;  // 1
echo $second->value; // 0
~~~

**Explanation:** Each `new Counter()` creates an instance with independent state.

### 44. Assign an object to an alias, mutate it through the alias, and check identity.

**Answer:**

~~~php
$original = new Counter();
$alias = $original;

$alias->increment();

var_dump($original->value);  // int(1)
var_dump($original === $alias); // bool(true)
~~~

**Explanation:** Both variables identify the same object, so the mutation is visible through either one.

### 45. Create two objects with equal values and compare both value equality and identity.

**Answer:**

~~~php
class Badge
{
    public string $label = 'Ready';
}

$first = new Badge();
$second = new Badge();

var_dump($first == $second);  // bool(true)
var_dump($first === $second); // bool(false)
~~~

**Explanation:** The objects have equal class and property values, but they are separate instances.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [061 - Debugging, diagnostics, and application logging](../lessons/061-debugging-diagnostics-and-application-logging.md) | [Read lesson](../lessons/062-classes-objects-and-object-oriented-php.md) | [Open example](../examples/062-classes-objects-and-object-oriented-php.md) | [063 - Properties, methods, and object creation](../lessons/063-properties-methods-and-object-creation.md) |
