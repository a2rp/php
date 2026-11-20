# 055 - Encoding and Decoding JSON: Q&A

Review JSON values, PHP array and object shapes, UTF-8, encoding flags, decoding errors, numeric precision, and input validation. Try each question before reading its answer. See the [chapter lesson](../lessons/055-encoding-and-decoding-json.md).

## Multiple-choice questions

### 1. Which character encoding must PHP JSON strings use?

A. UTF-8
B. UTF-16 only
C. ASCII only
D. Any encoding; PHP converts it automatically

**Answer:** A. UTF-8.

**Explanation:** **json_encode()** and **json_decode()** expect UTF-8 strings. They do not automatically convert arbitrary source encodings.

### 2. What does **json_encode()** return when encoding succeeds?

A. A PHP object
B. A JSON text string
C. A file handle
D. A boolean value only

**Answer:** B. A JSON text string.

**Explanation:** The caller can send, store, or further process that string as JSON data.

### 3. What does **json_decode()** return for a JSON object by default?

A. An associative array
B. A **stdClass** object
C. A resource
D. A serialized PHP string

**Answer:** B. A **stdClass** object.

**Explanation:** Pass **true** as the second argument when JSON objects should become associative arrays.

### 4. Which option makes **json_encode()** throw **JsonException** on JSON encoding errors?

A. **JSON_PRETTY_PRINT**
B. **JSON_THROW_ON_ERROR**
C. **JSON_NUMERIC_CHECK**
D. **JSON_FORCE_OBJECT**

**Answer:** B. **JSON_THROW_ON_ERROR**

**Explanation:** The flag is available in PHP 7.3 and later and applies to both encoding and decoding.

### 5. What does **JSON_PRETTY_PRINT** do?

A. Adds whitespace to make JSON easier to read
B. Validates the application schema
C. Converts numbers to strings
D. Replaces invalid UTF-8 with a marker

**Answer:** A. Adds whitespace to make JSON easier to read.

**Explanation:** Pretty printing changes formatting, not the represented data.

### 6. What happens to a sequential PHP array with keys starting at zero when encoded?

A. It becomes a JSON list
B. It becomes a JSON string
C. It becomes a PHP object
D. Encoding always fails

**Answer:** A. It becomes a JSON list.

**Explanation:** A PHP array with string keys or non-sequential numeric keys is encoded as a JSON object.

### 7. What can happen when a middle key is removed from a numeric PHP array before encoding?

A. The array may be encoded as a JSON object with numeric-looking keys
B. PHP always renumbers it automatically
C. The JSON encoder inserts null for every removed element
D. The result is always invalid JSON

**Answer:** A. The array may be encoded as a JSON object with numeric-looking keys.

**Explanation:** Use **array_values()** if the intended JSON shape is a compact list.

### 8. What does an empty PHP array encode as by default?

A. **{}**
B. **[]**
C. **null**
D. **false**

**Answer:** B. **[]**

**Explanation:** Cast an empty array to an object or intentionally construct an object if the contract requires **{}**.

### 9. What does **JSON_NUMERIC_CHECK** do?

A. Converts numeric-looking strings to JSON numbers
B. Checks whether the JSON syntax is valid
C. Forces every PHP array to remain a list
D. Preserves a float's zero fraction

**Answer:** A. Converts numeric-looking strings to JSON numbers.

**Explanation:** This can change strings such as identifiers or values with leading zeroes, so avoid it for general data.

### 10. Which flag decodes oversized integer tokens as strings?

A. **JSON_FORCE_OBJECT**
B. **JSON_BIGINT_AS_STRING**
C. **JSON_UNESCAPED_UNICODE**
D. **JSON_PRESERVE_ZERO_FRACTION**

**Answer:** B. **JSON_BIGINT_AS_STRING**

**Explanation:** It helps avoid converting integers outside the platform's integer range into floating-point values.

### 11. What does **JSON_UNESCAPED_UNICODE** change?

A. It leaves Unicode characters visible in the JSON text instead of escaping them
B. It converts data from a legacy encoding to UTF-8
C. It disables JSON string escaping entirely
D. It converts all strings to objects

**Answer:** A. It leaves Unicode characters visible in the JSON text instead of escaping them.

**Explanation:** The represented text is the same; only the JSON string representation changes.

### 12. What does **JSON_INVALID_UTF8_SUBSTITUTE** do?

A. Replaces invalid UTF-8 sequences with the Unicode replacement character
B. Rejects every string containing a Unicode character
C. Converts every string to UTF-16
D. Removes every numeric value

**Answer:** A. Replaces invalid UTF-8 sequences with the Unicode replacement character.

**Explanation:** Substitution is an intentional recovery policy. Rejecting or repairing data at the input boundary may be more appropriate when exact data preservation matters.

### 13. When was **JSON_THROW_ON_ERROR** added?

A. PHP 5.2
B. PHP 7.3
C. PHP 8.3
D. PHP 8.5

**Answer:** B. PHP 7.3.

**Explanation:** The flag is available in PHP 7.3 and later.

### 14. In which PHP version was **json_validate()** introduced?

A. PHP 7.0
B. PHP 7.3
C. PHP 8.0
D. PHP 8.3

**Answer:** D. PHP 8.3.

**Explanation:** On older PHP versions, decoding with error handling can validate the syntax while also creating the PHP value.

### 15. What does **json_validate()** check?

A. JSON syntax
B. Whether decoded data matches the application's schema
C. Whether values are authorized
D. Whether an HTTP request succeeded

**Answer:** A. JSON syntax.

**Explanation:** It does not validate application-specific structure, types, ranges, or business rules.

## Short-answer questions

### 16. Why is **JSON_THROW_ON_ERROR** useful?

**Answer:** It converts JSON encoding or decoding errors into a **JsonException** that code can catch explicitly.

**Explanation:** This avoids confusing a failed decode with a valid JSON **null** result or continuing with an unusable payload.

### 17. What is the difference between a JSON object and a JSON array?

**Answer:** An object is a collection of string-named properties; an array is an ordered list of values.

**Explanation:** The API contract should state which shape is expected at each part of a payload.

### 18. How can a sparse numeric PHP array be encoded as a JSON list?

**Answer:** Reindex it with **array_values()** before encoding.

**Explanation:** A sequence beginning at key zero encodes as a JSON array. Gaps can cause the keys to be represented as JSON object properties instead.

### 19. How can PHP encode an empty object rather than an empty array?

**Answer:** Cast an empty array to an object, for example **(object) []**, or construct an object intentionally.

**Explanation:** An empty array becomes **[]** by default, while an empty object becomes **{}**.

### 20. What does **json_decode($json, true)** change?

**Answer:** It decodes JSON objects as associative PHP arrays instead of **stdClass** objects.

**Explanation:** JSON arrays remain PHP arrays either way, so choose the representation with the object/list distinction in mind.

### 21. Why can a direct check for **null** be ambiguous after **json_decode()**?

**Answer:** JSON **null** is a valid value, and a failed decode can also return **null** when exception throwing is not enabled.

**Explanation:** Use **JSON_THROW_ON_ERROR** or check the JSON error state immediately after decoding.

### 22. Name two reasons **json_encode()** can fail.

**Answer:** Invalid UTF-8 in a string, a resource value, a recursive reference, or a non-finite float such as **NAN** or **INF**.

**Explanation:** With **JSON_THROW_ON_ERROR**, such encoding errors raise **JsonException**.

### 23. What does **JSON_BIGINT_AS_STRING** help preserve?

**Answer:** Large JSON integer tokens that cannot be represented as a PHP integer on the current platform.

**Explanation:** Without it, an oversized integer may be decoded as a floating-point value and lose exact integer precision.

### 24. Why should **JSON_NUMERIC_CHECK** be used carefully?

**Answer:** It can change numeric-looking strings into numbers, altering identifiers, phone numbers, or strings with leading zeroes.

**Explanation:** JSON values should follow the data contract, not a global guess that every numeric-looking string is a number.

### 25. What does the depth argument to **json_decode()** limit?

**Answer:** The maximum nesting depth to process.

**Explanation:** A deliberate limit helps reject excessively nested input and must be greater than zero.

### 26. Why should a class implement **JsonSerializable**?

**Answer:** To control the value or structure returned when that object is encoded as JSON.

**Explanation:** This can expose a deliberate representation instead of relying on the object's public properties by default.

### 27. What should legacy code do if it does not use **JSON_THROW_ON_ERROR**?

**Answer:** Check **json_last_error()** and **json_last_error_msg()** immediately after the JSON operation.

**Explanation:** Another JSON call can replace the global last-error state.

### 28. Should **json_validate()** normally be called immediately before decoding the same payload?

**Answer:** No.

**Explanation:** **json_decode()** already parses and validates the JSON. Calling both in sequence parses the text twice without adding schema validation.

### 29. Does valid JSON guarantee that a request payload is acceptable to an application?

**Answer:** No. The decoded value must still be checked against the expected root type, fields, types, ranges, and business rules.

**Explanation:** JSON parsing checks syntax, not the application's schema or authorization policy.

## True or false

### 30. **json_encode()** automatically converts a Windows-1252 string to UTF-8.

**Answer:** False.

**Explanation:** String data supplied to the JSON encoder must already be UTF-8.

### 31. **json_decode()** returns **stdClass** objects by default for JSON objects.

**Answer:** True.

**Explanation:** The second argument can be set to **true** to decode JSON objects as associative arrays.

### 32. A sparse numeric PHP array always encodes as a JSON array.

**Answer:** False.

**Explanation:** Non-sequential numeric keys can cause it to encode as a JSON object.

### 33. **JSON_THROW_ON_ERROR** causes a **JsonException** when an encoding or decoding error occurs.

**Answer:** True.

**Explanation:** This flag is available starting in PHP 7.3.

### 34. **json_validate()** verifies that a JSON object contains all required application fields.

**Answer:** False.

**Explanation:** It validates JSON syntax only. Validate the decoded structure separately.

### 35. **JSON_UNESCAPED_UNICODE** changes which Unicode characters the JSON value represents.

**Answer:** False.

**Explanation:** It changes whether characters appear literally or as Unicode escape sequences in the encoded text.

### 36. Calling **json_validate()** and then immediately decoding the same payload parses it only once.

**Answer:** False.

**Explanation:** Both functions parse the input. Use **json_validate()** when the decoded value is not needed.

### 37. **JSON_BIGINT_AS_STRING** is a flag for decoding JSON.

**Answer:** True.

**Explanation:** It preserves oversized integer tokens as strings instead of converting them to imprecise floating-point values.

## Predict the output

### 38. What does this print?

~~~php
<?php

$json = json_encode(
    ['name' => 'Mina', 'tags' => ['php', 'json']],
    JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR
);

echo $json;
~~~

**Answer:**

~~~text
{"name":"Mina","tags":["php","json"]}
~~~

**Explanation:** The string keys form a JSON object, and the sequential **tags** array becomes a JSON list.

### 39. What does this print?

~~~php
<?php

echo json_encode(['42'], JSON_NUMERIC_CHECK | JSON_THROW_ON_ERROR);
~~~

**Answer:**

~~~text
[42]
~~~

**Explanation:** **JSON_NUMERIC_CHECK** converts the numeric-looking PHP string to a JSON number. Without the flag, the result would contain the string **"42"**.

### 40. What does this print?

~~~php
<?php

echo json_encode([], JSON_THROW_ON_ERROR) . "\n";
echo json_encode((object) [], JSON_THROW_ON_ERROR);
~~~

**Answer:**

~~~text
[]
{}
~~~

**Explanation:** An empty PHP array encodes as a JSON list, while an empty object encodes as a JSON object.

### 41. What does this print?

~~~php
<?php

$value = json_decode('{"active":true}', true, 512, JSON_THROW_ON_ERROR);

echo $value['active'] ? 'active' : 'inactive';
~~~

**Answer:**

~~~text
active
~~~

**Explanation:** The second argument asks for associative arrays, and JSON **true** decodes to PHP **true**.

### 42. What does this print?

~~~php
<?php

$data = json_decode('{"name":"Mina"}', false, 512, JSON_THROW_ON_ERROR);

echo get_class($data) . ':' . $data->name;
~~~

**Answer:**

~~~text
stdClass:Mina
~~~

**Explanation:** With **false** as the associative argument, a JSON object becomes a **stdClass** instance.

## Coding questions

### 43. Encode a PHP payload with readable Unicode and throw on errors.

**Answer:**

~~~php
<?php

$payload = [
    'city' => 'Montréal',
    'ready' => true,
];

try {
    $json = json_encode(
        $payload,
        JSON_PRETTY_PRINT
            | JSON_UNESCAPED_UNICODE
            | JSON_THROW_ON_ERROR
    );
} catch (JsonException $exception) {
    throw new RuntimeException('Could not encode the response.', 0, $exception);
}

echo $json;
~~~

**Explanation:** The flags control output formatting and Unicode presentation. **JSON_THROW_ON_ERROR** makes an invalid value or encoding error explicit.

### 44. Decode JSON as an object and reject a payload whose root is not an object.

**Answer:**

~~~php
<?php

try {
    $payload = json_decode($jsonText, false, 64, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    throw new RuntimeException('The input is not valid JSON.', 0, $exception);
}

if (!$payload instanceof stdClass) {
    throw new RuntimeException('Expected a JSON object at the root.');
}
~~~

**Explanation:** Parsing with **JSON_THROW_ON_ERROR** distinguishes invalid syntax from a valid JSON **null**. The root-type check enforces one part of the application's data contract.

### 45. On PHP 8.3 or later, check whether a string is JSON when the decoded value is not needed.

**Answer:**

~~~php
<?php

if (!json_validate($candidate)) {
    throw new RuntimeException('The input is not valid JSON.');
}

echo 'Valid JSON syntax.';
~~~

**Explanation:** **json_validate()** checks syntax without building a PHP value. If the application needs the decoded value, decode once with **JSON_THROW_ON_ERROR** instead of validating and then parsing the same string.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [054 - Streams, Wrappers, and Contexts](../lessons/054-streams-wrappers-and-contexts.md) | [Read lesson](../lessons/055-encoding-and-decoding-json.md) | [Open example](../examples/055-encoding-and-decoding-json.md) | [056 - Serialization and Safe Data Exchange](../lessons/056-serialization-and-safe-data-exchange.md) |
