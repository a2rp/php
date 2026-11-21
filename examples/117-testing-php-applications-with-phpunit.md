# 117 - Testing PHP applications with PHPUnit: Example

## Goal

Create a small project with a discount calculator and PHPUnit tests for normal results and invalid input.

## Question

How can a PHP project install PHPUnit locally and verify both the expected result and the calculator's validation behavior?

## Assumptions

- PHP 8.4 or newer and Composer are available.
- This example uses PHPUnit 13, which requires PHP 8.4. For a project using PHP 8.3, choose a compatible PHPUnit 12 release instead.
- Commands are run from the project root.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
phpunit-discount-example/
├── composer.json
├── src/
│   └── DiscountCalculator.php
└── tests/
    └── DiscountCalculatorTest.php
~~~

## 2. Configure Composer

Create `composer.json`:

~~~json
{
  "name": "learning/phpunit-discount-example",
  "type": "project",
  "require": {
    "php": "^8.4"
  },
  "require-dev": {
    "phpunit/phpunit": "^13.0"
  },
  "autoload": {
    "psr-4": {
      "Learning\\": "src/"
    }
  },
  "scripts": {
    "test": "phpunit tests"
  }
}
~~~

The PHPUnit dependency is under `require-dev` because it is needed to develop and test the application, not to handle normal production requests. Composer generates a lock file during dependency installation; commit that lock file in a real project to keep the tool version reproducible.

## 3. Implement the behavior

Create `src/DiscountCalculator.php`:

~~~php
<?php

declare(strict_types=1);

namespace Learning;

use InvalidArgumentException;

final class DiscountCalculator
{
    public function finalPriceInCents(int $priceInCents, int $discountPercent): int
    {
        if ($priceInCents < 0) {
            throw new InvalidArgumentException('Price cannot be negative.');
        }

        if ($discountPercent < 0 || $discountPercent > 100) {
            throw new InvalidArgumentException(
                'Discount must be between 0 and 100 percent.'
            );
        }

        // Round to the nearest cent, with a half cent rounded upward.
        return intdiv(
            ($priceInCents * (100 - $discountPercent)) + 50,
            100
        );
    }
}
~~~

The method uses integer cents to avoid representing ordinary money values as binary floating-point numbers. It rejects invalid input and documents the rounding rule. This teaching example assumes prices fit safely within the platform's integer range; a production payment domain should also define its currency, input limits, and rounding rules explicitly.

## 4. Write focused tests

Create `tests/DiscountCalculatorTest.php`:

~~~php
<?php

declare(strict_types=1);

namespace Learning\\Tests;

use InvalidArgumentException;
use Learning\\DiscountCalculator;
use PHPUnit\\Framework\\TestCase;

final class DiscountCalculatorTest extends TestCase
{
    public function testReturnsOriginalPriceWhenThereIsNoDiscount(): void
    {
        $calculator = new DiscountCalculator();

        $actual = $calculator->finalPriceInCents(1299, 0);

        $this->assertSame(1299, $actual);
    }

    public function testAppliesPercentageDiscountAndRoundsToCents(): void
    {
        $calculator = new DiscountCalculator();

        $actual = $calculator->finalPriceInCents(1299, 20);

        $this->assertSame(1039, $actual);
    }

    public function testRejectsNegativePrice(): void
    {
        $this->expectException(InvalidArgumentException::class);

        (new DiscountCalculator())->finalPriceInCents(-1, 10);
    }

    public function testRejectsDiscountAboveOneHundredPercent(): void
    {
        $this->expectException(InvalidArgumentException::class);

        (new DiscountCalculator())->finalPriceInCents(1299, 101);
    }
}
~~~

Each test follows Arrange, Act, Assert. The first two assert exact integer results. The last two state the expected exception before calling the method, so the test fails if invalid input is accepted.

## 5. Install and run the test suite

From the project root, run:

~~~sh
composer install
composer test
~~~

Composer installs the locked development dependency and generates the project's autoloader. The `test` script invokes the local PHPUnit binary against `tests/`. It does not require a global PHPUnit installation.

Representative output:

~~~text
PHPUnit 13.4.x
Runtime: PHP 8.4.x
....                                                                4 / 4 (100%)

OK (4 tests, 4 assertions)
~~~

The version and progress formatting vary by installed PHPUnit release. The important result is that four tests passed and no failures or errors were reported. If an assertion fails, inspect the test name and failure details to see which expected behavior differed from the actual result.

## Important notes

- Keep `composer.json` and `composer.lock` in version control; do not commit generated `vendor/` dependencies unless the project has a specific reason.
- PHPUnit belongs in `require-dev`; production code should not depend on the test runner.
- The test class and methods follow PHPUnit's test discovery naming conventions.
- `assertSame()` checks both the result's type and value.
- An expected exception is configured before invoking the method that should throw it.
- The tests use only the calculator's public API; they do not inspect private state.
- The example has no database, network calls, real payments, or shared mutable fixture state.
- PHPUnit major versions have PHP version requirements. Keep the selected PHPUnit release compatible with the project's supported PHP versions.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [116 - PHP extensions and the extension ecosystem](../lessons/116-php-extensions-and-the-extension-ecosystem.md) | [Q&A](../qnas/117-testing-php-applications-with-phpunit.md) | [Lesson](../lessons/117-testing-php-applications-with-phpunit.md) | [118 - Unit tests, fixtures, and data providers](../lessons/118-unit-tests-fixtures-and-data-providers.md) |
