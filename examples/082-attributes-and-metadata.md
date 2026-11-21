# 082 - Attributes and metadata: Example

## Question

How can a PHP application define route, tag, and range metadata, attach it to a class, method, and parameter, then inspect and use that metadata through Reflection?

## Complete PHP example

```php
<?php

declare(strict_types=1);

use Attribute;

#[Attribute(Attribute::TARGET_CLASS | Attribute::TARGET_METHOD)]
final class Route
{
    /** @param list<string> $methods */
    public string $path;
    public array $methods;

    public function __construct(string $path, array $methods = ['GET'])
    {
        $this->path = $path;
        $this->methods = $methods;
    }
}

#[Attribute(Attribute::TARGET_CLASS | Attribute::IS_REPEATABLE)]
final class Tag
{
    public string $name;

    public function __construct(string $name)
    {
        $this->name = $name;
    }
}

#[Attribute(Attribute::TARGET_PARAMETER)]
final class Range
{
    public int $min;
    public int $max;

    public function __construct(int $min, int $max)
    {
        $this->min = $min;
        $this->max = $max;
    }
}

#[Route('/reports', methods: ['GET'])]
#[Tag('api')]
#[Tag('internal')]
final class ReportController
{
    #[Route('/reports/{id}', methods: ['GET'])]
    public function show(#[Range(min: 1, max: 9999)] int $id): string
    {
        return 'Report ' . $id;
    }
}

$controllerReflection = new ReflectionClass(ReportController::class);

// Read a class-level Route attribute as reflection metadata first.
$classRouteMetadata = $controllerReflection->getAttributes(Route::class)[0];
echo 'Route attribute name: ' . $classRouteMetadata->getName() . "\n";
echo 'Raw route argument count: ' . count($classRouteMetadata->getArguments()) . "\n";

// Create the actual Route object only when newInstance() is called.
$classRoute = $classRouteMetadata->newInstance();
echo 'Class route: ' . $classRoute->path . ' ['
    . implode(', ', $classRoute->methods) . "]\n";

// Tag is repeatable, so reflection returns both uses in declaration order.
foreach ($controllerReflection->getAttributes(Tag::class) as $tagMetadata) {
    $tag = $tagMetadata->newInstance();
    echo 'Tag: ' . $tag->name . "\n";
}

$methodReflection = $controllerReflection->getMethod('show');
$methodRoute = $methodReflection->getAttributes(Route::class)[0]->newInstance();
echo 'Method route: ' . $methodRoute->path . ' ['
    . implode(', ', $methodRoute->methods) . "]\n";

$parameterReflection = $methodReflection->getParameters()[0];
$range = $parameterReflection->getAttributes(Range::class)[0]->newInstance();
echo 'ID range: ' . $range->min . '..' . $range->max . "\n";

// Metadata does not call the method; application code does that explicitly.
$controller = new ReportController();
echo 'Result: ' . $controller->show(42) . "\n";
```

This example requires PHP 8.0 or newer.

## Expected output

```text
Route attribute name: Route
Raw route argument count: 2
Class route: /reports [GET]
Tag: api
Tag: internal
Method route: /reports/{id} [GET]
ID range: 1..9999
Result: Report 42
```

## Explanation

1. **Define attribute classes:** `Route`, `Tag`, and `Range` are regular classes marked with `#[Attribute]`. Their constructors define the metadata they store.
2. **Set allowed targets:** `Route` allows classes and methods; `Tag` allows classes; `Range` allows parameters. A target flag is declared on the attribute class, not on the place where it is used.
3. **Allow repeated tags:** `Tag` combines `Attribute::TARGET_CLASS` with `Attribute::IS_REPEATABLE`, so `ReportController` can have both tags.
4. **Attach metadata:** The class, method, and parameter each have attributes suited to their declared target.
5. **Inspect metadata:** `ReflectionClass`, `ReflectionMethod`, and `ReflectionParameter` each provide `getAttributes()` for their declaration.
6. **Read before instantiating:** `$classRouteMetadata` is a `ReflectionAttribute`. Its name and raw constructor arguments are inspectable before creating the actual `Route` object.
7. **Create the attribute object:** Calling `newInstance()` invokes the selected attribute class constructor with its stored arguments.
8. **Use metadata explicitly:** The example prints the route and range data. It then calls `show(42)` itself; the attributes do not invoke or validate the method automatically.

## Try these changes

- Remove `Attribute::IS_REPEATABLE` from `Tag` and apply `#[Tag(...)]` twice. Observe that repeated use is not allowed when the attribute is instantiated.
- Add `Attribute::TARGET_PROPERTY` to `Range`, then attach it to a property and inspect it with `ReflectionProperty`.
- Change the route's named argument from `methods` to a name that is not in the constructor and observe the constructor argument error when `newInstance()` is called.
- Add a class-level attribute to a parent class and reflect a child class. Class-level attributes are not automatically inherited.
- Use `getArguments()` for the method's `Route` metadata before calling `newInstance()`.

## Key takeaways

- Attribute classes define metadata shape, allowed targets, and whether repeated use is allowed.
- Reflection returns metadata objects first; `newInstance()` constructs the actual attribute object.
- Different declaration types use their corresponding Reflection classes.
- Attributes do not register routes, validate values, or call methods unless application code implements that behavior.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [081 - Autoloading classes and files](../lessons/081-autoloading-classes-and-files.md) | [Q&A](../qnas/082-attributes-and-metadata.md) | Example | [083 - Reflection API](../lessons/083-reflection-api.md) |
