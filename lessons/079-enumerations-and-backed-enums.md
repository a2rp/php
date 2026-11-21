# 079 - Enumerations and backed enums

An enumeration (enum) defines a fixed set of named cases. It gives a domain such as an order status, direction, or access level its own type, so code can work with valid states instead of passing arbitrary strings or numbers.

## Learning goals

By the end of this chapter, you will be able to:

- declare and use a pure enum;
- distinguish an enum case's name from a backed value;
- choose between pure and backed enums;
- map scalar data with `from()` and `tryFrom()`;
- list cases with `cases()`;
- add methods, constants, and interfaces to enums;
- recognize enum restrictions and JSON behavior.

## What an enum represents

A regular string or integer can contain many values that your program does not support. An enum defines a finite set of legal cases and gives them a named type.

~~~php
enum Direction
{
    case North;
    case East;
    case South;
    case West;
}

function turn(Direction $direction): void
{
    echo $direction->name;
}

turn(Direction::East);
~~~

The `Direction` type accepts one of its declared cases. A string such as `'East'` is not itself a `Direction` case and cannot be passed to the typed function.

Enums were introduced in PHP 8.1. A case is accessed with the enum name and `::`, for example `Direction::North`.

## Pure enums

An enum without a backing type is called a pure enum. Its cases have no scalar value attached to them. A case is a singleton object of its enum type:

~~~php
enum Direction
{
    case North;
    case East;
}

$first = Direction::North;
$second = Direction::North;

var_dump($first === $second); // true
var_dump($first instanceof Direction); // true
var_dump($first->name); // North
~~~

All references to `Direction::North` refer to that same enum case. The case has a read-only `name` property. It does not become the string `'North'`; convert it deliberately when a string is needed.

Pure enums work well when your application only needs a known case, for example a direction used internally in program logic.

## Backed enums

A backed enum associates each case with an explicit scalar value. The backing type must be either `string` or `int`, and all cases in one enum use that one type.

~~~php
enum OrderStatus: string
{
    case Draft = 'draft';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Cancelled = 'cancelled';
}
~~~

A backed case still is an enum object. Its read-only `value` property exposes the scalar that was declared:

~~~php
$status = OrderStatus::Paid;

echo $status->name;  // Paid
echo $status->value; // paid
~~~

The case name (`Paid`) and backing value (`paid`) serve different purposes. The name is used in PHP source; the value can be used to exchange or persist a stable scalar. Pick explicit values. PHP does not generate sequential numbers for you.

Every backed case must have an explicit value, and those values must be unique within that enum. You cannot mix backed and pure cases in one enum or use a union backing type such as `int|string`.

## Convert between scalar values and cases

A backed enum automatically has two conversion methods:

- `OrderStatus::from($value)` returns the matching case. If the scalar has no matching case, it throws `ValueError`.
- `OrderStatus::tryFrom($value)` returns the matching case or `null` when the scalar has no match.

~~~php
$status = OrderStatus::from('paid');

$maybeStatus = OrderStatus::tryFrom('returned');
if ($maybeStatus === null) {
    echo 'Unknown status';
}
~~~

Use `from()` when a missing value means the data is invalid and should raise an error. Use `tryFrom()` when you want to handle unknown input or choose a fallback.

These methods accept `int|string` input. Strict and weak typing rules affect whether PHP accepts or coerces a scalar whose type does not match the enum's backing type. A value with the correct scalar type but no matching case still produces `ValueError` from `from()` or `null` from `tryFrom()`. For external input, validate its type and handle the no-match result deliberately.

Do not create your own `from()` or `tryFrom()` method on a backed enum. Those methods are provided by PHP and their names are reserved for that behavior.

## List cases with `cases()`

Both pure and backed enums provide a `cases()` method. It returns a packed array of the cases in declaration order:

~~~php
foreach (OrderStatus::cases() as $status) {
    echo $status->name . ': ' . $status->value . PHP_EOL;
}
~~~

`cases()` is useful for building select options, validation lists, or tests. You cannot define your own `cases()` method on an enum.

## Add behavior with methods

Enums can contain instance methods and static methods. Inside an instance method, `$this` is the case on which the method was called. A `match` expression is a clear way to map each case to a label or behavior.

~~~php
enum OrderStatus: string
{
    case Draft = 'draft';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Cancelled = 'cancelled';

    public function label(): string
    {
        return match ($this) {
            self::Draft => 'Draft order',
            self::Paid => 'Payment received',
            self::Shipped => 'On the way',
            self::Cancelled => 'Cancelled',
        };
    }
}

echo OrderStatus::Shipped->label(); // On the way
~~~

The label is derived behavior, not another mutable property. If a new case is added, a `match` expression without a default arm makes missing handling visible when that code path runs.

Static methods can provide alternative constructors or case-selection rules:

~~~php
enum Size
{
    case Small;
    case Medium;
    case Large;

    public static function forCentimeters(int $size): self
    {
        return match (true) {
            $size < 50 => self::Small,
            $size < 100 => self::Medium,
            default => self::Large,
        };
    }
}
~~~

## Constants and interfaces

Enums can declare constants, including a constant that aliases a case:

~~~php
enum Size
{
    case Small;
    case Medium;
    case Large;

    public const Standard = self::Medium;
}

$defaultSize = Size::Standard;
~~~

An enum may implement one or more interfaces. Every enum case then satisfies the interface contract:

~~~php
interface HasLabel
{
    public function label(): string;
}

enum Priority implements HasLabel
{
    case Low;
    case Normal;
    case High;

    public function label(): string
    {
        return match ($this) {
            self::Low => 'Low priority',
            self::Normal => 'Normal priority',
            self::High => 'High priority',
        };
    }
}
~~~

Enums may also use traits, but a trait used by an enum cannot declare properties. It may provide methods, static methods, and constants.

## Enums have no per-case mutable properties

Enum cases represent fixed values, not separately constructed records. PHP therefore forbids enum properties, constructors, destructors, inheritance, and cloning. Cases cannot be made with `new`.

If each choice needs its own mutable or independently constructed data, use a class or another data model. For static labels and behavior, use enum methods and constants; for metadata that is best kept outside the enum, use a separate map or model.

Enums cannot extend one another. They can share behavior through interfaces and property-free traits.

## JSON and persistence

A backed enum's `value` is often useful for storing a stable string or integer in a database or exchanging it with another system. Store deliberate values such as `'in_progress'` rather than relying on case order or display labels.

When a backed enum is encoded directly with `json_encode()`, PHP represents it by its backing scalar value. A pure enum cannot be JSON encoded by default; it needs an explicit JSON representation, such as implementing `JsonSerializable`. Decoding JSON does not automatically turn a scalar back into an enum case, so map it with `tryFrom()` or `from()` and handle invalid values.

Changing a backing value can affect stored records and external data. Treat backed values as part of a data contract and plan migrations when they change.

## Choosing pure or backed enums

Use a **pure enum** when code needs only a finite set of typed cases:

~~~php
enum Direction
{
    case North;
    case South;
}
~~~

Use a **backed enum** when each case needs a stable `int` or `string` representation for storage, API data, or another scalar boundary:

~~~php
enum PaymentState: string
{
    case Pending = 'pending';
    case Settled = 'settled';
}
~~~

Do not choose a backed enum only to make cases look like constants. If arbitrary values are expected, a string or value object may be a better fit.

## Common mistakes

- **Comparing a pure enum case to its name string:** `Direction::North` is an enum object, not `'North'`.
- **Confusing `name` and `value`:** the case name is PHP's identifier; the backing value is the explicitly assigned scalar.
- **Assuming backed values are auto-generated:** every backed case needs an explicit unique value.
- **Mixing integers and strings:** an enum has one backing type, either `int` or `string`.
- **Using `from()` for uncertain input without handling failure:** it throws `ValueError` for an unknown scalar; use `tryFrom()` when `null` is easier to handle.
- **Assuming `tryFrom()` accepts any input type:** invalid parameter types can still throw `TypeError`.
- **Trying to add properties or construct a case:** enums have fixed singleton cases and cannot store per-instance state.
- **Treating a display label as a permanent backing value:** labels can change; persisted values are data contracts.
- **Expecting JSON decoding to restore an enum automatically:** explicitly map the scalar to a case.
- **Trying to extend an enum:** enums do not support inheritance; use interfaces and property-free traits for shared behavior.

## Key takeaways

- An enum creates a type with a fixed set of cases.
- Pure cases are singleton objects and expose a read-only `name`.
- Backed enums use one scalar type (`int` or `string`) and explicit unique values.
- Use `from()` for required mapping and `tryFrom()` for nullable handling.
- `cases()` returns every case in declaration order.
- Enums can have methods, constants, static methods, interfaces, and property-free traits, but no properties or inheritance.
- Treat persisted backing values as stable data contracts.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [078 - Late static binding](078-late-static-binding.md) | [Q&A](../qnas/079-enumerations-and-backed-enums.md) | [Example](../examples/079-enumerations-and-backed-enums.md) | [080 - Namespaces, imports, and name resolution](080-namespaces-imports-and-name-resolution.md) |
