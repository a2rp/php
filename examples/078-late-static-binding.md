# 078 - Late static binding: Example

## Question

When a base class method is inherited, how can it call subclass behavior, identify the runtime called class, and create an instance of that subclass? How do `self::`, `static::`, `parent::`, and an explicit class call differ?

## Complete PHP example

```php
<?php

declare(strict_types=1);

class BaseFormatter
{
    public static function label(): string
    {
        return 'Base';
    }

    public static function viaSelf(): string
    {
        return self::label();
    }

    public static function viaStatic(): string
    {
        return static::label();
    }

    public static function definedClass(): string
    {
        return self::class;
    }

    public static function calledClass(): string
    {
        return static::class;
    }
}

final class JsonFormatter extends BaseFormatter
{
    public static function label(): string
    {
        return 'JSON';
    }
}

final class CsvFormatter extends BaseFormatter
{
    public static function label(): string
    {
        return 'CSV';
    }
}

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

final class Leaf extends Branch
{
}

abstract class Record
{
    public function __construct(public string $id)
    {
    }

    public static function create(string $id): static
    {
        return new static($id);
    }

    public function describe(): string
    {
        return static::class . ' #' . $this->id;
    }
}

final class UserRecord extends Record
{
}

class BaseModel
{
    protected static string $table = 'records';

    public static function tableName(): string
    {
        return static::$table;
    }
}

final class UserModel extends BaseModel
{
    protected static string $table = 'users';
}

echo 'self dispatch: ' . JsonFormatter::viaSelf() . "\n";
echo 'late-bound dispatch: ' . JsonFormatter::viaStatic() . "\n";
echo 'another subclass: ' . CsvFormatter::viaStatic() . "\n";
echo 'self class name: ' . CsvFormatter::definedClass() . "\n";
echo 'called class name: ' . CsvFormatter::calledClass() . "\n\n";

echo 'forwarded through self: ' . Leaf::throughSelf() . "\n";
echo 'forwarded through parent: ' . Leaf::throughParent() . "\n";
echo 'explicit Root call: ' . Leaf::throughExplicitClass() . "\n\n";

$record = UserRecord::create('user-42');
echo 'factory class: ' . get_class($record) . "\n";
echo 'factory result: ' . $record->describe() . "\n\n";

echo 'inherited static property: ' . UserModel::tableName() . "\n";
```

This example uses constructor property promotion and the `static` return type, so it requires PHP 8.0 or newer.

## Expected output

```text
self dispatch: Base
late-bound dispatch: JSON
another subclass: CSV
self class name: BaseFormatter
called class name: CsvFormatter

forwarded through self: Leaf
forwarded through parent: Leaf
explicit Root call: Root

factory class: UserRecord
factory result: UserRecord #user-42

inherited static property: users
```

## Explanation

### `self::` selects base behavior

`JsonFormatter::viaSelf()` enters the inherited method in `BaseFormatter`. That method calls `self::label()`, so it selects `BaseFormatter::label()` and returns `Base`.

### `static::` selects subclass behavior

`JsonFormatter::viaStatic()` uses the runtime called class for `static::label()`, so PHP selects the `JsonFormatter` override. Calling the same inherited method through `CsvFormatter` selects that class's override instead.

`self::class` reports where the expression is defined: `BaseFormatter`. `static::class` reports the runtime called class: `CsvFormatter` in this call.

### Forwarding calls retain the called class

`Leaf::throughSelf()` and `Leaf::throughParent()` call the root implementation through forwarding forms. The method in `Root` still sees `Leaf` as the called class when it evaluates `static::class`.

`Leaf::throughExplicitClass()` calls `Root::reportCalledClass()` by naming `Root` directly. That fully specified call resets the late-binding context to `Root`.

### `new static()` creates the subclass instance

`UserRecord::create()` inherits the factory from `Record`. The factory uses `new static($id)`, which constructs `UserRecord`. The `: static` return type documents that the result matches the class used for the call.

The factory assumes subclasses have a constructor compatible with the argument it supplies. If subclasses need different constructor parameters, give them an explicit factory contract or override the method.

### `static::$table` selects subclass configuration

`UserModel` redeclares the inherited static `$table` property. The inherited `tableName()` method uses `static::$table`, so it reads `UserModel::$table` and returns `users`.

## Try these changes

1. Add a `MarkdownFormatter` subclass and override `label()`. What do `viaSelf()` and `viaStatic()` return?
2. Add a `static::class` method to `Record`. Compare it with `self::class` when called through `UserRecord`.
3. Change `Leaf::throughExplicitClass()` to call `parent::reportCalledClass()`. What called class does the result show?
4. Add a `ProductRecord` subclass and call its inherited factory. What class should `get_class()` report?
5. Change `UserModel::tableName()` to use `self::$table`. Which table value does it select?
6. Remove the `static` return type from `Record::create()`. What type guarantee is lost from the method declaration?

## Key takeaways

- `self::` uses the class where the reference is written; `static::` uses the runtime called class.
- Forwarding calls through `self::` and `parent::` can preserve late-binding context.
- A fully specified class call resets the context to that named class.
- `new static()` creates the called subclass, and `: static` declares that return contract.
- `static::$property` can select subclass-specific configuration from inherited code.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [077 - Cloning and comparing objects](../lessons/077-cloning-and-comparing-objects.md) | [Lesson](../lessons/078-late-static-binding.md) | [Q&A](../qnas/078-late-static-binding.md) | [079 - Enumerations and backed enums](../lessons/079-enumerations-and-backed-enums.md) |
