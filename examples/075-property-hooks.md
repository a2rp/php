# Property hooks - Example

## Goal

Use backed property hooks to trim assigned names, virtual hooks to combine or redirect values, and a default value to observe when a set hook runs.

## Requirements

Property hooks require PHP 8.4 or later.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

final class Person
{
    public string $firstName {
        set {
            $name = trim($value);

            if ($name === '') {
                throw new InvalidArgumentException(
                    'First name cannot be empty.'
                );
            }

            $this->firstName = $name;
        }
    }

    public string $lastName {
        set {
            $name = trim($value);

            if ($name === '') {
                throw new InvalidArgumentException(
                    'Last name cannot be empty.'
                );
            }

            $this->lastName = $name;
        }
    }

    public string $fullName {
        get => $this->firstName . ' ' . $this->lastName;

        set {
            $parts = preg_split('/\s+/', trim($value), 2);

            if ($parts === false || count($parts) !== 2) {
                throw new InvalidArgumentException(
                    'Enter a first name and a last name.'
                );
            }

            $this->firstName = $parts[0];
            $this->lastName = $parts[1];
        }
    }

    public function __construct(string $firstName, string $lastName)
    {
        $this->firstName = $firstName;
        $this->lastName = $lastName;
    }
}

final class Rectangle
{
    public function __construct(
        public int $width,
        public int $height,
    ) {
        if ($width <= 0 || $height <= 0) {
            throw new InvalidArgumentException(
                'Width and height must be positive.'
            );
        }
    }

    public int $area {
        get => $this->width * $this->height;

        set {
            if ($value <= 0 || $value % $this->width !== 0) {
                throw new InvalidArgumentException(
                    'Area must be a positive multiple of the width.'
                );
            }

            $this->height = intdiv($value, $this->width);
        }
    }
}

final class Status
{
    public string $label = ' pending ' {
        set => trim($value);
    }
}

$person = new Person('  Ada  ', ' Lovelace ');
echo 'First name: ' . $person->firstName . PHP_EOL;
echo 'Last name: ' . $person->lastName . PHP_EOL;
echo 'Full name: ' . $person->fullName . PHP_EOL;

$person->fullName = '  Grace    Hopper ';
echo 'Updated full name: ' . $person->fullName . PHP_EOL;
echo 'Updated first name: ' . $person->firstName . PHP_EOL;
echo 'Updated last name: ' . $person->lastName . PHP_EOL;

$rectangle = new Rectangle(width: 4, height: 5);
echo 'Area before: ' . $rectangle->area . PHP_EOL;

$rectangle->area = 36;
echo 'Height after setting area: ' . $rectangle->height . PHP_EOL;
echo 'Area after: ' . $rectangle->area . PHP_EOL;

$status = new Status();
echo 'Default label: [' . $status->label . ']' . PHP_EOL;

$status->label = ' ready ';
echo 'Assigned label: [' . $status->label . ']' . PHP_EOL;

// This is invalid because fullName is a hooked property:
// unset($person->fullName);
~~~

## Expected output

~~~text
First name: Ada
Last name: Lovelace
Full name: Ada Lovelace
Updated full name: Grace Hopper
Updated first name: Grace
Updated last name: Hopper
Area before: 20
Height after setting area: 9
Area after: 36
Default label: [ pending ]
Assigned label: [ready]
~~~

## Explanation

### The name properties are backed

<code>firstName</code> and <code>lastName</code> each have a set hook that trims and validates input. Inside each setter, <code>$this-&gt;firstName</code> or <code>$this-&gt;lastName</code> accesses the backing value directly instead of calling that same hook again.

The class has no explicit property declarations for those fields; the properties are declared with their hooks.

### The full name is virtual

<code>fullName</code> never reads or writes <code>$this-&gt;fullName</code>. Its getter combines the two stored name properties. Its setter splits the input and assigns each part to the corresponding backed property.

Those assignments go through the name setters, so trimming and validation still happen. There is no separate full-name value stored on the object.

### The area is virtual and writable

The rectangle stores width and height. Its virtual area getter calculates their product. The setter validates the requested area and converts it into a new height.

Setting area to 36 while width is 4 changes height to 9. Reading area afterward calculates 4 multiplied by 9.

### A declared default skips the set hook

The <code>Status</code> property starts with the default value <code> pending </code>. PHP assigns that default directly, so the output includes the surrounding spaces.

When code assigns <code> ready </code> later, the set hook trims the incoming value. That output has no surrounding spaces.

### Unsetting a hooked property is not allowed

The commented <code>unset()</code> line would throw an <code>Error</code>. Properties with hooks cannot be unset, including virtual properties.

## Try these changes

1. Assign an empty first name. Which hook rejects it?
2. Set <code>fullName</code> to one word. What validation error is thrown?
3. Set the rectangle's area to 35. Why is that rejected when its width is 4?
4. Add a <code>get</code> hook to <code>firstName</code> that returns a transformed value. What value will callers read?
5. Change the initial <code>Status</code> default to include spaces. Does the set hook clean it during object creation?
6. Add a calculated <code>perimeter</code> virtual property to <code>Rectangle</code>.

## Key takeaways

- Backed property hooks can validate and normalize property writes.
- Accessing a backed property's own name inside its hook reads or writes the backing value without recursion.
- Virtual properties can derive values or redirect writes to other state.
- Defaults on backed hooked properties bypass the set hook.
- Hooked properties cannot be unset.
- Property hooks require PHP 8.4 or later.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [074 - Constructor property promotion](../lessons/074-constructor-property-promotion.md) | [Lesson](../lessons/075-property-hooks.md) | [Q&A](../qnas/075-property-hooks.md) | [076 - Magic methods](../lessons/076-magic-methods.md) |
