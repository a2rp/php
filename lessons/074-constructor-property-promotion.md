# Constructor property promotion

Constructor property promotion combines a constructor parameter with a property declaration and its initial assignment. It is a concise way to store constructor input when the parameter value should become object state.

The feature was introduced in PHP 8.0. A promoted parameter is still a constructor parameter, and it also creates a property with matching visibility and type.

## What you will learn

- promote constructor parameters into properties
- mix promoted and ordinary constructor parameters
- use visibility, defaults, and nullable types correctly
- understand when promotion happens in the constructor
- apply validation and normalization safely
- recognize syntax restrictions and version requirements
- decide when explicit property declarations are clearer

## The repeated pattern

Without promotion, a value object often repeats each value in a property declaration, a constructor parameter, and an assignment:

~~~php
class Point
{
    public int $x;
    public int $y;

    public function __construct(int $x, int $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
}
~~~

Promotion removes this repeated boilerplate:

~~~php
class Point
{
    public function __construct(
        public int $x,
        public int $y,
    ) {
    }
}
~~~

The public visibility modifier tells PHP to declare a public property for each promoted parameter. PHP then assigns the matching argument to that property during construction.

## Promote only values that should become properties

A constructor parameter without a visibility modifier is an ordinary parameter. It exists while the constructor runs but does not create a property:

~~~php
class User
{
    private string $email;

    public function __construct(
        public int $id,
        string $email,
    ) {
        $this->email = strtolower(trim($email));
    }

    public function email(): string
    {
        return $this->email;
    }
}
~~~

Here, <code>$id</code> is promoted to a property. <code>$email</code> is used as constructor input, normalized, and stored in an explicitly declared private property.

You can mix promoted and ordinary parameters in the same constructor. Use promotion when the input maps directly to stored state. Use an ordinary parameter when you need to transform it, use it only temporarily, pass it to a parent constructor, or store it in a differently named property.

## Choose the property visibility

The visibility modifier on a promoted parameter becomes the property's visibility:

~~~php
class Connection
{
    public function __construct(
        public string $host,
        protected int $port,
        private string $password,
    ) {
    }
}
~~~

- <code>public</code> allows callers to read and write the property.
- <code>protected</code> allows access in the class and its descendants.
- <code>private</code> allows access only in the declaring class.

Promotion does not make a property public automatically in every design. Choose visibility based on the class API. A private property with methods is often better when callers must not modify the value directly.

## Promotion with readonly

Readonly properties were added in PHP 8.1. A promoted property may be declared readonly:

~~~php
class UserId
{
    public function __construct(
        public readonly int $value,
    ) {
    }
}
~~~

The constructor initializes the property automatically. Later writes are rejected. Read the previous chapter, [Readonly properties and classes](072-class-constants-and-constant-expressions.md), for the one-time initialization and shallow immutability rules.

Promotion and readonly solve different parts of the declaration: promotion creates and initializes a property from a constructor parameter; readonly limits later writes.

## Defaults and nullable values

A default on a promoted parameter is a constructor parameter default. When a caller omits that argument, PHP assigns the default argument value to the promoted property for that object:

~~~php
class Pagination
{
    public function __construct(
        public int $page = 1,
        public int $perPage = 20,
    ) {
    }
}

$pagination = new Pagination();
echo $pagination->page; // 1
~~~

The default is not a property declaration default. The property still receives its value through constructor initialization.

A nullable type must be written explicitly. PHP does not infer a nullable property just because its parameter default is <code>null</code>:

~~~php
class Profile
{
    public function __construct(
        public ?string $nickname = null,
    ) {
    }
}
~~~

This creates a property that accepts a string or <code>null</code>, and allows callers to omit the constructor argument.

## When initialization occurs

Promoted properties are initialized automatically at the start of the constructor body, before the constructor's written statements run:

~~~php
class Label
{
    public function __construct(public string $text)
    {
        $this->text = trim($this->text);
    }
}

$label = new Label('  Ready  ');
echo $label->text; // Ready
~~~

The assignment in the constructor body is legal because <code>$text</code> is a normal mutable property. If the property were readonly, it would already be initialized by promotion, so assigning to it again would fail.

This order matters for validation and normalization:

- For a mutable promoted property, validate and then normalize by assigning the adjusted value in the constructor.
- For a readonly property, validate the parameter but do not assign the property a second time.
- If you must transform the value before the property receives it, declare the property explicitly and use a non-promoted constructor parameter.

Throw an exception when constructor input is invalid. Even though a promoted property receives its argument before the constructor body, a constructor that throws does not return a usable object.

## Transform input before assigning it

An explicit property is clearer when the stored value differs from the constructor input:

~~~php
class EmailAddress
{
    private string $value;

    public function __construct(string $value)
    {
        $normalized = strtolower(trim($value));

        if (filter_var($normalized, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException('Invalid email address.');
        }

        $this->value = $normalized;
    }

    public function value(): string
    {
        return $this->value;
    }
}
~~~

Promoting <code>$value</code> here would create a property with the original constructor input. An explicit property ensures only the validated normalized value becomes object state.

## Named arguments and constructor API

Promoted parameter names are also the property names. Callers may use those names with PHP named arguments:

~~~php
class Dimensions
{
    public function __construct(
        public int $width,
        public int $height,
    ) {
    }
}

$dimensions = new Dimensions(height: 600, width: 800);
~~~

Parameter names are part of the named-argument API. Renaming a promoted parameter can break callers that use named arguments, even if the property name also changes intentionally. Choose stable, descriptive names and document public constructors.

## Restrictions

Promotion has specific syntax and follows normal property rules:

- It is allowed only in a non-abstract constructor.
- It cannot be used in an ordinary method, abstract constructor, or interface declaration.
- The parameter needs a property visibility modifier such as <code>public</code>, <code>protected</code>, or <code>private</code>; <code>var</code> is not supported.
- A property with the same name must not already be declared in the class.
- A promoted parameter cannot use the <code>callable</code> type, because PHP properties cannot have that type.
- Variadic parameters cannot be promoted.
- Property and parameter naming and type rules both apply.

A promoted parameter must be a real constructor input. Do not add visibility merely to shorten a signature if the value should not be stored as a property.

## Mixing promotion with inheritance

A child constructor may promote its own properties and keep parent constructor inputs ordinary:

~~~php
class Entity
{
    public function __construct(protected int $id)
    {
    }
}

class Customer extends Entity
{
    public function __construct(
        int $id,
        public string $email,
    ) {
        parent::__construct($id);
    }
}
~~~

The <code>$email</code> argument becomes a property of <code>Customer</code>. The ordinary <code>$id</code> parameter is forwarded to the parent, where it initializes the parent's promoted property.

If both parent and child promote a property with the same name, they declare separate properties in their respective classes. Avoid accidental duplicate state; decide which class owns the value.

## Attributes and documentation

An attribute placed on a promoted parameter is available on both the constructor parameter and the generated property. This can be useful for validation, serialization, or mapping tools, but check that the attribute supports the targets your code uses.

A doc comment on the constructor parameter can document the argument. For property-focused tools, use documentation conventions supported by your project's analyzer and formatter. Promotion is syntax shorthand; reflection still exposes the resulting parameter and property.

## Reflection

Reflection can identify promoted parameters and properties:

~~~php
$parameter->isPromoted();
$property->isPromoted();
~~~

This is mainly useful when building frameworks, serializers, dependency injection tools, and code-analysis utilities. Most application code does not need to inspect promotion at runtime.

## When promotion helps

Promotion works well when constructor values map directly to properties, especially in:

- small value objects
- request and response data-transfer objects
- configuration objects
- immutable objects that use readonly promoted properties
- simple models with little initialization logic

Prefer explicit properties when you need to normalize values before storage, use a different property name, apply complex initialization, or make the state transitions easier to see.

## Version notes

| PHP version | Constructor property promotion feature |
| --- | --- |
| PHP 8.0 | Constructor property promotion |
| PHP 8.1 | Readonly properties can be promoted |

If your project supports PHP 8.0, do not use readonly promotion. Use ordinary promotion or explicit properties with the syntax available to your supported version.

## Common mistakes

- **Promoting every parameter:** temporary input does not need to become object state.
- **Assuming a bare parameter is promoted:** a visibility modifier is required.
- **Forgetting nullable syntax:** use <code>?Type</code> when <code>null</code> is a valid value.
- **Using a promoted readonly property as if it were uninitialized:** promotion initializes it before the constructor body.
- **Using promotion for normalization:** the unmodified argument is assigned before constructor statements run.
- **Using <code>callable</code> or a variadic parameter:** neither can be promoted.
- **Redeclaring the property:** the class cannot declare a property with the same name as a promoted property.
- **Changing public parameter names casually:** named-argument callers may depend on those names.
- **Using the feature before PHP 8.0:** constructor property promotion requires PHP 8.0 or later.

## Key takeaways

- A promoted constructor parameter creates a property and initializes it from the argument.
- Visibility and type describe the generated property.
- A parameter without visibility remains an ordinary parameter.
- Promotion assignments happen before the constructor body.
- Use explicit properties when validation or normalization must happen before storage.
- Readonly promotion requires PHP 8.1; ordinary constructor promotion requires PHP 8.0.
- Promotion is shorthand, not a requirement. Choose the form that makes object state easiest to understand.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [073 - Readonly properties and classes](073-readonly-properties-and-classes.md) | [Q&A](../qnas/074-constructor-property-promotion.md) | [Example](../examples/074-constructor-property-promotion.md) | [075 - Property hooks](075-property-hooks.md) |
