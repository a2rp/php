# 082 - Attributes and metadata: Q&A

Each answer follows its question and includes an explanation.

## Multiple-choice questions

### 1. In which PHP version were attributes introduced?

- A. PHP 5.6
- B. PHP 7.4
- C. PHP 8.0
- D. PHP 8.3

**Answer:** C. PHP 8.0.

**Explanation:** Native attributes became part of the PHP language in PHP 8.0.

### 2. Which syntax starts a PHP attribute?

- A. `@[`
- B. `#[`
- C. `<<`
- D. `/**`

**Answer:** B. `#[`.

**Explanation:** An attribute group begins with `#[` and ends with `]`.

### 3. What does an attribute provide?

- A. Structured metadata attached to a declaration.
- B. Automatic execution of a method.
- C. An automatic database schema migration.
- D. A replacement for every PHP comment.

**Answer:** A. Structured metadata attached to a declaration.

**Explanation:** Application or framework code must inspect the metadata and decide what to do with it.

### 4. What must a custom attribute class include to be usable as an attribute?

- A. A `#[Attribute]` declaration.
- B. An `autoload()` method.
- C. A static `register()` method.
- D. A docblock beginning with `@Attribute`.

**Answer:** A. A `#[Attribute]` declaration.

**Explanation:** The built-in `Attribute` attribute marks a class as an attribute class.

### 5. Which API reads attributes from a class declaration?

- A. `ReflectionClass::getAttributes()`
- B. `Class::attributes()`
- C. `get_class_comments()`
- D. `Attribute::read()`

**Answer:** A. `ReflectionClass::getAttributes()`.

**Explanation:** The Reflection API provides the appropriate `getAttributes()` method for classes and other reflected declarations.

### 6. What does `getAttributes()` return?

- A. Instances of every attribute class.
- B. An array of `ReflectionAttribute` objects.
- C. Only strings containing attribute names.
- D. A boolean indicating whether metadata exists.

**Answer:** B. An array of `ReflectionAttribute` objects.

**Explanation:** Each reflection object provides the name and arguments and can create the actual attribute instance when requested.

### 7. When is the actual custom attribute object created?

- A. When the PHP file is parsed.
- B. When Reflection's `newInstance()` is called.
- C. Whenever the class is instantiated.
- D. Only when PHP starts.

**Answer:** B. When Reflection's `newInstance()` is called.

**Explanation:** This separates reading metadata from running the attribute constructor.

### 8. Which flag allows an attribute to be applied more than once to the same declaration?

- A. `Attribute::TARGET_ALL`
- B. `Attribute::IS_REPEATABLE`
- C. `ReflectionAttribute::IS_INSTANCEOF`
- D. `Attribute::ALLOW_DUPLICATE`

**Answer:** B. `Attribute::IS_REPEATABLE`.

**Explanation:** This flag belongs in the `#[Attribute(...)]` declaration of the attribute class.

### 9. How are multiple allowed target types combined?

- A. With `&&`.
- B. With a comma in the constructor.
- C. With the bitwise OR operator `|`.
- D. With `+` in every case.

**Answer:** C. With the bitwise OR operator `|`.

**Explanation:** For example, `Attribute::TARGET_CLASS | Attribute::TARGET_METHOD` allows both targets.

### 10. Which argument forms are allowed in an attribute use?

- A. Any runtime expression, including function calls.
- B. Literal values and constant expressions.
- C. Variables only.
- D. Anonymous functions and objects.

**Answer:** B. Literal values and constant expressions.

**Explanation:** Attribute arguments are part of declaration metadata and cannot depend on arbitrary runtime evaluation.

### 11. Which reflection class is used to inspect a method's attributes?

- A. `ReflectionMethod`
- B. `ReflectionClassConstant`
- C. `ReflectionParameter`
- D. `ReflectionObject`

**Answer:** A. `ReflectionMethod`.

**Explanation:** Use the reflection class corresponding to the declaration you want to inspect.

### 12. Does a child class automatically inherit its parent's class-level attributes?

- A. Yes, all class attributes are inherited.
- B. No, class attributes are not inherited automatically.
- C. Only repeatable attributes are inherited.
- D. Only attributes with `TARGET_ALL` are inherited.

**Answer:** B. Class attributes are not inherited automatically.

**Explanation:** If an application wants inherited metadata, it must explicitly inspect parent classes or related declarations.

## Short-answer questions

### 13. What is the difference between an attribute and a comment?

**Answer:** An attribute is structured metadata available through Reflection; a comment is explanatory text for readers and documentation tools.

**Explanation:** Attributes are useful for machine-readable configuration, while comments explain intent and behavior in prose.

### 14. What is an attribute class?

**Answer:** A class marked with `#[Attribute]` whose constructor defines the metadata arguments.

**Explanation:** PHP uses that class when code calls `ReflectionAttribute::newInstance()`.

### 15. What does `Attribute::TARGET_METHOD` restrict?

**Answer:** It allows the attribute to be applied to methods.

**Explanation:** Target flags specify which declaration kinds accept an attribute. A method-only attribute should not be used on a class or property.

### 16. What does `Attribute::TARGET_ALL` mean?

**Answer:** The attribute may target any supported declaration kind.

**Explanation:** It is the default target setting if no flags are passed, though a narrower target is often clearer.

### 17. How do named attribute arguments improve readability?

**Answer:** They show which constructor parameter each value is intended for.

**Explanation:** For example, `#[Route(path: '/users', methods: ['GET'])]` is easier to scan than several positional arguments.

### 18. What information can a `ReflectionAttribute` expose?

**Answer:** Its name, arguments, target, and whether it was repeated; it can also create the represented attribute object.

**Explanation:** These operations are provided by methods such as `getName()`, `getArguments()`, `getTarget()`, `isRepeated()`, and `newInstance()`.

### 19. How can `getAttributes()` filter for a specific attribute class?

**Answer:** Pass the attribute class name, commonly `MyAttribute::class`, to `getAttributes()`.

**Explanation:** This returns only matching metadata instead of requiring the caller to filter every attribute manually.

### 20. What is `ReflectionAttribute::IS_INSTANCEOF` used for?

**Answer:** It allows attribute filtering by a class or interface relationship rather than only an exact class name.

**Explanation:** Pass it as the flags argument when calling a `getAttributes()` method with an attribute-name filter.

### 21. Does writing `#[Route('/users')]` register an HTTP route by itself?

**Answer:** No.

**Explanation:** A framework or application must reflect on the declaration and register or handle the route.

### 22. Why is it useful that `newInstance()` is separate from `getAttributes()`?

**Answer:** Code can inspect names and raw arguments before running the attribute constructor.

**Explanation:** The separation provides more control when an attribute class is missing or its arguments need to be checked.

## Scenario questions

### 23. An attribute should be valid only on methods and functions. Which target mask should its class declare?

**Answer:** `Attribute::TARGET_METHOD | Attribute::TARGET_FUNCTION`.

**Explanation:** The flags are combined with bitwise OR in the `#[Attribute(...)]` declaration.

### 24. A class uses `#[Tag('api')]` and `#[Tag('internal')]`, but `Tag` is not repeatable. What should be changed?

**Answer:** Add `Attribute::IS_REPEATABLE` to `Tag`'s `#[Attribute(...)]` declaration if repeated tags are intended.

**Explanation:** Repetition is controlled by the attribute class definition, not by the attribute use site.

### 25. A developer writes `#[Route(path: getPath())]`. Why is this invalid?

**Answer:** A function call is not a literal value or constant expression allowed as an attribute argument.

**Explanation:** Supply a literal or a constant expression instead of a runtime function result.

### 26. Reflection finds an attribute name, but `newInstance()` fails because its class is not declared with `#[Attribute]`. Why?

**Answer:** The reflected metadata name alone does not make a class an attribute class.

**Explanation:** `newInstance()` needs to load and instantiate a valid attribute class; that class must be marked with `#[Attribute]`.

### 27. A `Route` attribute is declared for methods but applied to a class. At what point can code encounter the target error?

**Answer:** When Reflection attempts to instantiate the attribute with `newInstance()`.

**Explanation:** PHP can represent the metadata through Reflection, but target validation is reported when the attribute instance is requested.

### 28. A framework expects a child controller to use its parent's class-level `#[ApiController]` metadata. What should the framework do?

**Answer:** Explicitly inspect the parent class if inherited behavior is intended.

**Explanation:** Class-level attributes are not automatically returned for a child class when reflecting the child.

### 29. An attribute class constructor accepts `$path` and `$methods`. Which named arguments match it?

**Answer:** `#[Route(path: '/users', methods: ['GET'])]`.

**Explanation:** Named argument names must match the constructor parameter names.

### 30. A codebase uses `#[SensitiveParameter]` or another built-in attribute. Does every attribute need to be custom-defined by the project?

**Answer:** No. PHP and libraries can provide built-in or framework-defined attributes.

**Explanation:** The syntax is shared; code should understand the particular attribute's documented meaning and target.

## Code-reading questions

For questions 31–35, assume this code:

```php
<?php

use Attribute;

#[Attribute(Attribute::TARGET_CLASS | Attribute::IS_REPEATABLE)]
final class Label
{
    public function __construct(public readonly string $text)
    {
    }
}

#[Label('public API')]
#[Label(text: 'stable')]
class UserEndpoint
{
}
```

### 31. Is `Label` allowed on a class?

**Answer:** Yes.

**Explanation:** Its target flags include `Attribute::TARGET_CLASS`.

### 32. Can `UserEndpoint` use `Label` twice?

**Answer:** Yes.

**Explanation:** The `Label` definition includes `Attribute::IS_REPEATABLE`.

### 33. How many `ReflectionAttribute` objects does `getAttributes(Label::class)` return for `UserEndpoint`?

**Answer:** Two.

**Explanation:** The class has two `Label` uses, and the attribute is repeatable.

### 34. What values do the two reflected attributes provide to `Label::__construct()`?

**Answer:** The strings `public API` and `stable`.

**Explanation:** The first argument is positional and the second is named; both map to the constructor's `$text` parameter.

### 35. Does calling `new ReflectionClass(UserEndpoint::class)` itself instantiate both `Label` objects?

**Answer:** No.

**Explanation:** Reflection creates `ReflectionAttribute` representations. The attribute objects are created only when `newInstance()` is called on each representation.

## Key takeaways

- `#[...]` attaches structured metadata to supported declarations.
- Mark each custom attribute class with `#[Attribute]` and specify suitable targets.
- Repeated use requires `Attribute::IS_REPEATABLE`.
- Attribute arguments are literals or constant expressions, not arbitrary runtime expressions.
- Reflection returns `ReflectionAttribute` objects; `newInstance()` creates the actual object.
- Attributes do not perform application behavior automatically, and class-level attributes are not inherited.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [081 - Autoloading classes and files](../lessons/081-autoloading-classes-and-files.md) | [Lesson](../lessons/082-attributes-and-metadata.md) | [Example](../examples/082-attributes-and-metadata.md) | [083 - Reflection API](../lessons/083-reflection-api.md) |
