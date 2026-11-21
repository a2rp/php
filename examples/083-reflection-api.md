# 083 - Reflection API: Example

## Question

How can a tool inspect a PHP class, its attribute, method parameters, property, and class constant, then create an object and invoke a method through Reflection?

## Complete PHP example: `reflection-demo.php`

```php
<?php

declare(strict_types=1);

use Attribute;

#[Attribute(Attribute::TARGET_CLASS)]
final class Component
{
    public function __construct(public string $name)
    {
    }
}

#[Component(name: 'billing')]
final class InvoiceFormatter
{
    public string $currency = 'USD';

    public const VERSION = '1.0';

    public function format(string $invoiceNumber, string $prefix = 'INV-'): string
    {
        return $prefix . $invoiceNumber;
    }
}

function normalizeCode(string $code): string
{
    return strtoupper(trim($code));
}

$classReflection = new ReflectionClass(InvoiceFormatter::class);

$sourceFile = $classReflection->getFileName();
echo 'Class: ' . $classReflection->getName() . "\n";
echo 'Source file: ' . ($sourceFile === false ? 'unavailable' : basename($sourceFile)) . "\n";
echo 'Can instantiate: ' . ($classReflection->isInstantiable() ? 'yes' : 'no') . "\n";

// Read the attribute metadata, then create its object explicitly.
$componentMetadata = $classReflection->getAttributes(Component::class)[0];
$component = $componentMetadata->newInstance();
echo 'Component: ' . $component->name . "\n";

$methodReflection = $classReflection->getMethod('format');
echo 'Method: ' . $methodReflection->getName() . "\n";

foreach ($methodReflection->getParameters() as $parameter) {
    $type = $parameter->getType();
    $typeName = $type === null ? 'no type declared' : (string) $type;
    $description = '$' . $parameter->getName() . ': ' . $typeName;

    if ($parameter->isDefaultValueAvailable()) {
        $default = var_export($parameter->getDefaultValue(), true);
        $description .= ' (optional, default: ' . $default . ')';
    } else {
        $description .= ' (required)';
    }

    echo $description . "\n";
}

// Create a valid object using the reflected class.
$formatter = $classReflection->newInstance();

$propertyReflection = $classReflection->getProperty('currency');
echo 'Currency initialized: '
    . ($propertyReflection->isInitialized($formatter) ? 'yes' : 'no') . "\n";
echo 'Property currency: ' . $propertyReflection->getValue($formatter) . "\n";

$constantReflection = $classReflection->getReflectionConstant('VERSION');
echo 'Constant VERSION: ' . $constantReflection->getValue() . "\n";

// Invoke the method with an object and its required argument.
echo 'Invocation result: ' . $methodReflection->invoke($formatter, '42') . "\n";

// ReflectionFunction can inspect and invoke a named function too.
$functionReflection = new ReflectionFunction('normalizeCode');
echo 'Function: ' . $functionReflection->getName() . "\n";
echo 'Function parameter: $' . $functionReflection->getParameters()[0]->getName() . "\n";
echo 'Function result: ' . $functionReflection->invoke(' inv-42 ') . "\n";
```

Save the code as `reflection-demo.php` and run it with PHP 8.0 or newer:

```sh
php reflection-demo.php
```

## Expected output

```text
Class: InvoiceFormatter
Source file: reflection-demo.php
Can instantiate: yes
Component: billing
Method: format
$invoiceNumber: string (required)
$prefix: string (optional, default: 'INV-')
Currency initialized: yes
Property currency: USD
Constant VERSION: 1.0
Invocation result: INV-42
Function: normalizeCode
Function parameter: $code
Function result: INV-42
```

## Explanation

1. **Reflect a class:** `ReflectionClass` describes `InvoiceFormatter`. The example reads its name, source filename, and whether it can be instantiated.
2. **Inspect an attribute:** `getAttributes(Component::class)` returns a `ReflectionAttribute`. Calling `newInstance()` creates the `Component` object with the stored name.
3. **Inspect a method:** `getMethod('format')` returns a `ReflectionMethod` for `format`.
4. **Inspect parameters:** Each `ReflectionParameter` provides its name, declared type, whether it has a default value, and the default value itself.
5. **Create an object:** `newInstance()` creates an `InvoiceFormatter`. This succeeds because it is concrete and has no required constructor arguments.
6. **Inspect a property:** `ReflectionProperty` reports the property and checks its initialization state for this object before reading its value.
7. **Inspect a class constant:** `getReflectionConstant('VERSION')` returns a `ReflectionClassConstant`, whose `getValue()` provides the constant's value.
8. **Invoke a method:** `ReflectionMethod::invoke()` receives the target object and method argument. The optional `$prefix` is omitted, so PHP uses `INV-`.
9. **Inspect a function:** `ReflectionFunction` describes `normalizeCode` and can invoke it with a supplied argument.

## Try these changes

- Change `$prefix` to another default and observe the reflected parameter description and method result.
- Add a required constructor argument to `InvoiceFormatter`; inspect `getConstructor()` and supply the argument with `newInstanceArgs()`.
- Add a second parameter to `normalizeCode()` and inspect it through `getParameters()`.
- Make the `currency` property uninitialized, then check `isInitialized()` before trying to read it.
- Pass an object with an invalid value to `invoke()` and observe PHP's normal type checking.

## Key takeaways

- Each Reflection class describes a different kind of declaration.
- Reflection lets tools inspect names, source, types, parameters, properties, constants, and attributes.
- `newInstance()` and `invoke()` perform dynamic operations and still follow the declarations' constructor, argument, and type rules.
- Use Reflection when runtime discovery is needed; direct code is clearer when the target is already known.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [082 - Attributes and metadata](../lessons/082-attributes-and-metadata.md) | [Q&A](../qnas/083-reflection-api.md) | Example | [084 - HTTP requests, responses, and the PHP request lifecycle](../lessons/084-http-requests-responses-and-the-php-request-lifecycle.md) |
