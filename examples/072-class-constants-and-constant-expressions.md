# Class constants and constant expressions - Example

## Goal

Create an HTTP response helper that keeps stable values in class constants, derives a value from other constants, and demonstrates how a child class can supply a specialized constant.

This main example uses typed class constants, so it requires PHP 8.3 or later.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

final class HttpResponse
{
    public const int OK = 200;
    public const int NOT_FOUND = 404;
    public const string MEDIA_TYPE = 'application/json';

    public const string CACHE_PREFIX =
        'response:' . self::MEDIA_TYPE . ':';

    private const array LABELS = [
        self::OK => 'OK',
        self::NOT_FOUND => 'Not Found',
    ];

    public static function labelFor(int $status): string
    {
        return self::LABELS[$status] ?? 'Unknown';
    }

    public static function contentTypeHeader(): string
    {
        return 'Content-Type: ' . self::MEDIA_TYPE;
    }

    public static function cacheKey(string $key): string
    {
        return self::CACHE_PREFIX . $key;
    }
}

class Document
{
    public const string CATEGORY = 'document';

    public static function fixedCategory(): string
    {
        return self::CATEGORY;
    }

    public static function calledCategory(): string
    {
        return static::CATEGORY;
    }
}

class Invoice extends Document
{
    public const string CATEGORY = 'invoice';
}

class Report
{
}

echo HttpResponse::OK . PHP_EOL;
echo HttpResponse::labelFor(HttpResponse::NOT_FOUND) . PHP_EOL;
echo HttpResponse::contentTypeHeader() . PHP_EOL;
echo HttpResponse::cacheKey('report-7') . PHP_EOL;

echo Document::fixedCategory() . PHP_EOL;
echo Invoice::fixedCategory() . PHP_EOL;

echo Document::calledCategory() . PHP_EOL;
echo Invoice::calledCategory() . PHP_EOL;

echo Report::class;
~~~

## Expected output

~~~text
200
Not Found
Content-Type: application/json
response:application/json:report-7
document
document
document
invoice
Report
~~~

## Follow the values

The constants on <code>HttpResponse</code> are stable class-level values:

~~~php
public const int OK = 200;
public const string MEDIA_TYPE = 'application/json';
~~~

<code>CACHE_PREFIX</code> is a constant expression that combines a literal with another class constant:

~~~php
public const string CACHE_PREFIX =
    'response:' . self::MEDIA_TYPE . ':';
~~~

The expression is resolved as part of the constant declaration. It does not call a runtime method.

<code>LABELS</code> is private because it is an internal lookup table. Callers use the public <code>labelFor()</code> method instead of depending on the internal constant directly.

## Compare self and static

<code>Document</code> declares CATEGORY, and <code>Invoice</code> redeclares it with a specialized value.

The inherited method using <code>self::CATEGORY</code> always resolves the constant from Document:

~~~php
Invoice::fixedCategory(); // document
~~~

The inherited method using <code>static::CATEGORY</code> follows the class that was called:

~~~php
Invoice::calledCategory(); // invoice
~~~

Use <code>static::</code> when child classes are intended to provide a replacement value. Use <code>self::</code> when the parent implementation must always use its own constant.

## Use a class constant as a type-independent name

<code>Report::class</code> produces the class name string:

~~~php
echo Report::class;
~~~

It does not create a Report object.

## Look up a constant by a variable name

PHP 8.3 or later supports dynamic class constant fetch:

~~~php
$name = 'NOT_FOUND';
echo HttpResponse::{$name};
~~~

Expected output:

~~~text
404
~~~

For ordinary fixed constants, use direct syntax such as <code>HttpResponse::NOT_FOUND</code>. Dynamic lookup is useful only when the constant name is actually selected at runtime.

## Visibility and final constants

Class constants are public by default. Visibility can hide implementation details:

~~~php
class InternalSettings
{
    public const string NAME = 'public value';
    protected const string EXTENSION = 'protected value';
    private const string SECRET_KEY_NAME = 'internal value';
}
~~~

Since PHP 8.1, a constant can be final so child classes cannot redeclare it:

~~~php
class Protocol
{
    final public const string VERSION = '1';
}
~~~

Final constants require PHP 8.1 or later. The typed declaration above also uses PHP 8.3 syntax.

## Before PHP 8.3

For a project that supports PHP 8.2 or earlier, remove the constant types:

~~~php
class HttpResponse
{
    public const OK = 200;
    public const NOT_FOUND = 404;
    public const MEDIA_TYPE = 'application/json';
    public const CACHE_PREFIX =
        'response:' . self::MEDIA_TYPE . ':';
}
~~~

The values and expressions remain constants; only the typed-constant syntax is unavailable.

## What to notice

- Constants are accessed with the class name and <code>::</code>.
- Class constants do not have a dollar sign and are not per-object properties.
- The public methods expose selected behavior without exposing the private map.
- Constant expressions can derive stable values from other constants.
- <code>self::</code> stays with the defining class; <code>static::</code> follows the called class.
- Typed constants require PHP 8.3 or later.

## Practice

1. Add a <code>BAD_REQUEST</code> constant with status code 400 and a label.
2. Add a public method that returns a cache key for any status code.
3. Change <code>Invoice::CATEGORY</code> and observe the calls using <code>self::</code> and <code>static::</code>.
4. Make <code>CATEGORY</code> final in the parent. Why can Invoice no longer redeclare it?
5. Which values in this example should be properties instead if they needed to change during a request?

## Key takeaways

- Use class constants for values that are stable and shared by the class.
- Use constant expressions to combine supported constant values.
- Keep internal constants private or protected when callers do not need them.
- A child may redeclare a non-final constant; a final constant prevents that.
- Typed class constants require PHP 8.3; final constants require PHP 8.1.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [071 - Traits and code reuse](../lessons/071-traits-and-code-reuse.md) | [Lesson](../lessons/072-class-constants-and-constant-expressions.md) | [Q&A](../qnas/072-class-constants-and-constant-expressions.md) | [073 - Readonly properties and classes](../lessons/073-readonly-properties-and-classes.md) |
