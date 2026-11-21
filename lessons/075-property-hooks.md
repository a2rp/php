# Property hooks

Property hooks let a class define what happens when an object property is read or written. They add behavior to normal property access without requiring callers to use a getter or setter method.

Hooks are useful for validating assigned values, normalizing input, and exposing values derived from other properties. They were introduced in PHP 8.4.

## What you will learn

- declare <code>get</code> and <code>set</code> hooks
- distinguish backed and virtual properties
- validate and normalize assigned values
- derive a virtual property from other state
- understand defaults, references, and visibility
- recognize restrictions and common recursion mistakes

## Basic syntax

A property with hooks uses a block after the property declaration:

~~~php
class Person
{
    public string $name {
        get => $this->name;
        set => $this->name = trim($value);
    }
}
~~~

The <code>get</code> hook runs when code reads <code>$person->name</code>. The <code>set</code> hook runs when code assigns to it. The special <code>$value</code> variable contains the incoming value in a set hook.

A property may define only a getter, only a setter, or both. A hook block must contain at least one hook.

## Backed properties

A backed property has a value stored by the object. A hook can access that stored value using the property's own name from inside its hook:

~~~php
class DisplayName
{
    public string $name {
        get => $this->name;
        set => $this->name = trim($value);
    }
}

$person = new DisplayName();
$person->name = '  Ada  ';
echo $person->name; // Ada
~~~

Inside the hook, <code>$this->name</code> accesses the backing value directly. It does not call the same hook recursively. Outside the hook, property reads and writes go through the relevant hook.

A backed property may define one or both hooks. If a hook is omitted, that operation uses the ordinary property behavior. For example, a backed property with only a <code>set</code> hook still uses the ordinary backing value when read.

## Validate and normalize assignments

A set hook can reject invalid values and store a normalized value:

~~~php
class UserName
{
    public string $name {
        set {
            $name = trim($value);

            if ($name === '') {
                throw new InvalidArgumentException(
                    'Name cannot be empty.'
                );
            }

            $this->name = $name;
        }
    }
}
~~~

Now every assignment is checked:

~~~php
$user = new UserName();
$user->name = '  Ada  ';
echo $user->name; // Ada
~~~

A block-bodied set hook must take responsibility for storing the value when the property is backed. If it only validates but never assigns to <code>$this->name</code>, the backing value is not updated.

If a set hook only transforms and stores one value, use the shorter expression form:

~~~php
class Slug
{
    public string $value {
        set => strtolower(trim($value));
    }
}
~~~

For an expression-bodied set hook, PHP stores the expression result as the backing value. The equivalent block form must assign explicitly:

~~~php
class Slug
{
    public string $value {
        set {
            $this->value = strtolower(trim($value));
        }
    }
}
~~~

The expression form is convenient for a single transformation. Use a block when validation needs multiple statements or when the hook performs additional work.

## Get hooks

A get hook can transform a stored value whenever callers read it:

~~~php
class Product
{
    public string $name {
        get => ucfirst($this->name);
        set => $this->name = strtolower(trim($value));
    }
}

$product = new Product();
$product->name = '  desk lamp  ';

echo $product->name; // Desk lamp
~~~

The backing value is normalized to lowercase. The getter controls the value callers observe.

A get hook's result must be compatible with the property type. If a getter derives a result, make sure every return path produces a valid value.

## Virtual properties

A virtual property has hooks but no backing value of its own. Its hooks do not access <code>$this->propertyName</code>. A virtual property can derive a value from other state:

~~~php
class Rectangle
{
    public function __construct(
        public int $width,
        public int $height,
    ) {
    }

    public int $area {
        get => $this->width * $this->height;
    }
}

$rectangle = new Rectangle(4, 5);
echo $rectangle->area; // 20
~~~

The <code>$area</code> property is virtual. The object stores width and height, not a separate area value. Reading <code>area</code> calculates it each time.

A virtual property can also have a setter that updates other properties:

~~~php
class Rectangle
{
    public function __construct(
        public int $width,
        public int $height,
    ) {
    }

    public int $area {
        get => $this->width * $this->height;

        set {
            if ($this->width <= 0 || $value < 0 || $value % $this->width !== 0) {
                throw new InvalidArgumentException(
                    'Area must be a non-negative multiple of the width.'
                );
            }

            $this->height = intdiv($value, $this->width);
        }
    }
}
~~~

Because these hooks never access <code>$this->area</code>, it remains virtual. Assigning an area changes the rectangle's height; reading area then recomputes the result.

A virtual property with no getter cannot be read. A virtual property with no setter cannot be written.

## Avoid accidental recursion

A hook becomes backed when one of its hooks directly accesses the same property as <code>$this->propertyName</code>. That access inside the hook refers to the backing value.

To calculate a different value without storing one, do not reference the virtual property itself:

~~~php
class PersonName
{
    public function __construct(
        public string $firstName,
        public string $lastName,
    ) {
    }

    public string $fullName {
        get => $this->firstName . ' ' . $this->lastName;
    }
}
~~~

The getter reads two different properties. If it instead tried to read <code>$this->fullName</code>, it would recursively invoke itself.

## Set hook parameter types

A set hook may declare its incoming parameter explicitly:

~~~php
class Title
{
    public string $value {
        set(string $incoming) {
            $this->value = trim($incoming);
        }
    }
}
~~~

When the set hook uses the property's type, the parameter declaration may be omitted and PHP supplies <code>$value</code>. A set hook may also accept a wider type than the property and convert it before storing. It must not store a value incompatible with the property's declared type.

For example, a string property can accept a <code>Stringable</code> object and convert it to a string. A set hook that accepts only a narrower type than the property is not valid.

## Defaults

A backed property with hooks can declare a default before the hook block:

~~~php
class Account
{
    public string $role = 'reader' {
        set => trim($value);
    }
}
~~~

The initial default is assigned directly; it does not pass through the set hook. Later writes use the hook.

A virtual property cannot declare a default because it has no backing storage in which to keep that value.

## Hooks and property visibility

Hooks do not replace property visibility. Access modifiers still determine who can read and write the property. PHP 8.4 also supports asymmetric visibility such as <code>public private(set)</code>, which can expose reads while restricting writes.

Property hooks are incompatible with readonly properties. If a property needs custom access behavior or restricted write access, use hooks and/or asymmetric visibility as appropriate rather than combining a hook with <code>readonly</code>.

## References and indirect modification

Property hooks intercept reads and writes. References and indirect changes can bypass the expected set-hook path, so hooked properties require care when used by reference.

For example, modifying an array offset is an indirect modification:

~~~php
$object->items['new'] = 'value';
~~~

If array changes must be validated, do not assume a set hook will run for every offset update. Prefer replacing the whole property through the setter path or expose a method that validates an individual item.

PHP supports a special by-reference getter for advanced cases, but it can expose the backing value and allow changes that bypass a set hook. Use it only when that reference behavior is intentional and documented.

## isset(), empty(), and unset()

When a property has a <code>get</code> hook, <code>isset()</code> and <code>empty()</code> evaluate the value returned by the getter. If that getter reads an uninitialized backing value, these operations can throw an <code>Error</code> just like a normal read.

A property with hooks cannot be unset. Calling <code>unset()</code> on it throws an <code>Error</code>, whether the property is backed or virtual.

Prefer initializing backed values to sensible defaults when possible. This makes getters and checks easier to reason about.

## Inheritance and interfaces

A child class can redefine a property's hooks while following normal property type and visibility compatibility rules. Hooks can also be declared final to prevent a child class from overriding that hook.

Interfaces may declare the read or write operations a property must support. This lets a class satisfy a property contract with hooks or, in some read-only cases, a compatible property declaration. Keep such contracts explicit so it is clear whether implementations must support reading, writing, or both.

## When to use property hooks

Property hooks are useful when property syntax should stay simple but reads or writes need targeted behavior:

- trim or normalize assigned strings
- validate values at the point they are written
- compute a property from other properties
- preserve a property-shaped API while adding behavior
- restrict reads or writes using property visibility

Prefer a method when the operation is substantial, has side effects that are not obvious from property syntax, or needs multiple arguments. A property read should generally remain simple and predictable.

## Version notes

| PHP version | Property-hook feature |
| --- | --- |
| PHP 8.4 | Property hooks for instance properties |

Property hooks require PHP 8.4 or later. They do not apply to static properties and cannot be combined with readonly properties.

## Common mistakes

- **Using hooks before PHP 8.4:** older runtimes cannot parse the syntax.
- **Referencing the same property in a virtual getter:** this can recursively invoke the hook.
- **Forgetting to store a value in a block-bodied backed setter:** the backing value will not be updated.
- **Assuming the default goes through the setter:** a backed property's declared default is assigned directly.
- **Giving a virtual property a default:** virtual properties have no backing storage.
- **Assuming a backed property with one hook loses the other default operation:** an omitted operation uses normal behavior on a backed property.
- **Assuming array-offset changes always call set:** indirect modifications have special reference behavior.
- **Trying to combine readonly and hooks:** they are incompatible.
- **Using a property for a complex operation:** a method may express the intent more clearly.

## Key takeaways

- Property hooks intercept reads and writes on object properties.
- A backed property stores a value; a virtual property derives or redirects behavior without its own backing value.
- Use <code>$this->propertyName</code> inside a hook to access its backing value.
- A block-bodied backed setter must write the backing value itself.
- A virtual property is useful for a derived value such as an area or full name.
- Hooks were introduced in PHP 8.4 and cannot be used on static or readonly properties.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [074 - Constructor property promotion](074-constructor-property-promotion.md) | [Q&A](../qnas/075-property-hooks.md) | [Example](../examples/075-property-hooks.md) | [076 - Magic methods](076-magic-methods.md) |
