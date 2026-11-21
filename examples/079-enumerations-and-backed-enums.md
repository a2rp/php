# 079 - Enumerations and backed enums: Example

## Question

How can an enum represent valid order states, provide labels and transition rules, map stored values safely, and still keep pure enum cases distinct from scalar values?

## Complete PHP example

```php
<?php

declare(strict_types=1);

enum Direction
{
    case North;
    case East;
    case South;
    case West;
}

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

    public function canTransitionTo(self $next): bool
    {
        return match ($this) {
            self::Draft => $next === self::Paid || $next === self::Cancelled,
            self::Paid => $next === self::Shipped,
            self::Shipped, self::Cancelled => false,
        };
    }
}

function describeStatus(OrderStatus $status): string
{
    return $status->label();
}

// A pure enum case is a typed singleton object with a name.
$direction = Direction::North;
echo 'Pure enum case name: ' . $direction->name . "\n";
echo 'Same pure case: ';
echo Direction::North === $direction ? 'yes' : 'no';
echo "\n\n";

// A backed enum case is still an enum object, with an explicit scalar value.
$storedValue = 'paid';
$status = OrderStatus::tryFrom($storedValue);

if ($status === null) {
    throw new UnexpectedValueException('Unknown order status.');
}

echo 'Stored status value: ' . $status->value . "\n";
echo 'Case identifier: ' . $status->name . "\n";
echo 'Display label: ' . describeStatus($status) . "\n";
echo 'May ship: ';
echo $status->canTransitionTo(OrderStatus::Shipped) ? 'yes' : 'no';
echo "\n";
echo 'May cancel: ';
echo $status->canTransitionTo(OrderStatus::Cancelled) ? 'yes' : 'no';
echo "\n\n";

// Unknown external values can be handled without throwing ValueError.
$unknownStatus = OrderStatus::tryFrom('refunded');
echo 'Unknown input: ' . ($unknownStatus?->label() ?? 'rejected') . "\n\n";

// cases() returns every case in declaration order.
echo "All order statuses:\n";
foreach (OrderStatus::cases() as $case) {
    echo $case->value . ' - ' . $case->label() . "\n";
}

// Backed enums use their scalar value in JSON by default.
echo "\nJSON representation: " . json_encode($status) . "\n";
```

This example requires PHP 8.1 or newer because it uses enums.

## Expected output

```text
Pure enum case name: North
Same pure case: yes

Stored status value: paid
Case identifier: Paid
Display label: Payment received
May ship: yes
May cancel: no

Unknown input: rejected

All order statuses:
draft - Draft order
paid - Payment received
shipped - On the way
cancelled - Cancelled

JSON representation: "paid"
```

## Explanation

### A pure enum is a finite type

`Direction` has exactly four cases. `$direction` holds the `Direction::North` case object, not the string `'North'`. Its `name` property gives the declared identifier. Reusing the same enum case refers to the same singleton, so strict identity comparison returns `true`.

### Backing values are explicit scalar representations

`OrderStatus` is backed by `string`. Each case has one unique value, such as `'paid'`. The case name (`Paid`) and backing value (`paid`) are separate: code can use the case name, while a database or API can use the stable scalar value.

### Type declarations make valid states explicit

`describeStatus()` accepts an `OrderStatus`, not an arbitrary string. PHP callers must map incoming scalar data to an enum case before passing it to this function.

### `tryFrom()` handles unknown values

The stored value `'paid'` maps to `OrderStatus::Paid`. The invalid value `'refunded'` returns `null`, so the example can show a validation result instead of raising `ValueError`.

`from()` is available when the value must match a case; it throws `ValueError` if no case exists. Both conversion methods still require an acceptable scalar argument type.

### Methods keep labels and rules with the cases

`label()` uses `match ($this)` to select a display label. `canTransitionTo()` keeps the example's order-state rules alongside the enum. Both methods return derived behavior without storing mutable per-case properties.

In a real application, more complex workflow rules may belong in a separate service. The enum is a good fit here because the small set of transitions is fixed and directly tied to each status.

### `cases()` provides the declared options

The loop displays every order status in declaration order. Applications can use this list for select inputs or validation, then display labels with `label()` while persisting the backing values.

### JSON uses the backing scalar

Because `OrderStatus` is backed by a string, `json_encode($status)` produces the JSON string `"paid"`. Decoding that JSON returns a string; it does not automatically create an enum case. Map the decoded value through `tryFrom()` or `from()`.

## Try these changes

1. Add a `Returned` case with a unique string value. What output appears in the `cases()` loop?
2. Add a new `Direction` case. Does the pure enum case expose a `value` property?
3. Change `$storedValue` to `'draft'`. Which label and transition results change?
4. Change `$storedValue` to `'refunded'`. Why does the example not throw `ValueError`?
5. Replace `tryFrom()` with `from()` for the unknown value. Which exception is thrown?
6. Change one backing value. Which database records or API clients may need a migration?
7. Add a `HasLabel` interface and implement it on `OrderStatus`. What contract does that give callers?

## Key takeaways

- Pure enum cases are singleton objects and do not have scalar values.
- Backed enum cases combine a typed case with one explicit unique `int` or `string` value.
- `tryFrom()` is useful when an external scalar may not match a case.
- Methods and `match` can keep labels and small fixed rules with the enum.
- `cases()` lists every case, and backed enum JSON output uses its scalar value by default.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [078 - Late static binding](../lessons/078-late-static-binding.md) | [Lesson](../lessons/079-enumerations-and-backed-enums.md) | [Q&A](../qnas/079-enumerations-and-backed-enums.md) | [080 - Namespaces, imports, and name resolution](../lessons/080-namespaces-imports-and-name-resolution.md) |
