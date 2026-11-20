# PHP Errors, Exceptions, and Error Categories: Example

## Question

How can you inspect PHP error categories, combine severity masks, and convert a selected diagnostic into an exception without confusing diagnostics with normal exception handling?

## Complete example

This standalone example works with PHP 8.0 and later. Save it as `error-categories.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

error_reporting(E_ALL);

$selectedSeverities = E_WARNING | E_USER_WARNING;

echo 'E_WARNING selected: '
    . (($selectedSeverities & E_WARNING) !== 0 ? 'yes' : 'no')
    . PHP_EOL;

echo 'E_USER_WARNING selected: '
    . (($selectedSeverities & E_USER_WARNING) !== 0 ? 'yes' : 'no')
    . PHP_EOL;

// This is only a mask demonstration. It does not change the runtime setting
// or recommend hiding deprecation notices.
$maskWithoutDeprecations = E_ALL & ~E_DEPRECATED;

echo 'E_DEPRECATED excluded from demonstration mask: '
    . (($maskWithoutDeprecations & E_DEPRECATED) === 0 ? 'yes' : 'no')
    . PHP_EOL;

// Convert this selected user-generated warning into ErrorException.
// Other severities are left to PHP's normal error handling.
set_error_handler(
    static function (
        int $severity,
        string $message,
        string $file,
        int $line
    ): bool {
        // Respect the active error_reporting mask, including when @ is used.
        if ((error_reporting() & $severity) === 0) {
            return false;
        }

        throw new ErrorException($message, 0, $severity, $file, $line);
    },
    E_USER_WARNING
);

try {
    trigger_error('Optional profile value is missing.', E_USER_WARNING);
} catch (ErrorException $exception) {
    echo 'Converted diagnostic: ' . $exception->getMessage() . PHP_EOL;
} finally {
    restore_error_handler();
}

// Invalid application input is better represented as an exception directly;
// it is not a PHP diagnostic.
try {
    $profileName = '';

    if ($profileName === '') {
        throw new InvalidArgumentException('A profile name is required.');
    }
} catch (InvalidArgumentException $exception) {
    echo 'Caught Throwable: ' . $exception::class . PHP_EOL;
}
~~~

## Expected output

~~~text
E_WARNING selected: yes
E_USER_WARNING selected: yes
E_DEPRECATED excluded from demonstration mask: yes
Converted diagnostic: Optional profile value is missing.
Caught Throwable: InvalidArgumentException
~~~

## Explanation

### 1. Error levels are bit flags

The constants such as `E_WARNING`, `E_USER_WARNING`, and `E_DEPRECATED` are severity bit flags. Combine selected levels with the bitwise OR operator (`|`). Check whether a level is present with bitwise AND (`&`).

Use the named constants instead of hard-coding their numeric values. Numeric values and the set of meaningful levels can vary across PHP versions.

### 2. A diagnostic can be converted into an exception

PHP normally reports a warning as a diagnostic. The custom handler above is registered only for `E_USER_WARNING`; it turns that selected diagnostic into an `ErrorException`. The `catch` block can then handle it using ordinary exception flow.

The handler checks the active `error_reporting()` mask before throwing. It returns `false` for a masked level so PHP can apply its normal handling. The `finally` block restores the previous handler even if the code throws.

This conversion is a deliberate application choice. It does not mean that every PHP warning is automatically an exception.

### 3. Invalid input can be an exception directly

The example throws `InvalidArgumentException` for invalid application input. This is different from reporting a runtime diagnostic: the program itself knows that the input violates a rule and chooses a suitable exception type.

Diagnostics and exceptions can both need handling, but they describe different situations. A warning also does not tell you whether a function returned `false`; check that function's documented return value separately.

### 4. Error levels can change across versions

The mask involving `E_DEPRECATED` is shown only to practice bitwise operations. It does not change PHP's active configuration. Avoid suppressing deprecations as a general fix; use them to find code that needs updating.

PHP 8.4 deprecates `E_STRICT` and `E_USER_ERROR`. Do not use `E_USER_ERROR` to terminate application logic; throw an appropriate exception instead. The example intentionally does not trigger that deprecated level.

### 5. Development and production settings

For local development, reporting all relevant levels makes problems easier to find. On a production server, configure errors to be logged while keeping detailed error display disabled for visitors. Configure these settings in the appropriate PHP or hosting environment rather than relying on this demonstration script.

## Try it yourself

1. Add `E_NOTICE` to `$selectedSeverities` and print whether it is selected.
2. Change the triggered diagnostic to `E_USER_NOTICE`, then update the handler's accepted-level mask to match.
3. Remove the custom handler and observe that the warning is handled by PHP's normal diagnostic configuration.
4. Change the invalid value and decide which exception type communicates the violated rule most clearly.

## Key takeaways

- Error-level constants are bit flags; combine them with `|` and inspect them with `&`.
- Keep diagnostic reporting separate from normal exception handling unless you intentionally install a converting handler.
- Check a function's return value independently of whether it also reports a diagnostic.
- Prefer named constants and version-aware code over hard-coded severity numbers.
- Log production errors without exposing sensitive details to visitors.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [058 - PHP exceptions and throwable types](../lessons/058-php-exceptions-and-throwable-types.md) | [Q&A](../qnas/059-php-errors-exceptions-and-error-categories.md) | [Lesson](../lessons/059-php-errors-exceptions-and-error-categories.md) | [060 - Error reporting, handlers, and assertions](../lessons/060-error-reporting-handlers-and-assertions.md) |
