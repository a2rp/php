# 065 - Visibility and Encapsulation: Example

## Question

How can a class keep its state private, validate every change, and expose only intentional operations? How does a child class use protected functionality without exposing it to unrelated callers?

## Complete example

Save the following PHP 8.0+ code as `visibility-and-encapsulation.php` and run it from a terminal with `php visibility-and-encapsulation.php`.

~~~php
<?php

declare(strict_types=1);

class BankAccount
{
    public const CURRENCY = 'USD';

    private int $balanceInCents = 0;

    public function __construct(
        private string $owner,
        int $openingBalanceInCents = 0,
    ) {
        $this->owner = trim($this->owner);

        if ($this->owner === '') {
            throw new InvalidArgumentException(
                'Account owner cannot be empty.'
            );
        }

        if ($openingBalanceInCents < 0) {
            throw new InvalidArgumentException(
                'Opening balance cannot be negative.'
            );
        }

        $this->balanceInCents = $openingBalanceInCents;
    }

    public function owner(): string
    {
        return $this->owner;
    }

    public function balanceInCents(): int
    {
        return $this->balanceInCents;
    }

    public function deposit(int $amountInCents): void
    {
        if ($amountInCents <= 0) {
            throw new InvalidArgumentException(
                'Deposit must be greater than zero.'
            );
        }

        $this->balanceInCents += $amountInCents;
    }

    public function withdraw(int $amountInCents): void
    {
        if ($amountInCents <= 0) {
            throw new InvalidArgumentException(
                'Withdrawal must be greater than zero.'
            );
        }

        if ($amountInCents > $this->balanceInCents) {
            throw new DomainException('Insufficient funds.');
        }

        $this->balanceInCents -= $amountInCents;
    }

    public function statement(): string
    {
        return $this->owner
            . ' balance: '
            . $this->formatAmount($this->balanceInCents);
    }

    private function formatAmount(int $amountInCents): string
    {
        $wholeUnits = intdiv($amountInCents, 100);
        $fractionalUnits = $amountInCents % 100;

        return '$'
            . $wholeUnits
            . '.'
            . str_pad((string) $fractionalUnits, 2, '0', STR_PAD_LEFT);
    }
}

class Message
{
    protected string $body;

    public function __construct(string $body)
    {
        $this->body = $body;
    }

    protected function cleanBody(): string
    {
        return trim($this->body);
    }
}

class Notice extends Message
{
    public function render(): string
    {
        return '[NOTICE] ' . $this->cleanBody();
    }
}

$account = new BankAccount(' Mina ', 2500);
$account->deposit(500);
$account->withdraw(700);

echo $account->owner() . PHP_EOL;
echo BankAccount::CURRENCY . PHP_EOL;
echo $account->statement() . PHP_EOL;

try {
    $account->deposit(0);
} catch (InvalidArgumentException $exception) {
    echo 'InvalidArgumentException: '
        . $exception->getMessage()
        . PHP_EOL;
}

try {
    $account->withdraw(5000);
} catch (DomainException $exception) {
    echo 'DomainException: '
        . $exception->getMessage()
        . PHP_EOL;
}

$notice = new Notice('  Planned maintenance starts at 20:00.  ');
echo $notice->render() . PHP_EOL;

// These direct accesses are intentionally not executed:
// echo $account->balanceInCents; // Error: the property is private.
// echo $notice->cleanBody(); // Error: the method is protected.
~~~

## Expected output

~~~text
Mina
USD
Mina balance: $23.00
InvalidArgumentException: Deposit must be greater than zero.
DomainException: Insufficient funds.
[NOTICE] Planned maintenance starts at 20:00.
~~~

## Explanation

### The account owns its state

The balance is private, so outside code cannot assign a value directly. The constructor checks the owner and opening amount. The public deposit and withdraw methods validate every transaction before changing the balance.

The balance starts at 2500 cents. Depositing 500 cents and withdrawing 700 cents leaves 2300 cents, which the private formatting method displays as $23.00.

### The public API describes allowed operations

Callers can ask for the owner's name, current balance, or a statement. They can request a deposit or withdrawal. They cannot set an arbitrary balance, and invalid operations throw exceptions.

The private formatAmount method is an implementation detail used by statement(). It is not part of the account's public API.

### The child class can use protected members

Message stores its text in a protected property and exposes a protected cleaning helper. Notice can use both because it extends Message. Code outside those classes cannot call cleanBody() directly; it must use Notice's public render() method.

### The constant is public

BankAccount::CURRENCY is a public class constant, so callers can refer to it with the scope-resolution operator. A class can also use protected or private constants for implementation details.

## PHP 8.4+ option: public read, private write

Asymmetric property visibility is a newer option when a typed property should be publicly readable but writable only from within its class. This separate snippet requires PHP 8.4 or later; it is not part of the PHP 8.0+ script above.

~~~php
class Article
{
    public private(set) string $title;

    public function __construct(string $title)
    {
        $this->rename($title);
    }

    public function rename(string $title): void
    {
        $title = trim($title);

        if ($title === '') {
            throw new InvalidArgumentException('Title cannot be empty.');
        }

        $this->title = $title;
    }
}

$article = new Article('Getting Started');
echo $article->title; // Public read.

$article->rename('Visibility in PHP'); // Validated class-controlled write.

// $article->title = 'Unvalidated title'; // Error outside the class.
~~~

A caller can read the title and use rename() to request a validated change. The set visibility prevents direct outside writes.

## Try these changes

1. Change the opening balance to a negative number. Which exception is thrown?
2. Try depositing a negative amount. Does the balance change?
3. Add a public method that returns a formatted account summary without exposing the private formatting method.
4. Create a child class of Message that changes how the body is displayed.
5. On PHP 8.4 or later, try to assign directly to Article::title from outside the class. What happens?

## Key takeaways

- Private state can be changed only through the declaring class.
- Public methods give callers intentional ways to use an object.
- Validation belongs beside the state change it protects.
- Protected members are available to subclasses but not unrelated outside code.
- A private helper keeps implementation details out of the public API.
- The main example uses PHP 8.0+; asymmetric property visibility requires PHP 8.4+.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [064 - Constructors and destructors](../lessons/064-constructors-and-destructors.md) | [Lesson](../lessons/065-visibility-and-encapsulation.md) | [Q&A](../qnas/065-visibility-and-encapsulation.md) | [066 - Static properties and methods](../lessons/066-static-properties-and-methods.md) |
