# 046 - Generators and yield

A generator produces values one at a time instead of building and returning an entire array. In PHP, a function that contains **yield** is a generator function. Calling it returns a **Generator** object, and iterating over that object resumes the function each time another value is requested.

Generators are useful when values can be computed as they are consumed, when a sequence is large, or when an iterator should keep its progress between values.

## What you will learn

- Recognize a generator function
- Create a **Generator** object and iterate over it
- Understand how **yield** pauses and resumes execution
- Yield values with automatic or explicit keys
- Delegate to another iterable with **yield from**
- Return a final value and read it with **getReturn()**
- Send a value back into a suspended generator
- Understand when a generator can reduce memory use

## A simple generator

A generator function looks like a normal function, but it uses **yield** to produce values:

~~~php
<?php
function countToThree(): Generator
{
    yield 1;
    yield 2;
    yield 3;
}

foreach (countToThree() as $number) {
    echo $number, "\n";
}
~~~

Output:

~~~text
1
2
3
~~~

The function returns a **Generator** object. The **foreach** loop requests each value in order. The generator pauses at each **yield** and resumes when the loop requests the next value.

## Generator functions pause and resume

A normal **return** ends a function. **yield** provides one value and suspends the generator so it can continue later:

~~~php
<?php
function steps(): Generator
{
    echo "Start\n";

    yield 'first';

    echo "Continue\n";

    yield 'second';

    echo "Finish\n";
}

$generator = steps();

echo "Generator created\n";

foreach ($generator as $step) {
    echo "Received: {$step}\n";
}
~~~

Output:

~~~text
Generator created
Start
Received: first
Continue
Received: second
Finish
~~~

Calling **steps()** creates the generator object; it does not run the function body up to the first **yield**. Iteration starts the function. Each later request resumes it from the point where it paused.

## Generate values in a loop

A generator can compute a sequence as needed:

~~~php
<?php
function squares(int $limit): Generator
{
    for ($number = 1; $number <= $limit; $number++) {
        yield $number * $number;
    }
}

foreach (squares(4) as $square) {
    echo $square, "\n";
}
~~~

Output:

~~~text
1
4
9
16
~~~

The function does not first create an array containing all the squares. It computes and yields each square when the caller iterates.

## Automatic keys

When a generator yields values without explicit keys, PHP assigns sequential integer keys, starting at **0**:

~~~php
function letters(): Generator
{
    yield 'A';
    yield 'B';
    yield 'C';
}

foreach (letters() as $key => $letter) {
    echo "{$key}: {$letter}\n";
}
~~~

Output:

~~~text
0: A
1: B
2: C
~~~

The keys make a generator iterable like an array or other iterator.

## Yield key/value pairs

Use **yield $key => $value** to choose an explicit key:

~~~php
<?php
function courses(): Generator
{
    yield 'php' => 'PHP';
    yield 'js' => 'JavaScript';
}

foreach (courses() as $code => $name) {
    echo "{$code}: {$name}\n";
}
~~~

Output:

~~~text
php: PHP
js: JavaScript
~~~

This is useful when consumers need to associate each yielded value with an identifier.

## Yield a null value

Using **yield** without an expression yields **null**:

~~~php
function emptySlots(): Generator
{
    yield;
    yield;
}

var_dump(iterator_to_array(emptySlots()));
~~~

The resulting values are both **null**. In normal application code, yielding explicit values is usually easier to understand.

## Delegate values with yield from

Use **yield from** to pass values from an array, another generator, or another **Traversable** object through the current generator:

~~~php
<?php
function allNumbers(): Generator
{
    yield 1;
    yield from [2, 3];
    yield 4;
}

foreach (allNumbers() as $number) {
    echo $number, "\n";
}
~~~

Output:

~~~text
1
2
3
4
~~~

The outer generator continues after the delegated iterable is exhausted.

### Delegate to another generator

~~~php
<?php
function firstPair(): Generator
{
    yield 'A';
    yield 'B';
}

function allLetters(): Generator
{
    yield from firstPair();
    yield 'C';
}

foreach (allLetters() as $letter) {
    echo $letter, "\n";
}
~~~

Output:

~~~text
A
B
C
~~~

This lets a generator compose smaller sequences without manually forwarding every value.

### Keys are preserved by yield from

**yield from** keeps the keys from the delegated iterable. When values are collected into an array, duplicate keys can overwrite earlier values:

~~~php
<?php
function values(): Generator
{
    yield 'start';
    yield from ['left' => 'L', 'right' => 'R'];
    yield 'end';
}

$preservingKeys = iterator_to_array(values());
$sequentialValues = iterator_to_array(values(), false);

echo implode(', ', $preservingKeys), "\n";
echo implode(', ', $sequentialValues), "\n";
~~~

The first collection preserves keys. The automatically assigned key after the keyed delegation may overlap with an existing integer key, so it can replace a value in the resulting array. Passing **false** as the second argument to **iterator_to_array()** discards keys and stores every yielded value at the next numeric index.

When keys matter, choose and manage them deliberately.

## Return a final value

A generator may use **return** to produce a final result after its yielded sequence is complete. That return value is not another yielded item. Retrieve it with **Generator::getReturn()** after iteration has finished:

~~~php
<?php
function generateReport(): Generator
{
    yield 'Reading records';
    yield 'Writing summary';

    return 'Done';
}

$report = generateReport();

foreach ($report as $message) {
    echo $message, "\n";
}

echo $report->getReturn(), "\n";
~~~

Output:

~~~text
Reading records
Writing summary
Done
~~~

Calling **getReturn()** before the generator has finished is an error. Finish iteration first.

## Send a value into a generator

The value returned from a **yield** expression can receive data sent by the caller with **Generator::send()**:

~~~php
<?php
function askForName(): Generator
{
    $name = yield 'What is your name?';

    yield "Hello, {$name}!";
}

$generator = askForName();

echo $generator->current(), "\n";

$generator->send('Asha');

echo $generator->current(), "\n";
~~~

Output:

~~~text
What is your name?
Hello, Asha!
~~~

The first **current()** starts the generator and reads its first yielded prompt. **send('Asha')** supplies the value of the suspended **yield** expression and resumes execution. The next **current()** reads the greeting that was yielded.

Sending values is an advanced use. For ordinary data sequences, **foreach** is simpler.

## Generators and memory

A generator avoids building the complete output sequence in an array before iteration. This can reduce memory use when a sequence is large and values can be produced one at a time.

A generator is not automatically more memory-efficient in every situation. If the input data is already stored in a large array, that array still occupies memory. Generators also provide sequential access rather than random access by index. Create an array when the program needs to retain, count, or repeatedly access all values.

## A generator is an iterator

Calling a generator function returns a **Generator** object, which implements the **Iterator** interface. Usually, use **foreach** to advance it. The object also provides methods such as **current()**, **key()**, **next()**, **valid()**, **send()**, and **getReturn()**.

A generator instance progresses through its sequence. To iterate again from the beginning, call the generator function again to create a new generator.

## Common mistakes

- Expecting a generator function to return an array.
- Expecting the function body to run all the way to the first **yield** when the generator is created.
- Using **return** when the intent is to yield another value.
- Expecting the generator's final **return** value to appear in the **foreach** loop.
- Calling **getReturn()** before the generator has completed.
- Treating a generator as an array with random index access.
- Forgetting that **yield from** preserves keys.
- Assuming **iterator_to_array()** always produces a list of sequential values.
- Expecting a generator to save memory when the full input is still stored in an array.
- Using **send()** when a simple **foreach** loop would be clearer.

## Try it yourself

1. Write a generator that yields the integers from **1** to **5**.
2. Write a generator that yields squares and use **foreach** to print them.
3. Yield a string keyed by a short code and print both key and value.
4. Use **yield from** to combine two small arrays into one sequence.
5. Return a final count from a generator and retrieve it with **getReturn()** after iteration.
6. Use **send()** to provide a name to a suspended generator.
7. Compare **iterator_to_array($generator)** with **iterator_to_array($generator, false)** when yielded keys can repeat.

## Key takeaways

- Any function containing **yield** is a generator function.
- Calling it returns a **Generator** object; iteration starts and resumes its body.
- **yield** produces a value and pauses execution.
- Generators can yield automatic integer keys or explicit key/value pairs.
- **yield from** delegates to an array, generator, or other traversable value and preserves its keys.
- A generator's final **return** value is available through **getReturn()** after completion.
- Generators can produce values on demand without first collecting all output in an array.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [045 - Passing and Returning Values by Reference](045-passing-and-returning-values-by-reference.md) | [Open Q&A](../qnas/046-generators-and-yield.md) | [Open example](../examples/046-generators-and-yield.md) | [047 - Iterators, Traversable, and Iterable Values](047-iterators-traversable-and-iterable-values.md) |
