# 082 - Attributes and metadata

## Learning goals

By the end of this lesson, you will be able to:

- Describe how PHP attributes store structured metadata on declarations.
- Write attributes using the `#[...]` syntax.
- Create an attribute class and choose where it may be used.
- Pass literal and constant-expression arguments to an attribute.
- Read attribute metadata with PHP's Reflection API.
- Explain what attributes do not do automatically.

## 1. What is an attribute?

An attribute is structured metadata attached to a declaration in PHP code. It can describe a class, method, function, property, class constant, or parameter. Frameworks and applications can inspect that metadata later through Reflection and decide how to use it.

For example, an application could attach a route description to a controller method:

```php
#[Route('/articles', methods: ['GET'])]
public function index(): Response
{
    // Return the article list.
}
```

The attribute records information in a language-supported structure. PHP does not automatically register a route, call the method, validate a request, or change the method's behavior just because the attribute appears there. Application or framework code must read and act on it.

Attributes were added in PHP 8.0.

## 2. Attribute syntax

An attribute starts with `#[` and ends with `]`. The attribute name refers to an attribute class. Arguments are optional and go in parentheses:

```php
#[Audit]
class AccountService
{
}

#[Route('/accounts', methods: ['GET'])]
public function list(): array
{
    return [];
}
```

You may put attributes on separate lines or separate them with commas in one attribute group:

```php
#[Audit]
#[Route('/accounts', methods: ['GET'])]
public function list(): array
{
    return [];
}
```

An attribute name can be unqualified, qualified, or fully qualified. Import names with `use` just as you would for other classes:

```php
use App\Attributes\Route;

#[Route('/accounts')]
class AccountController
{
}
```

The import makes the short name available; it does not instantiate the attribute or run its behavior.

## 3. Attribute arguments

Attribute arguments can be positional or named. They must be literal values or constant expressions; arbitrary runtime expressions such as variables or function calls are not allowed.

```php
#[Route('/accounts', ['GET', 'HEAD'])]
class AccountController
{
}

#[Route(path: '/accounts', methods: ['GET', 'HEAD'])]
class AccountController
{
}
```

Named arguments make metadata easier to read when an attribute accepts several values. The names must match the constructor parameter names of the attribute class when PHP eventually creates its object.

## 4. Define an attribute class

Create a regular class and mark it with the built-in `#[Attribute]` attribute. Its constructor receives the metadata values:

```php
<?php

declare(strict_types=1);

use Attribute;

#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
final class Route
{
    /** @param list<string> $methods */
    public function __construct(
        public readonly string $path,
        public readonly array $methods = ['GET'],
    ) {
    }
}
```

`#[Attribute]` is required for a class to be used as an attribute. The bitmask passed to it controls which declarations may use the attribute. Here, `Route` can appear on a class or method.

Common target flags include:

| Target flag | Declaration it allows |
| --- | --- |
| `Attribute::TARGET_CLASS` | Class-like declarations such as classes, interfaces, traits, and enums |
| `Attribute::TARGET_FUNCTION` | Functions |
| `Attribute::TARGET_METHOD` | Methods |
| `Attribute::TARGET_PROPERTY` | Properties |
| `Attribute::TARGET_CLASS_CONSTANT` | Class constants |
| `Attribute::TARGET_PARAMETER` | Parameters |
| `Attribute::TARGET_ALL` | Any supported target |

Combine target flags with the bitwise OR operator (`|`). If no target flags are specified, the default is `Attribute::TARGET_ALL`.

## 5. Restrict or allow repeated use

By default, an attribute can be applied only once to a declaration. Add `Attribute::IS_REPEATABLE` to allow multiple occurrences:

```php
<?php

use Attribute;

#[Attribute(Attribute::TARGET_CLASS | Attribute::IS_REPEATABLE)]
final class Tag
{
    public function __construct(public readonly string $name)
    {
    }
}

#[Tag('api')]
#[Tag('internal')]
class AccountController
{
}
```

`Attribute::IS_REPEATABLE` is a flag for the attribute class declaration. It is not a target; combine it with whichever target flags the attribute needs.

## 6. Read attributes with Reflection

Reflection lets code inspect declarations and retrieve their attributes. `getAttributes()` returns `ReflectionAttribute` objects. You can inspect each object's name and arguments, then instantiate the represented attribute with `newInstance()`:

```php
$reflection = new ReflectionClass(AccountController::class);

foreach ($reflection->getAttributes(Tag::class) as $metadata) {
    $tag = $metadata->newInstance();
    echo $tag->name . "\n";
}
```

The reflection object depends on the declaration being inspected:

| Declaration | Reflection API |
| --- | --- |
| Class, interface, trait, or enum | `ReflectionClass` |
| Function | `ReflectionFunction` |
| Method | `ReflectionMethod` |
| Property | `ReflectionProperty` |
| Parameter | `ReflectionParameter` |
| Class constant | `ReflectionClassConstant` |

A `ReflectionAttribute` also provides `getName()`, `getArguments()`, `getTarget()`, and `isRepeated()`. Use the `IS_INSTANCEOF` flag with `getAttributes()` when you want to match an attribute class or its subclasses rather than only an exact class name.

### Metadata inspection and object creation are separate

`getAttributes()` returns reflection representations. The attribute class is instantiated only when code calls `newInstance()`. This separation lets code examine an attribute name and its arguments before invoking the attribute constructor.

Calling `newInstance()` can fail if the attribute class cannot be loaded, is not marked with `#[Attribute]`, has invalid constructor arguments, is applied to a disallowed target, or is repeated without being declared repeatable.

## 7. Attributes are metadata, not automatic behavior

An attribute does not by itself:

- validate a property or request;
- enforce authorization;
- register a route or event handler;
- load a class file;
- call a method; or
- modify the declaration's behavior.

A framework or your own code must inspect the metadata and implement the behavior. Treat metadata as input to that code and validate any values that affect security-sensitive decisions.

## 8. Attributes and documentation comments

A documentation comment is text intended for people and documentation tools. An attribute is structured metadata represented in PHP syntax and available through Reflection. Attributes can make configuration easier to parse, while comments remain useful for prose, explanations, and generated API documentation.

Attributes do not replace all comments. Use each for its intended purpose: comments explain code to readers; attributes provide structured data for tools or application logic.

## 9. Inheritance behavior

Class attributes are not automatically inherited. Reflecting a child class returns attributes declared on that child, not attributes declared on its parent, interfaces, or traits. If an application wants inherited metadata, its Reflection-based logic must explicitly inspect the relevant parent or related declarations.

Attributes on reflected members such as inherited methods and properties are available when those members are reflected. Keep the distinction between class-level metadata and member-level metadata in mind.

## 10. Common mistakes

- **Forgetting `#[Attribute]` on the metadata class:** PHP cannot instantiate it as an attribute.
- **Assuming metadata runs automatically:** an application must inspect attributes and implement their meaning.
- **Using a target the attribute does not allow:** include the correct target flag in the `#[Attribute(...)]` declaration.
- **Repeating a non-repeatable attribute:** add `Attribute::IS_REPEATABLE` if repetition is part of the design.
- **Passing a variable or function call as an argument:** use a literal or constant expression instead.
- **Confusing an attribute name with an object:** it is instantiated only when Reflection's `newInstance()` is called.
- **Assuming a child class inherits its parent's class attributes:** inspect the parent explicitly if inherited behavior is desired.
- **Using attributes instead of explanatory comments:** metadata and prose solve different problems.

## Key takeaways

- Attributes add structured metadata to PHP declarations using `#[...]` syntax.
- Attribute classes must be marked with `#[Attribute]` and can restrict targets or allow repeated use.
- Arguments are literal values or constant expressions and may be positional or named.
- Reflection returns `ReflectionAttribute` objects; `newInstance()` creates the actual attribute object.
- Attributes have no automatic application behavior; code must read and act on them.
- Class attributes are not inherited automatically.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [081 - Autoloading classes and files](081-autoloading-classes-and-files.md) | [Q&A](../qnas/082-attributes-and-metadata.md) | [Example](../examples/082-attributes-and-metadata.md) | [083 - Reflection API](083-reflection-api.md) |
