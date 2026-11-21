# Interfaces and contracts - Example

## Goal

Build a notification service that can send a message through different providers without changing the service itself.

- <code>NotificationSender</code> defines the sending contract.
- <code>EmailSender</code> and <code>SmsSender</code> implement that contract.
- <code>AuditLogger</code> defines a separate logging capability.
- <code>NotificationService</code> depends on the two interfaces, not on specific implementations.

The main example uses PHP 8.0 or later.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

interface NotificationSender
{
    public function send(string $recipient, string $message): bool;
}

interface AuditLogger
{
    public function record(string $event): void;
}

class EmailSender implements NotificationSender
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Sending email to ' . $recipient . ': ' . $message . PHP_EOL;

        return true;
    }
}

class SmsSender implements NotificationSender
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Sending SMS to ' . $recipient . ': ' . $message . PHP_EOL;

        return true;
    }
}

class ConsoleAuditLogger implements AuditLogger
{
    public function record(string $event): void
    {
        echo 'AUDIT: ' . $event . PHP_EOL;
    }
}

final class NotificationService
{
    public function __construct(
        private NotificationSender $sender,
        private AuditLogger $logger
    ) {
    }

    public function notify(string $recipient, string $message): bool
    {
        $sent = $this->sender->send($recipient, $message);

        if ($sent) {
            $this->logger->record('notification.sent');
        } else {
            $this->logger->record('notification.failed');
        }

        return $sent;
    }
}

$logger = new ConsoleAuditLogger();

$emailService = new NotificationService(
    new EmailSender(),
    $logger
);
$emailService->notify(
    'reader@example.test',
    'Your report is ready.'
);

$smsService = new NotificationService(
    new SmsSender(),
    $logger
);
$smsService->notify(
    '+15550102030',
    'Your report is ready.'
);
~~~

## Expected output

~~~text
Sending email to reader@example.test: Your report is ready.
AUDIT: notification.sent
Sending SMS to +15550102030: Your report is ready.
AUDIT: notification.sent
~~~

## How the contract helps

The service constructor accepts any implementation of <code>NotificationSender</code> and <code>AuditLogger</code>:

~~~php
public function __construct(
    private NotificationSender $sender,
    private AuditLogger $logger
) {
}
~~~

The service uses only the methods promised by those interfaces. It does not need to know whether a message is sent by email or SMS, or whether audit events are printed, stored, or sent elsewhere.

The composition root at the bottom chooses the concrete objects:

~~~php
new NotificationService(new EmailSender(), $logger);
new NotificationService(new SmsSender(), $logger);
~~~

Both notification services use the same class. The implementation changes at object construction, so <code>NotificationService</code> does not need separate email and SMS branches.

## Add another implementation

A test or development version can implement the same contract without sending a real notification:

~~~php
class RecordingSender implements NotificationSender
{
    private array $messages = [];

    public function send(string $recipient, string $message): bool
    {
        $this->messages[] = [
            'recipient' => $recipient,
            'message' => $message,
        ];

        return true;
    }

    public function messages(): array
    {
        return $this->messages;
    }
}
~~~

It can be used by the same service:

~~~php
$recordingSender = new RecordingSender();

$service = new NotificationService(
    $recordingSender,
    new ConsoleAuditLogger()
);

$service->notify('reader@example.test', 'Test notification.');

var_export($recordingSender->messages());
~~~

Expected recorded value:

~~~text
array (
  0 =>
  array (
    'recipient' => 'reader@example.test',
    'message' => 'Test notification.',
  ),
)
~~~

The service depends on the contract, so it can work with this implementation too. In a real test suite, the recording object can help assert how the service collaborates with its sender.

## Implement multiple interfaces

One class can promise more than one capability when that design makes sense:

~~~php
interface HealthCheck
{
    public function isAvailable(): bool;
}

class MonitoredEmailSender implements NotificationSender, HealthCheck
{
    public function send(string $recipient, string $message): bool
    {
        echo 'Email sent to ' . $recipient . PHP_EOL;

        return true;
    }

    public function isAvailable(): bool
    {
        return true;
    }
}

function showAvailability(HealthCheck $service): void
{
    echo $service->isAvailable() ? 'Available' : 'Unavailable';
}

showAvailability(new MonitoredEmailSender());
~~~

Expected output:

~~~text
Available
~~~

Keep each interface focused. A class can implement separate small contracts rather than one broad interface that forces every implementation to provide unrelated operations.

## Version-specific interface features

The main example uses method contracts and works without PHP 8.4 interface properties. A method is a portable way to require a label:

~~~php
interface HasLabel
{
    public function label(): string;
}
~~~

On PHP 8.4 or later, an interface may instead require public property access:

~~~php
interface HasLabel
{
    public string $label { get; }
}

class Product implements HasLabel
{
    public string $label = 'Product';
}
~~~

Interface constants are also supported. Their inheritance and overriding behavior changed in PHP 8.1, so account for the target PHP version if relying on those details.

## What to notice

- Interfaces describe behavior but do not implement it.
- Implementing classes provide every required public method.
- The service types dependencies by interface.
- Multiple concrete senders can be substituted without editing the service.
- A class may implement multiple focused interfaces.
- The caller creates and wires the concrete objects.

## Practice

1. Add a <code>PushSender</code> that implements <code>NotificationSender</code>.
2. Add a <code>MemoryAuditLogger</code> that stores events in an array.
3. Make a recording sender return <code>false</code> and observe which audit event the service records.
4. Add a method that accepts <code>AuditLogger</code> and records an application-start event.
5. Decide whether retry behavior belongs in a new implementation, the service, or a separate collaborator. Explain the responsibility boundary.

## Key takeaways

- Type against an interface when a class needs a capability rather than one concrete implementation.
- Constructor injection lets the caller select implementations.
- Small contracts make implementations easier to substitute and test.
- A class can implement multiple interfaces, and a function can require just one capability.
- Use methods for broad PHP compatibility; interface property requirements need PHP 8.4 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [069 - Abstract classes and final declarations](../lessons/069-abstract-classes-and-final-declarations.md) | [Lesson](../lessons/070-interfaces-and-contracts.md) | [Q&A](../qnas/070-interfaces-and-contracts.md) | [071 - Traits and code reuse](../lessons/071-traits-and-code-reuse.md) |
