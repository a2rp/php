# Test doubles, stubs, and mocks

## Question

How can you test a shipping service without calling a real rate API or printing a real shipping label?

This example uses a stub to supply a predictable shipping rate and mocks to check whether the service prints a label only when it should. The service depends on interfaces, so each test can control its collaborators without changing production code.

## Project files

~~~text
project/
  src/
    ShippingService.php
  tests/
    Unit/
      ShippingServiceTest.php
~~~

PHPUnit must already be installed in the project and compatible with its PHP version. These tests are illustrative; the code and sample result below were not executed here.

## 1. Define the service and its dependency contracts

### File: src/ShippingService.php

~~~php
<?php

declare(strict_types=1);

interface ShippingRateProvider
{
    public function rateInCents(string $zone): int;
}

interface ShippingLabelPrinter
{
    public function print(string $orderId, int $priceInCents): void;
}

final class ShippingService
{
    public function __construct(
        private ShippingRateProvider $rates,
        private ShippingLabelPrinter $printer,
    ) {
    }

    public function quote(string $zone): int
    {
        $rate = $this->rates->rateInCents($zone);

        if ($rate < 0) {
            throw new InvalidArgumentException(
                'Shipping rate cannot be negative.'
            );
        }

        return $rate;
    }

    public function purchase(string $orderId, string $zone): int
    {
        $rate = $this->quote($zone);
        $this->printer->print($orderId, $rate);

        return $rate;
    }
}
~~~

The rate provider represents an external lookup. The label printer represents a side effect. Constructor injection makes both dependencies replaceable in tests while production can supply real implementations.

The service is deliberately final. PHPUnit can still create test doubles for the two interfaces, so the class under test does not need to be subclassed.

## 2. Write tests using a stub and mocks

### File: tests/Unit/ShippingServiceTest.php

~~~php
<?php

declare(strict_types=1);

use PHPUnit\Framework\TestCase;

require_once dirname(__DIR__, 2) . '/src/ShippingService.php';

final class ShippingServiceTest extends TestCase
{
    public function testQuoteReturnsTheRateSuppliedByAStub(): void
    {
        $rates = $this->createStub(ShippingRateProvider::class);
        $rates
            ->method('rateInCents')
            ->willReturn(725);

        $printer = $this->createStub(ShippingLabelPrinter::class);
        $service = new ShippingService($rates, $printer);

        self::assertSame(725, $service->quote('local'));
    }

    public function testPurchasePrintsOneLabelWithTheOrderAndRate(): void
    {
        $rates = $this->createStub(ShippingRateProvider::class);
        $rates
            ->method('rateInCents')
            ->willReturn(725);

        $printer = $this->createMock(ShippingLabelPrinter::class);
        $printer
            ->expects(self::once())
            ->method('print')
            ->with('ORDER-42', 725);

        $service = new ShippingService($rates, $printer);

        self::assertSame(725, $service->purchase('ORDER-42', 'local'));
    }

    public function testRateFailureDoesNotPrintALabel(): void
    {
        $rates = $this->createStub(ShippingRateProvider::class);
        $rates
            ->method('rateInCents')
            ->willThrowException(
                new RuntimeException('Rate service is unavailable.')
            );

        $printer = $this->createMock(ShippingLabelPrinter::class);
        $printer
            ->expects(self::never())
            ->method('print');

        $service = new ShippingService($rates, $printer);

        $this->expectException(RuntimeException::class);
        $this->expectExceptionMessage('Rate service is unavailable.');

        $service->purchase('ORDER-42', 'local');
    }

    public function testNegativeRateDoesNotPrintALabel(): void
    {
        $rates = $this->createStub(ShippingRateProvider::class);
        $rates
            ->method('rateInCents')
            ->willReturn(-1);

        $printer = $this->createMock(ShippingLabelPrinter::class);
        $printer
            ->expects(self::never())
            ->method('print');

        $service = new ShippingService($rates, $printer);

        $this->expectException(InvalidArgumentException::class);
        $this->expectExceptionMessage('Shipping rate cannot be negative.');

        $service->purchase('ORDER-42', 'local');
    }
}
~~~

The first test uses a stub because the test needs a controlled return value. The second uses a mock because it must verify an important interaction: one label is printed with the right order ID and rate. The last two tests check that invalid or unavailable rates raise an exception before printing.

## 3. Run the test file

On macOS or Linux:

~~~sh
vendor/bin/phpunit tests/Unit/ShippingServiceTest.php
~~~

On Windows with Composer's PHPUnit command shim:

~~~powershell
vendor/bin/phpunit.bat tests/Unit/ShippingServiceTest.php
~~~

### Expected result

All four tests pass. PHPUnit's exact summary and assertion count depend on the installed version.

## What this example demonstrates

- A stub controls an indirect input: the rate returned by the provider.
- A mock checks an important interaction: whether the printer receives the correct values.
- Exception stubs make failure paths reproducible without a real network request.
- A never expectation protects the side effect when rate lookup or validation fails.
- The final service class can be tested through its injected interface dependencies.

Do not mock every method just because a test framework allows it. Use a stub when the test needs a controlled value, and use a mock when a meaningful interaction is part of the behavior being checked. Prefer assertions about observable behavior over expectations tied to incidental call order.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [119 - Integration tests for databases and HTTP](../lessons/119-integration-tests-for-databases-and-http.md) | [Q&A](../qnas/120-test-doubles-stubs-and-mocks.md) | [Lesson](../lessons/120-test-doubles-stubs-and-mocks.md) | [121 - Static analysis with PHPStan and Psalm](../lessons/121-static-analysis-with-phpstan-and-psalm.md) |