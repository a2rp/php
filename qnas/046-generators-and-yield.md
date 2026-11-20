# 046 - Generators and yield: Q&A

Review generator functions, lazy iteration, **yield**, keys, **yield from**, final return values, and values sent into a generator. Try each question before reading its answer. See the [chapter lesson](../lessons/046-generators-and-yield.md).

## Multiple-choice questions

### 1. What makes a function a generator function?

A. It returns an array.
B. It contains **yield**.
C. It is declared with **Generator** as its name.
D. It uses **foreach**.

**Answer:** B. It contains **yield**.

**Explanation:** Any function containing **yield** is treated as a generator function.

### 2. What does calling a generator function return?

A. An array containing every yielded value
B. A **Generator** object
C. The first yielded value
D. A string

**Answer:** B. A **Generator** object.

**Explanation:** The generator object can be iterated to request values from the function.

### 3. When does a generator function start running?

A. When PHP parses the file
B. When the generator function is called
C. When the returned generator is first advanced or iterated
D. Only when **getReturn()** is called

**Answer:** C. When the returned generator is first advanced or iterated.

**Explanation:** Calling the function creates the generator object. Iteration starts its body and runs it to the first **yield**.

### 4. What does **yield** do?

A. Ends the entire script
B. Produces a value and suspends the generator until it is resumed
C. Creates an array
D. Returns from the function permanently

**Answer:** B. Produces a value and suspends the generator until it is resumed.

**Explanation:** The generator can continue from the yield point when another value is requested.

### 5. What keys are assigned when values are yielded without explicit keys?

A. Sequential integer keys
B. Random keys
C. String versions of the values
D. No keys

**Answer:** A. Sequential integer keys.

**Explanation:** A generator can be iterated as key/value pairs; unkeyed yields receive automatic integer keys.

### 6. Which syntax yields a value with an explicit key?

A. **yield $key => $value**
B. **yield ($key, $value)**
C. **return $key -> $value**
D. **yield key($value)**

**Answer:** A. **yield $key => $value**.

**Explanation:** This follows the familiar key/value form used in associative arrays.

### 7. What can **yield from** delegate values from?

A. Arrays only
B. Generators only
C. Arrays, generators, and other traversable values
D. Strings only

**Answer:** C. Arrays, generators, and other traversable values.

**Explanation:** **yield from** forwards an iterable's values through the current generator.

### 8. When can **Generator::getReturn()** retrieve a generator's final return value?

A. Before the generator starts
B. While it is paused at its first yield
C. After it has finished
D. Only inside the generator function

**Answer:** C. After it has finished.

**Explanation:** The final return value is available after the generator completes.

### 9. Does a generator's final **return** value appear as another value in a **foreach** loop?

A. Yes, always
B. No; use **getReturn()** after completion
C. Only if it is an integer
D. Only when using **yield from**

**Answer:** B. No; use **getReturn()** after completion.

**Explanation:** **yield** produces iterable values. **return** ends the generator and can supply a separate final value.

### 10. What does **yield;** produce?

A. An empty string
B. **null**
C. **false**
D. It causes a syntax error

**Answer:** B. **null**.

**Explanation:** A yield without an expression yields **null** with an automatic key.

### 11. What is a typical memory benefit of a generator?

A. It stores all values in a hidden array for fast random access.
B. It can produce values as they are consumed instead of building the whole output array first.
C. It always reduces total program memory to zero.
D. It compresses the input data.

**Answer:** B. It can produce values as they are consumed instead of building the whole output array first.

**Explanation:** It can reduce memory use for generated output, but existing input data may still occupy memory.

### 12. What does **Generator::send($value)** do?

A. Sends a value as the result of the suspended **yield** expression and resumes the generator
B. Changes the last yielded key
C. Adds the value to an array
D. Restarts the generator from the beginning

**Answer:** A. Sends a value as the result of the suspended **yield** expression and resumes the generator.

**Explanation:** The generator can receive input as it continues.

### 13. What does **yield from** do with keys from its delegated iterable?

A. It always resets them to zero.
B. It preserves them.
C. It removes them.
D. It converts them to uppercase.

**Answer:** B. It preserves them.

**Explanation:** When collecting values into an array, duplicate keys can therefore overwrite earlier values.

### 14. Which interface does the **Generator** class implement?

A. **Countable**
B. **Iterator**
C. **ArrayAccess**
D. **JsonSerializable**

**Answer:** B. **Iterator**.

**Explanation:** A generator object provides the iterator behavior used by **foreach**.

### 15. What happens if **yield** occurs inside a conditional branch of a function?

A. The function cannot be a generator.
B. The function is a generator function because it contains **yield**.
C. Only that branch becomes a closure.
D. PHP ignores the yield.

**Answer:** B. The function is a generator function because it contains **yield**.

**Explanation:** A function containing **yield** is a generator function, even if the yield executes only when a condition is met.

## Short-answer questions

### 16. What is the difference between **return** and **yield** in a generator?

**Answer:** **yield** provides one value and suspends execution. **return** ends the generator and may provide a final value.

**Explanation:** A return value is not part of the yielded sequence; retrieve it with **getReturn()** after completion.

### 17. Does a generator function build an array of all its values automatically?

**Answer:** No.

**Explanation:** It produces values as iteration requests them. The caller can choose to collect them into an array if needed.

### 18. How do you iterate through a generator's values in ordinary code?

**Answer:** Use a **foreach** loop.

**Explanation:** The loop requests the next value and resumes the generator as needed.

### 19. What does **yield $id => $record** produce?

**Answer:** A yielded key/value pair with **$id** as the key and **$record** as the value.

**Explanation:** The consumer can read both with **foreach ($generator as $id => $record)**.

### 20. What is the purpose of **yield from $iterable**?

**Answer:** It delegates values from another iterable to the current generator.

**Explanation:** The outer generator continues after the delegated iterable is exhausted.

### 21. Why can **iterator_to_array()** produce a shorter array after **yield from**?

**Answer:** The delegated keys are preserved, and duplicate keys in the collected array overwrite earlier values.

**Explanation:** Pass **false** as the second argument to **iterator_to_array()** when all values should be collected under sequential numeric keys.

### 22. When should you call **getReturn()**?

**Answer:** After the generator has completed iteration.

**Explanation:** Before completion, no final return value is available.

### 23. What does the **send()** method pass into a generator?

**Answer:** It supplies the value that becomes the result of the generator's currently suspended **yield** expression.

**Explanation:** The call also resumes execution.

### 24. How can a generator use less memory than building an array first?

**Answer:** It can compute and provide one value at a time instead of storing the entire output sequence.

**Explanation:** This helps when the output is large and consumed sequentially.

### 25. Does a generator provide random access by numeric index like an array?

**Answer:** No.

**Explanation:** A generator is an iterator that produces values in sequence.

### 26. How can you iterate through the same sequence again?

**Answer:** Call the generator function again to create a new generator object.

**Explanation:** An already advanced generator is a stateful one-pass sequence.

### 27. What is the value produced by **yield** with no expression?

**Answer:** **null**.

**Explanation:** It still yields a value and receives an automatic key.

### 28. What is the relationship between a **Generator** object and **foreach**?

**Answer:** The object implements **Iterator**, so **foreach** can request its current value and advance it.

**Explanation:** The generator pauses and resumes as the loop consumes values.

### 29. What does **yield from** return as an expression when delegating to a generator?

**Answer:** It evaluates to the value returned by the inner generator after that generator finishes.

**Explanation:** This can pass a delegated generator's final result back to the outer generator.

## True or false

### 30. A function that contains **yield** returns a **Generator** object when called.

**Answer:** True.

**Explanation:** Iteration over that object runs the function and consumes its yielded values.

### 31. Calling a generator function runs its body all the way to completion immediately.

**Answer:** False.

**Explanation:** The body starts when the generator is advanced and pauses at each **yield**.

### 32. A generator can preserve local state between yields.

**Answer:** True.

**Explanation:** Execution resumes from the suspended point, keeping its local variables.

### 33. A generator's **return** value is yielded automatically to a **foreach** loop.

**Answer:** False.

**Explanation:** Read it separately with **getReturn()** after the generator finishes.

### 34. **yield from** resets all delegated keys.

**Answer:** False.

**Explanation:** It preserves the keys from the delegated iterable.

### 35. A generator is always the best replacement for an array.

**Answer:** False.

**Explanation:** Arrays are useful for random access, counting, and repeated traversal. Generators are useful for sequential on-demand production.

### 36. **Generator::send()** can provide a value back to a suspended generator.

**Answer:** True.

**Explanation:** The sent value becomes the result of the current **yield** expression.

### 37. Generators have existed since PHP 5.5.

**Answer:** True.

**Explanation:** Generators were introduced in PHP 5.5; **yield from** arrived in PHP 7.

## Predict the output

### 38. What does this code print?

~~~php
<?php
function numbers(): Generator
{
    yield 2;
    yield 4;
}

foreach (numbers() as $number) {
    echo $number, "\n";
}
~~~

**Answer:**

~~~text
2
4
~~~

**Explanation:** The loop consumes the two yielded values in order.

### 39. What does this code print?

~~~php
<?php
function languages(): Generator
{
    yield 'php' => 'PHP';
    yield 'js' => 'JavaScript';
}

foreach (languages() as $code => $language) {
    echo "{$code}: {$language}\n";
}
~~~

**Answer:**

~~~text
php: PHP
js: JavaScript
~~~

**Explanation:** Each **yield** provides an explicit key and value.

### 40. What does this code print?

~~~php
<?php
function allNumbers(): Generator
{
    yield 1;
    yield from [2, 3];
    yield 4;
}

foreach (allNumbers() as $number) {
    echo $number, ' ';
}
~~~

**Answer:**

~~~text
1 2 3 4
~~~

**Explanation:** The outer generator yields **1**, delegates **2** and **3**, then resumes and yields **4**.

### 41. What does this code print?

~~~php
<?php
function report(): Generator
{
    yield 'Reading';
    yield 'Done';

    return 'Complete';
}

$generator = report();

foreach ($generator as $message) {
    echo $message, "\n";
}

echo $generator->getReturn(), "\n";
~~~

**Answer:**

~~~text
Reading
Done
Complete
~~~

**Explanation:** The first two strings are yielded. The final return value is retrieved separately after iteration.

### 42. What does this code print?

~~~php
<?php
function ask(): Generator
{
    $name = yield 'Name?';
    yield "Hello, {$name}!";
}

$generator = ask();

echo $generator->current(), "\n";
$generator->send('Mina');
echo $generator->current(), "\n";
~~~

**Answer:**

~~~text
Name?
Hello, Mina!
~~~

**Explanation:** The first **current()** advances to the prompt. **send('Mina')** supplies the suspended yield expression's value and resumes the generator.

## Write code

### 43. Write a generator that yields the even numbers from 2 through 8.

**Answer:**

~~~php
<?php
function evenNumbers(): Generator
{
    for ($number = 2; $number <= 8; $number += 2) {
        yield $number;
    }
}

foreach (evenNumbers() as $number) {
    echo $number, "\n";
}
~~~

Output:

~~~text
2
4
6
8
~~~

**Explanation:** Each loop iteration computes and yields one value.

### 44. Write a generator that yields **'start'**, delegates **['middle', 'more']**, then yields **'end'**.

**Answer:**

~~~php
<?php
function messages(): Generator
{
    yield 'start';
    yield from ['middle', 'more'];
    yield 'end';
}

foreach (messages() as $message) {
    echo $message, "\n";
}
~~~

Output:

~~~text
start
middle
more
end
~~~

**Explanation:** **yield from** forwards every value from the array before the outer generator continues.

### 45. Write a generator that yields two items and returns a count, then print the count after the loop.

**Answer:**

~~~php
<?php
function items(): Generator
{
    yield 'book';
    yield 'pen';

    return 2;
}

$generator = items();

foreach ($generator as $item) {
    echo $item, "\n";
}

echo 'Count: ', $generator->getReturn(), "\n";
~~~

Output:

~~~text
book
pen
Count: 2
~~~

**Explanation:** The yielded items are iterated normally. The final count is available with **getReturn()** after the generator has completed.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [045 - Passing and Returning Values by Reference](../lessons/045-passing-and-returning-values-by-reference.md) | [Read lesson](../lessons/046-generators-and-yield.md) | [Open example](../examples/046-generators-and-yield.md) | [047 - Iterators, Traversable, and Iterable Values](../lessons/047-iterators-traversable-and-iterable-values.md) |
