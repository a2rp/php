# 080 - Namespaces, imports, and name resolution: Q&A

These questions review namespace declarations, aliases, and how PHP resolves class-like names, functions, and constants. Each answer follows its question.

## Multiple-choice questions

### 1. What does a PHP namespace primarily organize?

- A. Directories and files on disk.
- B. Classes, interfaces, traits, enums, functions, and constants by name.
- C. Variables in a function.
- D. Composer packages only.

**Answer:** B. Classes, interfaces, traits, enums, functions, and constants by name.

**Explanation:** A namespace gives symbols a qualified name. It does not itself create directories or load files.

### 2. Where must a namespace declaration normally appear?

- A. At the end of a file.
- B. At the top of the file, before other code.
- C. Inside a class.
- D. Anywhere, including inside a function.

**Answer:** B. At the top of the file, before other code.

**Explanation:** A `declare` statement may precede it. Other code or output before the declaration is not allowed.

### 3. Which is a fully qualified class name?

- A. `Invoice`
- B. `Billing\Invoice`
- C. `\Storefront\Billing\Invoice`
- D. `namespace\Invoice`

**Answer:** C. `\Storefront\Billing\Invoice`.

**Explanation:** A fully qualified name starts with a leading namespace separator and is absolute from the global namespace.

### 4. Inside `Storefront\Billing`, how is an unimported qualified class name `Models\Invoice` resolved?

- A. `Models\Invoice`
- B. `Storefront\Billing\Models\Invoice`
- C. `Storefront\Models\Invoice`
- D. `\Models\Invoice`

**Answer:** B. `Storefront\Billing\Models\Invoice`.

**Explanation:** Without a matching imported prefix, PHP prepends the current namespace to a qualified name.

### 5. What does this import do?

~~~php
use Vendor\Billing\Invoice as Bill;
~~~

- A. Includes the file containing `Invoice`.
- B. Defines a new `Bill` class.
- C. Lets this file refer to `Vendor\Billing\Invoice` as `Bill`.
- D. Moves `Invoice` into the current namespace.

**Answer:** C. It lets this file refer to `Vendor\Billing\Invoice` as `Bill`.

**Explanation:** `use` creates a compile-time alias; it does not declare a class or load a file.

### 6. Which syntax imports a function?

- A. `use Vendor\Support\slugify;`
- B. `use function Vendor\Support\slugify;`
- C. `use const Vendor\Support\slugify;`
- D. `import function Vendor\Support\slugify;`

**Answer:** B. `use function Vendor\Support\slugify;`.

**Explanation:** Functions have their own import table and require `use function`.

### 7. In a namespace, if class `DateTimeImmutable` is not imported, where does `new DateTimeImmutable()` look?

- A. It always falls back to global `DateTimeImmutable`.
- B. It looks for the class in the current namespace.
- C. It searches every namespace.
- D. It calls a global function instead.

**Answer:** B. It looks for the class in the current namespace.

**Explanation:** Class-like names do not have the global fallback rule. Import the global class or write `new \DateTimeImmutable()`.

### 8. If an unqualified function is not imported or defined in the current namespace, what fallback can PHP use?

- A. A function in a parent namespace.
- B. The matching global function.
- C. A method with the same name.
- D. The first loaded function.

**Answer:** B. The matching global function.

**Explanation:** Unqualified function names can resolve first in the current namespace and then fall back to the global function.

### 9. What does `__NAMESPACE__` contain in global, non-namespaced code?

- A. `global`
- B. `root`
- C. An empty string.
- D. `\\`

**Answer:** C. An empty string.

**Explanation:** It contains the current namespace name, which is empty in the global namespace.

### 10. Do import aliases affect dynamic class names stored in strings?

- A. Yes, imports rewrite all matching string values.
- B. No, imports apply to statically written names, not dynamic strings.
- C. Only for classes, never for interfaces.
- D. Only in strict mode.

**Answer:** B. Imports apply to statically written names, not dynamic strings.

**Explanation:** A dynamic name should contain the fully qualified name it needs at runtime. `SomeClass::class` is a useful source for that string.

## Short-answer questions

### 11. What is an unqualified name?

**Answer:** A name with no namespace separator, such as `Invoice`.

**Explanation:** Its resolution depends on the current namespace and any applicable import.

### 12. What is a qualified name?

**Answer:** A name with a namespace separator but no leading backslash, such as `Models\Invoice`.

**Explanation:** PHP resolves it through an imported prefix if one matches; otherwise, it prepends the current namespace.

### 13. What is a fully qualified name?

**Answer:** A name beginning with a leading backslash, such as `\Vendor\Billing\Invoice`.

**Explanation:** It refers to that absolute name regardless of the current namespace or imports.

### 14. What does the `namespace\` prefix mean?

**Answer:** Resolve the following name relative to the current namespace.

**Explanation:** For example, `namespace\Support\Logger` inside `App` means `App\Support\Logger`.

### 15. Which symbol types can be imported with `use`?

**Answer:** Classes and other class-like names, namespaces, functions, and constants.

**Explanation:** Use `use function` for functions and `use const` for constants; a plain `use` imports class-like names or namespace prefixes.

### 16. What is a grouped import?

**Answer:** A compact syntax for importing multiple names from the same namespace.

**Explanation:** For example, `use Vendor\Package\{Client, Request as ApiRequest};` is shorthand for separate imports.

### 17. How are unqualified namespaced functions and constants resolved when no import applies?

**Answer:** PHP first looks in the current namespace and can then fall back to the global function or constant.

**Explanation:** This fallback is different from class resolution. A qualified function or constant name does not use this unqualified fallback rule.

### 18. Does importing `Vendor\Billing\Invoice` make the class available at runtime by itself?

**Answer:** No.

**Explanation:** The import only provides an alias. The file still needs to be included or the class must be found by an autoloader when it is used.

### 19. Are import aliases shared with a file that the current file includes?

**Answer:** No.

**Explanation:** Imports are scoped to the file and namespace declaration where they appear.

### 20. How does namespace organization relate to autoloading?

**Answer:** Namespaces provide symbol names; autoloading locates and loads files for class names.

**Explanation:** A project may map namespace prefixes to directories, but that mapping is a separate autoloader convention.

## Scenario questions

### 21. Inside `App\Clock`, you need PHP's global `DateTimeImmutable`. What are two valid ways to refer to it?

**Answer:** Import it with `use DateTimeImmutable;` or refer to it as `\DateTimeImmutable`.

**Explanation:** An unqualified class name otherwise resolves in the current namespace, with no global class fallback.

### 22. Inside `App\Errors`, you want to extend PHP's global `Exception`. What should the declaration use?

**Answer:** `class ApiException extends \Exception {}` or a `use Exception;` import.

**Explanation:** The leading backslash explicitly selects the global class.

### 23. Inside `App\Text`, there is no `App\Text\strlen()` function. What can an unqualified `strlen('PHP')` call?

**Answer:** PHP's global `strlen()` function.

**Explanation:** Unqualified function calls fall back to the global function if no namespaced function or import resolves the name.

### 24. Inside `App\Models`, an unimported `new User()` fails, but a global `User` class exists. Why?

**Answer:** PHP resolves the unqualified class-like name to `App\Models\User` and does not fall back to the global `User` class.

**Explanation:** Add `use User;` or write `new \User()` for the global class.

### 25. A developer writes `use App\Services\Mailer;` but PHP reports that the class cannot be found. What does the import not do?

**Answer:** It does not include or autoload the file.

**Explanation:** Configure an autoloader or include the file so PHP can find the class declaration.

### 26. Two packages both define `Logger`. How can one file use both?

**Answer:** Import each with a distinct alias, such as `use VendorA\Log\Logger as AuditLogger;` and `use VendorB\Log\Logger as AppLogger;`.

**Explanation:** The aliases avoid a collision in the current file's imported names.

### 27. A single demonstration file contains two named namespaces and global code. Which declaration style should it use?

**Answer:** Use bracketed namespace declarations for all sections and put global code in `namespace { ... }`.

**Explanation:** Global code cannot be combined with unbracketed namespace declarations, and bracketed and unbracketed styles cannot be mixed in one file.

### 28. An import alias is named `HttpClient`, while a class in the current namespace also has that name. What should the developer do?

**Answer:** Rename the imported alias or use the imported class's fully qualified name at the use site.

**Explanation:** A local symbol and an import should not compete for the same short name. A clear alias makes the intended class visible.

### 29. Inside namespace `App`, how can code explicitly refer to `App\Support\Formatter` without depending on an import for `Support`?

**Answer:** Use `namespace\Support\Formatter`.

**Explanation:** The `namespace\` prefix explicitly starts resolution in the current namespace.

### 30. A project maps `App\` to the `src/` directory using an autoloader. Is that mapping built into the namespace declaration?

**Answer:** No.

**Explanation:** The autoloader implements the mapping. A namespace declaration only names symbols.

## Code-reading questions

For questions 31–35, assume these declarations appear in one file:

~~~php
namespace Demo\Billing {
    class Invoice {}
}

namespace Demo\Checkout {
    use Demo\Billing\Invoice as Bill;

    function strlen(string $value): int
    {
        return \strlen($value) + 10;
    }
}
~~~

### 31. What class does `new Bill()` refer to inside `Demo\Checkout`?

**Answer:** `Demo\Billing\Invoice`.

**Explanation:** The `Bill` alias maps to the imported class name.

### 32. What does `Bill::class` evaluate to?

**Answer:** The fully qualified class-name string `Demo\Billing\Invoice`.

**Explanation:** The `::class` expression resolves the imported alias at compile time and returns the class name as a string.

### 33. If the code calls `strlen('PHP')` inside `Demo\Checkout`, which function is selected?

**Answer:** `Demo\Checkout\strlen()`.

**Explanation:** The local namespaced function exists, so it is selected before the global fallback.

### 34. What does `\strlen('PHP')` call from that namespace?

**Answer:** The global PHP `strlen()` function.

**Explanation:** The leading backslash makes the function name fully qualified.

### 35. What is the namespace string returned by `__NAMESPACE__` inside `Demo\Checkout`?

**Answer:** `Demo\Checkout`.

**Explanation:** `__NAMESPACE__` contains the namespace in which the code is declared.

## Deeper understanding

### 36. Why is a namespace not the same thing as a directory?

**Answer:** A namespace is part of a symbol's name; a directory is a filesystem location.

**Explanation:** An autoloader can connect namespace prefixes to directory paths, but PHP does not require the names to mirror folders.

### 37. What is the difference between `use Vendor\Tool as Tool` and `new $className` when `$className` is the string `'Tool'`?

**Answer:** The import affects the statically written `new Tool()` name; it does not rewrite the runtime string `'Tool'`.

**Explanation:** For dynamic construction, place the full class name in the variable, for example `Tool::class`.

### 38. Does a fully qualified name change when an alias with the same first segment is imported?

**Answer:** No.

**Explanation:** A fully qualified name starts with `\` and is absolute; imports only affect unqualified and qualified names.

### 39. What does a global function fallback mean for an unqualified function call inside a namespace?

**Answer:** PHP tries the namespaced function first, then tries the global function if no namespaced function is available.

**Explanation:** This behavior allows built-in functions to be called naturally, while still permitting a namespace to define a same-named function.

### 40. What should be checked when an imported class name still fails at runtime?

**Answer:** Check that the class is declared and that the file is loaded by an include or a working autoloader.

**Explanation:** `use` only creates an alias. It does not create the class, load its file, or configure autoloading.

## Key takeaways

- A namespace gives symbols a qualified name; it is not a directory or autoloader.
- `use`, `use function`, and `use const` create aliases scoped to a file and namespace declaration.
- Class-like names do not fall back to the global namespace; unqualified functions and constants can.
- Fully qualified names begin with `\`; `namespace\Name` explicitly starts in the current namespace.
- Dynamic name strings need the full name they will use at runtime.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [079 - Enumerations and backed enums](../lessons/079-enumerations-and-backed-enums.md) | [Lesson](../lessons/080-namespaces-imports-and-name-resolution.md) | [Example](../examples/080-namespaces-imports-and-name-resolution.md) | [081 - Autoloading classes and files](../lessons/081-autoloading-classes-and-files.md) |
