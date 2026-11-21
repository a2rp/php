# Static analysis with PHPStan and Psalm

## Question

How can static analysis help catch unsafe use of request data, and how can runtime validation give the analyzer a precise type to follow?

This example starts with an unsafe function that reads keys from an unknown value. It then replaces that function with a validated version and configures PHPStan and Psalm to analyze the project files.

## Project files

~~~text
project/
  src/
    SignupInput.php
  demo.php
  phpstan.neon
  psalm.xml
~~~

The example assumes Composer and compatible PHPStan and Psalm versions are already installed as development dependencies. The code and representative analyzer output below were not executed here.

## 1. Notice the unsafe starting point

~~~php
<?php

declare(strict_types=1);

function unsafeSignupSummary(mixed $input): array
{
    return [
        'email' => $input['email'],
        'marketing' => $input['marketing'] ?? false,
    ];
}
~~~

The parameter is mixed, so it could be a string, null, integer, or another value. The function reads an offset without checking that the input is an array, and it does not establish the types of the values it returns.

Depending on tool version and settings, an analyzer may report unsafe offset access or missing type information here. The exact wording differs between PHPStan and Psalm.

## 2. Validate the value and declare the result shape

### File: src/SignupInput.php

~~~php
<?php

declare(strict_types=1);

/**
 * @return array{email: non-empty-string, marketing: bool}
 */
function parseSignupInput(mixed $input): array
{
    if (!is_array($input)) {
        throw new InvalidArgumentException('Expected an object-like array.');
    }

    $email = $input['email'] ?? null;

    if (!is_string($email)) {
        throw new InvalidArgumentException('Email must be a string.');
    }

    $email = trim($email);

    if ($email === '') {
        throw new InvalidArgumentException('Email cannot be empty.');
    }

    $marketing = $input['marketing'] ?? false;

    if (!is_bool($marketing)) {
        throw new InvalidArgumentException('Marketing must be a boolean.');
    }

    return [
        'email' => $email,
        'marketing' => $marketing,
    ];
}
~~~

Each guard establishes a fact about the data before the function uses it. The PHPDoc return shape records the contract that callers receive an email string that is not empty and a boolean preference.

The PHPDoc alone would not make the input safe. The runtime checks are what validate the actual value.

## 3. Call the function

### File: demo.php

~~~php
<?php

declare(strict_types=1);

require_once __DIR__ . '/src/SignupInput.php';

$signup = parseSignupInput([
    'email' => 'sam@example.test',
    'marketing' => true,
]);

var_export($signup);
echo PHP_EOL;
~~~

Expected runtime output:

~~~text
array (
  'email' => 'sam@example.test',
  'marketing' => true,
)
~~~

If the email is missing, blank, or not a string, or if marketing is not a boolean, the function throws InvalidArgumentException instead of returning an unvalidated shape.

## 4. Configure PHPStan

### File: phpstan.neon

~~~neon
parameters:
    level: 6
    paths:
        - src
        - demo.php
~~~

PHPStan levels are cumulative. Level 6 asks for more explicit type information than the lower levels, but it is not equivalent to Psalm's error level.

## 5. Configure Psalm

### File: psalm.xml

~~~xml
<?xml version="1.0"?>
<psalm errorLevel="4">
    <projectFiles>
        <directory name="src" />
        <file name="demo.php" />
    </projectFiles>
</psalm>
~~~

The numbers and defaults in these two configuration files are separate settings. Tune each tool according to the project instead of assuming a matching number means matching rules.

## 6. Run the analyzers and the demo

From the project root:

~~~sh
vendor/bin/phpstan analyse
vendor/bin/psalm
php demo.php
~~~

On Windows, Composer's command shims can be used:

~~~powershell
vendor/bin/phpstan.bat analyse
vendor/bin/psalm.bat
php demo.php
~~~

### Expected result

- PHPStan reports no issues in the corrected files at the configured level.
- Psalm reports no issues in the corrected files at the configured error level.
- The demo prints the validated array shown above.

Exact analyzer output and wording vary by installed version and configuration. If a tool reports a finding, read its inferred type and message before changing the code or suppressing the issue.

## What this example demonstrates

- An unknown input must be validated before its keys and values are trusted.
- Type checks and control flow narrow broad values to usable types.
- A precise return shape documents what callers can rely on.
- PHPStan and Psalm share concepts but use independent configuration.
- Runtime output and static analysis provide different kinds of evidence.

Static analysis does not replace tests, input validation, or review. It reports problems it can infer from the source and configuration; it cannot prove every runtime path or business rule is correct.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [120 - Test doubles, stubs, and mocks](../lessons/120-test-doubles-stubs-and-mocks.md) | [Q&A](../qnas/121-static-analysis-with-phpstan-and-psalm.md) | [Lesson](../lessons/121-static-analysis-with-phpstan-and-psalm.md) | [122 - Formatting, linting, and code quality tools](../lessons/122-formatting-linting-and-code-quality-tools.md) |