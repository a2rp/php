# 030 - Conditional Branching with if and switch: Complete Example

This command-line program makes decisions with `if`, `elseif`, and `else`, then uses `switch` for a set of discrete choices. It demonstrates score ranges, combined requirements, strict comparisons, switch fall-through prevention, shared case labels, and a default branch.

## Question

How can a PHP program choose a grade from a score range, check multiple access requirements, distinguish a string input from an integer, and map format or day values to a result?

## Goal

Create this command-line project:

~~~text
conditional-branching-example/
â””â”€â”€ index.php
~~~

The program will:

- Assign a grade and feedback using an `if`/`elseif`/`else` chain
- Allow access only when an account is active and verified
- Compare a submitted string using strict identity
- Map known formats with `switch` and provide a `default`
- Group case labels that share one body
- Use `break` to stop accidental fall-through
- Skip a loop iteration from inside a switch with `continue 2`

## Complete code

Create `index.php` with this code:

~~~php
<?php
declare(strict_types=1);

function gradeFor(int $score): string
{
    if ($score >= 90) {
        return 'A';
    } elseif ($score >= 80) {
        return 'B';
    } elseif ($score >= 70) {
        return 'C';
    } elseif ($score >= 50) {
        return 'D';
    } else {
        return 'Needs more practice';
    }
}

function accessFor(bool $accountIsActive, bool $emailIsVerified): string
{
    if (!$accountIsActive) {
        return 'Denied: account inactive';
    } elseif (!$emailIsVerified) {
        return 'Denied: verify email';
    } else {
        return 'Allowed';
    }
}

echo "=== 1. Score ranges ===\n";

$score = 82;
$grade = gradeFor($score);
$feedback = $score >= 50 ? 'Pass' : 'Try again';

echo "Score: {$score}\n";
echo "Grade: {$grade}\n";
echo "Feedback: {$feedback}\n";

echo "\n=== 2. Combined access conditions ===\n";

echo 'Verified account: ', accessFor(true, true), "\n";
echo 'Unverified account: ', accessFor(true, false), "\n";
echo 'Inactive account: ', accessFor(false, true), "\n";

echo "\n=== 3. Strict input comparison ===\n";

$submittedCode = '2';

if ($submittedCode === 2) {
    echo "Integer two\n";
} elseif ($submittedCode === '2') {
    echo "String two\n";
} else {
    echo "Different value\n";
}

echo "\n=== 4. Switch by format ===\n";

$format = 'json';

switch ($format) {
    case 'html':
        $contentType = 'text/html';
        break;
    case 'json':
        $contentType = 'application/json';
        break;
    case 'text':
        $contentType = 'text/plain';
        break;
    default:
        $contentType = 'application/octet-stream';
}

echo "Content type: {$contentType}\n";

$unknownFormat = 'csv';

switch ($unknownFormat) {
    case 'html':
        $unknownContentType = 'text/html';
        break;
    case 'json':
        $unknownContentType = 'application/json';
        break;
    case 'text':
        $unknownContentType = 'text/plain';
        break;
    default:
        $unknownContentType = 'application/octet-stream';
}

echo "Unknown format: {$unknownContentType}\n";

echo "\n=== 5. Shared case labels and break ===\n";

$days = ['Sunday', 'Monday'];

foreach ($days as $day) {
    switch ($day) {
        case 'Saturday':
        case 'Sunday':
            $dayType = 'Weekend';
            break;
        default:
            $dayType = 'Weekday';
    }

    echo "{$day}: {$dayType}\n";
}

$plan = 'basic';

switch ($plan) {
    case 'basic':
        echo "Basic features\n";
        break;
    case 'pro':
        echo "Pro features\n";
        break;
    default:
        echo "Unknown plan\n";
}

echo "\n=== 6. Continue the outer loop ===\n";

$commands = ['skip', 'print'];

foreach ($commands as $command) {
    switch ($command) {
        case 'skip':
            continue 2;
        case 'print':
            echo "Processed print command\n";
            break;
    }
}
~~~

## Expected output

~~~text
=== 1. Score ranges ===
Score: 82
Grade: B
Feedback: Pass

=== 2. Combined access conditions ===
Verified account: Allowed
Unverified account: Denied: verify email
Inactive account: Denied: account inactive

=== 3. Strict input comparison ===
String two

=== 4. Switch by format ===
Content type: application/json
Unknown format: application/octet-stream

=== 5. Shared case labels and break ===
Sunday: Weekend
Monday: Weekday
Basic features

=== 6. Continue the outer loop ===
Processed print command
~~~

## How it works

### 1. Handle ranges with an `if` chain

`gradeFor()` checks the highest score threshold first. For a score of `82`, the checks for 90 and above fail, then the `80` check succeeds and returns `B`. Later branches are skipped. The feedback expression is a small two-value choice; the full tutorial on that expression is in the previous chapter.

### 2. Require both access conditions

`accessFor()` returns early when the account is inactive or the email is not verified. Only when those checks pass does the final branch return `Allowed`. The parameters use boolean types so the function receives explicit true/false values.

### 3. Preserve the input type

`$submittedCode` contains the string `'2'`. Strict comparisons distinguish it from the integer `2`, so the `elseif` branch prints `String two`. This is useful when input needs to be checked without implicit type conversion.

### 4. Choose a discrete format with `switch`

The switch maps each known format to a content type. Each case ends with `break`, so only the selected assignment runs. The second switch has no matching case; its `default` branch supplies a generic content type.

Switch case matching itself is loose. The separate strict comparison section shows how to check exact types when that distinction matters.

### 5. Share cases and prevent fall-through

`Saturday` and `Sunday` have no statements between their labels, so they share the `Weekend` body. The plan switch includes `break` after `basic`; without it, execution would continue into the `pro` body and print both messages.

### 6. Continue the enclosing loop

The `skip` command uses `continue 2` to leave the switch and advance the `foreach` loop. The `print` command reaches its case, prints its message, and exits the switch with `break`.

## Run it

This code uses scalar type declarations and return types, so use PHP 7.0 or later.

~~~sh
php --version
php index.php
~~~

The code is shown here as a learning example. Run it locally if you want to compare your output with the expected output above.

## Make a change

1. Set `$score` to `95`, `70`, `49`, and `120`; predict the grade and feedback.
2. Change the grade thresholds and keep the conditions in the correct order.
3. Try each combination of active/inactive and verified/unverified access values.
4. Change `$submittedCode` to integer `2`, string `'02'`, and string `'2'`.
5. Add an `xml` format and map it to a content type.
6. Change `$unknownFormat` to a supported value, then to an unsupported one.
7. Add `Friday` to the days and confirm it is a weekday.
8. Remove the `break` after the `basic` case and observe the fall-through.
9. Add another command that prints a message, then add a command that is skipped with `continue 2`.
10. Rewrite the format switch using `if`/`elseif` and compare readability.

## Common mistakes

- Putting a broad range condition before a more specific threshold
- Treating `'0'` or `0` as truthy without checking PHP's falsey rules
- Using `==` where input type matters
- Forgetting that `switch` uses loose case comparisons
- Forgetting `break` and accidentally executing later case bodies
- Adding statements between case labels that were meant to share one body
- Using `continue` when the intent is to continue an outer loop from inside a switch
- Omitting a `default` when unknown values need a defined result

## Key takeaways

- `if`/`elseif`/`else` works well for ranges and compound conditions; only the first true branch runs.
- Strict comparison distinguishes values such as integer `2` and string `'2'`.
- `switch` is useful for discrete alternatives, but its case matching is loose.
- `break` ends a case; adjacent empty labels intentionally share one body.
- Use `continue 2` to advance an outer loop from within a switch nested in that loop.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [029 - Ternary, Null Coalescing, and Nullsafe Operators](../lessons/029-ternary-null-coalescing-and-nullsafe-operators.md) | [Read this lesson](../lessons/030-conditional-branching-with-if-and-switch.md) | [Open Q&A](../qnas/030-conditional-branching-with-if-and-switch.md) | [031 - The match expression](../lessons/031-the-match-expression.md) |
