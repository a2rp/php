# Traits and code reuse - Example

## Goal

Build CSV exporters that share the same row formatting but choose their own columns.

- <code>CsvExporter</code> defines the public contract.
- <code>CsvExportBehavior</code> implements reusable CSV formatting and requires a concrete column list.
- <code>UserCsvExporter</code> and <code>ProductCsvExporter</code> reuse the trait in unrelated classes.

This example uses PHP 8.0 or later and does not require a CSV library.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

interface CsvExporter
{
    /**
     * @param array<int, array<string, scalar|null>> $rows
     */
    public function toCsv(array $rows): string;
}

trait CsvExportBehavior
{
    /**
     * @return array<int, string>
     */
    abstract protected function columns(): array;

    /**
     * @param array<int, array<string, scalar|null>> $rows
     */
    public function toCsv(array $rows): string
    {
        $columns = $this->columns();

        if ($columns === []) {
            throw new LogicException('An exporter must define at least one column.');
        }

        $headers = array_map(
            static fn (string $column): string => ucfirst($column),
            $columns
        );

        $lines = [$this->encodeRow($headers)];

        foreach ($rows as $row) {
            if (!is_array($row)) {
                throw new InvalidArgumentException('Each row must be an array.');
            }

            $values = [];

            foreach ($columns as $column) {
                $value = $row[$column] ?? '';

                if ($value !== null && !is_scalar($value)) {
                    throw new InvalidArgumentException(
                        'CSV values must be scalar or null.'
                    );
                }

                $values[] = $value === null ? '' : (string) $value;
            }

            $lines[] = $this->encodeRow($values);
        }

        return implode(PHP_EOL, $lines) . PHP_EOL;
    }

    /**
     * @param array<int, string> $values
     */
    private function encodeRow(array $values): string
    {
        $escaped = [];

        foreach ($values as $value) {
            $escaped[] = '"' . str_replace('"', '""', $value) . '"';
        }

        return implode(',', $escaped);
    }
}

class UserCsvExporter implements CsvExporter
{
    use CsvExportBehavior;

    protected function columns(): array
    {
        return ['name', 'email'];
    }
}

class ProductCsvExporter implements CsvExporter
{
    use CsvExportBehavior;

    protected function columns(): array
    {
        return ['sku', 'name', 'price'];
    }
}

$exporter = new UserCsvExporter();

$csv = $exporter->toCsv([
    [
        'name' => 'Asha',
        'email' => 'asha@example.test',
    ],
    [
        'name' => 'Sam "S"',
        'email' => 'sam@example.test',
    ],
]);

echo $csv;
~~~

## Expected output

~~~text
"Name","Email"
"Asha","asha@example.test"
"Sam ""S""","sam@example.test"
~~~

## How the trait and interface work together

The interface declares what a CSV exporter promises:

~~~php
interface CsvExporter
{
    public function toCsv(array $rows): string;
}
~~~

The trait provides the <code>toCsv()</code> implementation and requires the using class to provide <code>columns()</code>. The class uses the trait and supplies those columns:

~~~php
class UserCsvExporter implements CsvExporter
{
    use CsvExportBehavior;

    protected function columns(): array
    {
        return ['name', 'email'];
    }
}
~~~

After composition, the class has a public <code>toCsv()</code> method from the trait, so it fulfills the interface contract. The trait reuses implementation; the interface defines the type contract.

## Reuse the same behavior for products

The same trait works with another unrelated class:

~~~php
$productExporter = new ProductCsvExporter();

echo $productExporter->toCsv([
    [
        'sku' => 'BOOK-1',
        'name' => 'PHP Guide',
        'price' => 24.5,
    ],
]);
~~~

Expected output:

~~~text
"Sku","Name","Price"
"BOOK-1","PHP Guide","24.5"
~~~

Each class selects its columns, while the trait owns the common CSV encoding. Values are quoted, and embedded double quotes are doubled according to CSV escaping rules.

## Resolve conflicting methods

When two traits provide a method with the same name, specify which implementation to use. An alias can keep access to the other implementation:

~~~php
<?php
declare(strict_types=1);

trait FriendlyStatus
{
    public function status(): string
    {
        return 'Ready';
    }
}

trait MachineStatus
{
    public function status(): string
    {
        return 'READY';
    }
}

class DashboardStatus
{
    use FriendlyStatus, MachineStatus {
        MachineStatus::status insteadof FriendlyStatus;
        FriendlyStatus::status as friendlyStatus;
    }
}

$status = new DashboardStatus();

echo $status->status() . PHP_EOL;
echo $status->friendlyStatus();
~~~

Expected output:

~~~text
READY
Ready
~~~

<code>insteadof</code> selects the implementation for <code>status()</code>. The alias makes FriendlyStatus's version available under <code>friendlyStatus()</code>.

## Change an imported method's visibility

A trait method can be made protected when it is imported:

~~~php
trait DebugDetails
{
    public function debugDetails(): string
    {
        return 'Internal details';
    }
}

class Account
{
    use DebugDetails {
        debugDetails as protected;
    }

    public function supportSummary(): string
    {
        return $this->debugDetails();
    }
}

echo (new Account())->supportSummary();
~~~

Expected output:

~~~text
Internal details
~~~

External callers cannot call <code>debugDetails()</code> directly, but methods inside Account can use it.

## Understand precedence

A method written directly in the class overrides a method imported from a trait. A trait method overrides an inherited method from the parent:

~~~php
class BasePage
{
    public function heading(): string
    {
        return 'Parent heading';
    }
}

trait PageHeading
{
    public function heading(): string
    {
        return 'Trait heading';
    }
}

class HomePage extends BasePage
{
    use PageHeading;
}

class CustomHomePage extends BasePage
{
    use PageHeading;

    public function heading(): string
    {
        return 'Class heading';
    }
}

echo (new HomePage())->heading() . PHP_EOL;
echo (new CustomHomePage())->heading();
~~~

Expected output:

~~~text
Trait heading
Class heading
~~~

## Version-specific features

The main CSV example uses methods and works with PHP 8.0+. These additional features require newer versions:

- Trait constants require PHP 8.2.
- Marking an imported trait method final with <code>as final</code> requires PHP 8.3.
- Directly accessing static methods or properties through the trait name is deprecated as of PHP 8.1. Access them through a class using the trait.
- Trait static-property behavior in inheritance differs starting in PHP 8.3.
- Trait and parent class property/constant precedence changed in PHP 8.5.

Example trait constant for PHP 8.2+:

~~~php
trait JsonDefaults
{
    public const CONTENT_TYPE = 'application/json';
}

class JsonResponse
{
    use JsonDefaults;
}

echo JsonResponse::CONTENT_TYPE;
~~~

## Practice

1. Add an <code>OrderCsvExporter</code> with columns for order ID, customer, and total.
2. Reject a row that is missing a required value instead of using an empty cell.
3. Add a trait that provides a reusable <code>createdAt()</code> method.
4. Give two traits a method with the same name, then resolve the collision with <code>insteadof</code> and <code>as</code>.
5. Add a class method with the same name as a trait method. Which implementation runs, and why?

## Key takeaways

- A trait reuses implementation; it does not create a class or interface type.
- The abstract <code>columns()</code> method makes the trait's requirement explicit.
- The interface promises <code>toCsv()</code>; the trait supplies its shared implementation.
- Conflict adaptations let a class select, alias, or restrict imported methods.
- Traits work best when their behavior is cohesive and their dependencies are clear.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [070 - Interfaces and contracts](../lessons/070-interfaces-and-contracts.md) | [Lesson](../lessons/071-traits-and-code-reuse.md) | [Q&A](../qnas/071-traits-and-code-reuse.md) | [072 - Class constants and constant expressions](../lessons/072-class-constants-and-constant-expressions.md) |
