# 064 - Constructors and Destructors

A constructor prepares an object when it is created. A destructor runs when the object is being destroyed. Constructors help establish valid initial state; destructors can provide last-resort cleanup, but their timing is not a reliable place for essential application work.

## Learning goals

By the end of this chapter, you should be able to:

- Define a constructor with `__construct()` and initialize object state.
- Pass required and optional arguments when creating an object.
- Use constructor property promotion in PHP 8 or later.
- Understand constructor behavior in a child class.
- Define a destructor with `__destruct()`.
- Explain why essential cleanup should not depend on destructor timing.
- Choose a factory method when object creation needs named alternatives.

## What a constructor does

PHP calls a class's `__construct()` method when an instance is created with `new`. Use it to set required state and check the values needed for the object to be valid.

~~~php
class Product
{
    private string $name;
    private int $priceInCents;

    public function __construct(string $name, int $priceInCents)
    {
        if (trim($name) === '') {
            throw new InvalidArgumentException(
                'Product name cannot be empty.'
            );
        }

        if ($priceInCents < 0) {
            throw new InvalidArgumentException(
                'Price cannot be negative.'
            );
        }

        $this->name = trim($name);
        $this->priceInCents = $priceInCents;
    }

    public function label(): string
    {
        return $this->name . ' - ' . $this->priceInCents . ' cents';
    }
}

$product = new Product('Notebook', 1299);
echo $product->label();
~~~

The object is created with the required arguments, and the constructor validates and stores them. If construction throws, the `new` expression does not provide a usable completed object to the caller.

Constructors do not declare a return value. Return the object's data through ordinary methods after construction.

## Required and optional constructor arguments

Constructor parameters can use types and defaults just like parameters on other methods:

~~~php
class SearchOptions
{
    private string $query;
    private int $limit;

    public function __construct(string $query, int $limit = 20)
    {
        if ($limit < 1 || $limit > 100) {
            throw new InvalidArgumentException(
                'Limit must be from 1 to 100.'
            );
        }

        $this->query = trim($query);
        $this->limit = $limit;
    }
}

$defaultOptions = new SearchOptions('php');
$customOptions = new SearchOptions('php', 50);
~~~

The first argument is required. The second uses its default when omitted.

Since PHP 8.0, callers can use named arguments:

~~~php
$options = new SearchOptions(query: 'php', limit: 50);
~~~

Named arguments can improve readability when there are several optional parameters. Parameter names form part of the API when callers use named arguments, so changing a public parameter name can affect callers.

PHP does not support multiple constructors with different signatures in one class. Use required and optional parameters carefully, or add clearly named static factory methods when there are distinct creation paths.

## Constructor property promotion

As of PHP 8.0, a constructor parameter can declare a visibility modifier and type to create and initialize a property in one step:

~~~php
class Customer
{
    public function __construct(
        private string $name,
        private int $accountId,
        private ?string $email = null,
    ) {
        if (trim($this->name) === '') {
            throw new InvalidArgumentException(
                'Customer name cannot be empty.'
            );
        }

        $this->name = trim($this->name);
    }

    public function displayName(): string
    {
        return $this->name;
    }
}

$customer = new Customer('Mina', 42, 'mina@example.test');
~~~

The promoted parameters declare properties and assign the incoming values before the constructor body runs. The constructor can still perform validation or other initialization afterward.

Promotion is shorthand; use ordinary properties and assignments when the logic is clearer that way. Constructor promotion is available in PHP 8.0 and later, so use explicit declarations if a project must support an older PHP version.

## Constructor inheritance

If a child class does not define its own constructor, it can inherit an accessible parent constructor. If a child class defines its own constructor, PHP does not automatically run the parent's constructor. Call it explicitly when the parent initialization is required:

~~~php
class BaseRecord
{
    protected string $recordId;

    public function __construct(string $recordId)
    {
        $this->recordId = $recordId;
    }
}

class UserRecord extends BaseRecord
{
    private string $displayName;

    public function __construct(string $recordId, string $displayName)
    {
        parent::__construct($recordId);
        $this->displayName = $displayName;
    }
}
~~~

The call to `parent::__construct()` initializes the parent portion of the object. If that step is required for the object's invariants, do not omit it.

## Use named factory methods for alternate creation paths

A PHP class has one constructor. If data can be converted into the constructor's expected form in different ways, use a named static factory method to make the intent clear:

~~~php
class Color
{
    private function __construct(
        private int $red,
        private int $green,
        private int $blue,
    ) {
    }

    public static function fromRgb(int $red, int $green, int $blue): self
    {
        foreach ([$red, $green, $blue] as $channel) {
            if ($channel < 0 || $channel > 255) {
                throw new InvalidArgumentException(
                    'Color channels must be from 0 to 255.'
                );
            }
        }

        return new self($red, $green, $blue);
    }

    public static function black(): self
    {
        return new self(0, 0, 0);
    }
}

$customColor = Color::fromRgb(35, 120, 210);
$black = Color::black();
~~~

The private constructor prevents callers from skipping validation. The factory names communicate which creation path is being used.

## What a destructor does

PHP calls the `__destruct()` method when an object is being destroyed. This typically happens when no references to the object remain, or during script shutdown.

~~~php
class TemporaryReport
{
    public function __destruct()
    {
        // Optional best-effort cleanup.
        echo 'TemporaryReport is being destroyed.' . PHP_EOL;
    }
}

$report = new TemporaryReport();
unset($report);
~~~

When `unset($report)` removes the last reference, the object can be destroyed at that point. If another variable still refers to the object, it remains alive.

The destructor takes no arguments and must not return a value. A child class that defines its own destructor must explicitly call `parent::__destruct()` if the parent cleanup is required; it is not automatically invoked.

## Do not rely on destructor timing for essential work

Destructors are useful as a fallback for releasing an object's resources, but the exact time of destruction can depend on object references and shutdown behavior. During script shutdown, the order in which objects are destroyed is not guaranteed.

Avoid using a destructor as the only place to:

- Commit a database transaction.
- Save important application data.
- Send a required email or API request.
- Emit a response that must appear in a particular order.
- Throw an exception or terminate the application.

Perform essential work explicitly and make failures visible to the caller.

### Use finally for deterministic resource cleanup

For a resource that must be closed after an operation, use `try/finally`:

~~~php
$handle = fopen('report.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the report.');
}

try {
    while (($line = fgets($handle)) !== false) {
        processLine($line);
    }
} finally {
    fclose($handle);
}
~~~

The `finally` block runs as control leaves the `try` block, including when an exception is thrown. The explicit close makes the cleanup point clear. A destructor may still be a useful safety net, but it should not replace the explicit cleanup path.

Do not throw exceptions from a destructor during shutdown. If cleanup can fail, prefer an explicit method that reports that failure while normal exception handling is still available.

## Old-style constructor names

Always use `__construct()` in new code. Before PHP 8, a method with the same name as a class in the global namespace could act as an old-style constructor, but that behavior was deprecated. In PHP 8 and later, a same-named method is an ordinary method and is not a constructor.

## Common mistakes

- Forgetting the two leading underscores in `__construct` or `__destruct`.
- Creating an object without required constructor arguments.
- Assuming the parent constructor runs automatically when a child defines its own.
- Trying to define several constructors in one class instead of using optional parameters or named factories.
- Putting essential data saving in a destructor.
- Relying on a particular destructor order during script shutdown.
- Throwing an exception from a destructor during shutdown.
- Using constructor property promotion in a project that supports PHP earlier than 8.0.
- Using a same-named class method as a constructor instead of `__construct()`.

## Key takeaways

- `__construct()` runs when `new` creates an object and is the usual place to establish valid initial state.
- Constructor parameters can be typed, required, optional, and named by callers.
- PHP 8.0 introduced constructor property promotion.
- A child class that defines its own constructor must call `parent::__construct()` when parent setup is required.
- Use named factories for distinct creation paths instead of trying to overload constructors.
- `__destruct()` is a best-effort cleanup hook; its shutdown timing and order are not reliable for essential work.
- Use explicit cleanup and `finally` for operations that must happen predictably.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| :--- | :--- | :--- | :--- |
| [063 - Properties, methods, and object creation](063-properties-methods-and-object-creation.md) | [Open Q&A](../qnas/064-constructors-and-destructors.md) | [Open example](../examples/064-constructors-and-destructors.md) | [065 - Visibility and encapsulation](065-visibility-and-encapsulation.md) |
