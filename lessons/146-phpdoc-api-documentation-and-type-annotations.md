# PHPDoc, API documentation, and type annotations

Types and documentation explain what values code expects, what it returns, and how callers should use it. PHP can enforce many native type declarations at runtime. PHPDoc can describe additional details for maintainers, IDEs, and static-analysis tools, but PHP itself does not enforce those comments.

Good type annotations make APIs easier to call correctly. Good documentation explains intent and behavior that a type alone cannot show.

## Learning goals

After this chapter, you should be able to:

- Add native parameter, return, and property types where they clarify a contract.
- Explain the difference between runtime type declarations and PHPDoc.
- Document collections, array shapes, templates, exceptions, and deprecations.
- Use types without mistaking them for validation of untrusted input.
- Write concise public API documentation that stays consistent with behavior.
- Keep language features and PHPDoc syntax within the project's supported toolchain.

## Native types and PHPDoc have different jobs

A native type declaration is part of PHP syntax. The engine checks it when the program runs. Depending on the type, a mismatch can produce a TypeError. Parameter, return, and property types have different support histories, so verify a feature against the project's minimum PHP version.

~~~php
<?php
declare(strict_types=1);

function formatPrice(int|float $amount): string
{
    return number_format($amount, 2, '.', '');
}
~~~

A PHPDoc comment is a structured comment. PHP does not enforce its types at runtime, but editors and static-analysis tools can use them.

~~~php
<?php
/**
 * @param list<string> $names
 * @return list<string>
 */
function normalizeNames(array $names): array
{
    return array_map('trim', $names);
}
~~~

The native declaration says that the argument and result are arrays. The PHPDoc gives a more precise contract: each value is a string, and the arrays are sequential lists. The engine will not verify that every array element is a string; an analyzer can check the code against that promise.

Use native types for contracts the language can express. Use PHPDoc to add precision that the runtime type system does not express, and keep both descriptions consistent.

## Common native type declarations

Native declarations can describe many common function and object contracts:

~~~php
<?php
function findCustomer(int $id): ?Customer
{
    // Return a Customer or null when no record exists.
}

function parseInput(string|int $input): string
{
    return (string) $input;
}

function sendMessage(Message $message): void
{
    // Send the message.
}
~~~

Common types include scalar types such as int, string, bool, and float; class or interface names; nullable types; union types; and special return types such as void and never. Intersection and other newer type forms have minimum-version requirements. Check the PHP version targeted by the project before using them.

Use the narrowest honest type:

- Use a class or interface when a specific object contract is required.
- Use a union when callers may supply one of a small set of valid types.
- Use a nullable type only when null is a real and documented outcome.
- Use void when a function intentionally has no return value.
- Use never for a function that cannot return normally, such as one that always throws or exits.

Do not use mixed as a shortcut when a more accurate contract is available. If an external value genuinely has not been checked yet, keep it at the boundary and validate or narrow it before using it deeper in the application.

## Understand strict scalar typing

The strict_types declaration affects scalar argument and return coercion for calls made from the file where the call occurs. It is not input validation, and it does not make every PHP value strictly typed. Objects and arrays still need correct application-level validation.

~~~php
<?php
declare(strict_types=1);

function quantity(int $count): int
{
    return $count;
}

$result = quantity(3);
~~~

A value from an HTTP request, JSON body, environment variable, or database can still be malformed or out of the expected range. Validate external input explicitly before relying on the internal function contract.

Use a consistent strict-typing policy across project files. Adding strict_types to one file does not automatically change calls made from other files.

## Document array and collection contents

The runtime type array does not say whether a value is a list, a map, or an array of records. PHPDoc can make this intent clearer:

~~~php
<?php
/**
 * @param list<Customer> $customers
 */
function printCustomerNames(array $customers): void
{
    foreach ($customers as $customer) {
        echo $customer->name . PHP_EOL;
    }
}
~~~

Common analyzer-recognized forms include:

~~~text
array<string, Customer>
list<Customer>
non-empty-list<string>
array{id: int, email: string}
~~~

An array shape describes a record-like array with named keys and value types. A list describes sequential integer keys. These are PHPDoc type syntaxes interpreted by tools; PHP itself still sees an array. Supported details can differ between analyzers, so use the syntax supported by the project and configure the analyzer consistently.

Prefer a small value object over a complex array shape when the data has behavior, validation rules, or a stable public contract.

## Use templates for reusable type relationships

A generic PHPDoc template can preserve the relationship between an input type and an output type when native PHP syntax cannot express it.

~~~php
<?php
/**
 * @template T
 * @param list<T> $items
 * @return T|null
 */
function firstItem(array $items): mixed
{
    return $items[0] ?? null;
}
~~~

A compatible static analyzer can infer that the return value has the same element type as the input list, or null when the list is empty. The PHP engine enforces only the native array and mixed declarations here; it does not enforce the template.

Use templates when they make a reusable API more precise. Avoid advanced annotations that the project tools do not understand or that make a simple function harder to read.

## Write useful PHPDoc blocks

A concise PHPDoc block should explain information a caller cannot learn clearly from the signature or implementation.

~~~php
<?php
/**
 * Finds an active customer by its identifier.
 *
 * Returns null when the customer does not exist or is inactive.
 *
 * @param positive-int $id A persisted customer identifier.
 * @return Customer|null The active customer, or null when none matches.
 * @throws DatabaseException When the database query fails.
 */
function findActiveCustomer(int $id): ?Customer
{
    // ...
}
~~~

Some precise types in this example, such as positive-int, require a static-analysis tool to interpret them. PHP only enforces the native int and nullable Customer declaration. Validate that the ID is positive where it enters the application.

Useful tags include:

- **@param** to describe a parameter when the name and type are not enough.
- **@return** to explain a result or a more precise collection type.
- **@throws** to document exceptions callers may need to handle.
- **@var** to clarify a local or property type when inference is insufficient.
- **@deprecated** to mark an API that callers should stop using and point to its replacement.
- **@template** and related generic tags when type relationships need to be preserved.

Do not repeat the entire function signature in prose. Explain business meaning, null behavior, side effects, units, ordering, ownership, and failure behavior where they matter.

## Keep API documentation accurate

A public PHP API includes more than HTTP endpoints. It can include public classes, methods, function parameters, return values, exceptions, callbacks, and configuration.

For each public entry point, explain:

- What it does and when to use it.
- Required inputs, units, accepted formats, and meaningful boundaries.
- Whether null, false, an empty collection, or an exception signals a particular outcome.
- Whether it mutates state, performs I/O, sends a message, or has other side effects.
- Which exceptions may be thrown and what callers can do about them.
- Whether the behavior is stable, experimental, internal, or deprecated.
- A short example when usage is not obvious.

PHPDoc does not automatically define an HTTP API schema. If an HTTP API needs a machine-readable contract, maintain that contract in the format and tooling the API consumers use, and keep it aligned with the PHP implementation.

## Do not confuse annotations with validation

A docblock saying an array contains integer IDs does not make incoming JSON trustworthy. A native string type does not establish that a value is a valid email, path, URL, currency amount, or authorized identifier.

At trust boundaries:

1. Parse the external representation.
2. Check presence, type, range, and format.
3. Apply authorization and business rules.
4. Convert it into a validated domain value or object.
5. Pass the validated value through typed internal APIs.

Static analysis can find mistakes in program code. It does not validate data received from a browser or guarantee that a database row satisfies every comment.

## Keep annotations consistent with implementation

Incorrect PHPDoc can be worse than no PHPDoc because it gives callers and tools false confidence.

- Keep the native signature and docblock compatible.
- Update documentation whenever behavior or return values change.
- Do not claim a method throws only one exception if other documented failures can escape.
- Do not describe a nullable result as always present.
- Keep examples runnable and representative.
- Remove stale annotations when the implementation no longer matches them.
- Prefer inference when the type is obvious and explicit annotations when they clarify a boundary.

Static-analysis findings should be resolved deliberately. Do not silence them with broad ignore rules or overly permissive mixed types without understanding the underlying mismatch.

## Plan annotations around project compatibility

PHP language features have minimum-version requirements, and analyzer-specific PHPDoc types are understood only by compatible tools. Before adding a type:

- Confirm the syntax is supported by the project's minimum PHP version.
- Confirm the analyzer and IDE recognize the PHPDoc form.
- Keep PHPDoc aliases or analyzer-specific tags clearly identified.
- Run the project's static-analysis and test commands on supported runtimes.
- Update the Composer PHP requirement if the implementation now needs a newer language feature.

Do not claim older PHP compatibility just because a comment uses a type syntax the old interpreter ignores. Native syntax can make a file fail to parse on an older runtime.

## Common mistakes

- Treating PHPDoc as runtime validation.
- Giving an array a precise documented shape but never checking external data.
- Writing a native type that requires a newer PHP version than the project allows.
- Duplicating a signature in prose without explaining behavior.
- Letting the PHPDoc return type contradict the actual implementation.
- Using mixed everywhere to quiet a static analyzer.
- Documenting exceptions or side effects inaccurately.
- Assuming PHPDoc array shapes and generics have identical support in every tool.
- Using strict_types as a substitute for request validation.
- Treating PHPDoc as an HTTP API schema without maintaining the actual API contract.

## Documentation review checklist

- [ ] Public parameters and return values have useful native types.
- [ ] Null, empty, and error outcomes are documented accurately.
- [ ] Array contents, key shapes, and iterable element types are clear where needed.
- [ ] PHPDoc types do not conflict with native declarations.
- [ ] External inputs are validated rather than trusted from annotations.
- [ ] Language features match the minimum PHP version.
- [ ] Analyzer-specific types match the configured tool.
- [ ] Exceptions, side effects, units, and deprecations are documented where relevant.
- [ ] Examples match the current implementation.

## Key takeaways

- Native type declarations are checked by PHP; PHPDoc comments are interpreted by tools, not enforced by the engine.
- Use native types for runtime contracts and PHPDoc for useful additional detail such as collection element types and templates.
- Strict scalar typing is not input validation.
- Document API behavior that signatures alone cannot explain.
- Keep annotations accurate and compatible with the project's runtime and static-analysis tools.
- Validate all untrusted data at application boundaries.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [145 - PHP version compatibility and migration guides](145-php-version-compatibility-and-migration-guides.md) | [Q&A](../qnas/146-phpdoc-api-documentation-and-type-annotations.md) | [Example](../examples/146-phpdoc-api-documentation-and-type-annotations.md) | [147 - Code review, maintenance, and upgrades](147-code-review-maintenance-and-upgrades.md) |