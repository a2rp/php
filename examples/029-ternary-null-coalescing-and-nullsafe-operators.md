# 029 - Ternary, Null Coalescing, and Nullsafe Operators: Complete Example

This command-line example chooses values with ternary operators, supplies defaults for missing or `null` data, and reads values through optional object chains. It also demonstrates why `?:` and `??` behave differently for the string `'0'`.

## Question

How can a PHP program choose a display value, preserve meaningful falsey values, initialize optional settings, and safely read a city from a user profile that might be absent?

## Goal

Create this small command-line project:

~~~text
ternary-null-example/
└── index.php
~~~

The program will:

- Choose a status label with a full ternary
- Use a shorthand ternary for an empty nickname
- Compare `?:` and `??` with the string `'0'`
- Read settings with `??` and initialize them with `??=`
- Read an optional city and call a method with `?->`

## Complete code

Create `index.php` with this code:

~~~php
<?php
declare(strict_types=1);

class Address
{
    public function __construct(public string $city)
    {
    }
}

class UserProfile
{
    public function __construct(
        public string $name,
        public ?Address $address = null,
        public ?string $nickname = null,
    ) {
    }

    public function getLabel(): string
    {
        return $this->name;
    }
}

$profiles = [
    new UserProfile('Asha', new Address('Jaipur'), ''),
    new UserProfile('Dev'),
    null,
];

echo "=== 1. Ternary expressions ===\n";

$isApproved = true;
$status = $isApproved ? 'Approved' : 'Needs review';
$nickname = $profiles[0]->nickname ?: $profiles[0]->name;

$page = '0';
$pageWithTernaryFallback = $page ?: 'home';
$pageWithNullFallback = $page ?? 'home';

echo "Approval: {$status}\n";
echo "Nickname: {$nickname}\n";
echo "Page with ?: {$pageWithTernaryFallback}\n";
echo "Page with ??: {$pageWithNullFallback}\n";

echo "\n=== 2. Missing and null settings ===\n";

$settings = [
    'theme' => null,
    'show_tips' => false,
];

$theme = $settings['theme'] ?? 'light';
$showTips = $settings['show_tips'] ?? true;
$language = $settings['language'] ?? 'en';

$settings['theme'] ??= 'light';
$settings['language'] ??= 'en';
$settings['show_tips'] ??= true;

echo "Theme: {$theme}\n";
echo 'Show tips: ', var_export($showTips, true), "\n";
echo "Language: {$language}\n";
echo "Updated settings:\n";
var_export($settings);
echo "\n";

echo "\n=== 3. Nullsafe object access ===\n";

foreach ($profiles as $profile) {
    $name = $profile?->name ?? 'Guest';
    $city = $profile?->address?->city ?? 'Unknown city';

    echo "{$name} lives in {$city}\n";
}

$firstProfileLabel = $profiles[0]?->getLabel() ?? 'Guest';
$missingProfileLabel = $profiles[2]?->getLabel() ?? 'Guest';

echo "First profile label: {$firstProfileLabel}\n";
echo "Missing profile label: {$missingProfileLabel}\n";
~~~

## Expected output

~~~text
=== 1. Ternary expressions ===
Approval: Approved
Nickname: Asha
Page with ?: home
Page with ??: 0

=== 2. Missing and null settings ===
Theme: light
Show tips: false
Language: en
Updated settings:
array (
  'theme' => 'light',
  'show_tips' => false,
  'language' => 'en',
)

=== 3. Nullsafe object access ===
Asha lives in Jaipur
Dev lives in Unknown city
Guest lives in Unknown city
First profile label: Asha
Missing profile label: Guest
~~~

## How it works

### 1. Choose a value with ternary expressions

`$isApproved ? 'Approved' : 'Needs review'` returns one of two labels based on a boolean condition. The nickname expression uses shorthand ternary: the empty nickname is falsey, so the profile name becomes the display value.

The page value is the string `'0'`. PHP treats it as falsey, so `$page ?: 'home'` returns `home`. Null coalescing checks only whether the value is missing or `null`, so `$page ?? 'home'` preserves `0`.

### 2. Read and initialize settings

The theme exists but contains `null`, so `??` supplies `light`. The `show_tips` setting contains `false`, which is preserved; `??` does not replace falsey non-null values. The missing language key uses the fallback `en` without an undefined-key warning.

The `??=` lines initialize the missing or null entries in `$settings`. The existing `false` value for `show_tips` remains unchanged.

### 3. Read optional objects safely

The first profile has an address, so its city is `Jaipur`. The second profile exists but has no address; the nullsafe chain returns `null`, and `??` supplies `Unknown city`. The last list entry is itself `null`, so neither the property reads nor the method call continue.

The call to `getLabel()` demonstrates nullsafe method access. The first profile returns its name, while the missing profile falls back to `Guest`.

## Run it

Use PHP 8.0 or later because the example uses the nullsafe operator and constructor property promotion.

~~~sh
php --version
php index.php
~~~

The code is displayed here as a learning example. Run it locally if you want to compare your output with the expected output above.

## Make a change

1. Set `$isApproved` to `false` and check the status label.
2. Change the first nickname from an empty string to `Asha R.`.
3. Set `$page` to `2`, `''`, and `null`; compare the two fallback expressions each time.
4. Change `show_tips` to `null`, then to `0`, and observe what `??` and `??=` do.
5. Add a `country` property to `Address` and read it with a nullsafe chain.
6. Add a profile with an address but a different city.
7. Change the final list item from `null` to a profile without an address.
8. Replace the status ternary with an `if` statement and compare readability.

## Common mistakes

- Using `?:` when a valid value such as `0`, `'0'`, or `false` must be preserved
- Assuming `??` replaces every falsey value
- Assuming `??=` replaces an existing `false` value
- Forgetting that `?->` requires PHP 8.0 or later
- Expecting nullsafe access to hide a misspelled property or method on a non-null object
- Trying to assign through a nullsafe chain
- Writing deeply nested ternaries when an `if` statement would be clearer

## Key takeaways

- A full ternary selects a value using a condition; shorthand ternary selects based on truthiness.
- Use `??` and `??=` when only missing or `null` values should trigger a fallback.
- Nullsafe property and method access short-circuit an object chain at `null`.
- Combining `?->` with `??` is useful when the final display value should never be null.
- This example requires PHP 8.0 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [028 - String, Array, Bitwise, and Other Operators](../lessons/028-string-array-bitwise-and-other-operators.md) | [Read this lesson](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) | [Open Q&A](../qnas/029-ternary-null-coalescing-and-nullsafe-operators.md) | [030 - Conditional Branching with if and switch](../lessons/030-conditional-branching-with-if-and-switch.md) |
