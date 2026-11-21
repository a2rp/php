# PHPDoc, API documentation, and type annotations — Q&A

Review runtime type declarations, PHPDoc, API contracts, and validation. Each answer follows its question.

## Multiple-choice questions

### 1. Which statement best describes native PHP types and PHPDoc?

- A. Both are enforced identically by the PHP runtime.
- B. Native declarations are checked by PHP; PHPDoc is interpreted by tools and does not enforce values at runtime.
- C. PHPDoc replaces all native declarations.
- D. Native types only affect comments.
**Answer: B.** PHP checks native parameter, return, and property declarations during execution. PHPDoc can help IDEs and static-analysis tools understand more detail, but the engine does not enforce it.

### 2. What does PHP itself know about the element types of a parameter declared as array?

- A. It knows every element is a string.
- B. It knows every element is a Customer object.
- C. It knows the value is an array, but not the element types from that declaration alone.
- D. It knows the array is non-empty.
**Answer: C.** A native array declaration does not express the element type, key shape, or whether the array is a sequential list. PHPDoc and compatible analyzers can describe those details.

### 3. What does strict_types do?

- A. It validates every HTTP request and database result.
- B. It affects scalar argument and return coercion for calls made from the file where the declaration is enabled.
- C. It makes PHPDoc types runtime-enforced.
- D. It changes every included file automatically.
**Answer: B.** Strict scalar typing is a language behavior, not request validation. The declaration is per file, and values from external boundaries still need explicit checking.

### 4. Which annotation most clearly describes a sequential list of Customer objects to compatible static-analysis tools?

- A. array
- B. list<Customer>
- C. Customer
- D. string
**Answer: B.** A list type communicates both sequential integer keys and the element type to tools that support this PHPDoc syntax. PHP itself still sees an array.

### 5. Which statement about PHPDoc generics is correct?

- A. PHP executes generic type parameters at runtime.
- B. They can preserve relationships between input and output types for static analysis.
- C. They change the installed PHP version.
- D. They replace database validation.
**Answer: B.** Generic templates help compatible tools infer type relationships. PHP's runtime does not enforce the template itself.

## Short-answer questions

### 6. What does PHPDoc add when a function already declares a native array parameter?

**Answer:** It can explain whether the array is a list or map, the key and element types, or a record shape. This improves IDE help and static analysis, but does not make PHP enforce those details at runtime.

### 7. What is an array shape?

**Answer:** It is a PHPDoc type syntax that describes a record-like array with named keys and corresponding value types, such as an integer ID and a string email. Tool support varies, and PHP itself still treats the value as an array.

### 8. When should a generic template be used?

**Answer:** Use one when a reusable function or class needs to preserve a type relationship, such as returning an item of the same type stored in a list. Keep it only when the configured tools understand it and it makes the API clearer.

### 9. What does the @throws tag communicate?

**Answer:** It documents exceptions callers may need to handle. It does not cause PHP to throw an exception, prevent other exceptions, or enforce the documentation at runtime.

### 10. What should an @deprecated annotation include?

**Answer:** Mark that the API should no longer be used and, when possible, identify its replacement or migration path. Keep the annotation aligned with the actual deprecation and removal plan.

### 11. Why is an inaccurate PHPDoc type harmful?

**Answer:** It misleads callers, IDEs, and static analyzers about what the implementation returns or accepts. A type promise should match runtime behavior; otherwise fix the implementation or correct the documentation.

### 12. What should you document when a signature alone is not enough?

**Answer:** Explain business meaning, units, accepted formats, null or empty outcomes, side effects, ordering, failure behavior, and relevant exceptions. Avoid merely restating parameter names and types.

### 13. Why is a PHPDoc array shape not request validation?

**Answer:** A comment does not inspect or reject incoming data. Parse and validate untrusted values at the application boundary before passing them into internal typed code.

### 14. Does a native string type mean that a value is a valid email address?

**Answer:** No. It means the value is a string when the type declaration is checked. Validate its format and apply any required business or authorization rules separately.

### 15. What should happen when a PHP type feature is newer than the project's minimum runtime?

**Answer:** Either avoid that syntax or raise and document the minimum PHP version. New native syntax can make older interpreters fail to parse the file before the code runs.

## API and type scenarios

### 16. A method declares a native Customer return type but its PHPDoc says it can return null. What is wrong?

**Answer:** The contracts conflict. If null is a valid outcome, the native declaration should also allow it, such as a nullable Customer type. Otherwise change the PHPDoc or the implementation so all descriptions agree.

### 17. A public method returns an array whose keys each contain an ID and a name. What should its documentation describe?

**Answer:** If the method must return that shape, document the key names and value types with the PHPDoc syntax supported by the project's analyzer. For a stable object with behavior or validation, consider returning a value object instead.

### 18. A static analyzer reports a value may be null, but a developer adds an annotation claiming it is always present. Is that enough?

**Answer:** No. Investigate the source and control flow. Add a real check or narrow the value after a condition, then document the invariant only if the implementation guarantees it. Do not use comments to hide a possible runtime error.

### 19. An application accepts an integer ID in an internal function. Is the native integer type enough to prove the caller may access that record?

**Answer:** No. The type checks only the kind of value, subject to PHP's typing rules. The application must also validate the ID's range and authorize access to the requested record.

### 20. A reusable library uses a PHPDoc feature understood by one analyzer but not by the project's supported tools. What should its maintainers do?

**Answer:** Use syntax understood by the supported toolchain, add a clearly tool-specific annotation when appropriate, or simplify the type. Test static analysis as part of the project workflow and document tool-specific assumptions.

### 21. Does PHPDoc automatically create an HTTP API schema?

**Answer:** No. PHPDoc may help document PHP code, but an HTTP API's machine-readable contract must be maintained in the format and tooling its consumers use. Keep that contract aligned with the implementation.

### 22. Should every obvious local variable have a @var annotation?

**Answer:** No. Prefer type inference when the type is clear. Add an annotation when it resolves ambiguity, documents an external boundary or complex shape, or helps tools understand a real invariant.

## Practical exercise

### 23. A function accepts an array of customer rows and returns Customer objects. What should the type contract communicate?

**Answer:** The native signature can declare an array parameter and an array result. PHPDoc should describe the input row keys and value types, and the returned list's Customer element type. The implementation should still validate data that comes from outside the trusted application boundary.

**Explanation:** Native and documented types work together: the runtime enforces supported declarations, while PHPDoc gives compatible tools more precise collection information.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [145 - PHP version compatibility and migration guides](../lessons/145-php-version-compatibility-and-migration-guides.md) | [Lesson](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) | [Example](../examples/146-phpdoc-api-documentation-and-type-annotations.md) | [147 - Code review, maintenance, and upgrades](../lessons/147-code-review-maintenance-and-upgrades.md) |