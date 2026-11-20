# 055 - Encoding and Decoding JSON

JSON is a text format for exchanging structured data. PHP can encode arrays and objects as JSON and decode JSON into PHP values. Reliable JSON handling means using UTF-8, choosing a clear object/list shape, handling errors, and validating the decoded data for the application's needs.

## What you will learn

- Map common PHP values to JSON values.
- Encode data with **json_encode()** and decode it with **json_decode()**.
- Choose between PHP arrays and objects when decoding JSON objects.
- Handle errors with **JSON_THROW_ON_ERROR**.
- Understand lists, objects, numeric values, nesting depth, and UTF-8.
- Use encoding flags deliberately.
- Validate input structure after decoding.
- Know when **json_validate()** is useful.

## The JSON data model

JSON has these value types:

| JSON type | Example |
| --- | --- |
| Object | **{"name":"Mina"}** |
| Array | **["PHP","JSON"]** |
| String | **"hello"** |
| Number | **42** or **3.5** |
| Boolean | **true** or **false** |
| Null | **null** |

JSON objects have string property names. JSON arrays are ordered lists. A JSON document may have any JSON value at its root, but an API contract may require a specific root type such as an object.

## Encode PHP values

**json_encode()** converts a PHP value into a JSON string:

~~~php
<?php

$payload = [
    'course' => 'PHP',
    'active' => true,
    'lessons' => ['Files', 'Streams', 'JSON'],
    'metadata' => (object) [],
];

$json = json_encode(
    $payload,
    JSON_PRETTY_PRINT
        | JSON_UNESCAPED_UNICODE
        | JSON_UNESCAPED_SLASHES
        | JSON_THROW_ON_ERROR
);

echo $json;
~~~

The result is JSON text, not a PHP array. Strings supplied to the encoder must be valid UTF-8. Resources cannot be encoded. Unsupported values, recursive references, invalid UTF-8, or non-finite floats such as **NAN** and **INF** can cause an error.

When an object is encoded, public properties are included by default. Implement **JsonSerializable** when a class needs to define its JSON representation. Be deliberate about what data is exposed; encoding an application object directly can unintentionally publish fields.

## Lists and objects in PHP arrays

PHP arrays can represent both JSON lists and JSON objects. A sequential numeric array beginning at key zero is encoded as a JSON array:

~~~php
<?php

echo json_encode(['red', 'green', 'blue'], JSON_THROW_ON_ERROR);
~~~

A PHP array with string keys, or with non-sequential numeric keys, is encoded as a JSON object. Removing an item can leave a gap:

~~~php
<?php

$colors = ['red', 'green', 'blue'];
unset($colors[1]);

echo json_encode($colors, JSON_THROW_ON_ERROR);
~~~

This produces an object with the remaining keys rather than a compact list. Use **array_values()** when the intended JSON shape is a list:

~~~php
<?php

echo json_encode(array_values($colors), JSON_THROW_ON_ERROR);
~~~

An empty PHP array encodes as **[]**. If the contract requires an empty JSON object **{}**, cast the empty array to an object or construct an object intentionally. **JSON_FORCE_OBJECT** can convert numeric arrays throughout a nested structure, so avoid using it as a blanket fix when some values should remain lists.

## Decode JSON

**json_decode()** parses UTF-8 JSON and returns a matching PHP value. By default, JSON objects become **stdClass** objects and JSON arrays become PHP arrays:

~~~php
<?php

$json = '{"name":"Mina","active":true,"tags":["php","json"]}';

$data = json_decode($json, false, 512, JSON_THROW_ON_ERROR);

echo $data->name . "\n";
echo $data->tags[0] . "\n";
~~~

Pass **true** as the second argument to decode JSON objects as associative arrays:

~~~php
<?php

$data = json_decode($json, true, 512, JSON_THROW_ON_ERROR);

echo $data['name'] . "\n";
echo $data['tags'][0] . "\n";
~~~

Choose one representation consistently for the code that consumes the result. Associative arrays can blur the distinction between a JSON object and a JSON list, especially when an object has numeric-looking keys. Use objects when preserving that distinction matters.

JSON **null** decodes to PHP **null**. Without exception-based error handling, **null** can also be returned when decoding fails, so a direct null check alone is ambiguous.

## Handle errors

For PHP 7.3 and later, **JSON_THROW_ON_ERROR** makes encoding and decoding failures throw **JsonException**:

~~~php
<?php

try {
    $data = json_decode($requestBody, true, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    // Reject or report malformed JSON according to the application policy.
    throw new RuntimeException('The request body is not valid JSON.', 0, $exception);
}
~~~

It is usually clearer to handle exceptions than to continue with a failed or incomplete result. For legacy code that does not use **JSON_THROW_ON_ERROR**, check **json_last_error()** and **json_last_error_msg()** immediately after the JSON operation. Another JSON call can replace the last-error state.

Set a reasonable maximum nesting depth when handling data from outside the application. The depth argument to **json_decode()** and **json_encode()** limits recursive processing; it must be greater than zero.

Avoid **JSON_PARTIAL_OUTPUT_ON_ERROR** when a complete, trustworthy document is required. It substitutes values for some encoding errors instead of failing, and it takes precedence over **JSON_THROW_ON_ERROR**.

## Choose encoding flags deliberately

Flags are bitmasks and can be combined with the bitwise OR operator **|**.

| Flag | Effect |
| --- | --- |
| **JSON_PRETTY_PRINT** | Adds whitespace for readable output |
| **JSON_UNESCAPED_UNICODE** | Leaves Unicode characters visible instead of escaping them as sequences |
| **JSON_UNESCAPED_SLASHES** | Leaves forward slashes unescaped |
| **JSON_THROW_ON_ERROR** | Throws **JsonException** on a JSON encoding or decoding error |
| **JSON_INVALID_UTF8_SUBSTITUTE** | Replaces invalid UTF-8 sequences during encoding or decoding |
| **JSON_INVALID_UTF8_IGNORE** | Drops invalid UTF-8 sequences |
| **JSON_PRESERVE_ZERO_FRACTION** | Keeps a float such as **12.0** in a floating-point JSON representation |
| **JSON_BIGINT_AS_STRING** | Decodes integers too large for the platform integer as strings |

Pretty printing is useful for human-readable files and debugging, but usually unnecessary for compact API responses. **JSON_UNESCAPED_UNICODE** changes the visual representation of the JSON text, not the Unicode characters represented by it.

Use invalid-UTF-8 substitution or ignoring only as an intentional recovery policy. Substitution preserves a replacement marker; ignoring discards invalid bytes. Prefer fixing or rejecting data at the boundary when data integrity matters.

Avoid **JSON_NUMERIC_CHECK** for general application data. It converts numeric-looking strings to JSON numbers, which can change identifiers, telephone numbers, leading zeroes, or values whose contract requires a string.

## Preserve numeric meaning

PHP integers have platform-dependent limits, and floating-point numbers cannot represent every decimal value exactly. JSON numbers do not carry a separate decimal or integer precision guarantee for every consumer.

Use **JSON_BIGINT_AS_STRING** when decoding large integer tokens that must not be converted to a PHP float. For identifiers that must remain exact across systems, define the contract as a string. For exact decimal quantities, use a representation designed for the application's precision needs instead of assuming binary floating point is exact.

**JSON_PRESERVE_ZERO_FRACTION** keeps a float such as **12.0** encoded with a decimal fraction. It affects the JSON text representation; it does not make floating-point arithmetic exact.

## Decode first, then validate the data shape

Valid JSON is not automatically valid application input. After decoding, check the root type, required properties, value types, lengths, and allowed values:

~~~php
<?php

try {
    $payload = json_decode($requestBody, false, 64, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    throw new RuntimeException('Malformed JSON input.', 0, $exception);
}

if (!$payload instanceof stdClass) {
    throw new RuntimeException('The JSON root must be an object.');
}

if (!isset($payload->name) || !is_string($payload->name)) {
    throw new RuntimeException('A string name is required.');
}

$name = trim($payload->name);

if ($name === '' || strlen($name) > 120) {
    throw new RuntimeException('The name is outside the allowed range.');
}
~~~

Production validation should match the actual API contract. Do not trust decoded strings, numbers, or nested values merely because the JSON syntax is valid.

## Validate syntax without decoding

**json_validate()** is available in PHP 8.3 and later. It checks whether a string contains syntactically valid JSON without constructing the decoded PHP value:

~~~php
<?php

if (!json_validate($candidate)) {
    echo 'Invalid JSON';
}
~~~

Use it when you only need to know whether a string is valid JSON. Do not call **json_validate()** immediately before **json_decode()** for the same payload; decoding already parses and validates the text.

Syntax validation does not check whether the JSON matches an application schema or whether its values are acceptable.

## Send a JSON HTTP response

For an HTTP endpoint, encode the response, set the content type, and then send the JSON text:

~~~php
<?php

try {
    $body = json_encode(
        ['status' => 'ok', 'message' => 'Ready'],
        JSON_UNESCAPED_UNICODE | JSON_THROW_ON_ERROR
    );
} catch (JsonException $exception) {
    http_response_code(500);
    exit;
}

header('Content-Type: application/json; charset=utf-8');
echo $body;
~~~

Do not print warnings, debugging output, or HTML before the JSON response. The **Content-Type** header tells the client to interpret the body as JSON. A JSON string is not automatically safe to insert into HTML or JavaScript source; output encoding must match the destination context.

## Common mistakes

- Using single quotes around JSON property names or string values; JSON requires double quotes.
- Assuming **json_encode()** converts arbitrary source encodings to UTF-8.
- Treating **null** as proof that **json_decode()** succeeded.
- Decoding objects as arrays without considering object/list shape.
- Forgetting that sparse PHP numeric keys encode as JSON object properties.
- Assuming an empty PHP array means an empty JSON object.
- Using **JSON_NUMERIC_CHECK** on identifiers or numeric-looking strings.
- Assuming large integers and floating-point values preserve exact meaning in every consumer.
- Treating syntactically valid JSON as schema-valid or safe application input.
- Calling **json_validate()** and then immediately decoding the same payload.
- Mixing warnings, HTML, or debugging text into a JSON HTTP response.
- Embedding raw JSON in HTML or JavaScript without destination-specific handling.

## Try it yourself

1. Encode an associative array using pretty printing and unescaped Unicode.
2. Encode a sequential array, then remove a middle key and compare the result.
3. Use **array_values()** to turn the sparse numeric array back into a JSON list.
4. Encode an empty array and an empty object and compare **[]** with **{}**.
5. Decode a JSON object once as **stdClass** and once as an associative array.
6. Decode malformed JSON with **JSON_THROW_ON_ERROR** and catch **JsonException**.
7. Decode a large integer with and without **JSON_BIGINT_AS_STRING**.
8. Compare a numeric-looking identifier with **JSON_NUMERIC_CHECK** enabled and disabled.
9. On PHP 8.3 or later, use **json_validate()** only when you do not need to decode the same payload immediately.
10. Build an HTTP JSON response and confirm no output occurs before the content-type header.

## Key takeaways

- JSON is structured UTF-8 text; it is not PHP serialization.
- Use **json_encode()** and **json_decode()** with an explicit error policy.
- Choose objects and lists intentionally; PHP arrays can encode to either.
- Validate the decoded root and fields against the application contract.
- Treat large numbers, numeric-looking strings, and floating-point values deliberately.
- **json_validate()** checks syntax only and is useful when the decoded value is not needed.
- Send JSON with the correct content type and handle output for its destination context.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [054 - Streams, Wrappers, and Contexts](054-streams-wrappers-and-contexts.md) | [Open Q&A](../qnas/055-encoding-and-decoding-json.md) | [Open example](../examples/055-encoding-and-decoding-json.md) | [056 - Serialization and Safe Data Exchange](056-serialization-and-safe-data-exchange.md) |
