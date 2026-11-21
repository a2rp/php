# 083 - Reflection API: Q&A

Each answer appears directly after its question and includes an explanation.

## Multiple-choice questions

### 1. What is PHP Reflection mainly used for?

- A. Examining declarations and their details at runtime.
- B. Changing PHP's syntax before parsing.
- C. Automatically documenting every application.
- D. Loading all files in a project.

**Answer:** A. Examining declarations and their details at runtime.

**Explanation:** Reflection lets code inspect classes, methods, properties, functions, parameters, types, and related metadata.

### 2. Which class is suited to inspect a class by name?

- A. `ReflectionParameter`
- B. `ReflectionClass`
- C. `ReflectionProperty`
- D. `ReflectionFunction`

**Answer:** B. `ReflectionClass`.

**Explanation:** `ReflectionClass` describes a class-like declaration such as a class, interface, trait, or enum.

### 3. Which class reflects the runtime class of an existing object?

- A. `ReflectionObject`
- B. `ReflectionMethod`
- C. `ReflectionClassConstant`
- D. `ReflectionType`

**Answer:** A. `ReflectionObject`.

**Explanation:** `ReflectionObject` is constructed with an object and provides class information for that object's runtime class.

### 4. Which API describes a method?

- A. `ReflectionMethod`
- B. `ReflectionParameter`
- C. `ReflectionFunction`
- D. `ReflectionClassConstant`

**Answer:** A. `ReflectionMethod`.

**Explanation:** It exposes information about a method, including its declaring class, visibility, parameters, return type, and attributes.

### 5. Which API describes the parameters of a function or method?

- A. `ReflectionClass`
- B. `ReflectionParameter`
- C. `ReflectionProperty`
- D. `ReflectionObject`

**Answer:** B. `ReflectionParameter`.

**Explanation:** Obtain parameter reflection objects through a reflected function or method's `getParameters()` method.

### 6. What can `ReflectionMethod::invoke()` do?

- A. Invoke a reflected method with supplied arguments.
- B. Load every method from a file.
- C. Turn a method into a property.
- D. Change the method's declared return type.

**Answer:** A. Invoke a reflected method with supplied arguments.

**Explanation:** For an instance method, pass the target object and then the method arguments.

### 7. What does `ReflectionClass::isInstantiable()` tell you?

- A. Whether the class can be instantiated directly.
- B. Whether a method is static.
- C. Whether a property has a value.
- D. Whether a function has a doc comment.

**Answer:** A. Whether the class can be instantiated directly.

**Explanation:** Interfaces, traits, abstract classes, and classes with inaccessible constructors are not directly instantiable.

### 8. What does `ReflectionProperty::isInitialized($object)` check?

- A. Whether the property's declaration has a doc comment.
- B. Whether the property has been initialized on that object.
- C. Whether the class is abstract.
- D. Whether the property is public.

**Answer:** B. Whether the property has been initialized on that object.

**Explanation:** This is especially useful before reading a typed property that may not yet have a value.

### 9. What can `getType()` return for a parameter?

- A. Only a string.
- B. A Reflection type object, or `null` if there is no type declaration.
- C. Always a `ReflectionNamedType`.
- D. The parameter's runtime value.

**Answer:** B. A Reflection type object, or `null` if there is no type declaration.

**Explanation:** The type object can represent a named, union, or intersection type. Handle `null` before using it.

### 10. If a declaration has no available source file, what may `ReflectionClass::getFileName()` return?

- A. `false`
- B. An empty object
- C. The class name
- D. `0`

**Answer:** A. `false`.

**Explanation:** Internal declarations and dynamically created code may not correspond to an ordinary source file.

### 11. How do you inspect an attribute on a reflected method?

- A. Call `getAttributes()` on its `ReflectionMethod`.
- B. Call `getDocComment()` on its `ReflectionClass` only.
- C. Call `Attribute::getMethods()`.
- D. Call `ReflectionAttribute::getMethod()`.

**Answer:** A. Call `getAttributes()` on its `ReflectionMethod`.

**Explanation:** Reflection types expose attributes on the declaration they represent.

### 12. What does `ReflectionClass::newInstanceArgs($arguments)` do?

- A. Creates an instance using an array of constructor arguments.
- B. Invokes a static method with an array of names.
- C. Reads every class constant.
- D. Skips the constructor automatically.

**Answer:** A. It creates an instance using an array of constructor arguments.

**Explanation:** The method passes the supplied values to the constructor of an instantiable class.

## Short-answer questions

### 13. What is runtime introspection?

**Answer:** Examining information about declarations while a PHP program is running.

**Explanation:** Reflection exposes declaration details such as names, modifiers, parameters, types, source locations, and attributes.

### 14. How do you create a `ReflectionClass` for a class?

**Answer:** Pass a class name, commonly written as `SomeClass::class`, to its constructor.

**Explanation:** The `::class` expression gives a class-name string without requiring a class instance.

### 15. How do you get all methods from a reflected class?

**Answer:** Call `getMethods()` on its `ReflectionClass` object.

**Explanation:** The result is an array of `ReflectionMethod` objects. It can include inherited methods unless filters are provided.

### 16. What does `ReflectionMethod::getDeclaringClass()` return?

**Answer:** A `ReflectionClass` for the class where the method was originally declared.

**Explanation:** A method listed on a child class may have been declared in a parent class, and this method identifies its source declaration.

### 17. Why should code check whether `getType()` returned `null`?

**Answer:** A parameter or return value may have no declared type.

**Explanation:** Calling a type method on `null` would cause an error. Union and intersection types also mean the result is not always a named type.

### 18. What does casting a `ReflectionType` to a string provide?

**Answer:** A display form of its declared type.

**Explanation:** This is a convenient way to display named, union, or intersection type information without assuming it is one specific type class.

### 19. What does `ReflectionFunctionAbstract` provide?

**Answer:** Shared reflection behavior for functions and methods.

**Explanation:** Both `ReflectionFunction` and `ReflectionMethod` can expose parameters, return types, attributes, and other callable details.

### 20. What does `ReflectionClassConstant` inspect?

**Answer:** One class constant and its metadata.

**Explanation:** It can report the constant's name, value, declaring class, visibility, and attributes.

### 21. When should application code prefer a direct method call over Reflection invocation?

**Answer:** When the method is already known and can be called normally.

**Explanation:** A direct call is easier to understand and type-check. Reflection invocation is useful when a tool discovers its target dynamically.

### 22. What is a risk of `newInstanceWithoutConstructor()`?

**Answer:** It can create an object whose constructor-established invariants or required initialization are missing.

**Explanation:** Use it only for specialized tooling that knows how to initialize the object safely afterward.

## Scenario questions

### 23. A function has an optional parameter. Which Reflection API can check whether it has a default value and retrieve it?

**Answer:** Use its `ReflectionParameter` and call `isDefaultValueAvailable()` and `getDefaultValue()`.

**Explanation:** The availability check should come first so code does not request a default for a required parameter.

### 24. A class name is selected from a request parameter and passed directly to `new ReflectionClass($name)`. What should the application do instead?

**Answer:** Validate the choice against an allowlist of known classes.

**Explanation:** User-controlled names should not decide which class or method internal application code inspects or invokes.

### 25. A method is non-static. What must be supplied to `ReflectionMethod::invoke()`?

**Answer:** An instance of the class as the first argument, followed by the method's arguments.

**Explanation:** A non-static method operates on an object. A static method does not require a target object.

### 26. A reflected class is an interface. What should happen before calling `newInstance()`?

**Answer:** Check `isInstantiable()`; an interface cannot be instantiated directly.

**Explanation:** The application should select a concrete implementation instead.

### 27. A typed property has not been assigned yet. How can code safely check before reading it?

**Answer:** Use `ReflectionProperty::isInitialized($object)`.

**Explanation:** An uninitialized typed property cannot be read like a normal initialized property.

### 28. A method appears in `getMethods()` for a child class, but its declaring class is the parent. How can code confirm that?

**Answer:** Call `getDeclaringClass()` on the method's `ReflectionMethod`.

**Explanation:** Reflection distinguishes the class through which a method was found from the class where it was declared.

### 29. A framework wants to read an attribute constructor argument without constructing the attribute object. What can it call?

**Answer:** Call `getArguments()` on the corresponding `ReflectionAttribute`.

**Explanation:** Reflection exposes the stored arguments before `newInstance()` executes the attribute constructor.

### 30. A script expects `getFileName()` to always return a path, but it receives `false`. Is that necessarily a bug?

**Answer:** No.

**Explanation:** Some declarations do not have an ordinary source file, including internal declarations and certain dynamically defined code.

## Code-reading questions

For questions 31–35, assume the following declarations:

```php
<?php

final class Receipt
{
    public function format(int|string $number, string $prefix = 'R-'): string
    {
        return $prefix . $number;
    }
}
```

### 31. What does this expression describe?

```php
$method = new ReflectionMethod(Receipt::class, 'format');
```

**Answer:** The `format` method declared by `Receipt`.

**Explanation:** `ReflectionMethod` accepts a class name and method name to describe that method.

### 32. What can `$method->getParameters()` return for this method?

**Answer:** Two `ReflectionParameter` objects, for `$number` and `$prefix`.

**Explanation:** The reflected parameter list follows the method declaration order.

### 33. What type declaration does the first parameter's reflected type display?

**Answer:** `string|int` or an equivalent order such as `int|string`, depending on the display form.

**Explanation:** The first parameter has a union type containing `int` and `string`; it is not a single named type.

### 34. How can code check that the second parameter has a default value and retrieve it?

**Answer:** Get the second `ReflectionParameter`, check `isDefaultValueAvailable()`, then call `getDefaultValue()`.

**Explanation:** For `$prefix`, the default value is the string `R-`.

### 35. What does this call return?

```php
$receipt = new Receipt();
$result = $method->invoke($receipt, 42);
```

**Answer:** The string `R-42`.

**Explanation:** The reflected method receives `42` as `$number`; `$prefix` is omitted, so PHP uses its default value `R-`.

## Key takeaways

- Reflection inspects runtime declarations; it does not automatically change application behavior.
- Use the Reflection class that matches the declaration you need to inspect.
- Check nullable reflection results such as a missing type or source filename.
- `invoke()` and `newInstanceArgs()` are dynamic operations that need valid targets and arguments.
- Prefer direct calls for known targets, and restrict dynamically selected classes or methods to an allowlist.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [082 - Attributes and metadata](../lessons/082-attributes-and-metadata.md) | [Lesson](../lessons/083-reflection-api.md) | [Example](../examples/083-reflection-api.md) | [084 - HTTP requests, responses, and the PHP request lifecycle](../lessons/084-http-requests-responses-and-the-php-request-lifecycle.md) |
