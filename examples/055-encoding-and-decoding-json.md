# 055 - Encoding and Decoding JSON: Complete Example

This example encodes a PHP response, decodes it back into an object, validates selected fields, and demonstrates list shape, large integer handling, numeric strings, floating-point formatting, malformed JSON, and invalid UTF-8.

## Goal

Build a small JSON exchange with an explicit error policy and preserve the intended data types across the PHP-to-JSON boundary.

## Requirements

- PHP 8.1 or later.
- JSON extension enabled; it is included with standard PHP builds.
- Save the PHP source as UTF-8.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

$payload = [
    'id' => '00042',
    'name' => 'Mina',
    'city' => 'Montréal',
    'active' => true,
    'scores' => [88, 92],
    'metadata' => (object) [],
];

try {
    $json = json_encode(
        $payload,
        JSON_PRETTY_PRINT
            | JSON_UNESCAPED_UNICODE
            | JSON_UNESCAPED_SLASHES
            | JSON_THROW_ON_ERROR
    );
} catch (JsonException $exception) {
    throw new RuntimeException('Could not encode the response.', 0, $exception);
}

echo "Encoded response:\n";
echo $json . "\n";

// Decode as stdClass objects, then check the shape and types expected by this API.
try {
    $decoded = json_decode($json, false, 64, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    throw new RuntimeException('Could not decode the response.', 0, $exception);
}

if (
    !($decoded instanceof stdClass)
    || !isset($decoded->id)
    || !is_string($decoded->id)
    || !isset($decoded->name)
    || !is_string($decoded->name)
    || !isset($decoded->active)
    || !is_bool($decoded->active)
    || !isset($decoded->scores)
    || !is_array($decoded->scores)
    || !isset($decoded->metadata)
    || !($decoded->metadata instanceof stdClass)
) {
    throw new RuntimeException('The decoded response has an unexpected shape.');
}

echo 'Decoded object class: ' . get_class($decoded) . "\n";
echo 'Recipient: ' . $decoded->name . "\n";
echo 'Account ID remains a string: '
    . (is_string($decoded->id) ? 'yes' : 'no')
    . "\n";
echo 'Active: ' . ($decoded->active ? 'yes' : 'no') . "\n";
echo 'First score: ' . $decoded->scores[0] . "\n";

// A sparse numeric-key array encodes as an object. Reindex when a JSON list is intended.
$sparseColors = [
    0 => 'red',
    2 => 'blue',
];

$sparseJson = json_encode($sparseColors, JSON_THROW_ON_ERROR);
$listJson = json_encode(array_values($sparseColors), JSON_THROW_ON_ERROR);

echo 'Sparse keys encode: ' . $sparseJson . "\n";
echo 'Reindexed list: ' . $listJson . "\n";

// Preserve a large integer token as a string rather than converting it to a float.
$largeIntegerJson = '{"id":1234567890123456789012345}';
$largeIntegerData = json_decode(
    $largeIntegerJson,
    false,
    512,
    JSON_BIGINT_AS_STRING | JSON_THROW_ON_ERROR
);

echo 'Large identifier: ' . $largeIntegerData->id . "\n";

// Keep a floating-point value's zero fraction in the JSON representation.
$ratioJson = json_encode(
    ['ratio' => 12.0],
    JSON_PRESERVE_ZERO_FRACTION | JSON_THROW_ON_ERROR
);

echo 'Float with zero fraction: ' . $ratioJson . "\n";

// Reject malformed JSON and handle it as an exception.
$malformedJsonRejected = false;

try {
    json_decode('{"name":', false, 512, JSON_THROW_ON_ERROR);
} catch (JsonException) {
    $malformedJsonRejected = true;
}

echo 'Malformed JSON rejected: '
    . ($malformedJsonRejected ? 'yes' : 'no')
    . "\n";

// JSON strings must be UTF-8. Here C3 is deliberately followed by an invalid byte.
$malformedUtf8 = "\xC3\x28";
$malformedUtf8Rejected = false;

try {
    json_encode(
        ['text' => $malformedUtf8],
        JSON_THROW_ON_ERROR
    );
} catch (JsonException) {
    $malformedUtf8Rejected = true;
}

echo 'Malformed UTF-8 rejected: '
    . ($malformedUtf8Rejected ? 'yes' : 'no')
    . "\n";

// Substitution is an explicit alternative policy for this demonstration.
$substitutedJson = json_encode(
    ['text' => $malformedUtf8],
    JSON_UNESCAPED_UNICODE
        | JSON_INVALID_UTF8_SUBSTITUTE
        | JSON_THROW_ON_ERROR
);

echo 'Invalid UTF-8 substitution: ' . $substitutedJson . "\n";
~~~

## Expected output

~~~text
Encoded response:
{
    "id": "00042",
    "name": "Mina",
    "city": "Montréal",
    "active": true,
    "scores": [
        88,
        92
    ],
    "metadata": {}
}
Decoded object class: stdClass
Recipient: Mina
Account ID remains a string: yes
Active: yes
First score: 88
Sparse keys encode: {"0":"red","2":"blue"}
Reindexed list: ["red","blue"]
Large identifier: 1234567890123456789012345
Float with zero fraction: {"ratio":12.0}
Malformed JSON rejected: yes
Malformed UTF-8 rejected: yes
Invalid UTF-8 substitution: {"text":"�("}
~~~

## How it works

- **json_encode()** converts the PHP payload to JSON text. Pretty printing adds whitespace; unescaped Unicode keeps **Montréal** readable in the encoded output.
- The payload deliberately keeps **id** as a string so its leading zeroes are not lost.
- The empty **metadata** object is encoded as **{}**, while **scores** is a sequential PHP array and becomes a JSON list.
- **json_decode()** with **false** decodes JSON objects into **stdClass**. The example checks the root and several field types after parsing.
- The sparse **$sparseColors** array becomes a JSON object because its numeric keys are not a sequential list. **array_values()** makes the intended list shape explicit.
- **JSON_BIGINT_AS_STRING** preserves a large JSON integer token as a string.
- **JSON_PRESERVE_ZERO_FRACTION** keeps the float representation **12.0** instead of **12**.
- **JSON_THROW_ON_ERROR** makes malformed syntax and invalid UTF-8 visible as exceptions.
- **JSON_INVALID_UTF8_SUBSTITUTE** demonstrates an explicit replacement policy. It should be chosen intentionally rather than used to hide unexpected source-encoding problems.

## Try it yourself

1. Remove the **city** field and update the schema check to require it.
2. Change **id** to an integer and observe how a leading zero cannot be represented as part of a number.
3. Remove a different key from the sparse array and compare the encoded shape.
4. Decode the response with **true** as the second argument and use array access.
5. Remove **JSON_BIGINT_AS_STRING** and inspect how the large integer is represented on your platform.
6. Remove **JSON_PRESERVE_ZERO_FRACTION** and compare the encoding of **12.0**.
7. On PHP 8.3 or later, try **json_validate()** for a payload whose decoded value you do not need.
8. Put the payload in an HTTP response and set the **application/json; charset=utf-8** content type before output.

## Common mistakes

- Treating JSON as a PHP serialization format.
- Assuming JSON functions convert arbitrary input strings to UTF-8.
- Using numeric JSON values for identifiers that need leading zeroes.
- Assuming every PHP array becomes a JSON list.
- Expecting an empty PHP array to become an empty JSON object.
- Decoding JSON without an error policy and confusing a failed decode with a valid **null**.
- Trusting decoded data without validating its shape and value types.
- Silently substituting or ignoring invalid UTF-8 without a data policy.
- Encoding sensitive object properties unintentionally.
- Sending debug output before a JSON response or inserting raw JSON into HTML without context-specific handling.

## Key takeaways

- Encode and decode UTF-8 JSON with an explicit error policy.
- Decide whether each collection is a JSON object or list.
- Validate decoded values against the application's contract.
- Preserve identifiers and large numeric values with appropriate string representations.
- Use flags for specific documented needs, not as blanket fixes.
- Syntax-valid JSON still needs application-level validation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [054 - Streams, Wrappers, and Contexts](../lessons/054-streams-wrappers-and-contexts.md) | [Read lesson](../lessons/055-encoding-and-decoding-json.md) | [Open Q&A](../qnas/055-encoding-and-decoding-json.md) | [056 - Serialization and Safe Data Exchange](../lessons/056-serialization-and-safe-data-exchange.md) |
