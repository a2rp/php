# 056 - Serialization and Safe Data Exchange: Q&A

Review PHP serialization, object restoration, JSON interchange, and security rules for stored or received data. Try each question before reading its answer. See the [chapter lesson](../lessons/056-serialization-and-safe-data-exchange.md).

## Multiple-choice questions

### 1. What does serialize($value) return when it succeeds?

A. A PHP resource
B. A string representation of the value
C. A JSON object
D. A class definition

**Answer:** B. A string representation of the value.

**Explanation:** PHP serialize() encodes supported PHP values in a PHP-specific string format.

### 2. Which format is generally better for data exchanged by browsers, APIs, and different languages?

A. PHP serialization
B. var_export() output
C. JSON with a documented schema
D. print_r() output

**Answer:** C. JSON with a documented schema.

**Explanation:** JSON is language-neutral text. The receiver must still validate decoded data against its expected schema.

### 3. Why can restoring a serialized object be security-sensitive?

A. It always executes the text as PHP source
B. It may instantiate classes, autoload code, and invoke magic methods
C. It automatically grants database access
D. It disables type checks in PHP

**Answer:** B. It may instantiate classes, autoload code, and invoke magic methods.

**Explanation:** Object restoration can activate application behavior, so attacker-controlled serialized data must never be passed to unserialize().

### 4. Which input must never be passed to unserialize()?

A. An application-generated, authenticated payload under a controlled contract
B. Any attacker-controlled or otherwise untrusted serialized input
C. A fixed value in a local controlled exercise
D. A value just produced by trusted code

**Answer:** B. Any attacker-controlled or otherwise untrusted serialized input.

**Explanation:** Requests, cookies, uploads, and modifiable storage can be attacker-controlled. Use an appropriate safe interchange format for external input.

### 5. What does allowed_classes => false do?

A. Makes every malformed string harmless
B. Restricts normal class restoration but does not make arbitrary input safe
C. Encrypts serialized data
D. Validates the business schema

**Answer:** B. It restricts normal class restoration but does not make arbitrary input safe.

**Explanation:** Class restrictions may be defense in depth for controlled legacy data; they are not a safe parser boundary.

### 6. What is max_depth for in unserialize()?

A. Authenticating the payload
B. Limiting how deeply nested the decoded structure may be
C. Limiting character encoding
D. Preventing every object from being created

**Answer:** B. Limiting how deeply nested the decoded structure may be.

**Explanation:** It can limit one form of resource use, but does not authenticate the payload or make it safe.

### 7. Which statement about a serialized PHP string is correct?

A. It is always printable UTF-8
B. It can contain binary bytes, including NUL bytes
C. It is JSON
D. It can be safely truncated at any byte

**Answer:** B. It can contain binary bytes, including NUL bytes.

**Explanation:** Store serialized output in binary-safe storage and preserve all bytes.

### 8. What does a serialized object contain about its class?

A. The full PHP source code
B. Class identity and serialized state, not the implementation
C. Compiled copies of its methods
D. A database connection

**Answer:** B. Class identity and serialized state, not the implementation.

**Explanation:** The relevant class definition generally must be available when PHP restores the object.

### 9. Which magic method returns the state to serialize for a class?

A. __toString()
B. __serialize()
C. __invoke()
D. __clone()

**Answer:** B. __serialize().

**Explanation:** It returns an array describing the object's persisted state.

### 10. What is the return type of the modern __unserialize() hook?

A. string
B. bool
C. void
D. array

**Answer:** C. void.

**Explanation:** It receives an array of data and initializes the object after validating it.

### 11. Since which PHP version are __serialize() and __unserialize() available?

A. PHP 5.0
B. PHP 7.0
C. PHP 7.4
D. PHP 8.5

**Answer:** C. PHP 7.4.

**Explanation:** These are the modern object serialization hooks.

### 12. What happens if a class defines both __serialize() and the older __sleep()?

A. Both always run in sequence
B. __serialize() takes precedence
C. Serialization fails
D. PHP picks randomly

**Answer:** B. __serialize() takes precedence.

**Explanation:** The modern hooks take precedence over the older hooks.

### 13. As of PHP 8.5, how are __sleep() and __wakeup() classified?

A. Required for every class
B. Soft-deprecated
C. Removed
D. Encrypted

**Answer:** B. Soft-deprecated.

**Explanation:** New code should use __serialize() and __unserialize().

### 14. What does an HMAC provide for a stored payload?

A. Confidentiality and compression
B. Integrity and authenticity for parties protecting the key
C. JSON schema validation
D. A guarantee the payload is harmless

**Answer:** B. Integrity and authenticity for parties protecting the key.

**Explanation:** HMAC does not encrypt data and is useful only if the key is protected.

### 15. Which function is designed for timing-safe comparison of an expected MAC and supplied MAC?

A. strcmp()
B. hash_equals()
C. array_diff()
D. json_validate()

**Answer:** B. hash_equals().

**Explanation:** Verify the expected HMAC before parsing an authenticated legacy payload.

## Short-answer questions

### 16. Define serialization and deserialization.

**Answer:** Serialization converts an in-memory value into a representation that can be stored or transmitted. Deserialization parses that representation and reconstructs a value.

**Explanation:** PHP serialize()/unserialize() are one PHP-specific pair; JSON has separate encoding and decoding functions.

### 17. Why is PHP serialization a poor public API format?

**Answer:** It is PHP-specific, can encode PHP class identity and types, and may trigger object behavior when restored.

**Explanation:** Use a documented neutral format such as JSON across browsers, languages, and services, then validate its schema.

### 18. Does a serialized object contain its methods or PHP source?

**Answer:** No. It records class identity and object state, not the implementation.

**Explanation:** The receiving PHP process needs the class definition to reconstruct an instance.

### 19. Why should a database connection not be ordinary serialized object state?

**Answer:** It is live runtime state and cannot be meaningfully recreated from serialized bytes.

**Explanation:** Reopen the connection through the application's normal setup code.

### 20. Why store a serialized string without text assumptions?

**Answer:** It can contain binary bytes, including NUL bytes, and may not be printable UTF-8.

**Explanation:** Preserve the complete value in binary-safe storage; avoid text transformations and truncation.

### 21. What should __serialize() include?

**Answer:** Only the explicit stable state needed to restore the object, excluding secrets and runtime-only dependencies.

**Explanation:** An explicit state list is easier to review and less coupled to implementation details.

### 22. What should __unserialize() do with supplied state?

**Answer:** Validate required keys and value types, initialize valid state, and recreate or reset runtime dependencies separately.

**Explanation:** Successful parsing does not mean values satisfy application rules.

### 23. Why does allowed_classes => false not make attacker input safe?

**Answer:** It restricts class restoration but does not authenticate bytes or turn PHP serialization into a safe external format.

**Explanation:** Never pass untrusted data to unserialize().

### 24. What does max_depth help limit, and what does it not do?

**Answer:** It limits nesting depth and may reduce one kind of resource exhaustion. It does not authenticate data or make untrusted serialization safe.

**Explanation:** A parser limit is not a trust check.

### 25. What should happen before a controlled legacy serialized payload is parsed if storage may be modified?

**Answer:** Recalculate its HMAC with a protected key and verify the supplied MAC using hash_equals().

**Explanation:** Reject a mismatch before parsing. Protect the key from the party who can modify the payload.

### 26. Does HMAC encrypt a payload?

**Answer:** No. It authenticates data and detects unauthorized modification; it does not hide the contents.

**Explanation:** Confidentiality requires an appropriate encryption design and separate key management.

### 27. Why is false an awkward unserialize() failure signal?

**Answer:** A valid serialized boolean false also restores to false, while malformed input can return false and emit a warning.

**Explanation:** Prefer JSON decoding with JSON_THROW_ON_ERROR for new data protocols.

### 28. What does a version field provide in a durable JSON envelope?

**Answer:** It lets the receiver support, reject, or migrate known schema versions deliberately.

**Explanation:** A versioned contract prevents guessing how changed or old records should be interpreted.

## True or false

### 29. JSON parsing automatically validates the application's schema.

**Answer:** False.

**Explanation:** Decoding checks JSON syntax. The application must validate fields, types, allowed values, and relationships.

### 30. A serialized PHP object contains copies of its methods.

**Answer:** False.

**Explanation:** It stores class identity and state; the class definition provides behavior.

### 31. It is safe to call unserialize() on a request parameter if allowed_classes is false.

**Answer:** False.

**Explanation:** The option does not make attacker-controlled serialized data safe. Use JSON or another suitable external format.

### 32. max_depth proves that a serialized value came from a trusted producer.

**Answer:** False.

**Explanation:** It limits nesting depth, not provenance or integrity.

### 33. Modern serialization hooks may execute during object restoration.

**Answer:** True.

**Explanation:** __unserialize() participates in restoration, so object deserialization has behavioral and security implications.

### 34. PHP serialization is language-neutral.

**Answer:** False.

**Explanation:** It represents PHP values and classes in a PHP-specific format.

### 35. HMAC and encryption have the same purpose.

**Answer:** False.

**Explanation:** HMAC protects integrity and authenticity; encryption protects confidentiality.

### 36. A serialized string may contain non-printable bytes.

**Answer:** True.

**Explanation:** Serialized output is binary-safe data and may contain NUL bytes.

## Predict the result or behavior

### 37. What does this print?

~~~php
<?php

$value = ['ready' => true, 'count' => 3];
$copy = unserialize(serialize($value));
var_dump($copy === $value);
~~~

**Answer:**

~~~text
bool(true)
~~~

**Explanation:** The supported array values return with the same keys and values.

### 38. A class's __serialize() returns ['id' => 7]. Is a private $password automatically included?

**Answer:** No, unless the method explicitly includes it.

**Explanation:** The hook controls which state is represented, which helps omit secrets and transient fields.

### 39. What is the decoded root type?

~~~php
<?php

$value = json_decode('{"name":"Asha"}', true, 64, JSON_THROW_ON_ERROR);
echo gettype($value);
~~~

**Answer:**

~~~text
array
~~~

**Explanation:** The true argument decodes JSON objects as associative arrays.

### 40. If a serialized Sample object is restored with allowed_classes false, is it a Sample instance?

**Answer:** No. PHP represents the object payload as __PHP_Incomplete_Class.

**Explanation:** The option restricts class restoration; it does not make arbitrary serialized input safe.

### 41. Does this prove that parsing failed?

~~~php
<?php

$result = unserialize(serialize(false));
var_dump($result === false);
~~~

**Answer:** No. It prints bool(true), but false is the valid value that was serialized.

**Explanation:** Failure can also return false and issue a warning, so the result alone is ambiguous.

### 42. What if one payload byte changes but the original HMAC is supplied?

**Answer:** Verification should fail and the application should reject the payload before parsing.

**Explanation:** The expected MAC covers the exact payload bytes and changes when those bytes change.

## Coding questions

### 43. Write __serialize() to persist $id and $label while omitting runtime-only $connection.

**Answer:**

~~~php
public function __serialize(): array
{
    return [
        'id' => $this->id,
        'label' => $this->label,
    ];
}
~~~

**Explanation:** Include only stable state needed for restoration. Re-establish the connection separately.

### 44. Decode JSON with exceptions and reject it unless name is a string.

**Answer:**

~~~php
try {
    $data = json_decode($jsonText, true, 64, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    throw new RuntimeException('Invalid JSON.', 0, $exception);
}

if (!is_array($data) || !isset($data['name']) || !is_string($data['name'])) {
    throw new RuntimeException('Expected a string name field.');
}
~~~

**Explanation:** Exceptions handle malformed JSON; the explicit check validates one part of the schema.

### 45. Show the safe order for authenticating controlled legacy serialized data before parsing.

**Answer:**

~~~php
$expectedMac = hash_hmac('sha256', $serialized, $secretKey);

if (!hash_equals($expectedMac, $providedMac)) {
    throw new RuntimeException('Payload authentication failed.');
}

$value = unserialize($serialized, [
    'allowed_classes' => false,
    'max_depth' => 64,
]);
~~~

**Explanation:** Verify the exact bytes first. This is only for a controlled format with a protected key; it does not make arbitrary user input safe.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [055 - Encoding and decoding JSON](../lessons/055-encoding-and-decoding-json.md) | [Read lesson](../lessons/056-serialization-and-safe-data-exchange.md) | [Open example](../examples/056-serialization-and-safe-data-exchange.md) | [057 - Errors, exceptions, and Throwable](../lessons/057-errors-exceptions-and-throwable.md) |
