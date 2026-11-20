# 031 - The match Expression

PHP's `match` expression chooses a result by comparing one subject with several possible values. It is useful when one value maps to one of several results, such as a status label, command response, or configuration option.

Unlike `switch`, `match` compares with strict identity (`===`), returns a value, and does not fall through to later arms. A match expression should handle every possible value; otherwise PHP throws an `UnhandledMatchError` at runtime.

## What you will learn

- How to write a `match` expression and use its returned value
- How strict matching distinguishes values of different types
- How several values can share one arm
- Why `default` is important and what happens when no arm matches
- How to use `match (true)` for ordered conditions
- How `match` differs from `switch`, `if`, and ternary expressions
- Which PHP version supports `match`

## Basic syntax

A `match` expression has a subject, one or more arms, and a result expression for each arm:

~~~php
<?php
$dayNumber = 6;

$dayName = match ($dayNumber) {
    1 => 'Monday',
    2 => 'Tuesday',
    3 => 'Wednesday',
    4 => 'Thursday',
    5 => 'Friday',
    6 => 'Saturday',
    7 => 'Sunday',
    default => 'Unknown day',
};

echo $dayName;
~~~

Output:

~~~text
Saturday
~~~

Each arm has the form `condition => result`. The `default` arm handles any subject that did not match an earlier arm. Because `match` is an expression, it can be assigned to a variable, returned from a function, or passed to another expression.

~~~php
<?php
function labelForStatus(int $status): string
{
    return match ($status) {
        200 => 'Success',
        404 => 'Not found',
        default => 'Other response',
    };
}
~~~

## Matching is strict

`match` compares the subject and arm conditions with `===`. It does not convert a numeric string to an integer to make a match.

~~~php
<?php
$submittedValue = '2';

$result = match ($submittedValue) {
    2 => 'Integer two',
    '2' => 'String two',
    default => 'Something else',
};

echo $result;
~~~

Output:

~~~text
String two
~~~

This strict behavior is useful for values where the type matters. Validate and normalize external input when the application expects a specific type; do not depend on implicit conversion.

## Several values can share an arm

Separate alternatives with commas when they should produce the same result:

~~~php
<?php
$httpStatus = 201;

$category = match ($httpStatus) {
    200, 201, 204 => 'Success',
    301, 302, 307, 308 => 'Redirect',
    400, 401, 403, 404 => 'Client error',
    500, 502, 503 => 'Server error',
    default => 'Other response',
};

echo $category;
~~~

Output:

~~~text
Success
~~~

A comma between arm conditions means “match any of these values.” It is not the same as writing a sequence of operations.

## There is no fall-through

A matching arm produces its result and the match expression ends. Later arms do not run. There is no `break` keyword in a match expression.

~~~php
<?php
$plan = 'basic';

$message = match ($plan) {
    'basic' => 'Basic features',
    'pro' => 'Pro features',
    default => 'Unknown plan',
};

echo $message;
~~~

Only `Basic features` is returned. This differs from `switch`, where a case without `break` can fall through into later statements.

To have multiple values produce one result, group the labels with commas rather than relying on fall-through:

~~~php
<?php
$day = 'Sunday';

$dayType = match ($day) {
    'Saturday', 'Sunday' => 'Weekend',
    default => 'Weekday',
};
~~~

## Handle every possible value

A `match` expression must be exhaustive at runtime. If no arm matches and there is no `default`, PHP throws an `UnhandledMatchError`.

~~~php
<?php
try {
    $label = match ($code) {
        200 => 'Success',
        404 => 'Not found',
    };
} catch (UnhandledMatchError $error) {
    $label = 'Unsupported status';
}
~~~

For user input or values that may vary, a `default` arm is usually the clearest fallback. If an unmatched value means a programming or data error, the default arm can throw a more meaningful exception instead of silently returning a generic result.

~~~php
<?php
$label = match ($code) {
    200 => 'Success',
    404 => 'Not found',
    default => throw new InvalidArgumentException('Unsupported status code'),
};
~~~

`throw` can be used as an expression in PHP 8.0 and later. Use it when unsupported values should stop processing and be handled by the caller.

## Use `match (true)` for conditions

A match subject can be the boolean `true`. Each arm condition can then be a boolean expression. The first condition that evaluates to `true` matches the subject.

~~~php
<?php
$age = 22;

$group = match (true) {
    $age < 13 => 'Child',
    $age < 20 => 'Teenager',
    $age < 65 => 'Adult',
    default => 'Senior',
};

echo $group;
~~~

Output:

~~~text
Adult
~~~

Order the conditions carefully. For example, a broad condition such as `$age >= 18` would also match a senior age, so it must come after a more specific senior condition. For complicated ranges or actions, an `if`/`elseif` chain is often easier to understand.

## Arms return expressions

An arm contains an expression that produces a result, not a block of statements. Keep each result concise. If an arm needs several steps, put those steps in a function and call the function from the arm.

~~~php
<?php
function createReport(): string
{
    // Several steps could happen here.
    return 'Report created';
}

$action = 'report';
$result = match ($action) {
    'report' => createReport(),
    'help' => 'Show help',
    default => 'Unknown action',
};
~~~

The subject is evaluated once. Conditions are checked in order until one matches, and only the result expression for the matched arm is evaluated. Keep conditions and results easy to reason about, especially when a function call has side effects.

A match expression can also stand alone when its result is not needed, but it still needs a terminating semicolon:

~~~php
<?php
match ($event) {
    'saved' => writeLog('Saved'),
    'deleted' => writeLog('Deleted'),
    default => writeLog('Unrecognized event'),
};
~~~

For a standalone match, the expressions are still evaluated as the selected arm is reached. If the operation needs multiple statements, use a regular conditional or call a function that performs them.

## `match`, `switch`, `if`, or ternary?

| Choose | When it fits |
| --- | --- |
| `match` | One value maps to one result; strict identity and a returned value are useful |
| `switch` | One value maps to statement bodies; legacy fall-through or statement-oriented handling is needed |
| `if` / `elseif` | Ranges, compound conditions, or several different checks are needed |
| Ternary | One small condition chooses between two values |

`match` and `switch` both compare one subject with discrete alternatives, but their comparison and control-flow rules differ. Use `match` when each case should yield one result and strict matching is desirable.

## Version support

`match` is available in PHP 8.0 and later. PHP 7 and earlier cannot parse a `match` expression. The `UnhandledMatchError` class is also part of PHP 8's match behavior.

## Try it yourself

1. Map a number from 1 through 7 to a weekday and add a fallback for other numbers.
2. Compare the integer `2` and string `'2'` with separate match arms.
3. Group several successful HTTP status codes into one arm.
4. Use `match (true)` to label a score as `High`, `Medium`, or `Low`.
5. Remove the `default` arm and try a value that matches no other arm.
6. Replace a generic default result with an `InvalidArgumentException`.
7. Move a multi-step arm into a named function and call that function from the match.
8. Rewrite a small `switch` expression as `match` and compare their type behavior.
9. Decide whether an `if` chain would be clearer for a set of overlapping ranges.

## Key takeaways

- `match` is an expression that returns the result of the first matching arm.
- It compares with strict identity (`===`) and never falls through.
- Comma-separated conditions can share one result.
- Include a `default` when all values need a defined result; otherwise an unmatched value throws `UnhandledMatchError`.
- `match (true)` can express ordered conditions, but `if` is often clearer for complex ranges.
- `match` requires PHP 8.0 or later.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [030 - Conditional Branching with if and switch](030-conditional-branching-with-if-and-switch.md) | [Open Q&A](../qnas/031-the-match-expression.md) | [Open example](../examples/031-the-match-expression.md) | [032 - Loops with while, do-while, and for](032-loops-with-while-do-while-and-for.md) |
