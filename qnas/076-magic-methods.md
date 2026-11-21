# Magic methods - Q&A

These questions review PHP magic methods, their triggers, signatures, visibility, and safe use. Each question is followed immediately by its answer and explanation.

## Multiple-choice questions

### 1. What is a magic method?

- A. A method that PHP calls when a specific object operation occurs
- B. Any private method
- C. A function that runs only at compile time
- D. A method that must be static

**Answer: A.** Magic methods customize specific PHP behavior, such as reading an inaccessible property or converting an object to a string.

### 2. Which PHP method runs when a new object is created?

- A. <code>__start()</code>
- B. <code>__construct()</code>
- C. <code>__create()</code>
- D. <code>__init()</code>

**Answer: B.** PHP calls <code>__construct()</code> when a class is instantiated.

### 3. When is <code>__get()</code> called?

- A. On every property read
- B. When code reads an undefined or inaccessible property
- C. When code assigns a property
- D. When the object is destroyed

**Answer: B.** Normal accessible properties are read directly. <code>__get()</code> handles undefined or inaccessible properties.

### 4. When is <code>__set()</code> called?

- A. When code writes an undefined or inaccessible property
- B. On every assignment to a declared public property
- C. When code serializes an object
- D. When a method is called

**Answer: A.** <code>__set()</code> handles writes to undefined or inaccessible properties.

### 5. What is passed to <code>__call()</code>?

- A. The class name and a string
- B. The requested method name and an array of arguments
- C. Only the method's first argument
- D. The object's properties array

**Answer: B.** The first argument is the missing or inaccessible method name; the second is an array containing the arguments supplied by the caller.

### 6. Which method handles calls to undefined or inaccessible static methods?

- A. <code>__static()</code>
- B. <code>__call()</code>
- C. <code>__callStatic()</code>
- D. <code>__invoke()</code>

**Answer: C.** PHP calls <code>__callStatic()</code> for undefined or inaccessible static method calls.

### 7. What operation invokes <code>__toString()</code>?

- A. Using the object where a string is expected
- B. Cloning the object
- C. Calling an undefined property
- D. Using the object as an array

**Answer: A.** String conversion contexts such as <code>echo $object</code> invoke <code>__toString()</code>.

### 8. What operation invokes <code>__invoke()</code>?

- A. <code>$object-&gt;method()</code>
- B. <code>$object()</code>
- C. <code>clone $object</code>
- D. <code>isset($object)</code>

**Answer: B.** Calling an object as though it were a function invokes <code>__invoke()</code>.

### 9. Which magic method controls the information displayed by <code>var_dump()</code>?

- A. <code>__debugInfo()</code>
- B. <code>__varDump()</code>
- C. <code>__inspect()</code>
- D. <code>__print()</code>

**Answer: A.** If defined, <code>__debugInfo()</code> returns the data that <code>var_dump()</code> displays for the object.

### 10. Which method pair is preferred for custom serialization in modern PHP?

- A. <code>__store()</code> and <code>__restore()</code>
- B. <code>__serialize()</code> and <code>__unserialize()</code>
- C. <code>__sleep()</code> and <code>__clone()</code>
- D. <code>__export()</code> and <code>__import()</code>

**Answer: B.** The <code>__serialize()</code> and <code>__unserialize()</code> pair provides a controlled representation and restoration path.

### 11. In which PHP version were <code>__serialize()</code> and <code>__unserialize()</code> introduced?

- A. PHP 7.2
- B. PHP 7.3
- C. PHP 7.4
- D. PHP 8.0

**Answer: C.** These serialization hooks were introduced in PHP 7.4.

### 12. What happened to <code>__sleep()</code> and <code>__wakeup()</code> in PHP 8.5?

- A. They became required
- B. They were removed
- C. They were soft-deprecated
- D. They became static methods

**Answer: C.** PHP 8.5 soft-deprecated these legacy serialization methods. New code should use the newer pair.

### 13. Which visibility should most magic methods use?

- A. Private
- B. Protected
- C. Public
- D. Static

**Answer: C.** Most magic methods must be public. The common exceptions are <code>__construct()</code>, <code>__destruct()</code>, and <code>__clone()</code>.

### 14. Which three magic methods are exceptions to the general public-visibility requirement?

- A. <code>__get()</code>, <code>__set()</code>, <code>__call()</code>
- B. <code>__construct()</code>, <code>__destruct()</code>, <code>__clone()</code>
- C. <code>__toString()</code>, <code>__invoke()</code>, <code>__debugInfo()</code>
- D. <code>__serialize()</code>, <code>__unserialize()</code>, <code>__set_state()</code>

**Answer: B.** These three may use non-public visibility when the design requires it.

### 15. What return type must <code>__toString()</code> declare?

- A. <code>mixed</code>
- B. <code>string</code>
- C. <code>object</code>
- D. No return type is permitted

**Answer: B.** <code>__toString()</code> must return a string.

### 16. As of PHP 8.0, what interface does a class with <code>__toString()</code> implicitly implement?

- A. <code>Stringable</code>
- B. <code>Serializable</code>
- C. <code>StringObject</code>
- D. <code>Iterator</code>

**Answer: A.** PHP 8.0 made classes with <code>__toString()</code> implicitly implement <code>Stringable</code>.

### 17. Which method is called for classes exported by <code>var_export()</code> when the generated code is evaluated?

- A. <code>__set_state()</code>
- B. <code>__export()</code>
- C. <code>__unserialize()</code>
- D. <code>__restore()</code>

**Answer: A.** <code>__set_state()</code> receives the exported properties and returns a reconstructed object.

### 18. As of PHP 8.5, what is the status of returning null from <code>__debugInfo()</code>?

- A. Required
- B. Deprecated
- C. Converted automatically to an empty array
- D. A syntax error

**Answer: B.** Returning null from <code>__debugInfo()</code> is deprecated as of PHP 8.5. Return an array instead.

### 19. Why should code avoid unserializing untrusted input?

- A. It always converts every value to a string
- B. Deserialization can instantiate objects and trigger their behavior
- C. It cannot restore arrays
- D. It always deletes the input

**Answer: B.** Deserialization may instantiate classes and invoke hooks. Use safer formats such as JSON for external input and validate the data.

### 20. What happens if a magic method uses an incompatible signature?

- A. PHP silently changes the signature
- B. PHP may emit an error or fatal error
- C. The method becomes ordinary
- D. PHP makes it private

**Answer: B.** Magic method signatures are part of PHP's contract. Match the documented parameter and return types.

## Short-answer questions

### 21. Why should ordinary custom methods avoid names beginning with double underscores?

**Answer:** Names beginning with <code>__</code> are reserved for PHP's magic behavior.

**Explanation:** Use descriptive ordinary names such as <code>formatForDisplay()</code> instead of inventing another double-underscore method.

### 22. What is the difference between <code>__isset()</code> and <code>__get()</code>?

**Answer:** <code>__isset()</code> handles an <code>isset()</code> or <code>empty()</code> check for an undefined or inaccessible property; <code>__get()</code> handles reading the property's value.

**Explanation:** They represent different operations and have different return contracts.

### 23. What does <code>__unset()</code> handle?

**Answer:** It handles <code>unset()</code> on an undefined or inaccessible property.

**Explanation:** The class decides whether the requested property can be removed and what removal means for its internal state.

### 24. When might <code>__call()</code> be useful?

**Answer:** For a deliberate dynamic API, a proxy, or a builder that routes selected method names.

**Explanation:** Unsupported calls should fail clearly. Otherwise, magic handling can hide spelling mistakes and make code harder to discover.

### 25. What should a class consider before implementing <code>__toString()</code>?

**Answer:** It should choose one stable textual representation that does not expose secrets or trigger surprising side effects.

**Explanation:** String conversion may happen implicitly in output and logging contexts, so the result should be safe and predictable.

### 26. What is a useful case for <code>__invoke()</code>?

**Answer:** An object that represents a callable operation configured with stored state.

**Explanation:** For example, a prefixing or validation object can be passed to an API that accepts a callable.

### 27. What is a safe use of <code>__debugInfo()</code>?

**Answer:** Return a concise set of useful diagnostic fields while redacting secrets.

**Explanation:** The hook controls <code>var_dump()</code> output, so it can prevent passwords or tokens from appearing in debug dumps.

### 28. What should <code>__serialize()</code> return?

**Answer:** An array representing the state that should be stored.

**Explanation:** The array need not mirror every object property. It should contain only the data needed to safely reconstruct the object.

### 29. What should <code>__unserialize()</code> do with incoming data?

**Answer:** Validate its keys and value types, then restore valid object state.

**Explanation:** The hook receives data without normal constructor initialization. Invalid or incomplete data should be rejected.

### 30. When should <code>__set_state()</code> be used?

**Answer:** When a class needs to reconstruct instances from the PHP code representation produced by <code>var_export()</code>.

**Explanation:** It is not a general-purpose replacement for JSON serialization. Never evaluate exported PHP code from an untrusted source.

## Code reading and application

### 31. What does this print?

~~~php
class Label
{
    public function __construct(private string $value)
    {
    }

    public function __toString(): string
    {
        return $this->value;
    }
}

echo new Label('Ready');
~~~

**Answer: <code>Ready</code>.**

**Explanation:** Echoing the object in a string context invokes <code>__toString()</code>.

### 32. What does this print?

~~~php
class Prefixer
{
    public function __construct(private string $prefix)
    {
    }

    public function __invoke(string $value): string
    {
        return $this->prefix . $value;
    }
}

$prefix = new Prefixer('PHP: ');
echo $prefix('Objects');
~~~

**Answer: <code>PHP: Objects</code>.**

**Explanation:** The object call invokes <code>__invoke()</code> with <code>Objects</code> as its argument.

### 33. Which magic method runs here?

~~~php
class Box
{
    public function __get(string $name): mixed
    {
        return 'missing: ' . $name;
    }
}

$box = new Box();
echo $box->color;
~~~

**Answer:** <code>__get()</code>.

**Explanation:** <code>color</code> is undefined, so reading it invokes the property overloading hook.

### 34. Does <code>__get()</code> run for every read of a declared public property?

**Answer:** No.

**Explanation:** Property overloading handles undefined or inaccessible properties. A declared accessible public property is read normally.

### 35. What does this print?

~~~php
class Formatter
{
    public function __call(string $name, array $arguments): mixed
    {
        return strtoupper($name) . ':' . $arguments[0];
    }
}

$formatter = new Formatter();
echo $formatter->format('php');
~~~

**Answer: <code>FORMAT:php</code>.**

**Explanation:** Since <code>format()</code> is not declared, PHP invokes <code>__call()</code> with the method name and its argument list.

### 36. What issue is present in this declaration?

~~~php
class Report
{
    protected function __toString(): string
    {
        return 'Report';
    }
}
~~~

**Answer:** <code>__toString()</code> is not public.

**Explanation:** Most magic methods must be public. <code>__toString()</code> is not one of the visibility exceptions.

### 37. What should this class return from <code>__debugInfo()</code>?

~~~php
class Token
{
    public function __construct(
        private string $value,
    ) {
    }

    public function __debugInfo(): array
    {
        return [
            'value' => '[redacted]',
        ];
    }
}
~~~

**Answer:** It returns an array containing the redacted value.

**Explanation:** When <code>var_dump()</code> displays a Token, it uses this array instead of exposing the private token value.

### 38. Which method pair should replace this legacy pair in new code?

~~~php
public function __sleep(): array
{
    return ['name'];
}

public function __wakeup(): void
{
    // Restore state.
}
~~~

**Answer:** Use <code>__serialize(): array</code> and <code>__unserialize(array $data): void</code>.

**Explanation:** The old pair is soft-deprecated as of PHP 8.5. The newer pair provides an explicit data representation and restoration hook.

### 39. Why is this code unsafe if <code>$input</code> comes from a user?

~~~php
$object = unserialize($input);
~~~

**Answer:** It unserializes untrusted data and may instantiate classes or trigger object hooks.

**Explanation:** Avoid unserializing user-controlled values. Parse a safer format such as JSON and validate the resulting data.

### 40. What is one reason to avoid accepting every name in <code>__call()</code>?

**Answer:** It can hide misspelled method calls and make the object's API difficult to discover.

**Explanation:** Handle a deliberate set of supported operations and throw <code>BadMethodCallException</code> for everything else.

## Key takeaways

- PHP calls magic methods for specific operations on objects.
- Property and method overloading only handle undefined or inaccessible members.
- Most magic methods must be public and use their documented signatures.
- <code>__toString()</code> converts an object to text; <code>__invoke()</code> makes it callable.
- Use <code>__debugInfo()</code> to keep sensitive values out of debug output.
- Prefer <code>__serialize()</code> and <code>__unserialize()</code> over the legacy pair.
- Never unserialize data from an untrusted source.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [075 - Property hooks](../lessons/075-property-hooks.md) | [Lesson](../lessons/076-magic-methods.md) | [Example](../examples/076-magic-methods.md) | [077 - Cloning and comparing objects](../lessons/077-cloning-and-comparing-objects.md) |
