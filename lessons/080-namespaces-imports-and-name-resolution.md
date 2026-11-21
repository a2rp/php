# 080 - Namespaces, imports, and name resolution

Namespaces organize PHP classes, interfaces, traits, enums, functions, and constants under named scopes. Imports create shorter aliases for those names. Understanding how PHP resolves each kind of name helps prevent collisions and the common mistake of expecting a namespace import to load a file.

## Learning goals

By the end of this chapter, you will be able to:

- declare a namespace correctly;
- distinguish unqualified, qualified, fully qualified, and relative names;
- import classes, functions, constants, and namespace prefixes;
- explain the different fallback rules for classes, functions, and constants;
- use `namespace\` and `__NAMESPACE__`;
- understand why a `use` statement does not autoload or affect dynamic name strings.

## What a namespace does

A namespace gives a name to a set of related PHP symbols. It helps two libraries use the same short class name without colliding:

~~~php
namespace Storefront\Billing;

class Invoice {}
~~~

That class has the fully qualified name `Storefront\Billing\Invoice`. Another namespace can define its own `Invoice` class:

~~~php
namespace Storefront\Shipping;

class Invoice {}
~~~

Namespaces affect classes (including interfaces, traits, and enums), functions, and constants. They do not create directories, load files, or change variable names. A project may choose to map namespaces to directories through an autoloader, but that mapping is a separate convention.

## Declare a namespace at the top of a file

A file with a namespace declaration must put it before other PHP code. The only statement allowed before it is `declare`, commonly used for strict typing:

~~~php
<?php
declare(strict_types=1);

namespace Storefront\Billing;

class Invoice {}
~~~

Do not place output, executable statements, or HTML before the declaration. Whitespace before the opening PHP tag can also violate this rule.

For ordinary project code, use one unbracketed namespace per file:

~~~php
namespace Storefront\Billing;

class Invoice {}
~~~

PHP also supports bracketed declarations such as `namespace Storefront\Billing { ... }`. Bracketed syntax is useful for demonstrations or when one file truly needs multiple namespaces. Do not mix bracketed and unbracketed namespace declarations in one file. To combine named namespaces with global code, use bracketed declarations and wrap global code in `namespace { ... }`.

## Four ways to write a name

### 1. Unqualified name

An unqualified name has no namespace separator, such as `Invoice`:

~~~php
namespace Storefront\Billing;

new Invoice();
~~~

For a class-like name, PHP looks for `Storefront\Billing\Invoice`, unless an import alias changes the name.

### 2. Qualified name

A qualified name has a namespace separator but no leading backslash, such as `Models\Invoice`:

~~~php
namespace Storefront\Billing;

new Models\Invoice();
~~~

Without an import that matches its first segment, PHP prefixes the current namespace, producing `Storefront\Billing\Models\Invoice`. If the first segment is an imported alias, PHP expands that alias instead.

### 3. Fully qualified name

A fully qualified name begins with a backslash and is absolute from the global namespace:

~~~php
new \DateTimeImmutable();
new \Vendor\Package\Invoice();
~~~

Imports do not change fully qualified names. The leading backslash is optional in some import declarations, but use it in code when you want a direct, unambiguous global name without an import.

### 4. Relative name

The `namespace\` prefix explicitly starts a name in the current namespace:

~~~php
namespace Storefront\Billing;

new namespace\Models\Invoice();
~~~

This resolves to `Storefront\Billing\Models\Invoice`, even if an imported alias has a similar first segment.

## Import names with `use`

An import gives a fully qualified name a shorter local alias. Imports are scoped to the current file and namespace declaration:

~~~php
namespace Storefront\Checkout;

use Storefront\Billing\Invoice;
use Storefront\Catalog\Product as CatalogProduct;

$invoice = new Invoice();
$product = new CatalogProduct();
~~~

If the imported name and a local name would collide, give the import a clear alias with `as`. Importing a namespace prefix is also possible:

~~~php
use Storefront\Catalog;

$product = new Catalog\Product();
~~~

### Import functions and constants separately

Class-like imports, function imports, and constant imports use separate import tables:

~~~php
namespace Storefront\Checkout;

use function Storefront\Support\normalizeCode;
use const Storefront\Support\DEFAULT_CURRENCY;

$code = normalizeCode(' INV-42 ');
echo DEFAULT_CURRENCY;
~~~

Use `use function` for a function and `use const` for a constant. A plain `use` imports a class-like name or namespace prefix.

### Group imports

Names from the same namespace can be grouped:

~~~php
use Storefront\Catalog\{Category, Product as CatalogProduct};
use function Storefront\Support\{normalizeCode, slugify};
use const Storefront\Support\{DEFAULT_CURRENCY, TAX_RATE};
~~~

Grouping is only shorthand; it does not change how the imported names resolve.

### Imports do not load files

`use` is a compile-time alias declaration. It does not include or require a file and does not register an autoloader. The class must still be declared by an included file or found by an autoloader when PHP uses it.

Import rules are per file. Including another PHP file does not make the current file's imports available inside the included file, or vice versa.

## Name resolution differs by symbol type

Name resolution is one of the most important namespace rules.

### Classes, interfaces, traits, and enums

Inside `Storefront\Checkout`, an unqualified class-like name `Invoice` resolves to `Storefront\Checkout\Invoice` unless an import applies. PHP does not fall back to a global `Invoice` class if the namespaced class is missing.

To use a global or other external class, import it or write its fully qualified name:

~~~php
namespace Storefront\Checkout;

use DateTimeImmutable;

$now = new DateTimeImmutable();
$alsoNow = new \DateTimeImmutable();
~~~

### Functions and constants

An unqualified function or constant name inside a namespace first checks that namespace, then falls back to the global function or constant if no namespaced version exists:

~~~php
namespace Storefront\Text;

function strlen(string $value): int
{
    return \strlen($value) + 1;
}

$localCount = strlen('PHP'); // Storefront\Text\strlen()
$globalCount = \strlen('PHP'); // global strlen()
~~~

The leading backslash makes the global function choice explicit. Constants follow the same namespaced-then-global fallback rule.

A qualified name such as `Helpers\format()` is resolved relative to the current namespace or an imported prefix. It does not use the unqualified global fallback rule.

## The global namespace

Code without a namespace declaration belongs to the global namespace. Within a namespace, start a global class, function, or constant name with `\`:

~~~php
namespace Storefront\Checkout;

$length = \strlen('PHP');
$clock = new \DateTimeImmutable();
~~~

When a file deliberately declares several namespaces with bracketed syntax, global code can be placed in an empty namespace block:

~~~php
namespace {
    echo \strlen('PHP');
}
~~~

Most application files do not need multiple namespace declarations. Keeping one namespace per file is easier to navigate.

## `__NAMESPACE__` and the `namespace` keyword

`__NAMESPACE__` is a magic constant containing the current namespace name as a string. It is an empty string in global code:

~~~php
namespace Storefront\Billing;

echo __NAMESPACE__; // Storefront\Billing
~~~

The `namespace` keyword is used as a relative-name prefix, as in `namespace\Invoice`. It is not a string and cannot be concatenated like `__NAMESPACE__`.

## Dynamic names are strings

Imports affect statically written names at compile time. They do not rewrite a class name stored in a variable:

~~~php
namespace Storefront\Checkout;

use Storefront\Billing\Invoice;

$className = Invoice::class; // Fully qualified name: Storefront\Billing\Invoice
$invoice = new $className();
~~~

Using `new $className()` works because the variable contains the fully qualified class name. A string containing only `'Invoice'` is not automatically expanded through the import. For dynamic functions and constants, provide the appropriate full name as well.

Prefer `SomeClass::class` over manually typing a fully qualified name when building a class-name string. It avoids spelling mistakes and follows a future class rename.

## Namespaces and autoloading

A namespace is a symbol name, not a filesystem path. PHP can load a class through an autoloader after resolving its fully qualified name. Composer's PSR-4 autoloading convention often maps a namespace prefix to a directory, but the import itself does not perform that mapping or load the file.

Keep the responsibilities distinct:

- namespace declarations name symbols;
- imports shorten names in source files;
- autoloaders locate and load class files.

Autoloading is covered in a later chapter.

## Common mistakes

- **Putting executable code before `namespace`:** move the declaration to the top, after an optional `declare` statement.
- **Assuming a namespace creates a directory or loads a file:** file layout and autoloading are separate concerns.
- **Expecting a missing namespaced class to fall back to a global class:** import it or use a fully qualified name.
- **Expecting a class import to apply to a function or constant:** use `use function` or `use const`.
- **Expecting `use` to include the class file:** configure an autoloader or explicitly include the file.
- **Expecting imports to cross file boundaries:** imports apply only to the file and namespace scope where they are declared.
- **Assuming imports rewrite dynamic strings:** store a fully qualified name, such as the result of `SomeClass::class`.
- **Mixing bracketed and unbracketed namespace declarations:** choose one style for the file.
- **Using a relative qualified name when a global name is intended:** start it with `\` or import the symbol.
- **Confusing `namespace\Name` with `__NAMESPACE__ . '\\Name'`:** the first is syntax for a relative name; the second is string construction.

## Key takeaways

- Namespaces organize class-like symbols, functions, and constants.
- Declare a namespace at the top of its file; one namespace per file is a practical convention.
- Unqualified, qualified, fully qualified, and `namespace\` relative names resolve differently.
- `use`, `use function`, and `use const` create file-scoped aliases; they do not load files.
- Namespaced classes do not fall back to global classes, while unqualified functions and constants can.
- Dynamic names are strings and must contain the full name needed at runtime.
- Autoloading maps resolved class names to files; it is separate from namespaces and imports.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [079 - Enumerations and backed enums](079-enumerations-and-backed-enums.md) | [Q&A](../qnas/080-namespaces-imports-and-name-resolution.md) | [Example](../examples/080-namespaces-imports-and-name-resolution.md) | [081 - Autoloading classes and files](081-autoloading-classes-and-files.md) |
