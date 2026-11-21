# 080 - Namespaces, imports, and name resolution: Example

## Question

How can one PHP file declare symbols in separate namespaces, import a class, a function, and a constant, distinguish namespaced and global functions, and instantiate a class from a runtime name?

## Complete PHP example

```php
<?php

declare(strict_types=1);

namespace Demo\Billing {
    const CURRENCY = 'USD';

    class Invoice
    {
        public function __construct(public string $number)
        {
        }
    }

    function normalizeNumber(string $number): string
    {
        return strtolower(trim($number));
    }
}

namespace Demo\App {
    use Demo\Billing\Invoice as Bill;
    use function Demo\Billing\normalizeNumber;
    use const Demo\Billing\CURRENCY;

    // This function belongs to Demo\App, not the global namespace.
    function strlen(string $value): int
    {
        return \strlen($value) + 100;
    }

    function describeNamespace(): string
    {
        return __NAMESPACE__;
    }

    function run(): void
    {
        $invoice = new Bill(normalizeNumber(' INV-42 '));

        echo "Invoice number: {$invoice->number}\n";
        echo 'Currency: ' . CURRENCY . "\n";

        // An unqualified function name resolves to Demo\App\strlen here.
        echo 'Namespaced strlen: ' . strlen('PHP') . "\n";

        // A leading backslash explicitly selects the global PHP function.
        echo 'Global strlen: ' . \strlen('PHP') . "\n";

        // namespace\ starts name resolution in the current namespace.
        echo 'Current namespace: ' . namespace\describeNamespace() . "\n";

        // ::class returns the resolved class name as a string.
        $className = Bill::class;
        echo 'Imported class name: ' . $className . "\n";

        // Dynamic construction uses the runtime class-name string.
        $copy = new $className(normalizeNumber(' INV-99 '));
        echo 'Dynamic class name: ' . $copy->number . "\n";
    }
}

namespace {
    // The global namespace block can call the namespaced entry point by its
    // fully qualified name.
    \Demo\App\run();
}
```

This example requires PHP 8.0 or newer because it uses constructor property promotion.

## Expected output

```text
Invoice number: inv-42
Currency: USD
Namespaced strlen: 103
Global strlen: 3
Current namespace: Demo\App
Imported class name: Demo\Billing\Invoice
Dynamic class name: inv-99
```

## Explanation

1. **Bracketed namespace declarations:** `Demo\Billing`, `Demo\App`, and the global namespace each have their own block. PHP requires one namespace style per file, so all three use braces. In a real project, keeping one namespace per file is usually clearer.
2. **Class alias:** `use Demo\Billing\Invoice as Bill;` lets this namespace refer to that class as `Bill`. It creates an alias; it does not load the class file.
3. **Function import:** `use function ...` imports `normalizeNumber`. It trims the invoice number and converts it to lowercase.
4. **Constant import:** `use const ...` makes the `CURRENCY` constant available by its short name.
5. **Function name resolution:** `strlen('PHP')` calls the function declared in `Demo\App`; that function explicitly delegates to global `\strlen()` and adds 100. Writing `\strlen()` at the call site also selects the global function directly.
6. **Namespace-relative name:** `namespace\describeNamespace()` means `Demo\App\describeNamespace()` from inside the `Demo\App` namespace.
7. **Dynamic class name:** `Bill::class` resolves the imported alias to the string `Demo\Billing\Invoice`. The `new $className(...)` expression then constructs that class. Imports do not rewrite arbitrary strings at runtime.
8. **Global namespace block:** `namespace { ... }` makes the final call from the global namespace. Its leading backslash makes the entry-point name fully qualified.

## Try these changes

- Change `CURRENCY` to another value and update the output.
- Remove the `Demo\App\strlen()` function, then observe that an unqualified `strlen()` call can fall back to PHP's global function.
- Replace `new Bill(...)` with `new \Demo\Billing\Invoice(...)` and compare the explicit fully qualified name with the alias.
- Change `$className` to `'Bill'`. It will not use the import alias as a runtime string; use `Bill::class` or the full class name instead.

## Key takeaways

- Namespaces are part of symbol names; they do not automatically correspond to folders or load files.
- Use `use`, `use function`, and `use const` to create readable aliases in the current file and namespace.
- Class-like names do not fall back to global names; unqualified function and constant names can.
- A leading `\\` makes a name fully qualified, while `namespace\\` makes it relative to the current namespace.
- Dynamic class names are resolved at runtime from their string value.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [079 - Enumerations and backed enums](../lessons/079-enumerations-and-backed-enums.md) | [Q&A](../qnas/080-namespaces-imports-and-name-resolution.md) | Example | [081 - Autoloading classes and files](../lessons/081-autoloading-classes-and-files.md) |
