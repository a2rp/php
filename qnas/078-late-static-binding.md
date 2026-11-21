# 078 - Late static binding: Q&A

Review how PHP resolves `self::`, `static::`, and `parent::` in inheritance. Each answer follows its question, with an explanation where it helps clarify the behavior.

## Multiple-choice questions

### 1. What does `self::` refer to in a class method?

- A. The class that was called most recently at runtime in every use.
- B. The class where the reference is written.
- C. The parent class in every use.
- D. The current object instance.

**Answer:** B. The class where the reference is written.

**Explanation:** `self::class` and references to members through `self::` are tied to the class scope where they appear. A `self::method()` call can still forward late-binding context for later `static::` resolution, which is covered separately below.

### 2. What does `static::` refer to in a late static binding context?

- A. Only the base class where the method is defined.
- B. The runtime called class.
- C. The parent of the current class.
- D. An object stored in `$this` only.

**Answer:** B. The runtime called class.

**Explanation:** PHP resolves `static::` using runtime class information from the original call.

### 3. A child class inherits a method that returns `self::class`. What name does that method return?

- A. The child class name.
- B. The class where the method is defined.
- C. The most derived class name in all cases.
- D. The parent class of the child.

**Answer:** B. The class where the method is defined.

**Explanation:** `self::class` is resolved in the source class where that expression is written. Use `static::class` when you need the called class.

### 4. Why would an inherited factory use `new static()`?

- A. To create an object of the class that called the inherited factory.
- B. To call the constructor without arguments.
- C. To make the factory method non-static.
- D. To create a parent object.

**Answer:** A. To create an object of the class that called the inherited factory.

**Explanation:** `new static()` uses the late-bound called class, so a child can inherit the factory and receive its own class instance.

### 5. What class does `new self()` create inside an inherited base method?

- A. The class where `new self()` is written.
- B. The runtime called child class.
- C. The parent's parent class.
- D. A `stdClass` object.

**Answer:** A. The class where `new self()` is written.

**Explanation:** `self` is lexically bound. Use `new static()` if the factory is meant to construct the runtime called class.

### 6. Which syntax can forward late-binding context while invoking the parent implementation?

- A. `parent::method()`
- B. `ParentClass::method()`
- C. `new parent()`
- D. `$this::parent()`

**Answer:** A. `parent::method()`.

**Explanation:** `parent::` is a forwarding call. An explicitly named class call such as `ParentClass::method()` is fully resolved and resets the called-class context.

### 7. What does `Child::inheritedMethod()` return if the method calls `static::label()` and `Child` overrides `label()`?

- A. The base `label()` result.
- B. The child `label()` result.
- C. A syntax error.
- D. The parent's parent result.

**Answer:** B. The child `label()` result.

**Explanation:** The inherited method's `static::` reference uses `Child` as the runtime called class and dispatches to the override.

### 8. Since which PHP version is `static` supported as a return-only type?

- A. PHP 7.4
- B. PHP 8.0
- C. PHP 8.2
- D. PHP 8.4

**Answer:** B. PHP 8.0.

**Explanation:** The `static` return type expresses a return value compatible with the late-bound called class.

### 9. What does `get_called_class()` return in a valid late static binding context?

- A. The class name associated with the call.
- B. The parent method name.
- C. The current file name.
- D. The class's namespace only.

**Answer:** A. The class name associated with the call.

**Explanation:** It returns a string containing the called class, including its namespace when applicable.

### 10. Which statement about `static::` in a non-static method is accurate?

- A. Late static binding only works in static methods.
- B. The called class can be based on the object's runtime class.
- C. It always refers to `self::class`.
- D. It turns the method into a static method.

**Answer:** B. The called class can be based on the object's runtime class.

**Explanation:** Late static binding can be used in non-static contexts too. In that case the called class is based on the object instance.

## Short-answer questions

### 11. What problem does late static binding solve?

**Answer:** It lets inherited code use the class that was actually called at runtime when it performs a `static::` reference.

**Explanation:** Without this, a base method that uses only `self::` may keep selecting base-class behavior even when a subclass calls it.

### 12. What is the shortest distinction between `self::` and `static::`?

**Answer:** `self::` is tied to the class where the reference is written; `static::` uses the runtime called class.

**Explanation:** The nuance is that `self::method()` and `parent::method()` can forward called-class information for a later `static::` resolution.

### 13. How do you get the runtime class name inside an inherited method?

**Answer:** Use `static::class` or call `get_called_class()` in a valid context.

**Explanation:** `self::class` instead identifies the class where that expression is written.

### 14. What is a forwarding call?

**Answer:** A call made through a forwarding form such as `self::`, `parent::`, or `static::` that preserves the called-class context for later late static binding.

**Explanation:** A direct fully resolved call like `BaseClass::method()` does not preserve the earlier called class; it establishes `BaseClass` as the new context.

### 15. What is the difference between `new self()` and `new static()`?

**Answer:** `new self()` constructs the class where the code is written; `new static()` constructs the runtime called class.

**Explanation:** This distinction lets an inherited factory either deliberately return a base object or preserve the subclass type.

### 16. What contract does a `: static` return type express?

**Answer:** The method returns an instance compatible with the class used for the call.

**Explanation:** It is useful for subclass-aware factories and fluent methods. The return-only `static` type was introduced in PHP 8.0.

### 17. Can `static` be used as a static property type in the same sense as a return type?

**Answer:** No.

**Explanation:** Here `static` is the late static binding scope or a special return-only type, not a property type declaration.

### 18. What does an explicitly named call such as `Base::method()` do to late-binding context?

**Answer:** It creates a fully resolved call using `Base` as the called class for that method call.

**Explanation:** Unlike forwarding forms, it does not carry an earlier subclass context into the method.

### 19. Can late static binding be used from a non-static method?

**Answer:** Yes.

**Explanation:** In a non-static context, PHP can use the class of the object instance as the called class. Use it only when subclass dispatch is intentional.

### 20. When should a factory use a declared return type such as `self` instead of `static`?

**Answer:** When its contract always promises the defining class's type and is not meant to preserve subclass-specific return types.

**Explanation:** `static` is more specific about late-bound subclasses. A method should declare the actual guarantee it can uphold.

## Scenario questions

### 21. `Child::identify()` is inherited from `Base`, and `identify()` returns `static::class`. What does it return?

**Answer:** `Child`.

**Explanation:** `static::class` uses the called class, even though the method was defined in `Base`.

### 22. The same inherited method returns `self::class`. What does it return?

**Answer:** `Base`.

**Explanation:** The class name is resolved where the expression is written.

### 23. A base static factory currently uses `new self()`, but calling `AdminUser::create()` returns a `User`. What change preserves the subclass?

**Answer:** Use `new static()` and a compatible `: static` return type.

**Explanation:** `new static()` constructs the late-bound class. `: static` documents the factory's subclass-aware result.

### 24. `Leaf::forward()` is defined in `Branch` and calls `parent::report()`. `Root::report()` returns `static::class`. What does it return?

**Answer:** `Leaf`.

**Explanation:** `parent::` calls the parent's implementation while forwarding the called-class context.

### 25. `Leaf::fixed()` is defined in `Branch` and calls `Root::report()` directly. What does `Root::report()` return?

**Answer:** `Root`.

**Explanation:** The explicit `Root::` call is fully resolved and resets the context.

### 26. A model base class defines `protected static string $table = 'records'`. A subclass sets its own `$table = 'users'`. What should the inherited method use to return `users`?

**Answer:** `static::$table`.

**Explanation:** It reads the static property in the late-bound called class. `self::$table` refers to the base-class scope.

### 27. A method should always return a `BaseQuery`, even when inherited by `UserQuery`. Is `: static` automatically correct?

**Answer:** No.

**Explanation:** Use `: static` only if the implementation guarantees an instance compatible with the called subclass. Otherwise declare the base contract or redesign the method.

### 28. A subclass overrides a constructor with required extra parameters. What risk does it introduce for an inherited `new static($id)` factory?

**Answer:** The factory may call that constructor with an incompatible argument list.

**Explanation:** Subclass-aware construction relies on compatible constructors. Override the factory or use an explicit creation contract when subclasses need different construction.

### 29. A base method should never allow subclasses to change its helper behavior. Should it use `static::helper()`?

**Answer:** Usually no; use `self::helper()` or make the helper final if that fixed behavior is required.

**Explanation:** `static::` is an extension point. Use it only when subclass dispatch is intentional.

### 30. An instance method inherited by `MailService` calls `static::name()`. Which implementation can PHP select?

**Answer:** The implementation associated with `MailService`, if it overrides the method.

**Explanation:** Late static binding can use the runtime class of the object even when the calling method is non-static.

## Code-reading questions

For questions 31–34, assume:

~~~php
class Base
{
    public static function name(): string
    {
        return 'base';
    }

    public static function viaSelf(): string
    {
        return self::name();
    }

    public static function viaStatic(): string
    {
        return static::name();
    }
}

class Child extends Base
{
    public static function name(): string
    {
        return 'child';
    }
}
~~~

### 31. What does `Child::viaSelf()` return?

**Answer:** `base`.

**Explanation:** `self::name()` in `Base::viaSelf()` selects `Base::name()`.

### 32. What does `Child::viaStatic()` return?

**Answer:** `child`.

**Explanation:** `static::name()` uses `Child` as the called class and selects its override.

### 33. What does `Child::class` evaluate to?

**Answer:** The class-name string `Child`.

**Explanation:** `Child::class` is a compile-time class-name expression, independent of method dispatch.

### 34. What would `static::class` return inside `Base::viaStatic()` when invoked as `Child::viaStatic()`?

**Answer:** `Child`.

**Explanation:** The expression is evaluated in the runtime called-class context.

### 35. In an inherited factory, what does `return new static($value);` create when called as `SpecialRecord::create($value)`?

**Answer:** A `SpecialRecord` instance, provided its constructor is compatible.

**Explanation:** `static` is resolved to the called class. The constructor compatibility is part of the inherited factory's design contract.

## Deeper understanding

### 36. Is `static` in `public static function run()` the same feature as `static::run()`?

**Answer:** No.

**Explanation:** In a declaration, `static` marks the method as callable without an object instance. In `static::`, it is the late-bound class scope used for member resolution.

### 37. Why can `self::method()` still forward late-binding context if `self` is lexically bound?

**Answer:** The selected method is resolved in the current class scope, while the forwarding call can preserve the runtime called-class information for later `static::` use.

**Explanation:** Method selection and called-class context are related but distinct parts of PHP's resolution rules.

### 38. Does `static::` guarantee a child override exists?

**Answer:** No.

**Explanation:** If the subclass does not override the referenced member, PHP can use an inherited implementation. An abstract contract or interface may be needed when an override is mandatory.

### 39. What should a library document when it uses `new static()` in a base factory?

**Answer:** It should document constructor requirements and any override points subclasses must preserve.

**Explanation:** The factory's call is dynamic, so subclasses with incompatible constructors can break it.

### 40. When is late static binding a poor fit?

**Answer:** When subclass dispatch is accidental, hidden, or harder to understand than an explicit dependency or factory.

**Explanation:** Dynamic dispatch should express a real extension contract. Otherwise a normal instance method or explicit factory usually makes dependencies clearer.

## Key takeaways

- `self::` resolves the class where a reference is written; `static::` resolves the runtime called class.
- `self::` and `parent::` method calls can forward late-binding context.
- Fully specified class calls establish their named class as the context.
- `new static()` and `: static` support subclass-aware factories and fluent APIs.
- Late static binding works in static and instance contexts, but should be an intentional extension point.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [077 - Cloning and comparing objects](../lessons/077-cloning-and-comparing-objects.md) | [Lesson](../lessons/078-late-static-binding.md) | [Example](../examples/078-late-static-binding.md) | [079 - Enumerations and backed enums](../lessons/079-enumerations-and-backed-enums.md) |
