# 020 - Nullable, Union, and Intersection Types: Complete Example

This PHP CLI example demonstrates a nullable name, a union that accepts an integer, string, or null, an intersection requiring two interfaces, and a PHP 8.2 DNF type that also accepts a **Stringable** object.

## Question

How can a PHP function express that it accepts one of several scalar values, while another function requires an object with multiple capabilities?

## Goal

Create this project:

~~~text
composite-types-practice/
|-- README.md
|-- composite-types.php
~~~

The **composite-types.php** script demonstrates:

- A **?string** parameter accepting a string or null
- An **int|string|null** union parameter
- An intersection requiring both **Loggable** and **Cacheable**
- A DNF type accepting either that intersection or **Stringable**
- A **TypeError** caught for a value outside the union
- A **TypeError** caught for an object that does not satisfy the full intersection

## Complete code: composite-types.php

Save this file at the project root:

~~~php
<?php
declare(strict_types=1);

interface Loggable
{
    public function logMessage(): string;
}

interface Cacheable
{
    public function cacheKey(): string;
}

class Report implements Loggable, Cacheable
{
    public function logMessage(): string
    {
        return "Report generated";
    }

    public function cacheKey(): string
    {
        return "report:weekly";
    }
}

class TextLabel implements Stringable
{
    public function __toString(): string
    {
        return "Weekly summary";
    }
}

class LogOnly implements Loggable
{
    public function logMessage(): string
    {
        return "Log only";
    }
}

function displayName(?string $name): string
{
    return $name ?? "Guest";
}

function formatIdentifier(int|string|null $identifier): string
{
    if ($identifier === null) {
        return "No identifier";
    }

    return "ID: " . $identifier;
}

function publish(Loggable&Cacheable $item): string
{
    return $item->logMessage() . " [" . $item->cacheKey() . "]";
}

function describeTarget((Loggable&Cacheable)|Stringable $item): string
{
    if ($item instanceof Stringable) {
        return "Stringable: " . (string) $item;
    }

    return publish($item);
}

echo "Nullable values:" . PHP_EOL;
echo displayName("Asha") . PHP_EOL;
echo displayName(null) . PHP_EOL;

echo "Union values:" . PHP_EOL;
echo formatIdentifier(42) . PHP_EOL;
echo formatIdentifier("A-42") . PHP_EOL;
echo formatIdentifier(null) . PHP_EOL;

$report = new Report();

echo "Intersection:" . PHP_EOL;
echo publish($report) . PHP_EOL;

echo "DNF alternatives:" . PHP_EOL;
echo describeTarget($report) . PHP_EOL;
echo describeTarget(new TextLabel()) . PHP_EOL;

try {
    formatIdentifier([]);
} catch (TypeError $error) {
    echo "Rejected invalid union value." . PHP_EOL;
}

try {
    publish(new LogOnly());
} catch (TypeError $error) {
    echo "Rejected incomplete intersection object." . PHP_EOL;
}
~~~

## Complete code: README.md

Save this as **README.md** at the project root:

~~~~markdown
# Nullable, Union, and Intersection Types Practice

A PHP CLI script for practicing nullable, union, intersection, and DNF type declarations.

## Requirements

- PHP 8.2 or later
- PHP CLI available as **php** in the terminal

PHP 8.2 is required because the script uses a DNF declaration that combines an intersection and a union.

## Check and run

~~~sh
php -l composite-types.php
php composite-types.php
~~~

The first command checks PHP syntax. The second runs the script.

## Expected output

~~~text
Nullable values:
Asha
Guest
Union values:
ID: 42
ID: A-42
No identifier
Intersection:
Report generated [report:weekly]
DNF alternatives:
Report generated [report:weekly]
Stringable: Weekly summary
Rejected invalid union value.
Rejected incomplete intersection object.
~~~

## What the script demonstrates

- **?string** accepts a string or **null**.
- **int|string|null** accepts one of three listed types.
- **Loggable&Cacheable** requires an object implementing both interfaces.
- **(Loggable&Cacheable)|Stringable** accepts either an object implementing both interfaces or a **Stringable** object.
- **declare(strict_types=1);** prevents scalar coercion at calls made from this file.
- A type declaration checks the broad type contract, not business rules such as whether an identifier is allowed.

## Why the invalid calls fail

- An array matches none of the members of **int|string|null**.
- **LogOnly** implements **Loggable** but not **Cacheable**, so it does not satisfy **Loggable&Cacheable**.
- Each **TypeError** is caught so the script can print the rejection message and continue.

## Experiment

1. Add a different allowed string identifier and call **formatIdentifier()** with it.
2. Change **displayName()** to use a different fallback for null.
3. Create another class that implements both **Loggable** and **Cacheable**.
4. Add a second class implementing **Stringable** and pass it to **describeTarget()**.
5. Try an integer with **describeTarget()** inside a **try/catch**; it satisfies neither DNF alternative.
6. Run the code on PHP 8.1 and observe that the DNF signature is not supported there.

## Important distinction

A union provides alternatives: a value needs one matching member. An intersection combines requirements: an object needs every listed class or interface. A DNF type groups these ideas, as in **(A&B)|C**.
~~~~

## Understand the output

### Nullable name

**displayName()** accepts a string or null. The null coalescing operator returns **"Guest"** when the value is null. Because the parameter has no default, callers still need to pass an argument.

### Union identifier

**formatIdentifier()** accepts an integer, a string, or null. Each call uses one listed member. The array in the try block matches none of them, so PHP throws **TypeError** and the catch block prints a message.

### Intersection capabilities

**Report** implements both **Loggable** and **Cacheable**. It satisfies the intersection parameter and provides both the log message and cache key. **LogOnly** implements only the first interface, so it is rejected.

### DNF alternatives

The declaration **(Loggable&Cacheable)|Stringable** accepts either:

- A value satisfying both **Loggable** and **Cacheable**, such as **Report**
- A value satisfying **Stringable**, such as **TextLabel**

The grouped intersection is one alternative in the union.

## Make a change

1. Call **formatIdentifier()** with another integer and another string.
2. Pass **null** to **displayName()** and then pass an ordinary name.
3. Add a class implementing both interfaces and pass it to **publish()**.
4. Add a class implementing only **Cacheable** and confirm that it fails the intersection check.
5. Add a class implementing **Stringable** and pass it to **describeTarget()**.
6. Call **describeTarget()** with an array inside a **try/catch** and observe the **TypeError**.
7. Change the union to **int|string** and remove the null branch. Then confirm that null is rejected.
8. Change the function to **?string $name = null** and test that the argument can now be omitted.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming nullable means optional | A nullable argument is still required without a default. | Use **?Type $value = null** when omission is allowed. |
| Treating a union as a list of required types | A union needs only one matching member. | Read **A|B** as “A or B.” |
| Treating an intersection as alternatives | An intersection requires every member. | Read **A&B** as “A and B.” |
| Passing an object that implements only one interface | It does not satisfy the full intersection. | Implement every required interface. |
| Omitting DNF parentheses | The parser needs the intersection group marked. | Write **(A&B)|C**. |
| Using DNF on PHP 8.1 | DNF types require PHP 8.2 or later. | Check the project's minimum PHP version. |
| Assuming a union validates business rules | A value can have the right type and still be invalid for the application. | Validate allowed values, ranges, and formats separately. |
| Assuming null is accepted without declaring it | A non-nullable declaration does not permit null. | Add **null** to the union or use **?Type** for a single type. |

## Key takeaways

- Nullable syntax **?T** accepts **T** or null.
- A union accepts a value matching at least one member.
- An intersection requires an object matching every class or interface member.
- DNF syntax such as **(A&B)|C** combines both patterns and requires PHP 8.2.
- A default value controls whether an argument can be omitted.
- Type declarations do not enforce application-specific validation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [019 - Type declarations and strict typing](../lessons/019-type-declarations-and-strict-typing.md) | [Read this lesson](../lessons/020-nullable-union-and-intersection-types.md) | [Open Q&A](../qnas/020-nullable-union-and-intersection-types.md) | [021 - Mixed, void, never, and literal types](../lessons/021-mixed-void-never-and-literal-types.md) |
