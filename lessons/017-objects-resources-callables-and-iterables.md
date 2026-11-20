# 017 - Objects, Resources, Callables, and Iterables

PHP has several value types that represent more than a simple scalar or an array. An object groups state and behavior, a resource refers to an external handle, a callable can be invoked, and an iterable can provide values for iteration. These ideas are related in programs, but they are different types with different rules.

## What you will learn

- What an object represents and how to read its properties
- How a resource refers to an external handle such as a stream
- How to check and release a resource safely
- What makes a value callable and how to check it
- How a callable can be a function name or a closure
- What PHP accepts as an iterable
- Why an object is not automatically iterable
- How **is_object()**, **is_resource()**, **is_callable()**, and **is_iterable()** inspect values

## Objects

An object is an instance of a class. A class defines the object's structure and behavior; a later lesson covers how to declare classes. For now, PHP's built-in **stdClass** can provide a small object with properties:

~~~php
<?php
$profile = new stdClass();

$profile->name = "Asha";
$profile->chapter = 17;

var_dump(is_object($profile));
echo $profile->name . PHP_EOL;
echo $profile->chapter . PHP_EOL;
~~~

Output:

~~~text
bool(true)
Asha
17
~~~

The **new** expression creates an object. The arrow operator **->** accesses a property or method on that object. Array values use square brackets; object properties use **->**.

Objects can hold state and expose behavior through methods. The class lesson later explains how to define properties and methods for your own object types. **stdClass** is useful for a small demonstration, but structured application data often benefits from a class with a clearly defined shape.

Assigning an object to another variable does not automatically clone it. Both variables can refer to the same object instance, as introduced in the type-system lesson.

## Resources

A resource is a special value that refers to an external handle managed by PHP. Streams, such as opened files, are a common example. Functions that create resources usually return a resource when successful and **false** when they fail.

The **php://temp** stream provides temporary storage for this example, so it does not create a project data file:

~~~php
<?php
$stream = fopen("php://temp", "w+");

if ($stream === false) {
    exit("Could not open the temporary stream." . PHP_EOL);
}

var_dump(is_resource($stream));
var_dump(get_resource_type($stream));

fwrite($stream, "Temporary stream data");
rewind($stream);

echo fread($stream, 1024) . PHP_EOL;

fclose($stream);
var_dump(is_resource($stream));
~~~

Output:

~~~text
bool(true)
string(6) "stream"
Temporary stream data
bool(false)
~~~

Check a function's result before using it. Here **fopen()** can return **false**, so the script checks for failure before calling stream functions.

A resource has a lifetime. Close a stream with **fclose()** when you are finished with it. After closing this stream, **is_resource()** returns false.

Not every external handle in every PHP extension is represented as a resource in modern PHP. Some APIs use objects instead. Check the type and lifecycle rules of the function you are using.

## Callables

A callable is a value that PHP can invoke as a function or method. Functions that accept callbacks commonly use callables. A callable can be a function-name string, a closure, an object-method pair, or an invokable object.

### A function name can be callable

A string containing the name of an existing function can be checked with **is_callable()** and invoked:

~~~php
<?php
$formatter = "strtoupper";

var_dump(is_callable($formatter));
echo $formatter("php") . PHP_EOL;
~~~

Output:

~~~text
bool(true)
PHP
~~~

**is_callable()** checks whether the value can be called from the current scope. A string is not automatically callable merely because it looks like a function name; the function must exist and be accessible.

### A closure can be callable

An anonymous function creates a closure object that can be stored in a variable and called:

~~~php
<?php
$makeLabel = function ($title) {
    return "Chapter: " . $title;
};

var_dump(is_callable($makeLabel));
echo $makeLabel("Callables") . PHP_EOL;
~~~

Output:

~~~text
bool(true)
Chapter: Callables
~~~

The closure can be passed to another function that expects a callback. The functions and callbacks lesson later explores that pattern in more detail.

Do not invoke a value from an untrusted source just because it is a string. Check that a callback is allowed for the task; **is_callable()** only answers whether PHP can call the value, not whether the operation is safe or appropriate.

## Iterables

An iterable is a value PHP accepts for iteration. The **iterable** type represents either an array or an object that implements **Traversable**. A generator is also iterable.

~~~php
<?php
$list = ["HTML", "CSS", "PHP"];
$iterator = new ArrayIterator($list);
$plainObject = new stdClass();

var_dump(is_iterable($list));
var_dump(is_iterable($iterator));
var_dump(is_iterable($plainObject));
~~~

Output:

~~~text
bool(true)
bool(true)
bool(false)
~~~

Arrays are iterable. An object is iterable only when it follows the **Traversable** contract. A plain **stdClass** object does not become iterable just because it has properties.

The **is_iterable()** function checks whether a value can be used as an iterable. The **foreach** lesson later shows how to process each value and key.

## Compare the four concepts

| Concept | What it represents | Useful check or example |
| --- | --- | --- |
| Object | An instance with properties and possibly methods | **is_object($value)** |
| Resource | A PHP-managed handle to an external resource | **is_resource($value)** |
| Callable | A function or method that can be invoked | **is_callable($value)** |
| Iterable | An array or Traversable object that can be iterated | **is_iterable($value)** |

A value can satisfy more than one description. For example, a closure is an object and is callable. An **ArrayIterator** is an object and is iterable. The checks answer different questions; one does not replace the others.

## Inspect values carefully

Use **var_dump()** to see a value's type during development. Use the specific checks when a decision depends on one of these categories:

~~~php
<?php
$object = new stdClass();
$callback = "strtoupper";
$values = ["one", "two"];

var_dump(is_object($object));
var_dump(is_callable($callback));
var_dump(is_iterable($values));
var_dump(is_resource($object));
~~~

Output:

~~~text
bool(true)
bool(true)
bool(true)
bool(false)
~~~

A type check tells you what PHP can do with a value. It does not establish that a resource is open, that a callback is authorized for a particular task, or that an iterable contains the values your application expects.

## Try it yourself

1. Create a **stdClass** object with **name** and **chapter** properties. Read both properties using **->**.
2. Open a **php://temp** stream, write a short message, rewind it, read it, then close the stream.
3. Check a value returned by **fopen()** before calling **fwrite()** or **fread()**.
4. Check whether **"strtoupper"** is callable, then invoke it with a lowercase word.
5. Create a closure that adds a prefix to a title. Store it in a variable and invoke it.
6. Use **is_iterable()** with an array, an **ArrayIterator**, and a **stdClass** object. Compare the results.
7. Explain why **is_callable()** does not prove that invoking a callback is safe for your application.
8. Explain why a resource should be closed according to the API that created it.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Treating an object as an associative array | Properties and array entries use different access syntax. | Use **->** for object properties and brackets for array keys. |
| Assuming every object is iterable | A plain object does not automatically implement **Traversable**. | Use **is_iterable()** or verify the object's interface. |
| Using a resource result without checking for failure | Functions such as **fopen()** can return **false**. | Check the result before passing it to resource functions. |
| Forgetting to close a stream | The handle can remain open longer than needed. | Close it with the matching API, such as **fclose()**. |
| Assuming every external handle is a resource | Some modern PHP APIs use objects for handles. | Inspect the API's return type and lifecycle requirements. |
| Calling a string without checking what it names | A string may not identify an accessible function. | Check an expected callback with **is_callable()** and apply an allowlist when needed. |
| Assuming is_callable() means a callback is authorized | It checks whether PHP can invoke the value, not whether your application should invoke it. | Validate the operation against application rules. |
| Calling every callable a closure | A closure is one kind of callable; function names and method pairs can also be callable. | Treat callable as the broader category. |

## Key takeaways

- An object is an instance of a class; use **->** to access its properties and methods.
- A resource is a special handle created by PHP functions; check failures and close streams when finished.
- A callable is any supported value PHP can invoke, including function names and closures.
- An iterable is an array or an object implementing **Traversable**; a plain object is not automatically iterable.
- **is_object()**, **is_resource()**, **is_callable()**, and **is_iterable()** answer different type questions.
- A type check alone does not prove that a value is valid or safe for an application.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [016 - Indexed and associative arrays](016-indexed-and-associative-arrays.md) | [Open Q&A](../qnas/017-objects-resources-callables-and-iterables.md) | [Open example](../examples/017-objects-resources-callables-and-iterables.md) | [018 - Type juggling, casting, and conversion](018-type-juggling-casting-and-conversion.md) |
