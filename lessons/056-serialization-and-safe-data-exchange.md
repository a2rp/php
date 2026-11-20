# 056 - Serialization and Safe Data Exchange

Serialization converts a value in memory into a representation that can be stored or transmitted. Deserialization reconstructs a value from that representation. PHP has its own `serialize()` and `unserialize()` format, while JSON is a language-neutral text format commonly used for APIs and other data exchange.

These formats have different purposes. PHP serialization is coupled to PHP values and classes. JSON is usually the better choice when data crosses a process, language, browser, API, or trust boundary.

## What PHP serialization does

`serialize($value)` returns a string describing a supported PHP value. `unserialize($text)` parses that string and rebuilds the value.

~~~php
<?php

$original = [
    'name' => 'Asha',
    'topics' => ['PHP', 'security'],
    'active' => true,
];

$serialized = serialize($original);
$restored = unserialize($serialized);

var_dump($restored === $original); // bool(true)
~~~

PHP serialization can represent common scalar values, arrays, and objects. It preserves PHP-specific distinctions such as integer versus string keys, references in supported structures, and object class information. It is not a stable, language-neutral interchange format or a substitute for a documented data schema.

### Values and limitations

- **Scalars and arrays:** strings, integers, floats, booleans, `null`, and arrays can be serialized.
- **Objects:** serialized object data includes its class name and serialized state. The class definition normally needs to be available when the data is restored.
- **Resources:** resources cannot be restored as usable resources. Do not try to persist a file handle, database connection, socket, or similar live resource this way.
- **Closures and runtime state:** closures and other runtime-only state are not a portable persistence contract. Store the data needed to recreate behavior, not the live runtime object.
- **Binary strings:** the resulting serialized string can contain binary data, including NUL bytes. Store it in a binary-safe field and do not assume it is plain text or safe to truncate.
- **Class changes:** serialized objects depend on class names and state shape. Renaming a class or changing its expected properties can affect old saved data.

Serialization stores state, not a method implementation. An object payload names a class; it does not carry the class's PHP source code.

## Serializing objects deliberately

For a class that genuinely needs PHP's object serialization, define the exact persisted state with `__serialize()` and restore it with `__unserialize()`:

~~~php
<?php

final class ReadingList
{
    private string $owner;
    private array $titles;
    private ?PDO $connection = null;

    public function __construct(string $owner, array $titles)
    {
        $this->owner = $owner;
        $this->titles = $titles;
    }

    public function __serialize(): array
    {
        return [
            'owner' => $this->owner,
            'titles' => $this->titles,
        ];
    }

    public function __unserialize(array $data): void
    {
        if (!isset($data['owner']) || !is_string($data['owner'])) {
            throw new UnexpectedValueException('Invalid owner value.');
        }

        if (!isset($data['titles']) || !is_array($data['titles'])) {
            throw new UnexpectedValueException('Invalid titles value.');
        }

        foreach ($data['titles'] as $title) {
            if (!is_string($title)) {
                throw new UnexpectedValueException('Every title must be a string.');
            }
        }

        $this->owner = $data['owner'];
        $this->titles = $data['titles'];
        $this->connection = null; // Recreate runtime-only dependencies separately.
    }
}
~~~

The example persists only the owner's name and titles. It deliberately omits a database connection because a connection is runtime state that must be opened and configured by the application. Validate restored values before treating them as usable state.

`__serialize(): array` and `__unserialize(array $data): void` are the modern object hooks, available since PHP 7.4. They take precedence over the older `__sleep()` and `__wakeup()` hooks when both are defined. The older hooks are soft-deprecated as of PHP 8.5; new code should use the modern hooks. The older `Serializable` interface is also a legacy compatibility mechanism and has been deprecated since PHP 8.1 when implemented without the modern magic methods.

These hooks run during object serialization and restoration. Keep their work predictable. Do not perform network requests, open privileged resources, or trust restored values just because PHP populated an object.

## The security boundary: never deserialize untrusted input

**Do not pass untrusted data to `unserialize()`.** This includes data submitted by a browser, received from an API, uploaded in a file, or stored somewhere an attacker can modify. Restoring objects can instantiate classes, trigger autoloading, and invoke magic methods such as `__unserialize()` or `__wakeup()`. Applications with suitable classes or dependency chains may expose object-injection vulnerabilities.

The `allowed_classes` option can restrict which classes PHP restores, and `false` prevents normal application classes from being restored. This can be useful as defense in depth for an already trusted legacy payload. It does **not** turn `unserialize()` into a safe parser for attacker-controlled input. Do not use it as a substitute for choosing a safe format.

The `max_depth` option can cap nested structure depth and help limit one form of resource consumption. It does not authenticate the data or make an untrusted payload safe.

For data that must be read from a location that could be modified, a keyed message authentication code (HMAC) can prove that a payload was produced by a party holding the secret key. Verify it with `hash_equals()` **before** calling `unserialize()`:

~~~php
<?php

$expectedMac = hash_hmac('sha256', $serialized, $secretKey);

if (!hash_equals($expectedMac, $providedMac)) {
    throw new RuntimeException('The stored payload failed authentication.');
}

// Only consider unserialization after authenticity is verified and the
// producer, key handling, format, and expected classes are controlled.
$value = unserialize($serialized, [
    'allowed_classes' => false,
    'max_depth' => 64,
]);
~~~

This is a narrowly scoped pattern for authenticated, controlled legacy storage. It is not permission to deserialize arbitrary user input. Keep the key outside the data store, protect it from the same attacker, and use a separate encryption mechanism if confidentiality is needed. HMAC provides integrity and authenticity, not encryption.

A serialized boolean `false` is valid data, while `unserialize()` also returns `false` on failure and emits a warning. That makes error handling subtle. Do not build a new external data protocol around this ambiguous legacy format; prefer JSON with `JSON_THROW_ON_ERROR`.

## Use JSON for exchange between systems

For APIs, browser requests, files exchanged with other applications, and cross-language communication, use a documented JSON schema or equivalent structured format. Decode with an explicit error policy, then validate the shape and values:

~~~php
<?php

try {
    $payload = json_decode($jsonText, true, 64, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    throw new RuntimeException('The request body is not valid JSON.', 0, $exception);
}

if (!is_array($payload)
    || !isset($payload['name'])
    || !is_string($payload['name'])
    || !isset($payload['topics'])
    || !is_array($payload['topics'])) {
    throw new RuntimeException('The request body does not match the expected schema.');
}

foreach ($payload['topics'] as $topic) {
    if (!is_string($topic)) {
        throw new RuntimeException('Every topic must be a string.');
    }
}
~~~

Valid JSON syntax does not prove that the data has the fields, types, ranges, or relationships your application requires. Enforce those rules after decoding. JSON also does not recreate PHP classes, preserve every PHP type, or keep object identity and references; that is usually an advantage at a system boundary because the data contract is explicit.

A versioned envelope helps evolve an exchange format deliberately:

~~~json
{
  "version": 1,
  "type": "reading-list",
  "data": {
    "owner": "Asha",
    "titles": ["PHP basics", "Safe data exchange"]
  }
}
~~~

The receiver can reject unknown versions or apply a deliberate migration. Avoid silently guessing what an old or malformed payload means.

## Choose a format by purpose

| Need | Suitable approach |
| --- | --- |
| Short-lived, internal PHP cache under your control | PHP serialization may be appropriate, with expiry and trusted storage |
| Existing authenticated PHP session or legacy data | Follow the system's trusted storage contract; never accept attacker-controlled serialized bytes |
| Public API or browser/server exchange | JSON, explicit schema, and field validation |
| Interoperability with other languages | JSON or another documented neutral format |
| Confidential data | Encryption designed for the purpose, with key management; serialization alone is not encryption |
| Tamper detection | A keyed MAC/signature verified before parsing or acting on the data |
| Durable records across software versions | A versioned schema and migration strategy, rather than coupling the record to PHP class internals |

PHP's session subsystem commonly serializes session values internally. Store simple data such as identifiers, strings, and arrays in session state. Do not put live service objects, connections, or resources there, and do not treat a session value as trustworthy until the session's integrity and application-level rules have been established.

## Common mistakes

- Calling `unserialize()` on request data because it came from a hidden form field or cookie.
- Assuming `allowed_classes => false` makes arbitrary serialized input safe.
- Treating `max_depth` as an authenticity or security check.
- Storing a database connection, file handle, socket, or other runtime resource.
- Persisting every private property without considering secrets or internal implementation details.
- Assuming an object payload contains the class code or can be read by another language.
- Assuming serialized strings are printable text or safe to truncate.
- Treating HMAC as encryption or storing its secret beside the payload.
- Forgetting to verify a MAC before parsing a legacy serialized value.
- Treating syntactically valid JSON as valid application data.
- Depending on an unversioned serialized object format for long-term durable storage.

## Try it yourself

1. Serialize and restore an associative array containing strings, booleans, integers, and nested arrays.
2. Add a resource to the data model and explain why it should be reopened instead of persisted.
3. Implement `__serialize()` so only the stable, non-sensitive fields of a small value object are included.
4. Validate every field in `__unserialize()` before assigning it to the object.
5. Convert the same value object to a plain array, encode it as JSON, and decode it without creating a PHP class.
6. Add a `version` property to a JSON envelope and reject a version the program does not support.
7. In a controlled local demo, change one byte in an HMAC-protected stored payload and confirm that verification fails before parsing.
8. Explain why the HMAC example is only safe when the key is protected from the party who can modify the stored payload.

## Key takeaways

- `serialize()` and `unserialize()` are PHP-specific and can restore PHP objects.
- Never call `unserialize()` on untrusted input, even with `allowed_classes` set to `false`.
- Modern object hooks let a class define and validate its serialized state, but those hooks execute during restoration.
- Use JSON and an explicit schema for APIs, browser data, and cross-language exchange.
- Validate decoded data; valid syntax alone is not a valid application payload.
- HMAC can authenticate controlled stored data but does not encrypt it or make arbitrary input safe.
- Version durable data formats and keep runtime-only resources out of serialized state.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [055 - Encoding and decoding JSON](055-encoding-and-decoding-json.md) | [Open Q&A](../qnas/056-serialization-and-safe-data-exchange.md) | [Open example](../examples/056-serialization-and-safe-data-exchange.md) | [057 - Errors, exceptions, and Throwable](057-errors-exceptions-and-throwable.md) |
