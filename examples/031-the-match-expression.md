# 031 - The match Expression: Complete Example

This command-line program uses PHP's `match` expression to map HTTP status codes to labels, distinguish integer input from string input, choose a score band, and map commands to messages. It also catches an `UnhandledMatchError` to show what happens when no arm matches and there is no `default`.

## Question

How can a PHP program map discrete values to results with strict type checks, combine values that share a result, choose a range-based label, and handle an unmatched value safely?

## Goal

Create this command-line project:

~~~text
match-expression-example/
└── index.php
~~~

The program will:

- Map status codes to categories with `match`
- Group several codes into one arm
- Distinguish the integer `404` from the string `'404'`
- Select a score band using ordered conditions
- Map multiple commands to a shared result
- Catch the error caused by a match with no matching arm and no `default`

## Complete code

Create `index.php` with this code:

~~~php
<?php
declare(strict_types=1);

function labelForStatus(int $status): string
{
    return match ($status) {
        200, 201, 204 => 'Success',
        301, 302, 307, 308 => 'Redirect',
        400, 401, 403, 404 => 'Client error',
        500, 502, 503 => 'Server error',
        default => 'Other response',
    };
}

function gradeFor(int $score): string
{
    return match (true) {
        $score >= 90 => 'A',
        $score >= 80 => 'B',
        $score >= 70 => 'C',
        $score >= 50 => 'D',
        default => 'Needs more practice',
    };
}

echo "=== 1. Status categories ===\n";

$statuses = [200, 201, 404, 503, 418];

foreach ($statuses as $status) {
    echo "{$status}: ", labelForStatus($status), "\n";
}

echo "\n=== 2. Strict type matching ===\n";

$inputCode = '404';
$codeKind = match ($inputCode) {
    404 => 'integer code',
    '404' => 'string code',
    default => 'unknown code',
};

echo "String input maps to: {$codeKind}\n";

echo "\n=== 3. Ordered score conditions ===\n";

$score = 86;
echo "Score: {$score}\n";
echo 'Grade: ', gradeFor($score), "\n";

echo "\n=== 4. Shared command results ===\n";

$commands = ['save', 'submit', 'cancel', 'refresh'];

foreach ($commands as $command) {
    $message = match ($command) {
        'save', 'submit' => 'Save complete',
        'cancel' => 'No changes',
        default => 'Unknown command',
    };

    echo "{$command}: {$message}\n";
}

echo "\n=== 5. Unmatched value ===\n";

try {
    $stateLabel = match ('paused') {
        'ready' => 'Ready',
        'running' => 'Running',
    };
} catch (UnhandledMatchError $error) {
    $stateLabel = 'No arm matched';
}

echo "State: {$stateLabel} (UnhandledMatchError caught)\n";
~~~

## Expected output

~~~text
=== 1. Status categories ===
200: Success
201: Success
404: Client error
503: Server error
418: Other response

=== 2. Strict type matching ===
String input maps to: string code

=== 3. Ordered score conditions ===
Score: 86
Grade: B

=== 4. Shared command results ===
save: Save complete
submit: Save complete
cancel: No changes
refresh: Unknown command

=== 5. Unmatched value ===
State: No arm matched (UnhandledMatchError caught)
~~~

## How it works

### 1. Map status codes

`labelForStatus()` returns a string from the match expression. Related status codes share an arm through comma-separated conditions. The `default` arm gives every other integer a defined category.

### 2. Match the input type exactly

`$inputCode` is the string `'404'`. Match uses strict identity, so it selects the string arm rather than the integer arm. If the input had been integer `404`, the integer arm would match.

### 3. Choose an ordered score band

`gradeFor()` uses `match (true)` to check conditions from the highest threshold to the lowest. A score of `86` fails the `90` check and then matches `80`, producing grade `B`.

### 4. Reuse a result for several commands

Both `save` and `submit` return `Save complete`. There is no fall-through and no `break`; only the selected result is evaluated. `refresh` is not listed, so the default result is used.

### 5. Handle an unhandled match

The final match has no `default`, and `'paused'` matches neither listed string. PHP throws `UnhandledMatchError`; the `catch` block handles it and supplies a display value. In production code, either include a suitable default or catch the error at an appropriate boundary.

## Run it

This example requires PHP 8.0 or later.

~~~sh
php --version
php index.php
~~~

The code is shown here as a learning example. Run it locally if you want to compare your output with the expected output above.

## Make a change

1. Add status codes `202` and `429` with appropriate categories.
2. Change `$inputCode` from string `'404'` to integer `404`, then to string `'404'`.
3. Try scores `92`, `73`, and `49`; predict the grade before running the program.
4. Add a `review` command that maps to `No changes`.
5. Add a `paused` match arm so the last section no longer throws.
6. Replace the `catch` result with a message that includes the exception class name.
7. Remove a `default` from `labelForStatus()` and consider how unmatched status codes should be handled.
8. Rewrite `gradeFor()` with an `if`/`elseif` chain and decide which version is easier to read.

## Common mistakes

- Expecting match to consider integer `404` equal to string `'404'`
- Forgetting a `default` when unknown values need a normal fallback
- Assuming an unmatched match expression returns `null`
- Expecting a case to fall through into the next arm
- Adding `break` to a match arm
- Putting multiple statements directly in an arm instead of calling a helper function
- Ordering overlapping `match (true)` conditions too broadly
- Using `match (true)` when a regular `if` chain would be clearer
- Using match syntax on PHP versions earlier than 8.0

## Key takeaways

- Match uses strict identity and returns the selected arm's expression.
- Comma-separated conditions share one result; match arms never fall through.
- A `default` arm defines behavior for unmatched values. Without one, PHP throws `UnhandledMatchError` if nothing matches.
- `match (true)` can express ordered conditions, but range logic should stay easy to read.
- This example requires PHP 8.0 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [030 - Conditional Branching with if and switch](../lessons/030-conditional-branching-with-if-and-switch.md) | [Read this lesson](../lessons/031-the-match-expression.md) | [Open Q&A](../qnas/031-the-match-expression.md) | [032 - Loops with while, do-while, and for](../lessons/032-loops-with-while-do-while-and-for.md) |
