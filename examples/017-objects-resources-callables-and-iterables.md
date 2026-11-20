# 017 - Objects, Resources, Callables, and Iterables: Complete Example

This command-line example creates a small object, safely uses a temporary stream resource, invokes two callable values, and checks which values are iterable.

## Question

How can one PHP script demonstrate objects, resources, callables, and iterables while checking each value with the appropriate PHP function?

## Goal

Create this project:

~~~text
objects-resources-callables-and-iterables-practice/
|-- README.md
|-- values.php
~~~

The **values.php** script demonstrates:

- Creating an object with **stdClass** and reading its properties
- Opening a temporary stream and checking for failure
- Inspecting a resource and closing it when finished
- Checking and invoking a function-name callable
- Creating, checking, and invoking a closure
- Checking an array and **ArrayIterator** as iterable values
- Confirming that a plain **stdClass** object is not automatically iterable

## Complete code: values.php

Save this file at the project root:

~~~php
<?php

$profile = new stdClass();
$profile->name = "Asha";
$profile->chapter = 17;

var_dump(is_object($profile));
echo "Profile: " . $profile->name . PHP_EOL;
echo "Chapter: " . $profile->chapter . PHP_EOL;

$stream = fopen("php://temp", "w+");

if ($stream === false) {
    exit("Could not open the temporary stream." . PHP_EOL);
}

var_dump(is_resource($stream));
echo "Resource type: " . get_resource_type($stream) . PHP_EOL;

fwrite($stream, "Temporary stream data");
rewind($stream);

echo "Stream contents: " . fread($stream, 1024) . PHP_EOL;

fclose($stream);
var_dump(is_resource($stream));

$formatter = "strtoupper";

var_dump(is_callable($formatter));
echo "Formatted: " . $formatter("php") . PHP_EOL;

$makeLabel = function ($topic) {
    return "Chapter: " . $topic;
};

var_dump(is_callable($makeLabel));
echo $makeLabel("objects and related values") . PHP_EOL;

$values = ["first", "second"];
$iterator = new ArrayIterator($values);
$plainObject = new stdClass();

var_dump(is_iterable($values));
var_dump(is_iterable($iterator));
var_dump(is_iterable($plainObject));
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Objects, Resources, Callables, and Iterables Practice

A small PHP CLI script for inspecting several PHP value types.

## Requirements

- PHP CLI installed and available as **php** in the terminal
- A working directory where you can create the two project files

## Check and run

~~~sh
php -l values.php
php values.php
~~~

The first command checks PHP syntax. The second runs the script.

## What the script demonstrates

- **stdClass** creates an object whose properties are read with **->**.
- **fopen()** can return a resource or **false**. The script handles failure before using the stream.
- **is_resource()** and **get_resource_type()** inspect the open stream.
- **fclose()** releases the stream; afterward, the resource check returns false.
- A function-name string and a closure can both be checked with **is_callable()** and invoked.
- An array and an **ArrayIterator** are iterable.
- A plain **stdClass** object is not iterable just because it has properties.

## Expected output

~~~text
bool(true)
Profile: Asha
Chapter: 17
bool(true)
Resource type: stream
Stream contents: Temporary stream data
bool(false)
bool(true)
Formatted: PHP
bool(true)
Chapter: objects and related values
bool(true)
bool(true)
bool(false)
~~~

The first **bool(false)** is printed after the stream is closed. The final three booleans check the array, the iterator object, and the plain object, in that order.
~~~~

## Understand the output

### The profile is an object

**new stdClass()** creates an object. The script assigns **name** and **chapter** properties and reads them with **->**. **is_object()** confirms the value's type.

### The stream is a resource with a lifecycle

**fopen("php://temp", "w+")** opens a temporary in-memory stream. The script checks for **false** before using the returned value. While the stream is open, **is_resource()** returns true and **get_resource_type()** reports **"stream"**.

The script writes text, moves the stream position back to the beginning with **rewind()**, and reads the text. After **fclose()**, the variable no longer contains an active resource, so **is_resource()** returns false.

### Both callback values are callable

The **$formatter** variable contains a function-name string, **"strtoupper"**. The closure stored in **$makeLabel** is another callable value. Each value is checked with **is_callable()** and then invoked.

A successful callability check does not guarantee that a callback accepts every argument. Invoke it with arguments appropriate for that function.

### Iterability is its own property

The array and **ArrayIterator** both pass **is_iterable()**. The plain **stdClass** does not: having object properties alone does not make an object iterable. An object must implement **Traversable** to satisfy PHP's iterable type.

## Make a change

1. Change the profile's **name** and **chapter** values, then read the updated properties.
2. Change the text written to the temporary stream and compare it with the text read back.
3. Replace **strtoupper** with another built-in function that accepts one string, then update the input to match.
4. Change the closure so it returns a different label.
5. Add another value to **$values** and construct the **ArrayIterator** again.
6. Check **is_object($iterator)** as well as **is_iterable($iterator)** to see that an iterator can be both an object and iterable.
7. Remove **fclose($stream)** temporarily and observe why explicitly closing a resource is important. Restore the close call afterward.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Reading an object property with square brackets | Brackets are for array keys; object properties use a different access operator. | Use **$profile->name** for an object property. |
| Assuming **fopen()** always succeeds | It can return **false**, and stream functions require a valid open stream. | Check the result against **false** before using it. |
| Calling **get_resource_type()** on a failed open result | The value may be **false**, not a resource. | Handle the failure before inspecting the resource. |
| Forgetting to close a stream | The handle stays open longer than needed. | Call **fclose()** when finished. |
| Treating every string as a callable | A string is callable only when it resolves to a valid callable in the current scope. | Use **is_callable()** when the value may vary. |
| Assuming **is_callable()** validates every argument | The callback can still fail if it is called with unsuitable arguments or raises an error. | Read the callback's contract and provide suitable arguments. |
| Assuming every object is iterable | Ordinary objects do not automatically implement **Traversable**. | Use **is_iterable()** or check the object's documented interfaces. |
| Confusing an object with a resource | These are separate PHP value types with different APIs. | Inspect the returned value and use the functions appropriate to its type. |

## Key takeaways

- **stdClass** can demonstrate an object and its properties.
- Use **->** to access object properties.
- A resource is a managed handle; check resource-creating functions for failure and close streams when finished.
- **is_resource()** and **get_resource_type()** inspect resources.
- Callables include function-name strings and closures; **is_callable()** checks whether a value can be called.
- PHP's iterable type accepts arrays and objects implementing **Traversable**.
- A plain object is not automatically iterable.
- A value can be an object and iterable at the same time, as **ArrayIterator** demonstrates.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [016 - Indexed and associative arrays](../lessons/016-indexed-and-associative-arrays.md) | [Read this lesson](../lessons/017-objects-resources-callables-and-iterables.md) | [Open Q&A](../qnas/017-objects-resources-callables-and-iterables.md) | [018 - Type juggling, casting, and conversion](../lessons/018-type-juggling-casting-and-conversion.md) |
