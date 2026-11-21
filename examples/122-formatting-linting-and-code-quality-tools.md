# Formatting, linting, and code quality tools

## Question

How can a PHP project check syntax, enforce a shared coding style, and preview automated formatting changes?

This example uses PHP's syntax linter and PHP_CodeSniffer. It starts with inconsistent formatting, applies the fixer, then shows the corrected source and the checks to run. PHP CS Fixer and Rector are included as optional alternatives for formatting and refactoring workflows.

## Project files

~~~text
project/
  src/
    PriceFormatter.php
  demo.php
  phpcs.xml.dist
  composer.json
~~~

Install PHP_CodeSniffer as a development dependency:

~~~sh
composer require --dev squizlabs/php_codesniffer
~~~

If the project already has a composer.json file, merge the example scripts below into its existing scripts object.

## 1. Start with valid but inconsistently formatted PHP

### File: src/PriceFormatter.php before formatting

~~~php
<?php
declare(strict_types=1);
final class PriceFormatter
{
    public function format( int $amountInCents ): string
    {
        if($amountInCents<0){throw new InvalidArgumentException("Amount must be non-negative.");}
        return '$'.number_format($amountInCents/100,2,'.','');
    }
}
~~~

This code is syntactically valid, but its spacing and layout do not follow the rules in the project ruleset.

## 2. Configure the coding standard

### File: phpcs.xml.dist

~~~xml
<?xml version="1.0"?>
<ruleset name="Project">
    <description>Shared coding rules for this project.</description>

    <file>src</file>
    <file>demo.php</file>

    <exclude-pattern>*/vendor/*</exclude-pattern>

    <rule ref="PSR12"/>
</ruleset>
~~~

The committed ruleset tells PHP_CodeSniffer which files and rules to check. Keep the target paths focused on project-owned code.

## 3. Check and fix the style

Run PHP_CodeSniffer to see the violations:

~~~sh
vendor/bin/phpcs
~~~

Use PHP Code Beautifier and Fixer for the violations it can correct:

~~~sh
vendor/bin/phpcbf
~~~

Review the diff after it runs. The corrected source should follow the project's configured rules.

### File: src/PriceFormatter.php after formatting

~~~php
<?php

declare(strict_types=1);

final class PriceFormatter
{
    public function format(int $amountInCents): string
    {
        if ($amountInCents < 0) {
            throw new InvalidArgumentException(
                'Amount must be non-negative.'
            );
        }

        return '$' . number_format(
            $amountInCents / 100,
            2,
            '.',
            ''
        );
    }
}
~~~

A fixer changes style mechanically; it does not decide whether the class's behavior is correct. Keep the diff review in the workflow.

## 4. Add shared Composer commands

### Add to composer.json

~~~json
{
    "scripts": {
        "lint:syntax": [
            "@php -l src/PriceFormatter.php",
            "@php -l demo.php"
        ],
        "style:check": "phpcs",
        "style:fix": "phpcbf"
    }
}
~~~

These are the relevant entries to merge into the existing Composer configuration. The array for lint:syntax runs a syntax check for each PHP file in this small example.

Run the commands:

~~~sh
composer lint:syntax
composer style:check
~~~

Use style:fix during development when you intend to change files, then inspect the diff and run style:check again.

## 5. Execute the demonstration

### File: demo.php

~~~php
<?php

declare(strict_types=1);

require_once __DIR__ . '/src/PriceFormatter.php';

$formatter = new PriceFormatter();

echo $formatter->format(1999), PHP_EOL;
~~~

Run it from the project root:

~~~sh
php demo.php
~~~

Expected output:

~~~text
$19.99
~~~

The linter checks that the two PHP files parse. The style checker checks the selected coding standard. Running the demo shows the formatter's output. Each command answers a different question.

## Optional formatter and refactoring tools

PHP CS Fixer is an alternative formatter. Avoid configuring competing formatters with overlapping rules.

~~~sh
composer require --dev friendsofphp/php-cs-fixer
vendor/bin/php-cs-fixer fix --dry-run --diff
vendor/bin/php-cs-fixer fix
~~~

The first command that runs the fixer previews formatting changes without writing them. The second applies its configured rules.

Rector can propose larger syntax-aware refactors and upgrades:

~~~sh
composer require --dev rector/rector
vendor/bin/rector process --dry-run
~~~

The dry run shows proposed changes without modifying files. Apply a refactor only after reviewing its diff, then run the project's static analysis and tests.

## What this example demonstrates

- PHP's syntax linter parses files without executing them.
- PHP_CodeSniffer checks the coding standard selected by the project.
- PHP Code Beautifier and Fixer applies supported mechanical style fixes.
- Composer scripts give local development and CI shared commands.
- Formatters and refactoring tools can modify code, so review their diffs.

The PHP and tool output shown here is representative and was not executed. Exact reports vary by installed tool version and configuration.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [121 - Static analysis with PHPStan and Psalm](../lessons/121-static-analysis-with-phpstan-and-psalm.md) | [Q&A](../qnas/122-formatting-linting-and-code-quality-tools.md) | [Lesson](../lessons/122-formatting-linting-and-code-quality-tools.md) | [123 - Secure coding and threat modeling](../lessons/123-secure-coding-and-threat-modeling.md) |