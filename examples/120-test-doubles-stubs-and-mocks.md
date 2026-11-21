# Test doubles, stubs, and mocks

## Question

How can you test a shipping service without calling a real rate API or printing a real shipping label?

This example uses a stub to provide a predictable rate, a mock to verify a meaningful label-print call, and another stub to simulate a rate-service failure. The production code depends on interfaces so the tests can supply controlled collaborators.

## Project files

~~~text
project/
├── src/
│   └── ShippingService.php
└── tests/
    └── Unit/
        └── ShippingServiceTest.php
~~~

PHPUnit must be installed in the project. Choose a PHPUnit version compatible with the PHP version required by the application.

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
            throw new InvalidArgumentException('Shipping rate cannot be negative.');
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

The rate provider represents an external lookup, while the printer represents an external side effect. The service receives both through its constructor, so a test can replace them without changing the service's business logic.
