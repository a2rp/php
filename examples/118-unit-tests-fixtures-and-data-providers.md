# 118 - Unit tests, fixtures, and data providers: Example

## Goal

Build a small cart and test its total calculation with a fresh fixture for each test, named data-provider cases, and explicit validation tests.

## Question

How can a PHPUnit test suite reuse one calculation test across several cart scenarios without sharing mutated state between cases?

## Assumptions

- PHP 8.4 or newer and Composer are available.
- This example uses PHPUnit 13, which requires PHP 8.4. For PHP 8.3, choose a compatible PHPUnit 12 release instead.
- Commands are run from the project root.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
cart-tests/
├── composer.json
├── src/
│   └── Cart.php
└── tests/
    └── CartTest.php
~~~

## 2. Configure Composer

Create `composer.json`:

~~~json
{
  "name": "learning/cart-tests",
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

PHPUnit is a development dependency, and Composer maps the `Learning` namespace to `src/`. The `test` script invokes the project-local PHPUnit binary. Composer will create a lock file when dependencies are resolved; keep it in version control for a real application.

## 3. Implement a small cart

Create `src/Cart.php`:

~~~php
<?php

declare(strict_types=1);

namespace Learning;

use InvalidArgumentException;

final class Cart
{
    /** @var list<array{unitPriceCents: int, quantity: int}> */
    private array $items = [];

    public function addItem(int $unitPriceCents, int $quantity): void
    {
        if ($unitPriceCents < 0) {
            throw new InvalidArgumentException('Unit price cannot be negative.');
        }

        if ($quantity < 1) {
            throw new InvalidArgumentException('Quantity must be at least one.');
        }

        $this->items[] = [
            'unitPriceCents' => $unitPriceCents,
            'quantity' => $quantity,
        ];
    }

    public function totalInCents(): int
    {
        return array_sum(array_map(
            static fn (array $item): int =>
                $item['unitPriceCents'] * $item['quantity'],
            $this->items
        ));
    }
}
~~~

The cart stores prices as integer cents and rejects negative prices and invalid quantities. It is intentionally small so the tests can focus on fixture setup, repeated scenarios, and the public behavior.

## 4. Add fixture setup and a named provider

Create `tests/CartTest.php`:

~~~php
<?php

declare(strict_types=1);

namespace Learning\Tests;

use InvalidArgumentException;
use Learning\Cart;
use PHPUnit\Framework\Attributes\DataProvider;
use PHPUnit\Framework\TestCase;

final class CartTest extends TestCase
{
    private Cart $cart;

    protected function setUp(): void
    {
        $this->cart = new Cart();
    }

    public function testCartStartsEmpty(): void
    {
        $this->assertSame(0, $this->cart->totalInCents());
    }

    /** @return array<string, array{0: list<array{0: int, 1: int}>, 1: int}> */
    public static function totalCases(): array
    {
        return [
            'one line with two units' => [
                [[500, 2]],
                1000,
            ],
            'two different lines' => [
                [[500, 2], [125, 4]],
                1500,
            ],
            'several prices and quantities' => [
                [[199, 3], [250, 2]],
                1097,
            ],
        ];
    }

    #[DataProvider('totalCases')]
    public function testCalculatesTotalForItems(array $items, int $expectedTotal): void
    {
        foreach ($items as [$unitPriceCents, $quantity]) {
            $this->cart->addItem($unitPriceCents, $quantity);
        }

        $this->assertSame($expectedTotal, $this->cart->totalInCents());
    }

    public function testRejectsNegativeUnitPrice(): void
    {
        $this->expectException(InvalidArgumentException::class);

        $this->cart->addItem(-1, 1);
    }

    public function testRejectsQuantityBelowOne(): void
    {
        $this->expectException(InvalidArgumentException::class);

        $this->cart->addItem(500, 0);
    }
}
~~~

`setUp()` creates a new cart for every test execution, including each data-provider case. The provider is `public static` and returns only arrays of values; it does not use the fixture or create services. Its string keys give each scenario a useful name in PHPUnit's output.

No `tearDown()` is needed because this fixture owns no file, process, database connection, or other resource requiring explicit release. The invalid-input tests use separate methods because they verify different validation behaviors from the total cases.

## 5. Install dependencies and run the suite

From `cart-tests/`, run:

~~~sh
composer install
composer test
~~~

`composer install` installs the project dependencies and creates `vendor/`. The `test` script runs the local PHPUnit executable against the `tests/` directory.

Representative output:

~~~text
PHPUnit 13.4.x
Runtime: PHP 8.4.x
......                                                              6 / 6 (100%)

OK (6 tests, 6 assertions)
~~~

The three named provider cases count as three tests, in addition to the empty-cart test and two invalid-input tests. The version and progress display can vary between PHPUnit releases. The PHP code and output above are provided as a learning example and were not executed here.

## Important notes

- A fresh cart in `setUp()` prevents one test's mutations from affecting another.
- The provider is static because PHPUnit evaluates providers before per-test setup.
- Named provider keys help identify the failing scenario.
- Use one provider only when cases share the same behavior and assertion structure.
- Add `tearDown()` only when a test must release an explicitly acquired resource.
- Keep file and database fixtures inside isolated test-owned locations.
- PHPUnit 13 requires PHP 8.4; select a compatible PHPUnit line for the project's minimum PHP version.
- The snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [117 - Testing PHP applications with PHPUnit](../lessons/117-testing-php-applications-with-phpunit.md) | [Q&A](../qnas/118-unit-tests-fixtures-and-data-providers.md) | [Lesson](../lessons/118-unit-tests-fixtures-and-data-providers.md) | [119 - Integration tests for databases and HTTP](../lessons/119-integration-tests-for-databases-and-http.md) |
