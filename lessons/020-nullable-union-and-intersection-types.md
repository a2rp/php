# 020 - Nullable, Union, and Intersection Types

A type declaration can describe one accepted type or a deliberate combination of types. A nullable type accepts a value or **null**. A union accepts a value matching any one listed type. An intersection accepts an object that satisfies every listed class or interface type.

## What you will learn

- How to declare a nullable type with **?Type**
- How nullable syntax relates to a union with **null**
- How a union type uses the **|** operator
- How an intersection type uses the **&** operator
- Why union and intersection express different rules
- How to combine them with DNF syntax in PHP 8.2 and later
- Which PHP versions introduced these features
- How to avoid redundant or invalid type combinations

## Nullable types

A nullable type accepts either its base type or **null**. Prefix a single type with a question mark:

~~~php
<?php
declare(strict_types=1);

function displayName(?string $name): string
{
    if ($name === null) {
        return "Guest";
    }

    return $name;
}

echo displayName("Asha") . PHP_EOL;
echo displayName(null) . PHP_EOL;
~~~

Output:

~~~text
Asha
Guest
~~~

The declaration **?string** means **string|null**. It does not mean that the argument is optional: the caller still has to pass an argument unless the parameter also has a default.

~~~php
<?php
function displayName(?string $name = null): string
{
    return $name ?? "Guest";
}
~~~

Here the parameter accepts a string or **null**, and its default makes the argument optional.

### Nullable syntax with larger unions

Question-mark syntax applies to one base type. For more than one non-null type plus null, write every choice explicitly:

~~~php
<?php
function formatIdentifier(int|string|null $identifier): string
{
    if ($identifier === null) {
        return "No identifier";
    }

    return "Identifier: " . $identifier;
}
~~~

The declaration **int|string|null** accepts an integer, a string, or **null**. Do not write **?int|string**; use **int|string|null** when a union has several members.

## Union types

A union lists types separated by **|**. A value is accepted when it satisfies at least one member of that union.

~~~php
<?php
declare(strict_types=1);

function formatIdentifier(int|string $identifier): string
{
    return "Identifier: " . $identifier;
}

echo formatIdentifier(42) . PHP_EOL;
echo formatIdentifier("A-42") . PHP_EOL;
~~~

Output:

~~~text
Identifier: 42
Identifier: A-42
~~~

Both calls are valid: one supplies an integer, the other a string. An array would not satisfy **int|string**, so PHP would throw a **TypeError**.

Choose the narrowest union that matches the function's actual contract. A union that lists many unrelated types may make the function harder to use correctly.

## Intersection types

An intersection uses **&** between class or interface types. A value must satisfy all the listed types. Intersection types are available in PHP 8.1 and later.

The following example defines two interfaces and a class that implements both:

~~~php
<?php
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

function publish(Loggable&Cacheable $item): string
{
    return $item->logMessage() . " [" . $item->cacheKey() . "]";
}

echo publish(new Report()) . PHP_EOL;
~~~

Output:

~~~text
Report generated [report:weekly]
~~~

**Loggable&Cacheable** requires an object that implements both interfaces. An object that implements only one does not satisfy the declaration.

Intersection members must be class or interface types. Scalar types such as **int** and **string** cannot be used in an intersection.

## Union versus intersection

The operators describe different requirements:

| Declaration | Meaning | Example |
| --- | --- | --- |
| **A|B** | A value can satisfy **A** or **B** | **int|string** |
| **A&B** | An object must satisfy **A** and **B** | **Loggable&Cacheable** |

A union widens the set of acceptable values. An intersection narrows it to objects that satisfy every required type.

## Combining unions and intersections

PHP 8.2 added **Disjunctive Normal Form (DNF)** types, which let a declaration combine unions and intersections. Put parentheses around each intersection used as one member of a union:

~~~php
<?php
function publishItem((Loggable&Cacheable)|Stringable $item): void
{
    echo "Item accepted." . PHP_EOL;
}
~~~

This means the value must either:

- Implement both **Loggable** and **Cacheable**, or
- Implement **Stringable**

The parentheses make the grouping explicit: **(A&B)|C** means “both A and B, or C.” DNF types require PHP 8.2 or later.

Do not confuse **(A&B)|C** with **A&(B|C)**. DNF type declarations use a union of intersections, with each intersection grouped in parentheses.

## Version support

| Feature | Minimum PHP version |
| --- | --- |
| Nullable types such as **?string** | PHP 7.1 |
| Union types such as **int|string** | PHP 8.0 |
| Intersection types such as **Loggable&Cacheable** | PHP 8.1 |
| DNF types such as **(A&B)|C** | PHP 8.2 |
| Standalone **true**, **false**, and **null** types | PHP 8.2 |

A codebase that supports older PHP versions must not use syntax introduced after its minimum supported version. Check the project's required PHP version before adding a newer declaration.

## Avoid redundant or invalid combinations

Some combinations are invalid because one type already includes another:

- **bool|false** is redundant because **bool** already includes both true and false.
- **object|SomeClass** is redundant because **SomeClass** is already an object.
- **iterable|array** is redundant because arrays are already iterable.
- **mixed** cannot be combined with another union member because it already accepts every type.
- **never** cannot be combined with another type because no value can have type **never**.
- Intersections are for class and interface types, not scalar types.
- **?Type** is shorthand for one type plus **null**; use an explicit union for several alternatives.

PHP detects many redundant combinations when the declaration is compiled and reports an error before the function is called.

## Nullable parameters and defaults

A nullable declaration and a default value answer different questions:

~~~php
<?php
function findTitle(?string $title): ?string
{
    return $title;
}

function useDefaultTitle(?string $title = null): string
{
    return $title ?? "Untitled";
}
~~~

- **?string** says that **null** is an accepted value.
- **= null** says that the caller may omit the argument.
- A nullable parameter without a default is still required.

Use **?Type** rather than relying on an unmarked class-typed parameter with a **null** default. Implicitly nullable parameter declarations are deprecated as of PHP 8.4.

## Type declarations still need validation

A union or nullable type checks the broad kind of value, not its meaning. A **string** member can still be empty, and an **int** member can still be negative. Validate application rules such as allowed values, ranges, or formats separately.

Strict scalar coercion rules from the previous chapter still matter when a union includes scalar types. With **declare(strict_types=1);**, pass a value whose actual type is one of the declared members.

## Try it yourself

1. Write a function with a **?string** parameter and return a fallback when the argument is **null**.
2. Call that function once with a string and once with **null**.
3. Make a function accept an **int|string** identifier and try it with both allowed types.
4. Pass an array to the **int|string** parameter and observe the **TypeError**.
5. Define two small interfaces, create a class implementing both, and accept it using an intersection type.
6. Write down in plain language what **A|B** and **A&B** each require.
7. Convert **?int|string** into a valid explicit union declaration.
8. Check the project's minimum PHP version before using intersection or DNF syntax.

## Common mistakes

| Mistake | Why it causes trouble | Better approach |
| --- | --- | --- |
| Assuming **?string** makes a parameter optional | It accepts null, but the argument is still required without a default. | Add **= null** when callers may omit it. |
| Using **?int|string** for a multi-member nullable union | Question-mark syntax applies to a single base type. | Write **int|string|null**. |
| Confusing union and intersection | A union needs one matching member; an intersection needs every member. | Read **A|B** as “A or B” and **A&B** as “A and B.” |
| Using scalar types in an intersection | Intersections require class or interface types. | Use a union for scalar alternatives or define an interface contract for objects. |
| Forgetting parentheses in a DNF type | The parser needs the intersection group to be explicit. | Write **(A&B)|C**. |
| Using DNF syntax on PHP 8.1 | DNF types were introduced in PHP 8.2. | Check the minimum runtime version or use a supported alternative. |
| Adding a type already covered by another member | PHP rejects many redundant combinations. | Remove the redundant member; for example, use **bool** instead of **bool|false**. |
| Treating a nullable type as input validation | It only permits null in addition to the base type. | Validate the value's format and business rules separately. |
| Forgetting that **null** must be explicitly included | A non-nullable type does not accept null. | Use **?Type** or an explicit union with **null**. |

## Key takeaways

- **?Type** means **Type|null** and is available from PHP 7.1.
- A union **A|B** accepts a value matching at least one member.
- An intersection **A&B** requires an object matching every class or interface member.
- Union types require PHP 8.0; intersections require PHP 8.1.
- DNF syntax such as **(A&B)|C** requires PHP 8.2.
- Nullable syntax does not make an argument optional; a default value does.
- Avoid redundant combinations and use explicit unions when several alternatives are accepted.
- Type declarations do not replace validation of application-specific rules.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [019 - Type declarations and strict typing](019-type-declarations-and-strict-typing.md) | [Open Q&A](../qnas/020-nullable-union-and-intersection-types.md) | [Open example](../examples/020-nullable-union-and-intersection-types.md) | [021 - Mixed, void, never, and literal types](021-mixed-void-never-and-literal-types.md) |
