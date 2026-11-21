# 113 - Dependency injection and service design: Example

## Goal

Build a small receipt-sending use case whose repository and delivery mechanism are supplied through constructor injection and connected at a command-line composition root.

## Question

How can an application service send an order receipt without choosing its own database or mail provider, while keeping object construction explicit and easy to follow?

## Assumptions

- PHP 8.1 or newer and Composer are available.
- Commands are run from the project root.
- The repository and sender in this demo are in-memory and console implementations. They demonstrate boundaries and do not store orders durably or send real email.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
receipt-demo/
├── bin/
│   └── send-receipt.php
├── src/
│   ├── Application/
│   │   └── Order/
│   │       ├── OrderRepository.php
│   │       ├── ReceiptSender.php
│   │       └── SendOrderReceipt.php
│   ├── Domain/
│   │   └── Order/
│   │       └── Order.php
│   └── Infrastructure/
│       ├── Mail/
│       │   └── ConsoleReceiptSender.php
│       └── Persistence/
│           └── InMemoryOrderRepository.php
├── .gitignore
└── composer.json
~~~

The application layer defines the behaviors its use case needs. The infrastructure layer supplies concrete implementations. The command-line script chooses and connects those objects.

## 2. Configure Composer autoloading

Create `composer.json`:

~~~json
{
    "name": "example/receipt-demo",
    "type": "project",
    "description": "A small PHP example of constructor dependency injection",
    "require": {
        "php": "^8.1"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
~~~

Create `.gitignore`:

~~~gitignore
/vendor/
~~~

From the project root, generate Composer's autoloader:

~~~sh
composer install
~~~

## 3. Create the domain order

Create `src/Domain/Order/Order.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Domain\Order;

use InvalidArgumentException;

final class Order
{
    public function __construct(
        private readonly string $number,
        private readonly string $recipient,
        private readonly int $totalInCents,
    ) {
        if (trim($number) === '') {
            throw new InvalidArgumentException('An order number is required.');
        }

        if (trim($recipient) === '') {
            throw new InvalidArgumentException('A receipt recipient is required.');
        }

        if ($totalInCents < 0) {
            throw new InvalidArgumentException('The order total cannot be negative.');
        }
    }

    public function number(): string
    {
        return $this->number;
    }

    public function recipient(): string
    {
        return $this->recipient;
    }

    public function totalInCents(): int
    {
        return $this->totalInCents;
    }
}
~~~

The domain object validates the values needed by this demonstration. It does not know whether the order came from a web request, which repository stores it, or which delivery provider sends a receipt.

## 4. Define the repository contract

Create `src/Application/Order/OrderRepository.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

use App\Domain\Order\Order;

interface OrderRepository
{
    public function findByNumber(string $number): ?Order;
}
~~~

The use case needs to find an order. This contract says what it needs without deciding how data is stored.

## 5. Define the receipt sender contract

Create `src/Application/Order/ReceiptSender.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

interface ReceiptSender
{
    public function send(string $recipient, string $subject, string $body): void;
}
~~~

The use case needs a way to deliver a message. The contract does not expose SMTP configuration, API credentials, or provider-specific classes.

## 6. Implement the application use case

Create `src/Application/Order/SendOrderReceipt.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Application\Order;

use DomainException;

final class SendOrderReceipt
{
    public function __construct(
        private readonly OrderRepository $orders,
        private readonly ReceiptSender $sender,
    ) {
    }

    public function execute(string $orderNumber): void
    {
        $order = $this->orders->findByNumber($orderNumber);

        if ($order === null) {
            throw new DomainException('The requested order was not found.');
        }

        $total = $order->totalInCents();
        $wholeUnits = intdiv($total, 100);
        $fraction = $total % 100;
        $body = sprintf(
            'Order %s total: $%d.%02d',
            $order->number(),
            $wholeUnits,
            $fraction,
        );

        $this->sender->send(
            $order->recipient(),
            'Your order receipt',
            $body,
        );
    }
}
~~~

`SendOrderReceipt` has one clear responsibility: find the requested order, build the receipt message, and ask its sender dependency to deliver it. The constructor makes both collaborators explicit. The use case does not call `new PdoOrderRepository()`, read environment variables, or create an SMTP client.

## 7. Implement an in-memory repository

Create `src/Infrastructure/Persistence/InMemoryOrderRepository.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Infrastructure\Persistence;

use App\Application\Order\OrderRepository;
use App\Domain\Order\Order;

final class InMemoryOrderRepository implements OrderRepository
{
    /** @var array<string, Order> */
    private array $orders = [];

    public function __construct(Order ...$orders)
    {
        foreach ($orders as $order) {
            $this->orders[$order->number()] = $order;
        }
    }

    public function findByNumber(string $number): ?Order
    {
        return $this->orders[$number] ?? null;
    }
}
~~~

This adapter provides the repository behavior for the demonstration. A database-backed implementation could satisfy the same interface while the application service continued to depend on `OrderRepository`.

## 8. Implement a console sender

Create `src/Infrastructure/Mail/ConsoleReceiptSender.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Infrastructure\Mail;

use App\Application\Order\ReceiptSender;

final class ConsoleReceiptSender implements ReceiptSender
{
    public function send(string $recipient, string $subject, string $body): void
    {
        printf(
            "To: %s\nSubject: %s\n%s\n",
            $recipient,
            $subject,
            $body,
        );
    }
}
~~~

This implementation prints the message instead of contacting a mail provider. A real mail adapter can implement `ReceiptSender` and be selected by the application bootstrap.

## 9. Wire the dependencies in the composition root

Create `bin/send-receipt.php`:

~~~php
<?php

declare(strict_types=1);

use App\Application\Order\SendOrderReceipt;
use App\Domain\Order\Order;
use App\Infrastructure\Mail\ConsoleReceiptSender;
use App\Infrastructure\Persistence\InMemoryOrderRepository;

require dirname(__DIR__) . '/vendor/autoload.php';

$orders = new InMemoryOrderRepository(
    new Order('ORD-1001', 'customer@example.test', 3000),
);
$sender = new ConsoleReceiptSender();
$sendOrderReceipt = new SendOrderReceipt($orders, $sender);

$sendOrderReceipt->execute('ORD-1001');
~~~

This script is the composition root. It selects the in-memory repository and console sender, then injects them into the use case. This is dependency injection using ordinary constructors; no service container is required.

## 10. Run the demonstration

From the project root:

~~~sh
php bin/send-receipt.php
~~~

## Expected output

~~~text
To: customer@example.test
Subject: Your order receipt
Order ORD-1001 total: $30.00
~~~

The order total is stored as 3000 cents, then formatted as dollars and cents with integer arithmetic. The message is printed by the console adapter; it is not emailed.

## Swap an implementation at the composition root

The use case has no provider-specific setup. A production bootstrap can construct a database repository and a mail-provider adapter, then inject them in the same way:

~~~php
$orders = new PdoOrderRepository($pdo);
$sender = new ProviderReceiptSender($mailClient);
$sendOrderReceipt = new SendOrderReceipt($orders, $sender);
~~~

Those concrete classes and their configuration belong to infrastructure and the composition root. The application service continues to call the same contracts.

For a focused test, a small recording `ReceiptSender` can save the recipient, subject, and body in memory. The test can inspect that record without sending an actual email. The same technique can substitute an in-memory repository for a database connection.

## What this example demonstrates

1. The use case declares its required dependencies in the constructor.
2. Application contracts describe the repository and sending behavior it needs.
3. Infrastructure adapters implement those contracts.
4. The composition root chooses concrete implementations and connects the object graph.
5. Service behavior remains focused on one use case.
6. Manual wiring is enough for a small dependency graph; a container is optional.
7. A test can substitute a recording or in-memory collaborator without changing the use case.

## Important notes

- The example's `.test` email address is illustrative and no email is sent.
- The in-memory repository loses its data when the process exits.
- The contracts are useful here to show real persistence and delivery boundaries; do not automatically create interfaces for every class.
- A real mail adapter must handle provider configuration, failures, retries, and sensitive data appropriately.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [112 - Routing, controllers, and middleware](../lessons/112-routing-controllers-and-middleware.md) | [Q&A](../qnas/113-dependency-injection-and-service-design.md) | [Lesson](../lessons/113-dependency-injection-and-service-design.md) | [114 - Application configuration and logging](../lessons/114-application-configuration-and-logging.md) |
