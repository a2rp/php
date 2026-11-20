# 017 - Objects, Resources, Callables, and Iterables: Questions and Answers

Use this Q&A to review [017 - Objects, Resources, Callables, and Iterables](../lessons/017-objects-resources-callables-and-iterables.md). Each answer follows its question and includes an explanation where useful.

## Multiple-choice questions

### 1. What does an object represent in PHP?

A. A value that can only contain one integer<br>
B. An instance of a class that can hold state and provide behavior<br>
C. A file stored on disk<br>
D. A function that always returns an array

**Answer: B.** An object is an instance of a class. Its properties can hold state, and its methods can provide behavior.

### 2. Which expression creates an instance of PHP's built-in **stdClass**?

A. **stdClass()**<br>
B. **new stdClass()**<br>
C. **object stdClass**<br>
D. **create stdClass**

**Answer: B.** The **new** expression creates the object.

### 3. How do you read the **name** property from **$profile**?

A. **$profile["name"]**<br>
B. **$profile->name**<br>
C. **$profile::name**<br>
D. **name($profile)**

**Answer: B.** Object properties are accessed with the **->** operator. Square brackets are commonly used to access array entries.

### 4. Which function checks whether a value is an object?

A. **is_resource()**<br>
B. **is_object()**<br>
C. **is_iterable()**<br>
D. **is_array()**

**Answer: B.** **is_object()** returns true when its argument is an object.

### 5. What does assigning an object to another variable usually do?

A. It automatically clones every property into an independent object<br>
B. Both variables refer to the same object instance<br>
C. It converts the object to an array<br>
D. It closes the object

**Answer: B.** Assigning an object copies its object identifier. Both variables can refer to the same instance; use **clone** when an independent shallow copy is needed.

### 6. What is a PHP resource?

A. A special value referring to an external handle managed by PHP<br>
B. A PHP keyword for creating a class<br>
C. A string containing a URL<br>
D. Any object with a public property

**Answer: A.** Resources represent handles such as opened streams. Many modern APIs also represent handles as objects, so not every handle is a resource.

### 7. What can **fopen()** return when it cannot open the requested stream?

A. An empty object<br>
B. **false**<br>
C. An empty array<br>
D. The integer **0**

**Answer: B.** **fopen()** returns a resource on success or **false** on failure. Check the result before calling stream functions.

### 8. Which check is appropriate before using a value expected to be a resource?

A. **is_resource($value)**<br>
B. **is_object($value)**<br>
C. **is_callable($value)**<br>
D. **count($value)**

**Answer: A.** **is_resource()** tells you whether the value is currently a resource.

### 9. Which function reports a resource's type?

A. **resource_type()**<br>
B. **get_resource_type()**<br>
C. **gettype_resource()**<br>
D. **stream_name()**

**Answer: B.** **get_resource_type()** returns a string describing a resource, such as **"stream"** for an opened stream.

### 10. What should a program normally do when it has finished using a stream opened with **fopen()**?

A. Call **fclose()** on it<br>
B. Convert it to an integer<br>
C. Call **is_callable()** on it<br>
D. Rename the resource

**Answer: A.** Close a stream with **fclose()** when finished. This releases the handle.

### 11. Which value can be callable?

A. Only a function-name string<br>
B. Only an anonymous function<br>
C. A function name, closure, method callable, or invokable object<br>
D. Any string, whether or not it names a function

**Answer: C.** PHP supports several callable forms. A random string is not callable just because it is a string.

### 12. Which function checks whether a value can currently be called?

A. **is_callable()**<br>
B. **is_function()**<br>
C. **callable_exists()**<br>
D. **can_call()**

**Answer: A.** **is_callable()** checks whether a value is callable in the current scope.

### 13. What is **$formatter** in this code?

~~~php
$formatter = "strtoupper";
~~~

A. A resource handle<br>
B. A string that can name a callable function<br>
C. An object created by **new**<br>
D. An iterable by definition

**Answer: B.** The string names PHP's **strtoupper()** function, so it can be used as a callable.

### 14. Which value is an iterable according to PHP's iterable type?

A. Every object<br>
B. An array or an object implementing **Traversable**<br>
C. Only a string<br>
D. Only an object created with **stdClass**

**Answer: B.** PHP's iterable type accepts arrays and objects that implement **Traversable**.

### 15. Which function checks whether a value is iterable?

A. **is_array()** only<br>
B. **is_traversable()**<br>
C. **is_iterable()**<br>
D. **can_iterate()**

**Answer: C.** **is_iterable()** returns true for arrays and **Traversable** objects.

### 16. Is a plain **stdClass** object automatically iterable?

A. Yes, every object is iterable<br>
B. No, an object must implement **Traversable** to qualify<br>
C. Yes, but only when it has a **name** property<br>
D. No, because only resources are iterable

**Answer: B.** A plain object does not become iterable just because it has properties. It must implement **Traversable**.

### 17. What does this expression return?

~~~php
is_iterable(["red", "blue"])
~~~

A. **true**<br>
B. **false**<br>
C. **null**<br>
D. A resource

**Answer: A.** Every PHP array is iterable, including this indexed array.

### 18. What is the role of **ArrayIterator** in this example?

~~~php
$items = new ArrayIterator(["red", "blue"]);
~~~

A. It wraps an array in an object that supports iteration<br>
B. It converts an array into a resource<br>
C. It declares a new PHP class<br>
D. It closes an array

**Answer: A.** **ArrayIterator** is a built-in iterator object and implements the interfaces needed to be traversable.

## True or false

### 19. An array property is accessed with **->**.

**Answer: False.** Use square brackets for an array entry, such as **$items["name"]**. The **->** operator accesses an object property or method.

### 20. **is_object([])** returns **true**.

**Answer: False.** An array is not an object, so **is_object([])** returns **false**.

### 21. A successful call to **fopen()** gives a value that should be checked before use.

**Answer: True.** Check for **false** before passing the result to functions such as **fwrite()**, **fread()**, or **fclose()**.

### 22. A stream resource remains a resource after it has been closed with **fclose()**.

**Answer: False.** Once the stream has been closed, **is_resource()** returns false for that variable.

### 23. Every external handle used by every PHP extension must be a resource.

**Answer: False.** Some modern PHP APIs use objects for handles. Follow the return type and lifecycle rules of the specific API.

### 24. If **is_callable($value)** returns true, the value can be invoked as a callback at that point.

**Answer: True.** It is a useful callability check. It does not guarantee that invoking the callback will succeed for every possible argument or that the callback will complete without an error.

### 25. Every object is iterable because objects have properties.

**Answer: False.** Iterability is not the same as having properties. The object must implement **Traversable**; otherwise it does not satisfy PHP's **iterable** type.

## Read the code

### 26. What does this code print?

~~~php
$profile = new stdClass();
$profile->name = "Asha";

echo $profile->name;
~~~

**Answer:**

~~~text
Asha
~~~

**Explanation:** The property is assigned and read with the **->** operator.

### 27. What does this code print?

~~~php
$first = new stdClass();
$first->status = "ready";

$second = $first;
$second->status = "updated";

echo $first->status;
~~~

**Answer:**

~~~text
updated
~~~

**Explanation:** Both variables refer to the same object instance. Changing a property through **$second** is visible through **$first**.

### 28. What does **is_resource($stream)** return after the successful **fopen()** call?

~~~php
$stream = fopen("php://temp", "w+");
var_dump(is_resource($stream));
fclose($stream);
~~~

**Answer:** It prints **bool(true)** before the stream is closed.

**Explanation:** The example assumes **fopen()** succeeded. Production code should still check for **false** before using the result.

### 29. What does the resource type check print for an opened stream?

~~~php
$stream = fopen("php://temp", "w+");
echo get_resource_type($stream);
fclose($stream);
~~~

**Answer:**

~~~text
stream
~~~

**Explanation:** An opened stream is a resource of type **"stream"**. The example assumes **fopen()** succeeded.

### 30. What does the final check print?

~~~php
$stream = fopen("php://temp", "w+");
fclose($stream);

var_dump(is_resource($stream));
~~~

**Answer:**

~~~text
bool(false)
~~~

**Explanation:** **fclose()** closes the stream. The variable still exists, but it no longer holds an active resource.

### 31. What does this code print?

~~~php
$formatter = "strtoupper";

var_dump(is_callable($formatter));
echo $formatter("php");
~~~

**Answer:**

~~~text
bool(true)
PHP
~~~

**Explanation:** The string names the built-in **strtoupper()** function, and the callable variable can be invoked.

### 32. Is this closure callable, and what does the last line print?

~~~php
$label = function ($topic) {
    return "Topic: " . $topic;
};

var_dump(is_callable($label));
echo $label("PHP");
~~~

**Answer:**

~~~text
bool(true)
Topic: PHP
~~~

**Explanation:** A closure is a callable object. It can be stored in a variable and invoked through that variable.

### 33. What does this code print?

~~~php
$items = ["one", "two"];
$object = new stdClass();

var_dump(is_iterable($items));
var_dump(is_iterable($object));
~~~

**Answer:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Arrays are iterable. A plain **stdClass** does not implement **Traversable**, so it is not iterable.

### 34. What do these checks print?

~~~php
$items = new ArrayIterator(["one", "two"]);

var_dump(is_object($items));
var_dump(is_iterable($items));
~~~

**Answer:**

~~~text
bool(true)
bool(true)
~~~

**Explanation:** **ArrayIterator** is an object that implements the iterator interfaces, so it is also iterable.

## Write code

### 35. Create an object with a **title** property and print it.

**Answer:**

~~~php
$chapter = new stdClass();
$chapter->title = "PHP Types";

echo $chapter->title . PHP_EOL;
~~~

**Output:**

~~~text
PHP Types
~~~

**Explanation:** **stdClass** makes a convenient small object for this exercise. A later lesson covers declaring your own classes.

### 36. Open a temporary stream, check for failure, write a message, and close it.

**Answer:**

~~~php
$stream = fopen("php://temp", "w+");

if ($stream === false) {
    exit("Could not open the stream." . PHP_EOL);
}

fwrite($stream, "Saved temporarily");
fclose($stream);
~~~

**Explanation:** Always handle the **false** failure result before calling stream functions. Close the stream when the work is done.

### 37. Store a built-in function name in a variable, check it, and call it.

**Answer:**

~~~php
$formatter = "strtoupper";

if (is_callable($formatter)) {
    echo $formatter("php") . PHP_EOL;
}
~~~

**Output:**

~~~text
PHP
~~~

**Explanation:** **is_callable()** checks the value before it is invoked. The callback's argument requirements must still be respected.

### 38. Check whether an array and a plain object are iterable.

**Answer:**

~~~php
$values = ["first", "second"];
$record = new stdClass();

var_dump(is_iterable($values));
var_dump(is_iterable($record));
~~~

**Output:**

~~~text
bool(true)
bool(false)
~~~

**Explanation:** Arrays are iterable. A plain object needs to implement **Traversable** to qualify.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [016 - Indexed and associative arrays](../lessons/016-indexed-and-associative-arrays.md) | [Read this lesson](../lessons/017-objects-resources-callables-and-iterables.md) | [Open example](../examples/017-objects-resources-callables-and-iterables.md) | [018 - Type juggling, casting, and conversion](../lessons/018-type-juggling-casting-and-conversion.md) |
