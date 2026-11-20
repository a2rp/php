# 065 - Visibility and Encapsulation

Visibility controls which code can access a class property, method, or class constant. Encapsulation uses that control to protect an object's state and expose a small, reliable public API.

## Learning goals

By the end of this chapter, you should be able to:

- Explain the difference between public, protected, and private members.
- Choose a visibility level based on how a class should be used.
- Protect object state and preserve invariants with encapsulation.
- Validate changes through public methods instead of exposing writable state.
- Understand visibility when a class is extended.
- Declare visibility for class constants.
- Recognize asymmetric property visibility introduced in PHP 8.4.

## What visibility controls

A class member is a property, method, or class constant. Its visibility determines which code is allowed to read, write, or call it.

| Visibility | Declaring class | Child classes | Unrelated outside code |
| --- | --- | --- | --- |
| `public` | Yes | Yes | Yes |
| `protected` | Yes | Yes | No |
| `private` | Yes | No | No |

For methods and class constants, the class member name uses `::` when accessed statically or when referring to a constant. Object properties and methods use `->`.

Always write the visibility explicitly. Although PHP treats an omitted visibility modifier on a property, method, or class constant as public, explicit declarations make the class API easier to review.

~~~php
class Status
{
    public const READY = 'ready';

    public string $label = 'Available';

    public function describe(): string
    {
        return $this->label . ': ' . self::READY;
    }
}
~~~

## Public members

A public member is part of the class's externally available API. Callers can use it from anywhere they can access the object or class.

~~~php
class Greeting
{
    public function sayHello(string $name): string
    {
        return 'Hello, ' . $name . '!';
    }
}

$greeting = new Greeting();
echo $greeting->sayHello('Asha');
~~~

Public methods are usually the operations that make sense to a caller. Public properties can also be appropriate for simple data transfer objects, but a public writable property lets any caller replace its value without asking the class to validate the change.

## Private members

A private member is accessible only from code declared in the class that defines it. It is not accessible from outside code or from a child class.

~~~php
class ApiToken
{
    private string $value;

    public function __construct(string $value)
    {
        if ($value === '') {
            throw new InvalidArgumentException('Token cannot be empty.');
        }

        $this->value = $value;
    }

    public function masked(): string
    {
        return str_repeat('*', max(0, strlen($this->value) - 4))
            . substr($this->value, -4);
    }
}

$token = new ApiToken('abc12345');
echo $token->masked(); // ****2345
~~~

The caller can use the public operation but cannot read or replace `$value` directly. The private property is an implementation detail.

Private access is based on the class where the code is declared, not on which object is being used. A method of `ApiToken` could access the private state of another `ApiToken` instance. Unrelated classes and subclasses still cannot.

Private methods are useful for internal steps that support a public operation:

~~~php
class Slug
{
    public static function fromTitle(string $title): string
    {
        return self::normalize($title);
    }

    private static function normalize(string $title): string
    {
        $title = strtolower(trim($title));
        $title = preg_replace('/[^a-z0-9]+/', '-', $title) ?? '';

        return trim($title, '-');
    }
}

echo Slug::fromTitle('PHP Visibility Basics'); // php-visibility-basics
~~~

A child class may define a method with the same name as a parent's private method, but it is a separate method. It does not override or gain access to the parent's private method.

## Protected members

A protected member is accessible to the class that declares it and to classes that extend it. It is not accessible to unrelated outside code.

~~~php
class Message
{
    protected string $text;

    public function __construct(string $text)
    {
        $this->text = $text;
    }

    protected function clean(): string
    {
        return trim($this->text);
    }
}

class Notice extends Message
{
    public function display(): string
    {
        return '[' . $this->clean() . ']';
    }
}

$notice = new Notice('  Maintenance at 8 PM  ');
echo $notice->display(); // [Maintenance at 8 PM]
~~~

The child can use `$text` and `clean()`, while outside code must call the public `display()` method. Protected visibility is useful when a class intentionally offers an internal extension point to subclasses.

Do not make a member protected just because it might be useful someday. Every protected member becomes a design decision that child classes may rely on. Prefer private until subclasses have a real need to use or customize the member.

## Encapsulation protects valid state

Encapsulation means keeping data and the rules for changing that data together. A class can reject invalid changes and ensure that its own state stays consistent.

A public writable balance has no place to enforce rules:

~~~php
class UnsafeAccount
{
    public int $balanceInCents = 0;
}

$account = new UnsafeAccount();
$account->balanceInCents = -50000; // The class cannot prevent this.
~~~

Use a private property and public operations that validate input:

~~~php
class BankAccount
{
    private int $balanceInCents = 0;

    public function __construct(
        private string $owner,
        int $openingBalanceInCents = 0,
    ) {
        $owner = trim($owner);

        if ($owner === '') {
            throw new InvalidArgumentException('Owner cannot be empty.');
        }

        if ($openingBalanceInCents < 0) {
            throw new InvalidArgumentException(
                'Opening balance cannot be negative.'
            );
        }

        $this->owner = $owner;
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
}

$account = new BankAccount('Mina', 2500);
$account->deposit(500);
$account->withdraw(700);

echo $account->owner() . PHP_EOL;
echo $account->balanceInCents() . PHP_EOL;
~~~

The account's rules are enforced no matter which caller uses `deposit()` or `withdraw()`. Storing money as integer cents also avoids binary floating-point rounding in this simple example.

### Getters and setters are not the goal

A public method for every property is not automatically good encapsulation. A setter that accepts any value without validation merely moves the unrestricted write behind a method call.

Prefer methods that describe meaningful operations:

~~~php
// Weak API: callers can set any state without domain rules.
$account->setBalanceInCents(-500);

// Better API: the operation can enforce the account rules.
$account->deposit(500);
$account->withdraw(200);
~~~

Use a getter when a caller has a real need to read a value. Use a setter only when callers should be allowed to change that value and the method can define the rules for doing so.

## Visibility and inheritance

A child class inherits the parent's public and protected members. It does not gain access to the parent's private members.

~~~php
class Document
{
    private string $internalId;
    protected string $title;

    public function __construct(string $internalId, string $title)
    {
        $this->internalId = $internalId;
        $this->title = $title;
    }

    public function title(): string
    {
        return $this->title;
    }
}

class Report extends Document
{
    public function heading(): string
    {
        // The protected title is available to this child.
        return 'Report: ' . $this->title;
    }

    // The child's code cannot read Document::$internalId.
}
~~~

When overriding an inherited public or protected method, the child may keep the same visibility or make it more accessible. It cannot make the method less accessible. For example, a protected method may become public, but a public method cannot become protected. Constructors have special rules and may have their visibility restricted in a child.

Keep the public contract stable. Changing a public method to protected or private breaks callers that rely on it. Changing a private implementation detail usually has a smaller impact because outside callers could not access it directly.

## Visibility for class constants

Since PHP 7.1, class constants can be public, protected, or private. An omitted visibility modifier means public.

~~~php
class Shipping
{
    public const STANDARD_DAYS = 5;
    protected const INTERNAL_ROUTE = 'warehouse-a';
    private const MAX_RETRIES = 3;

    public function retryLimit(): int
    {
        return self::MAX_RETRIES;
    }
}

echo Shipping::STANDARD_DAYS; // 5
echo (new Shipping())->retryLimit(); // 3
~~~

Use public constants for values that callers are expected to use. Keep implementation-only values protected or private.

## Asymmetric property visibility in PHP 8.4+

PHP 8.4 introduced asymmetric property visibility: reading and writing a typed property can have different visibility. The `set` visibility must be the same as or more restrictive than the read visibility.

~~~php
class Article
{
    public private(set) string $title;

    public function __construct(string $title)
    {
        $this->title = $title;
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
echo $article->title; // Reading is public.
$article->rename('Visibility in PHP'); // The class controls writes.

// $article->title = 'Bypass validation'; // Error: writing is private.
~~~

This can be convenient when a property should be publicly readable but only writable inside its class. It does not replace choosing a sound public API: a method such as `rename()` can validate the new value and communicate the operation's intent.

This syntax requires PHP 8.4 or later. The rest of this chapter's examples use traditional visibility and work with PHP 8.0 or later unless a section says otherwise.

## Visibility is not a security boundary

Visibility is a language-level API and encapsulation tool. It helps prevent accidental invalid access through ordinary PHP code, but it is not a substitute for authentication, authorization, input validation at trust boundaries, or safe secret storage.

Do not expose passwords, tokens, or sensitive personal data through public methods merely because the underlying property is private. Decide what information a caller is allowed to receive, and protect the application at its actual trust boundaries.

## Common mistakes

- **Leaving properties public by default:** callers can change values without preserving the class's rules.
- **Making everything protected:** child classes gain coupling to details that may later change.
- **Treating private as inherited:** a child class cannot directly use a parent's private member.
- **Adding empty getters and setters everywhere:** accessors do not provide value unless they express a useful API or enforce a rule.
- **Reducing visibility in an override:** an inherited public or protected method cannot be made less accessible.
- **Using visibility as application security:** visibility does not replace authorization or safe handling of secrets.
- **Using asymmetric visibility on an older PHP version:** `private(set)` requires PHP 8.4 or later.

## Key takeaways

- `public` is available everywhere; `protected` is available in the declaring class and its children; `private` belongs only to the declaring class.
- Encapsulation keeps state private and exposes intentional operations.
- Validate changes at the point where the object changes its own state.
- Prefer private for implementation details and use protected only for intentional subclass extension points.
- Public visibility is part of a class's API, so changing it can affect callers.
- PHP 8.4+ supports separate read and write visibility for typed properties.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [064 - Constructors and destructors](064-constructors-and-destructors.md) | [Q&A](../qnas/065-visibility-and-encapsulation.md) | [Example](../examples/065-visibility-and-encapsulation.md) | [066 - Static properties and methods](066-static-properties-and-methods.md) |
