# 029 - Ternary, Null Coalescing, and Nullsafe Operators

PHP offers compact expressions for choosing between values and safely handling optional data. The ternary operator chooses a value from a condition, null coalescing supplies a default for a missing or `null` value, and the nullsafe operator stops a property or method chain when an object is `null`.

These operators make common cases shorter. Use them when the expression stays easy to read; use a regular `if` statement when the decision needs several steps or explanations.

## What you will learn

- How to write a full ternary expression and a shorthand ternary expression
- How truthiness affects the shorthand ternary operator
- How `??` differs from `?:` when values such as `false`, `0`, and an empty string are valid
- How to set defaults with `??=`
- How to use `?->` with optional objects
- How to avoid precedence and readability problems
- Which PHP versions introduced these operators

## The full ternary operator

The ternary operator evaluates a condition and returns one of two expressions:

~~~php
<?php
$age = 20;
$category = $age >= 18 ? 'Adult' : 'Minor';

echo $category;
~~~

Output:

~~~text
Adult
~~~

The general form is:

~~~php
$condition ? $valueWhenTrue : $valueWhenFalse
~~~

A ternary is an expression, so it can be assigned, returned, or used as an argument. Both branches should produce values that make sense for the same purpose.

~~~php
<?php
function accessLabel(bool $allowed): string
{
    return $allowed ? 'Access granted' : 'Access denied';
}
~~~

### Choose `if` when the branches do more

A ternary is useful for one small choice. A normal conditional is clearer when branches contain multiple statements or have different side effects.

~~~php
<?php
if ($paymentIsValid) {
    savePayment();
    sendReceipt();
} else {
    showPaymentError();
}
~~~

Avoid placing assignments, output, or other side effects in a ternary just to save a few lines.

## The shorthand ternary operator

The shorthand form omits the middle expression:

~~~php
<?php
$nickname = '';
$label = $nickname ?: 'No nickname';

echo $label;
~~~

Output:

~~~text
No nickname
~~~

Its general form is:

~~~php
$firstValue ?: $fallback
~~~

PHP returns the first value when it is truthy; otherwise it returns the fallback. The first expression is evaluated once. This is often called the Elvis operator because of its shape.

This operator uses PHP truthiness. The following values are falsey:

- `false`
- `0` and `0.0`
- `''` and `'0'`
- `[]`
- `null`

That matters when a falsey value is meaningful. For example, if the string `'0'` is a valid page number, `?:` will replace it with the fallback. Use `??` when only missing and `null` values should trigger a default.

~~~php
<?php
$page = '0';

$withTernaryFallback = $page ?: 'home';
$withNullFallback = $page ?? 'home';

echo $withTernaryFallback, "\n";
echo $withNullFallback, "\n";
~~~

Output:

~~~text
home
0
~~~

## Null coalescing with `??`

The null coalescing operator returns its left value when that value exists and is not `null`. Otherwise, it returns the right value.

~~~php
<?php
$settings = [
    'theme' => null,
    'show_tips' => false,
];

$theme = $settings['theme'] ?? 'light';
$showTips = $settings['show_tips'] ?? true;
$language = $settings['language'] ?? 'en';

var_dump($theme, $showTips, $language);
~~~

Output:

~~~text
string(5) "light"
bool(false)
string(2) "en"
~~~

`??` uses behavior similar to `isset()`: a missing variable or array key, and a value that is `null`, use the fallback. It does not emit the undefined-variable or undefined-array-key notice for the left operand in these common cases.

A value such as `false`, `0`, `'0'`, `''`, or an empty array is still a value, so `??` preserves it. That makes `??` a better defaulting operator when falsey values are allowed.

### Chain fallbacks

You can chain null coalescing operators. PHP checks from left to right and uses the first value that is set and not `null`.

~~~php
<?php
$displayName = $profile['display_name']
    ?? $profile['username']
    ?? $profile['email']
    ?? 'Guest';
~~~

The right-hand expressions are only needed when the earlier candidates are missing or `null`. Add parentheses when mixing `??` with other operators so the intended grouping is obvious.

~~~php
<?php
$label = 'Page: ' . ($query['page'] ?? '1');
~~~

## Assign a default with `??=`

The null coalescing assignment operator assigns its right-hand value only when the left-hand variable or array element is missing or `null`.

~~~php
<?php
$options = [
    'theme' => null,
    'show_tips' => false,
];

$options['theme'] ??= 'light';
$options['language'] ??= 'en';
$options['show_tips'] ??= true;

var_export($options);
~~~

Output:

~~~text
array (
  'theme' => 'light',
  'show_tips' => false,
  'language' => 'en',
)
~~~

The existing `false` value stays `false`; `??=` is not a truthiness check. It was added in PHP 7.4. The left side must be a writable variable or array element.

## Nullsafe property and method access with `?->`

An object-valued property or method result can be absent. Before PHP 8.0, code often checked every object in a chain before continuing. The nullsafe operator makes this case shorter:

~~~php
<?php
$city = $user?->address?->city;
~~~

If `$user` is `null`, PHP returns `null` for the chain. If `$user` exists but its `address` is `null`, the remaining chain is also skipped. If the chain completes, `$city` contains the city value.

A method can be part of the chain as well:

~~~php
<?php
$city = $user?->getAddress()?->getCity();
~~~

Nullsafe access is helpful when `null` is an expected outcome. A null result can be combined with `??` to provide a display value:

~~~php
<?php
$cityLabel = $user?->address?->city ?? 'Unknown city';
~~~

The nullsafe operator was introduced in PHP 8.0. It does not suppress errors for an undefined property on a non-null object, a misspelled method, or an invalid method call. It only short-circuits when the value to its left is `null`.

### Nullsafe access is read-only

Use `?->` to read a value or call a method. It cannot be used as the target of an assignment, reference, or `unset()` operation.

~~~php
<?php
// Reading is valid:
$city = $user?->address?->city;

// This is invalid:
// $user?->address?->city = 'Pune';
~~~

If you need to change an object, first check that it exists, then assign through a regular property or method.

## Operator precedence and parentheses

Compact operators can be confusing when combined. Parentheses make the intended order explicit.

~~~php
<?php
$page = $query['page'] ?? '1';
$title = 'Page ' . $page;

$summary = ($user?->name ?? 'Guest') . ' opened the page';
~~~

In PHP 8 and later, the full ternary operator is non-associative: nested ternaries must be parenthesized. Older versions evaluated unparenthesized nested full ternaries left-to-right, and PHP 7.4 deprecated that behavior.

~~~php
<?php
$result = $firstCondition
    ? 'first'
    : ($secondCondition ? 'second' : 'fallback');
~~~

Parentheses work across PHP versions and make the nesting easier to read. For several conditions, prefer `if`/`elseif` or `match` where appropriate instead of a chain of nested ternaries.

## Which operator should you choose?

| Need | Operator | What triggers the fallback? |
| --- | --- | --- |
| Choose one of two values based on a condition | `condition ? yes : no` | The condition is false |
| Use a fallback for any falsey value | `value ?: fallback` | `false`, `0`, `'0'`, `''`, `[]`, or `null` |
| Use a fallback only when a value is missing or `null` | `value ?? fallback` | Missing or `null` |
| Set a variable or array element only when missing or `null` | `value ??= fallback` | Missing or `null` |
| Read through an optional object chain | `object?->member` | A value to the left of `?->` is `null` |

## Version support

- `?:` and `??` are available from PHP 7.0.
- `??=` is available from PHP 7.4.
- `?->` is available from PHP 8.0.
- PHP 8.0 made unparenthesized nested full ternary expressions invalid; parenthesize nested ternaries for clear, compatible code.

Use syntax supported by the PHP version your application declares. A newer operator in one file can make that file fail to parse on an older runtime, even if execution would never reach that line.

## Try it yourself

1. Set `$age` to `16`, then use a full ternary to produce `Minor`.
2. Compare `'' ?: 'fallback'` with `'' ?? 'fallback'`.
3. Try the same comparison with `'0'`, `false`, and `null`.
4. Read a missing `theme` key with `??` and confirm that no warning is produced.
5. Use `??=` to add a missing key while preserving a key whose value is `false`.
6. Build an object with an optional address and read its city with `?->`.
7. Add `?? 'Unknown'` to the nullsafe chain so it always produces a display string.
8. Parenthesize a nested ternary and then rewrite it with `if`/`elseif`.
9. Explain which operators need PHP 7.4 or PHP 8.0.

## Key takeaways

- The full ternary operator chooses a value from a condition; the shorthand form chooses the first truthy value or a fallback.
- `?:` treats values such as `0`, `'0'`, `false`, and `''` as falsey.
- `??` and `??=` use missing-or-null behavior, preserving other falsey values.
- `?->` safely short-circuits an object chain when an object value is `null`; it does not hide unrelated errors.
- Parenthesize nested or mixed expressions, and use `if` when a compact expression becomes difficult to understand.
- `??` is from PHP 7.0, `??=` from PHP 7.4, and `?->` from PHP 8.0.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [028 - String, Array, Bitwise, and Other Operators](028-string-array-bitwise-and-other-operators.md) | [Open Q&A](../qnas/029-ternary-null-coalescing-and-nullsafe-operators.md) | [Open example](../examples/029-ternary-null-coalescing-and-nullsafe-operators.md) | [030 - Conditional Branching with if and switch](030-conditional-branching-with-if-and-switch.md) |
