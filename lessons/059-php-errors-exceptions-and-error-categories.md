# 059 - PHP Errors, Exceptions, and Error Categories

PHP reports problems in two main ways: diagnostic errors with severity levels, and Throwable objects that are thrown and caught. Knowing which category a failure belongs to helps you choose the right response and avoid treating every problem as the same kind of error.

## Two reporting systems

### Diagnostic errors

A diagnostic is reported with a severity such as E_WARNING or E_DEPRECATED. PHP may display or log it according to configuration. Most non-fatal diagnostics allow execution to continue, though the operation that raised one may return false or an unusable result.

### Throwable objects

A Throwable object is thrown with throw and can be caught with try/catch. The two main branches are Exception and Error. Exceptions often represent failures an application can handle; Error subclasses often indicate a programming or engine-level problem.

~~~text
Throwable
├── Exception
│   └── application and library exceptions
└── Error
    ├── TypeError
    ├── ValueError
    └── other engine errors
~~~

These mechanisms are related but not interchangeable. For example, E_WARNING is a diagnostic level, while TypeError is a Throwable object. Some legacy engine errors that used to be diagnostics have changed category across PHP versions.

## Diagnostic error levels

PHP error levels are named constants used as bit flags. They describe when and how a diagnostic was raised; they are not exception classes.

| Level | General meaning | Typical effect |
| --- | --- | --- |
| E_ERROR | Fatal runtime diagnostic | Script execution stops |
| E_WARNING | Runtime warning | Execution usually continues |
| E_PARSE | Parser syntax error | The affected file cannot execute |
| E_NOTICE | Runtime notice | Signals something that may indicate a problem |
| E_CORE_ERROR | Fatal error during PHP core startup | Happens before normal script execution |
| E_CORE_WARNING | Warning during PHP core startup | May occur before normal script execution |
| E_COMPILE_ERROR | Fatal compilation error | The affected code cannot execute |
| E_COMPILE_WARNING | Compilation warning | The affected code may still run |
| E_USER_ERROR | User-generated fatal diagnostic | Legacy; use an exception or exit instead |
| E_USER_WARNING | User-generated warning | Created by trigger_error() |
| E_USER_NOTICE | User-generated notice | Created by trigger_error() |
| E_DEPRECATED | PHP reports use of behavior that may be removed in a future version | Update the code before upgrading |
| E_USER_DEPRECATED | Application-generated deprecation diagnostic | Created by trigger_error() |
| E_STRICT | Historical compatibility suggestions | Unused and deprecated as of PHP 8.4 |
| E_RECOVERABLE_ERROR | Legacy recoverable engine diagnostic | Effectively unused in current PHP |

The E_USER_* levels are for diagnostics deliberately raised by application code. They do not create custom Throwable classes. For new code, prefer a meaningful exception when a caller should handle a failure.

### Version-sensitive levels

Diagnostic categories and severities can change between PHP releases. In PHP 8.4, E_STRICT became unused and deprecated. Passing E_USER_ERROR to trigger_error() also became deprecated; new code should throw an exception or call exit() when terminating the script is intentional.

E_ALL is a bitmask containing the diagnostic levels. Its numeric value can change as PHP evolves. Use the named constants and bitwise operators; do not copy a numeric mask from an old tutorial.

## Combining and filtering levels

Use bitwise OR to combine levels and bitwise AND to check whether a level is included:

~~~php
<?php

$levels = E_WARNING | E_USER_WARNING;

if (($levels & E_WARNING) !== 0) {
    echo "Warnings are included.\n";
}
~~~

To exclude a level from a mask, use bitwise NOT with AND:

~~~php
<?php

$levelsWithoutDeprecations = E_ALL & ~E_DEPRECATED;
~~~

Filtering should be deliberate. In development, reporting E_ALL helps reveal compatibility problems early. Avoid excluding deprecations just to silence them during upgrades.

## Configure reporting, display, and logging separately

These settings have different jobs:

- **error_reporting** selects which diagnostic levels PHP reports.
- **display_errors** controls whether diagnostics are printed into program output.
- **log_errors** controls whether diagnostics are written to the configured error log.

For development, use E_ALL and display diagnostics so problems are visible. For production web applications, keep display_errors disabled and enable protected logging. Error messages can reveal source paths, SQL, configuration, or secret values.

Set important reporting configuration in php.ini or the relevant server configuration. Some problems happen before a PHP script starts, so a runtime call near the top of the script cannot report every startup or parse failure. A runtime setting also cannot execute if a fatal error prevents the script from reaching it.

Example development setting:

~~~ini
error_reporting = E_ALL
display_errors = On
log_errors = On
~~~

Example production web setting:

~~~ini
error_reporting = E_ALL
display_errors = Off
log_errors = On
~~~

The exact log destination belongs to the deployment configuration and should be protected. Do not include credentials, access tokens, or unnecessary personal data in logs.

## What error_reporting() does

error_reporting() can set the current request's diagnostic mask or return the current mask:

~~~php
<?php

error_reporting(E_ALL);
$currentMask = error_reporting();
~~~

It does not convert diagnostics into exceptions. It also does not determine whether the message appears in a web response; that depends on display_errors and the active handler.

A custom handler may still be called when a diagnostic is suppressed with @. If a handler converts diagnostics into ErrorException, it should check whether the severity is enabled before throwing. Avoid @ as a normal control-flow or error-handling strategy.

## User-generated diagnostic levels

trigger_error() can raise selected user-level diagnostics:

~~~php
<?php

if ($profileName === '') {
    trigger_error(
        'The profile name is empty.',
        E_USER_WARNING
    );
}
~~~

Use E_USER_WARNING, E_USER_NOTICE, or E_USER_DEPRECATED only when a diagnostic message is appropriate. As of PHP 8.4, using E_USER_ERROR with trigger_error() is deprecated. Prefer a typed exception for a failure that callers need to catch:

~~~php
<?php

if ($profileName === '') {
    throw new InvalidArgumentException(
        'A profile name is required.'
    );
}
~~~

On PHP 8 and later, trigger_error() throws ValueError if the requested level is not one of its allowed E_USER_* levels. Check the manual for the PHP version your project supports before relying on version-specific behavior.

## Diagnostics and exceptions at a glance

| Situation | Category | Common handling |
| --- | --- | --- |
| A function emits E_WARNING and returns false | Diagnostic | Check the return value; log or handle the failure |
| Code uses a deprecated feature | E_DEPRECATED diagnostic | Update the code and test against supported versions |
| Application rejects an invalid argument | Exception | Throw InvalidArgumentException and catch at a useful layer |
| A value has the wrong runtime type | TypeError Throwable | Fix the programming error or validate the boundary |
| The source contains invalid syntax | E_PARSE diagnostic | Fix the source; runtime code in that file cannot catch it |
| PHP cannot allocate memory for an operation | E_ERROR diagnostic | Inspect protected logs and address the resource problem |

Do not assume every failure can be caught by the same mechanism. Many diagnostics can be routed through a custom error handler, but fatal and parse-level errors have important limits. Throwable objects use exception control flow instead.

## Legacy and compatibility notes

Old tutorials may show categories or behavior that no longer applies:

- E_STRICT is unused and deprecated as of PHP 8.4.
- E_USER_ERROR passed to trigger_error() is deprecated as of PHP 8.4.
- E_RECOVERABLE_ERROR is effectively unused in current PHP.
- PHP versions may change a condition from a diagnostic into a Throwable, or change its severity.
- E_ALL numeric values are not a stable value to hard-code.

When maintaining older applications, read the migration notes for the PHP versions being adopted and test with E_ALL. Do not suppress a new diagnostic until its cause is understood.

## Common mistakes

- Calling every PHP problem an exception.
- Treating E_WARNING as a Throwable object.
- Assuming every diagnostic stops execution or that every warning is harmless.
- Hard-coding the numeric value of E_ALL or a severity mask.
- Assuming error_reporting() controls display_errors and log_errors.
- Displaying internal diagnostics to production users.
- Using E_USER_ERROR with trigger_error() in new code on PHP 8.4 or later.
- Treating E_STRICT as a useful modern diagnostic level on PHP 8.4 or later.
- Ignoring E_DEPRECATED during development and discovering incompatibilities during an upgrade.
- Assuming a runtime handler can catch syntax errors in the same file.

## Try it yourself

1. Print the current error_reporting() mask after setting E_ALL.
2. Combine E_WARNING and E_USER_WARNING with the bitwise OR operator.
3. Check whether E_NOTICE is present in a mask using bitwise AND.
4. Exclude E_DEPRECATED from a temporary mask and explain why this should not replace fixing the deprecated code.
5. Compare a function's false return value after a warning with a thrown InvalidArgumentException.
6. On PHP 8.4 or later, replace a trigger_error(..., E_USER_ERROR) example with an exception.
7. Configure a local development environment to show E_ALL and a production-like environment to log without displaying details.
8. Explain why code in a file cannot catch a parse error that prevents the file from executing.

## Key takeaways

- Diagnostic errors have severity levels; Throwable objects follow throw/catch control flow.
- E_WARNING, E_PARSE, E_DEPRECATED, and TypeError are different categories.
- Use named error constants and bitwise masks rather than hard-coded numeric levels.
- Development should surface E_ALL; production should log privately and avoid exposing internals.
- PHP 8.4 deprecates E_STRICT and E_USER_ERROR use with trigger_error().
- Error categories can change between PHP versions, so check compatibility when upgrading.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [058 - try, catch, finally, and throwing exceptions](058-try-catch-finally-and-throwing-exceptions.md) | [Open Q&A](../qnas/059-php-errors-exceptions-and-error-categories.md) | [Open example](../examples/059-php-errors-exceptions-and-error-categories.md) | [060 - Error reporting, handlers, and assertions](060-error-reporting-handlers-and-assertions.md) |
