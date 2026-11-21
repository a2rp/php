# Magic methods

Magic methods are special methods that PHP calls when a particular operation is performed on an object. They let a class customize behavior such as property access, missing method calls, string conversion, invocation, debugging, and serialization.

The double-underscore names are reserved for PHP behavior. Define a magic method only when the class needs to support the operation it represents.

## What you will learn

- recognize the common PHP magic methods
- understand when PHP calls property and method overloading hooks
- convert an object to a string or make it callable
- customize debugging and serialization
- use the required visibility and signatures
- avoid surprising behavior and unsafe deserialization

## Magic methods are event hooks

A magic method is not normally called directly. PHP invokes it when code performs the associated operation:

~~~php
echo $object;        // May call __toString()
$value = $object();  // Calls __invoke()
var_dump($object);   // May call __debugInfo()
~~~

Magic methods do not make every unknown operation valid automatically. Each method should handle supported cases clearly and reject invalid ones.

## Constructors and destructors

### __construct()

<code>__construct()</code> runs when a new object is created. It is the usual place to initialize required state and validate constructor input:

~~~php
class User
{
    public function __construct(
        public readonly int $id,
        public string $name,
    ) {
        if ($id < 1) {
            throw new InvalidArgumentException(
                'User ID must be positive.'
            );
        }

        $this->name = trim($name);
    }
}
~~~

A constructor has no return type declaration. It may accept parameters and may throw an exception if it cannot create a valid object.

### __destruct()

<code>__destruct()</code> runs when an object is destroyed, usually when it is no longer referenced or during request shutdown. It is rarely necessary in application classes. Do not rely on a particular destruction order or shutdown timing for important work.

Destructors also have no return type declaration. Constructors and destructors may use non-public visibility when that is part of the design; most other magic methods must be public.

## Property overloading

Property overloading responds to access to a property that is undefined or inaccessible from the current scope. It does not replace ordinary property access for properties that are already accessible.

| Magic method | Operation |
| --- | --- |
| <code>__get(string $name): mixed</code> | Read an undefined or inaccessible property |
| <code>__set(string $name, mixed $value): void</code> | Write an undefined or inaccessible property |
| <code>__isset(string $name): bool</code> | Check an undefined or inaccessible property with <code>isset()</code> or <code>empty()</code> |
| <code>__unset(string $name): void</code> | Unset an undefined or inaccessible property |

A small property bag can use these hooks to expose a controlled set of dynamic-looking values:

~~~php
class Settings
{
    private array $values = [];

    public function __get(string $name): mixed
    {
        if (!array_key_exists($name, $this->values)) {
            throw new OutOfBoundsException(
                'Unknown setting: ' . $name
            );
        }

        return $this->values[$name];
    }

    public function __set(string $name, mixed $value): void
    {
        if ($name !== 'theme' || !is_string($value)) {
            throw new InvalidArgumentException(
                'Unsupported setting.'
            );
        }

        $this->values[$name] = $value;
    }

    public function __isset(string $name): bool
    {
        return isset($this->values[$name]);
    }

    public function __unset(string $name): void
    {
        unset($this->values[$name]);
    }
}
~~~

The backing array is private, so callers interact through the magic methods rather than gaining access to internal storage.

Property overloading has limits. For example, an operation such as <code>$settings->items[] = 'x'</code> may not behave like a simple assignment through <code>__set()</code>. Prefer explicit methods when nested or reference-based mutation must be controlled.

## Method overloading

Method overloading responds to calls to undefined or inaccessible methods:

| Magic method | Operation |
| --- | --- |
| <code>__call(string $name, array $arguments): mixed</code> | Call an undefined or inaccessible instance method |
| <code>__callStatic(string $name, array $arguments): mixed</code> | Call an undefined or inaccessible static method |

A handler receives the requested name and the arguments as an array:

~~~php
class Formatter
{
    public function __call(string $name, array $arguments): mixed
    {
        if ($name === 'uppercase' && count($arguments) === 1) {
            return strtoupper((string) $arguments[0]);
        }

        throw new BadMethodCallException(
            'Unsupported method: ' . $name
        );
    }
}

$formatter = new Formatter();
echo $formatter->uppercase('php');
~~~

This can be useful for carefully defined dynamic APIs, proxies, or query builders. It can also hide typos and make IDE navigation harder. Prefer explicit methods for a stable application API.

## String conversion: __toString()

<code>__toString(): string</code> defines the result when an object is used where a string is expected:

~~~php
class ProductCode
{
    public function __construct(private string $value)
    {
    }

    public function __toString(): string
    {
        return $this->value;
    }
}

echo new ProductCode('SKU-42');
~~~

Since PHP 8.0, a class that defines <code>__toString()</code> implicitly implements the <code>Stringable</code> interface. Explicitly declaring <code>implements Stringable</code> can make the contract clearer.

A string conversion should be predictable and should not expose secrets. Use <code>__toString()</code> for a useful textual representation, not as a substitute for JSON serialization or a general export format.

## Callable objects: __invoke()

<code>__invoke()</code> runs when an object is called like a function:

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

$addLabel = new Prefixer('PHP: ');
echo $addLabel('Magic methods');
~~~

An object with <code>__invoke()</code> can be used anywhere a callable is accepted. Use this when the object represents a small operation that needs stored configuration.

## Debug output: __debugInfo()

<code>__debugInfo(): array</code> controls the information shown by <code>var_dump()</code>:

~~~php
class Credentials
{
    public function __construct(
        private string $username,
        private string $password,
    ) {
    }

    public function __debugInfo(): array
    {
        return [
            'username' => $this->username,
            'password' => '[redacted]',
        ];
    }
}
~~~

Use it to present useful debug state while excluding secrets or internal details. As of PHP 8.5, returning <code>null</code> from <code>__debugInfo()</code> is deprecated; return an array, including an empty array when no details should be displayed.

## Serialization: __serialize() and __unserialize()

Serialization turns an object into a storable representation. Since PHP 7.4, classes can define:

~~~php
public function __serialize(): array
public function __unserialize(array $data): void
~~~

<code>__serialize()</code> returns the data to store. <code>__unserialize()</code> restores the object from that data:

~~~php
class Coordinate
{
    public function __construct(
        private int $x,
        private int $y,
    ) {
    }

    public function __serialize(): array
    {
        return ['x' => $this->x, 'y' => $this->y];
    }

    public function __unserialize(array $data): void
    {
        if (!isset($data['x'], $data['y'])
            || !is_int($data['x'])
            || !is_int($data['y'])) {
            throw new UnexpectedValueException(
                'Invalid coordinate data.'
            );
        }

        $this->x = $data['x'];
        $this->y = $data['y'];
    }
}
~~~

Serialization hooks are useful for controlling which state is stored and for rebuilding an object. Validate data during restoration, especially when serialized values may have been modified.

Never call <code>unserialize()</code> on untrusted data. Deserialization can instantiate classes and invoke object behavior. For external data, use a format such as JSON and validate its shape explicitly.

### Legacy __sleep() and __wakeup()

<code>__sleep()</code> and <code>__wakeup()</code> are the older serialization hooks. They remain for compatibility, but PHP 8.5 soft-deprecated them. Prefer <code>__serialize()</code> and <code>__unserialize()</code> in new code and migrate older implementations when practical.

## Exported object state: __set_state()

<code>__set_state(array $properties): object</code> is called by code generated by <code>var_export()</code> when the exported object is evaluated back into PHP code:

~~~php
class Color
{
    public function __construct(
        public string $name,
        public string $hex,
    ) {
    }

    public static function __set_state(array $properties): object
    {
        return new self(
            $properties['name'],
            $properties['hex'],
        );
    }
}
~~~

<code>var_export()</code> produces PHP code, not JSON. Only evaluate exported code from trusted sources; do not use <code>eval()</code> on untrusted input.

## Cloning: __clone()

<code>__clone()</code> runs on the newly created copy after the <code>clone</code> operation. It can customize clone behavior. Ordinary object cloning is shallow by default; nested objects are not automatically cloned.

The next chapter covers cloning and comparing objects in more detail. Use <code>__clone()</code> only when copying the object requires deliberate adjustments.

## Required visibility and signatures

Most magic methods must be public. The exceptions are <code>__construct()</code>, <code>__destruct()</code>, and <code>__clone()</code>, which may use other visibility where appropriate.

Use the signature specified for each magic method. A mismatched signature can cause a fatal error. Constructors and destructors must not declare a return type. In PHP 8 and later, PHP checks magic method signatures more strictly.

| Method | Required signature |
| --- | --- |
| <code>__get</code> | <code>public function __get(string $name): mixed</code> |
| <code>__set</code> | <code>public function __set(string $name, mixed $value): void</code> |
| <code>__isset</code> | <code>public function __isset(string $name): bool</code> |
| <code>__unset</code> | <code>public function __unset(string $name): void</code> |
| <code>__call</code> | <code>public function __call(string $name, array $arguments): mixed</code> |
| <code>__callStatic</code> | <code>public static function __callStatic(string $name, array $arguments): mixed</code> |
| <code>__toString</code> | <code>public function __toString(): string</code> |
| <code>__serialize</code> | <code>public function __serialize(): array</code> |
| <code>__unserialize</code> | <code>public function __unserialize(array $data): void</code> |
| <code>__debugInfo</code> | <code>public function __debugInfo(): array</code> |
| <code>__set_state</code> | <code>public static function __set_state(array $properties): object</code> |

## Choosing whether to add a magic method

Magic methods are useful when they preserve a natural object interface and have one clear behavior. They can make code difficult to discover when they silently accept arbitrary names or perform surprising work.

Before adding one, ask:

- Is the triggering PHP operation obvious to someone reading the call site?
- Will unsupported names fail clearly?
- Can a normal method or explicit property express the behavior better?
- Could the hook expose private data or secrets?
- Does the code validate values restored from serialized data?

## Common mistakes

- **Using a reserved double-underscore name for an ordinary method:** names beginning with <code>__</code> are reserved for PHP behavior.
- **Making an ordinary magic method non-public:** most magic methods must be public.
- **Using the wrong signature:** follow the documented parameters and return type.
- **Expecting <code>__get()</code> for every property read:** it applies to undefined or inaccessible properties.
- **Using overloading to hide typos:** reject unsupported property or method names.
- **Returning a non-string from <code>__toString()</code>:** the method must return a string.
- **Exposing credentials in <code>__debugInfo()</code>:** redact secrets.
- **Trusting serialized input:** validate restoration data and never unserialize untrusted input.
- **Starting new serialization code with <code>__sleep()</code> and <code>__wakeup()</code>:** use the newer serialization pair.
- **Assuming clone makes a deep copy:** cloning is shallow unless the class handles nested state.

## Key takeaways

- Magic methods customize specific PHP operations on objects.
- Property and method overloading handle undefined or inaccessible members.
- <code>__toString()</code> converts to text; <code>__invoke()</code> makes an object callable.
- <code>__debugInfo()</code> can keep sensitive values out of debug dumps.
- Prefer <code>__serialize()</code> and <code>__unserialize()</code>; the legacy pair is soft-deprecated as of PHP 8.5.
- Validate deserialized state and never unserialize untrusted data.
- Use magic methods sparingly and preserve predictable behavior.

## Version notes

| PHP version | Magic method detail |
| --- | --- |
| PHP 7.4 | <code>__serialize()</code> and <code>__unserialize()</code> |
| PHP 8.0 | Strict magic method signature checks; <code>__toString()</code> classes implement <code>Stringable</code> |
| PHP 8.5 | <code>__sleep()</code> and <code>__wakeup()</code> soft-deprecated; null from <code>__debugInfo()</code> deprecated |

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [075 - Property hooks](075-property-hooks.md) | [Q&A](../qnas/076-magic-methods.md) | [Example](../examples/076-magic-methods.md) | [077 - Cloning and comparing objects](077-cloning-and-comparing-objects.md) |
