# Constructor property promotion - Example

## Goal

Create a customer profile with promoted properties, validate and normalize constructor input, and return a new profile when a name changes.

## Requirements

This example uses readonly property promotion and requires PHP 8.1 or later. Ordinary constructor property promotion requires PHP 8.0; readonly properties were added in PHP 8.1.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

final class CustomerProfile
{
    private readonly DateTimeImmutable $registeredAt;
    private string $email;

    public function __construct(
        public readonly int $id,
        public string $name,
        DateTimeImmutable $registeredAt,
        string $email,
        public ?string $nickname = null,
    ) {
        if ($id < 1) {
            throw new InvalidArgumentException(
                'Customer ID must be positive.'
            );
        }

        $normalizedName = trim($name);

        if ($normalizedName === '') {
            throw new InvalidArgumentException(
                'Customer name cannot be empty.'
            );
        }

        // A mutable promoted property can be normalized in the constructor.
        $this->name = $normalizedName;

        $normalizedEmail = strtolower(trim($email));

        if (filter_var($normalizedEmail, FILTER_VALIDATE_EMAIL) === false) {
            throw new InvalidArgumentException(
                'Email address is invalid.'
            );
        }

        // This input is ordinary, not promoted. Store the normalized value.
        $this->email = $normalizedEmail;
        $this->registeredAt = $registeredAt;
    }

    public function email(): string
    {
        return $this->email;
    }

    public function registeredAt(): DateTimeImmutable
    {
        return $this->registeredAt;
    }

    public function withName(string $name): self
    {
        return new self(
            id: $this->id,
            name: $name,
            registeredAt: $this->registeredAt,
            email: $this->email,
            nickname: $this->nickname,
        );
    }
}

$profile = new CustomerProfile(
    id: 42,
    name: '  Ada Lovelace  ',
    registeredAt: new DateTimeImmutable(
        '2026-03-08T09:15:00+00:00'
    ),
    email: ' ADA@EXAMPLE.COM ',
    nickname: 'Ada',
);

$renamedProfile = $profile->withName('Ada King');

echo 'ID: ' . $profile->id . PHP_EOL;
echo 'Name: ' . $profile->name . PHP_EOL;
echo 'Email: ' . $profile->email() . PHP_EOL;
echo 'Nickname: ' . ($profile->nickname ?? 'none') . PHP_EOL;
echo 'Registered: '
    . $profile->registeredAt()->format('Y-m-d H:i')
    . PHP_EOL;
echo 'Original: ' . $profile->name . PHP_EOL;
echo 'Copy: ' . $renamedProfile->name . PHP_EOL;

// This write is invalid because id is readonly and already initialized:
// $profile->id = 43;
~~~

## Expected output

~~~text
ID: 42
Name: Ada Lovelace
Email: ada@example.com
Nickname: Ada
Registered: 2026-03-08 09:15
Original: Ada Lovelace
Copy: Ada King
~~~

## Explanation

### Which parameters are promoted?

The constructor promotes <code>$id</code>, <code>$name</code>, and <code>$nickname</code>. Their modifiers and types define properties, and the constructor arguments initialize those properties automatically.

The <code>$registeredAt</code> and <code>$email</code> parameters have no visibility modifier, so they are ordinary parameters. They do not create properties automatically.

### Why is email stored explicitly?

The constructor trims and lowercases the email, validates the normalized value, and then assigns it to the private <code>$email</code> property. Promoting the input would first store the unnormalized string as a property, which is not the representation this class wants to expose.

The explicit property makes the intended invariant clear: a profile stores only a validated, normalized email address.

### Why can the name be assigned twice?

Promotion initializes <code>$name</code> before the constructor body begins. It is a regular mutable property, so the constructor can replace the initial value with its trimmed form.

The <code>$id</code> property is readonly. It is initialized by promotion and cannot be assigned a second time. The constructor validates it without rewriting it.

### Why does the replacement method return a new profile?

<code>withName()</code> uses named arguments to construct another profile. The original remains named <code>Ada Lovelace</code>, while the returned object contains <code>Ada King</code>.

Named arguments also show that constructor parameter names are part of the calling API. Keep public parameter names clear and stable.

## Try these changes

1. Omit <code>nickname</code> and confirm that it becomes <code>null</code>.
2. Pass an empty name and observe the validation exception.
3. Pass an invalid email and observe the email validation exception.
4. Change the email input to mixed case and confirm that the stored result is lowercase.
5. Add a promoted <code>public readonly string $countryCode</code> and pass it by name.
6. Convert <code>$name</code> to readonly. What should change in the constructor's normalization logic?
7. Expand one promoted property into an explicit property declaration and constructor assignment.

## Key takeaways

- Promotion creates properties for parameters with property modifiers.
- Ordinary parameters can be transformed before their values are stored.
- Promotion initializes properties before the constructor body runs.
- Mutable promoted properties can be normalized in the body; readonly ones cannot be assigned twice.
- Named constructor arguments use parameter names, so those names are part of the public API.
- Readonly promotion requires PHP 8.1 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [073 - Readonly properties and classes](../lessons/073-readonly-properties-and-classes.md) | [Lesson](../lessons/074-constructor-property-promotion.md) | [Q&A](../qnas/074-constructor-property-promotion.md) | [075 - Property hooks](../lessons/075-property-hooks.md) |
