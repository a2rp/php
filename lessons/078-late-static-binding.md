# 078 - Late static binding

Late static binding lets an inherited method refer to the class that was actually called at runtime. It is especially useful for inheritance-aware factories, static configuration, and methods that must return the most specific subclass.

## Learning goals

By the end of this chapter, you will be able to:

- distinguish `self::` from `static::`;
- explain the runtime called class;
- understand forwarding calls through `self::` and `parent::`;
- use `new static()` to build subclass instances;
- use the `static` return type introduced in PHP 8.0;
- recognize when late static binding makes an API too implicit.

## The problem: inherited code can refer to its defining class

`self::` refers to the class where the method or property reference is written. That is often correct: a base class may need to use its own implementation regardless of subclass overrides.

It can be limiting when a base method is intended to work with a subclass. Consider a base method that dispatches to another overridable method:

~~~php
class BaseFormatter
{
    public static function format(): string
    {
        return self::label();
    }

    public static function label(): string
    {
        return 'Base';
    }
}

class JsonFormatter extends BaseFormatter
{
    public static function label(): string
    {
        return 'JSON';
    }
}

echo JsonFormatter::format(); // Base
~~~

The call inside `format()` uses `self::label()`, so PHP selects `BaseFormatter::label()`. The subclass override is not selected for that call.

## Use `static::` for the runtime called class

Late static binding provides the `static::` scope-resolution form. It resolves using the class that was initially called at runtime, rather than only the class where the current method was defined:

~~~php
class BaseFormatter
{
    public static function format(): string
    {
        return static::label();
    }

    public static function label(): string
    {
        return 'Base';
    }
}

class JsonFormatter extends BaseFormatter
{
    public static function label(): string
    {
        return 'JSON';
    }
}

echo JsonFormatter::format(); // JSON
~~~

`JsonFormatter::format()` starts the call with `JsonFormatter` as the called class. The inherited `format()` method uses `static::label()`, so PHP dispatches to `JsonFormatter::label()`.

A useful shorthand:

- `self::` means the class where the reference is defined;
- `static::` means the runtime called class tracked by late static binding.

## The called class and `static::class`

Inside an inherited method, `static::class` gives the class name associated with the current late static binding context. `get_called_class()` retrieves the called class name as a string from a valid class context.

~~~php
class BaseReport
{
    public static function definedClass(): string
    {
        return self::class;
    }

    public static function calledClass(): string
    {
        return static::class;
    }
}

class SalesReport extends BaseReport {}

// BaseReport
echo SalesReport::definedClass(), PHP_EOL;

// SalesReport
echo SalesReport::calledClass(), PHP_EOL;
~~~

`self::class` is resolved in the base class source. `static::class` uses the class that initiated the call.

## Forwarding calls: `self::` and `parent::`

There is an important nuance: `self::method()` selects a method using the current class scope, but it can still forward the called-class context to a later `static::` call. `parent::method()` also forwards that context.

~~~php
class Root
{
    public static function reportCalledClass(): string
    {
        return static::class;
    }
}

class Branch extends Root
{
    public static function throughSelf(): string
    {
        return self::reportCalledClass();
    }

    public static function throughParent(): string
    {
        return parent::reportCalledClass();
    }

    public static function throughExplicitClass(): string
    {
        return Root::reportCalledClass();
    }
}

class Leaf extends Branch {}

echo Leaf::throughSelf();          // Leaf
echo Leaf::throughParent();        // Leaf
echo Leaf::throughExplicitClass(); // Root
~~~

`self::` and `parent::` are forwarding calls in this context, so `Leaf` remains the called class. `Root::reportCalledClass()` explicitly names the class for a fully resolved call, so the late binding context is `Root`.

Do not confuse method forwarding with class-name constants. `self::class` always describes the class where that name is written, while `static::class` represents the runtime called class.

## Inherited static properties

Use `static::$property` when an inherited method should read a static property from the called subclass:

~~~php
class BaseModel
{
    protected static string $table = 'records';

    public static function tableName(): string
    {
        return static::$table;
    }
}

class UserModel extends BaseModel
{
    protected static string $table = 'users';
}

echo UserModel::tableName(); // users
~~~

A static property belongs to a class-level API rather than to one object instance. When a child redeclares the property, `static::$table` selects the property in the late-bound class context. `self::$table` would refer to the property selected from the class where that reference is written.

## Build the called subclass with `new static()`

A base factory can use `new static()` to instantiate the class that called the inherited method:

~~~php
abstract class Message
{
    public function __construct(public string $text) {}

    public static function create(string $text): static
    {
        return new static($text);
    }
}

class EmailMessage extends Message {}

$message = EmailMessage::create('Welcome');
echo get_class($message); // EmailMessage
~~~

The return type `static` says that the method returns an instance of the called class (or a compatible subtype), not merely an instance of the class where the method is defined. The `static` return type became available in PHP 8.0.

The subclass must remain compatible with the constructor call made by the inherited factory. If subclasses need different creation requirements, override the factory or use a separate factory abstraction.

## `self` and `static` in return types

`self` as a return type promises an instance compatible with the class where the method is declared. `static` promises an instance compatible with the late-bound called class. Use `static` for extensible fluent methods or factories that intentionally preserve the more specific subclass type.

~~~php
class Query
{
    public function where(string $condition): static
    {
        // Add the condition to this query.
        return $this;
    }
}

class UserQuery extends Query {}

$query = (new UserQuery())->where('active = 1');
// The declared return type preserves UserQuery.
~~~

Do not use `static` automatically. If a method always returns a specific base type, declare that actual contract instead. The `static` return type is a type declaration; it does not itself instantiate an object or make a method static.

## Late static binding also works in instance methods

The called class can come from an object context as well as a static call. A method inherited by a subclass instance can use `static::` to reach subclass behavior:

~~~php
class BaseService
{
    public function serviceName(): string
    {
        return static::name();
    }

    protected static function name(): string
    {
        return 'base';
    }
}

class MailService extends BaseService
{
    protected static function name(): string
    {
        return 'mail';
    }
}

echo (new MailService())->serviceName(); // mail
~~~

In non-static context, the called class is based on the class of the object instance. Choose `static::` only when subclass dispatch is part of the intended behavior.

## When to use late static binding

It is useful when all of these are true:

- a method is inherited and should vary by subclass;
- the called subclass is a meaningful part of the operation;
- the extension point is intentional and documented.

Common uses include:

- factories shared by a base class;
- subclass-specific static configuration;
- fluent APIs that preserve subclass types;
- template methods that call a subclass override.

A normal instance method, dependency injection, or an explicit factory may be easier to understand when runtime subclass dispatch is not the goal.

## Common mistakes

- **Using `self::` when subclass override dispatch is required:** use `static::` for the called-class behavior.
- **Replacing every `self::` with `static::`:** some base behavior should stay fixed; dynamic dispatch changes the contract.
- **Assuming `self::class` is late-bound:** it is tied to the class where it is written. Use `static::class` for the called class.
- **Assuming `parent::` loses the called class:** it forwards the late-binding context.
- **Treating `Base::method()` as a forwarding call:** an explicitly named class call resets the context to that class.
- **Using `new self()` in a subclass-aware factory:** it constructs the defining class; use `new static()` when the called class should be created.
- **Using `static` return type before PHP 8.0:** that return-only type requires PHP 8.0 or newer.
- **Building an implicit API that subclasses cannot safely support:** document constructor and override requirements or use an explicit factory.

## Key takeaways

- `self::` resolves to the class where a reference is written; `static::` uses the runtime called class.
- `static::` enables inherited code to dispatch to subclass methods and properties.
- `self::` and `parent::` method calls can forward late-binding context; a fully specified class call does not.
- `new static()` constructs the called class, which is useful in inherited factories.
- The `static` return type has been available since PHP 8.0 and preserves subclass-aware return contracts.
- Late static binding is an intentional extension point, not a default replacement for `self::`.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [077 - Cloning and comparing objects](077-cloning-and-comparing-objects.md) | [Q&A](../qnas/078-late-static-binding.md) | [Example](../examples/078-late-static-binding.md) | [079 - Enumerations and backed enums](079-enumerations-and-backed-enums.md) |
