# 083 - Reflection API

## Learning goals

By the end of this lesson, you will be able to:

- Explain how PHP Reflection describes declarations at runtime.
- Choose the correct Reflection class for a class, object, function, method, property, parameter, or class constant.
- Inspect methods, properties, parameters, types, modifiers, and source information.
- Invoke a reflected method and instantiate a class dynamically.
- Read attributes through Reflection.
- Use reflection carefully instead of making application behavior depend on unchecked names.

## 1. What is Reflection?

Reflection is PHP's API for examining classes, interfaces, traits, enums, functions, methods, properties, parameters, and constants at runtime. It can also perform certain operations, such as creating an object from reflected class information or invoking a reflected method.

Reflection does not run just because a class exists. Your code explicitly creates a Reflection object and calls its methods. This makes Reflection useful for framework internals, dependency injection, test tools, command-line tools, serializers, documentation generators, and attribute processing.

## 2. Choose the matching Reflection class

| What you want to inspect | Reflection class |
| --- | --- |
| Class, interface, trait, enum, or class-like declaration by name | `ReflectionClass` |
| An existing object and its runtime class | `ReflectionObject` |
| Function | `ReflectionFunction` |
| Method | `ReflectionMethod` |
| Property | `ReflectionProperty` |
| Function or method parameter | `ReflectionParameter` |
| Class constant | `ReflectionClassConstant` |

`ReflectionObject` extends `ReflectionClass`, so many class inspection operations are available for an existing object as well.

## 3. Inspect a class

Pass a class name or an object to `ReflectionClass`:

```php
$reflection = new ReflectionClass(Invoice::class);

 echo $reflection->getName() . "\n";
echo $reflection->getFileName() . "\n";
echo $reflection->isFinal() ? 'final' : 'not final';
```

Common `ReflectionClass` methods include:

- `getName()` for the fully qualified class name.
- `getFileName()`, `getStartLine()`, and `getEndLine()` for source location information.
- `getMethods()` and `getMethod()` for methods.
- `getProperties()` and `getProperty()` for properties.
- `getConstants()` and `getReflectionConstants()` for class constants.
- `getParentClass()` for parent class information.
- `getInterfaces()`, `getTraits()`, and `getAttributes()` for related declarations and metadata.
- `isInterface()`, `isTrait()`, `isEnum()`, `isAbstract()`, `isFinal()`, and `isInstantiable()` for declaration checks.

Some source details may be unavailable for internal PHP classes or dynamically defined code. For example, `getFileName()` can return `false` when there is no source file.

## 4. Inspect methods and their parameters

A `ReflectionMethod` describes one method. It can report the method name, declaring class, visibility, static status, return type, parameters, and attributes.

```php
$method = new ReflectionMethod(InvoiceService::class, 'create');

foreach ($method->getParameters() as $parameter) {
    $type = $parameter->getType();
    $typeName = $type === null ? 'no type declaration' : (string) $type;

    echo '$' . $parameter->getName() . ': ' . $typeName . "\n";
}
```

For each `ReflectionParameter`, useful methods include `getName()`, `getType()`, `isOptional()`, `isVariadic()`, `isPassedByReference()`, `isDefaultValueAvailable()`, and `getDefaultValue()`.

A type returned by `getType()` may be a `ReflectionNamedType`, `ReflectionUnionType`, or `ReflectionIntersectionType`, depending on the declaration. Casting a `ReflectionType` to a string is a convenient way to display its declared form. Check for `null` first because a parameter may have no declared type.

`ReflectionFunctionAbstract` is the common base for reflected functions and methods. It offers methods such as `getParameters()`, `getReturnType()`, `getNumberOfParameters()`, and `getNumberOfRequiredParameters()`.

## 5. Inspect properties and class constants

A `ReflectionProperty` can report its name, declaring class, type, visibility, static status, readonly status, default value, and attributes. With an object, `isInitialized($object)` checks whether a typed property has been initialized before it is read.

```php
$property = new ReflectionProperty(UserProfile::class, 'displayName');

echo $property->getName() . "\n";
echo $property->getType() . "\n";
echo $property->isInitialized($profile) ? 'initialized' : 'uninitialized';
```

Use `ReflectionClassConstant` for information about one class constant, including its name, value, visibility, declaring class, and attributes.

Reflection is a powerful inspection tool, but it should not be used as a reason to disregard normal class interfaces and encapsulation. Prefer public APIs in application code; reserve deeper inspection for tooling and framework code that has a clear contract.

## 6. Invoke a method through Reflection

`ReflectionMethod::invoke()` calls a method. An instance method needs an object; a static method does not:

```php
$service = new InvoiceService();
$method = new ReflectionMethod(InvoiceService::class, 'format');

$result = $method->invoke($service, 'INV-42');
```

`invokeArgs()` accepts its arguments as an array. Reflection still applies PHP's normal method rules, including argument and type constraints. A reflection call can throw the same kinds of exceptions or errors that the method itself can throw.

For ordinary application code, direct method calls are usually easier to understand. Reflection invocation is most useful when a tool must operate on a method it discovered dynamically.

## 7. Create an object dynamically

`ReflectionClass::newInstance()` creates an object by passing constructor arguments. `newInstanceArgs()` accepts an array of constructor arguments:

```php
$reflection = new ReflectionClass(Invoice::class);
$invoice = $reflection->newInstanceArgs(['INV-42']);
```

Check `isInstantiable()` before attempting to create an object. Interfaces, traits, abstract classes, and classes with inaccessible constructors cannot be instantiated directly.

`newInstanceWithoutConstructor()` deliberately skips the constructor. This can leave an object in an invalid state, so it is mainly appropriate for specialized tools such as certain serializers and test frameworks. Do not use it as a routine shortcut for object creation.

## 8. Reflection and attributes

Reflection is how application code reads PHP attributes. For example, a `ReflectionClass`, `ReflectionMethod`, `ReflectionProperty`, or `ReflectionParameter` can call `getAttributes()` for the declaration it represents.

That method returns `ReflectionAttribute` objects. Call `getName()` or `getArguments()` to inspect metadata, and call `newInstance()` when you need the actual attribute object. Attribute constructors are not run merely because the declaration was reflected.

See the previous chapter, [Attributes and metadata](082-attributes-and-metadata.md), for the full attribute syntax and a complete example.

## 9. Inspect functions and source documentation

`ReflectionFunction` describes a named function. It provides access to its parameters, return type, attributes, source file, source lines, and doc comment where available.

`getDocComment()` returns a documentation comment as a string when one exists, or `false` when none is available. Comments are documentation text; Reflection does not interpret arbitrary prose as validation rules.

## 10. Common mistakes and safety

- **Choosing the wrong Reflection class:** use the reflection class that matches the declaration being inspected.
- **Assuming inspection instantiates everything:** metadata such as attributes may need an explicit `newInstance()` call.
- **Assuming a declaration always has a source file:** internal or dynamically created declarations may not.
- **Assuming every parameter has a type:** `getType()` can return `null`.
- **Calling a method without the required object:** instance methods need an object for `invoke()`.
- **Creating an abstract class:** check `isInstantiable()` before dynamic construction.
- **Using `newInstanceWithoutConstructor()` casually:** it skips initialization and may produce an invalid object.
- **Passing unchecked user input into reflection:** map external choices through an allowlist of known classes and methods.
- **Using Reflection where a direct call is clearer:** use it when runtime discovery is required, not as a default replacement for normal code.

## Key takeaways

- Reflection describes PHP declarations at runtime and can also perform controlled operations.
- Use `ReflectionClass`, `ReflectionMethod`, `ReflectionProperty`, `ReflectionParameter`, and related classes for the declaration type you need.
- Reflection exposes declared types, parameters, modifiers, source details, and attributes.
- `invoke()` and `newInstance()` are dynamic operations; check that the target is valid and supply the required arguments.
- Prefer ordinary public APIs in application code, and constrain names that come from external input.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [082 - Attributes and metadata](082-attributes-and-metadata.md) | [Q&A](../qnas/083-reflection-api.md) | [Example](../examples/083-reflection-api.md) | [084 - HTTP requests, responses, and the PHP request lifecycle](084-http-requests-responses-and-the-php-request-lifecycle.md) |
