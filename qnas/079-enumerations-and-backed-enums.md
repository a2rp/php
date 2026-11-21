# 079 - Enumerations and backed enums: Q&A

Use these questions to review enum cases, backed scalar values, conversion methods, and enum constraints. Each answer follows its question, with an explanation where it helps.

## Multiple-choice questions

### 1. What is the main purpose of an enum?

- A. To store any number of unrelated dynamic properties.
- B. To define a type with a fixed set of valid cases.
- C. To replace every class in a PHP application.
- D. To convert all strings to integers.

**Answer:** B. To define a type with a fixed set of valid cases.

**Explanation:** A typed enum parameter restricts a function to the declared cases instead of accepting arbitrary scalar values.

### 2. How do you refer to a case named `Ready` in an enum named `TaskState`?

- A. `TaskState->Ready`
- B. `TaskState::Ready`
- C. `new TaskState('Ready')`
- D. `TaskState.Ready`

**Answer:** B. `TaskState::Ready`.

**Explanation:** Enum cases use the enum name, the scope-resolution operator, and the case name.

### 3. What does a pure enum case represent?

- A. A singleton object of the enum type.
- B. A string with the same spelling as the case.
- C. An automatically assigned integer.
- D. A new instance each time it is referenced.

**Answer:** A. A singleton object of the enum type.

**Explanation:** Repeated references to the same case return the same case instance, so strict identity comparison succeeds.

### 4. What does the `name` property of a case contain?

- A. The enum's backing type.
- B. The case's declared identifier.
- C. A localized display label.
- D. The enum's namespace only.

**Answer:** B. The case's declared identifier.

**Explanation:** For `TaskState::Ready`, the read-only `name` is `Ready`.

### 5. Which backing types can a backed enum use?

- A. `bool` or `float`
- B. `array` or `object`
- C. `int` or `string`
- D. Any union type

**Answer:** C. `int` or `string`.

**Explanation:** A backed enum uses exactly one of these scalar types for all its cases.

### 6. What is required for each case in a backed enum?

- A. A unique explicit backing value.
- B. A constructor argument at runtime.
- C. A public instance property.
- D. A default auto-incremented number.

**Answer:** A. A unique explicit backing value.

**Explanation:** PHP does not generate values automatically, and two cases in one backed enum cannot share a value.

### 7. What happens when `from()` receives a valid scalar type that has no matching case?

- A. It returns `null`.
- B. It returns the first case.
- C. It throws `ValueError`.
- D. It creates a new case.

**Answer:** C. It throws `ValueError`.

**Explanation:** `from()` is for required mappings where a missing value is an error.

### 8. What happens when `tryFrom()` receives a valid scalar type that has no matching case?

- A. It returns `null`.
- B. It throws `ValueError`.
- C. It creates a temporary case.
- D. It returns `false`.

**Answer:** A. It returns `null`.

**Explanation:** The caller can handle an unknown value or provide a fallback.

### 9. In what order does `EnumName::cases()` return cases?

- A. Alphabetical order by case name.
- B. Numerical order by backing value.
- C. Declaration order.
- D. Random order.

**Answer:** C. Declaration order.

**Explanation:** The method returns a packed array containing every case in the order declared in the enum.

### 10. In which PHP version were enums introduced?

- A. PHP 7.4
- B. PHP 8.0
- C. PHP 8.1
- D. PHP 8.3

**Answer:** C. PHP 8.1.

**Explanation:** Enum syntax and cases require PHP 8.1 or newer.

## Short-answer questions

### 11. What is the difference between a case's `name` and `value`?

**Answer:** `name` is the case identifier in PHP source; `value` is the scalar assigned to a backed case.

**Explanation:** For `OrderStatus::Paid = 'paid'`, the name is `Paid` and the value is `'paid'`.

### 12. How can you check that a value belongs to a pure enum type?

**Answer:** Use a type declaration or `instanceof`, such as `$direction instanceof Direction`.

**Explanation:** An enum case is an object of its enum type. A same-spelled string is not a case.

### 13. Can one backed enum mix integer and string values?

**Answer:** No.

**Explanation:** Each backed enum declares one backing type, either `int` or `string`, and all cases use it.

### 14. What does `OrderStatus::Paid->value` return?

**Answer:** The scalar backing value declared for the case.

**Explanation:** The `value` property exists only on backed enum cases and is read-only.

### 15. Name three kinds of behavior or declarations enums can contain.

**Answer:** Methods, static methods, constants, and interface implementations are allowed; any three are sufficient.

**Explanation:** Enums are object-like types with a constrained set of singleton cases, but they cannot store arbitrary per-case mutable properties.

### 16. What does `EnumName::cases()` return?

**Answer:** A packed array of every enum case in declaration order.

**Explanation:** PHP provides the method automatically for both pure and backed enums.

### 17. Can you implement your own `cases()` method inside an enum?

**Answer:** No.

**Explanation:** `cases()` is provided by PHP through the internal `UnitEnum` interface, and redefining it is an error.

### 18. How do `from()` and `tryFrom()` differ for an unknown backing scalar?

**Answer:** `from()` throws `ValueError`; `tryFrom()` returns `null`.

**Explanation:** Choose based on whether an unknown scalar should stop the operation or be handled as a normal possibility.

### 19. How can an enum return a different label for each case?

**Answer:** Add an instance method that uses `match ($this)` to map each case to a label.

**Explanation:** This keeps derived behavior with the enum without adding mutable state.

### 20. Why might an enum implement an interface?

**Answer:** To make each case satisfy a shared behavior contract, such as providing a label.

**Explanation:** A function can accept the interface type and work with any enum that implements it.

## Scenario questions

### 21. An API sends an unknown status string. Which conversion method is usually easier to handle: `from()` or `tryFrom()`?

**Answer:** `tryFrom()`.

**Explanation:** It returns `null` when the string has no matching case, allowing the application to return a validation error or use an intentional fallback.

### 22. A database column contains `'paid'`. How do you map it to a string-backed `OrderStatus` and handle invalid stored data?

**Answer:** Use `OrderStatus::tryFrom($value)` and handle a `null` result.

**Explanation:** This makes the mapping explicit and avoids an unhandled `ValueError` for an unknown scalar. Ensure the input is a string or deliberately validate/coerce it first.

### 23. Why should a backed value such as `'in_progress'` be chosen carefully?

**Answer:** It may be persisted or sent to other systems, so changing it can break existing data or integrations.

**Explanation:** Treat backing values as stable data contracts. Display labels can change independently through an enum method or translation layer.

### 24. Why does PHP prevent enum cases from having ordinary instance properties?

**Answer:** Cases are fixed singleton values and are not constructed as independent stateful records.

**Explanation:** If each choice needs independently stored or mutable data, use a class or another data model.

### 25. Each workflow option needs its own mutable metadata object. Should the options be enum cases?

**Answer:** Probably not; use classes or a separate data model for the mutable metadata.

**Explanation:** Enums define a closed set of fixed cases and cannot declare per-case properties.

### 26. How can a form generate options from every case in a backed enum?

**Answer:** Loop over `EnumName::cases()` and read each case's name, value, or a label method.

**Explanation:** The returned array is in declaration order and contains case objects, not plain scalars.

### 27. A UI label changes from “On the way” to “Shipped.” Should the stored backing value necessarily change too?

**Answer:** No.

**Explanation:** Keep stable storage values such as `'shipped'` and change the presentation label separately. A label is user-facing text, not necessarily a data contract.

### 28. A new enum case is added, and a `match ($this)` method has no default arm. What should the developer check?

**Answer:** Update the match expression to cover the new case.

**Explanation:** An unhandled value can cause `UnhandledMatchError` when that path runs. Explicit handling makes the expanded set visible in the code review.

### 29. What is the risk of calling `from()` on a scalar from an untrusted request without handling failure?

**Answer:** An unknown value raises `ValueError`, and a value of an invalid input type can raise `TypeError`.

**Explanation:** Validate the input type and use `tryFrom()` when an unknown case should be handled as a normal validation outcome.

### 30. Can you add a property to a trait and then use that trait in an enum?

**Answer:** No.

**Explanation:** A trait used by an enum may contain methods, static methods, and constants, but not properties.

## Code-reading questions

For questions 31–35, assume:

~~~php
enum OrderStatus: string
{
    case Draft = 'draft';
    case Paid = 'paid';
    case Shipped = 'shipped';
}
~~~

### 31. What does `OrderStatus::Paid->name` return?

**Answer:** The string `Paid`.

**Explanation:** `name` is the declared case identifier.

### 32. What does `OrderStatus::Paid->value` return?

**Answer:** The string `paid`.

**Explanation:** It is the scalar explicitly assigned to that backed case.

### 33. What does `OrderStatus::tryFrom('cancelled')` return?

**Answer:** `null`.

**Explanation:** There is no case with that backing value.

### 34. What happens with `OrderStatus::from('cancelled')`?

**Answer:** It throws `ValueError`.

**Explanation:** `from()` requires a corresponding backed case.

### 35. What does `OrderStatus::cases()` contain, in order?

**Answer:** `OrderStatus::Draft`, `OrderStatus::Paid`, then `OrderStatus::Shipped`.

**Explanation:** PHP preserves the order in which cases are declared.

## Deeper understanding

### 36. Is `OrderStatus::Paid` the same thing as the string `'paid'`?

**Answer:** No.

**Explanation:** `OrderStatus::Paid` is an enum case object. Its backing value is available as `OrderStatus::Paid->value`.

### 37. What does `json_encode(OrderStatus::Paid)` produce by default for a string-backed enum?

**Answer:** The JSON string `"paid"`.

**Explanation:** A backed enum is represented by its scalar value in JSON. A pure enum cannot be JSON encoded by default unless it provides an explicit representation, such as through `JsonSerializable`.

### 38. Does decoding a JSON string such as `"paid"` automatically create an `OrderStatus` case?

**Answer:** No.

**Explanation:** JSON decoding produces a scalar string. Map it explicitly with `OrderStatus::tryFrom()` or `from()` and handle an unknown value.

### 39. Can an enum case be cloned or instantiated with `new`?

**Answer:** No.

**Explanation:** Each case is a singleton. PHP forbids cloning cases and direct construction.

### 40. Can enums extend a base enum to inherit its cases?

**Answer:** No.

**Explanation:** Enums do not support inheritance. Share contracts through interfaces and share eligible behavior through traits without properties.

## Key takeaways

- Enums provide a fixed set of typed cases, introduced in PHP 8.1.
- A pure case is a singleton object; a backed case also has an explicit scalar `value`.
- `from()` throws for an unknown scalar, while `tryFrom()` returns `null`.
- `cases()` lists cases in declaration order.
- Enums support methods and interfaces, but not properties, constructors, cloning, or inheritance.
- Treat backing values as stable data contracts and map external data deliberately.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [078 - Late static binding](../lessons/078-late-static-binding.md) | [Lesson](../lessons/079-enumerations-and-backed-enums.md) | [Example](../examples/079-enumerations-and-backed-enums.md) | [080 - Namespaces, imports, and name resolution](../lessons/080-namespaces-imports-and-name-resolution.md) |
