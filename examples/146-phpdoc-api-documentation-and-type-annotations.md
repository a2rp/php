# PHPDoc, API documentation, and type annotations — Example

## Question

How can a function safely convert untrusted customer data into typed objects, document the collection it returns, and make the contract clear to people and static-analysis tools?

## Complete example

Save this as **customer-docs.php** and run it with PHP 8.0 or later.

~~~php
<?php
declare(strict_types=1);

final class Customer
{
    public function __construct(
        public int $id,
        public string $name,
    ) {
    }
}

/**
 * Convert decoded customer data into Customer objects.
 *
 * The input is checked at runtime because it may come from an external source.
 * The return annotation tells compatible static-analysis tools that the result
 * is a sequential list containing only Customer objects.
 *
 * @return list<Customer>
 *
 * @throws InvalidArgumentException If the input or any customer row is invalid.
 */
function customersFrom(mixed $input): array
{
    if (!is_array($input)) {
        throw new InvalidArgumentException('Customer data must be an array.');
    }

    $customers = [];

    foreach ($input as $index => $row) {
        if (!is_array($row)) {
            throw new InvalidArgumentException(
                sprintf('Customer row %s must be an array.', (string) $index)
            );
        }

        if (!array_key_exists('id', $row) || !is_int($row['id'])) {
            throw new InvalidArgumentException(
                sprintf('Customer row %s must have an integer id.', (string) $index)
            );
        }

        if (!array_key_exists('name', $row) || !is_string($row['name'])) {
            throw new InvalidArgumentException(
                sprintf('Customer row %s must have a string name.', (string) $index)
            );
        }

        $name = trim($row['name']);

        if ($name === '') {
            throw new InvalidArgumentException(
                sprintf('Customer row %s must have a non-empty name.', (string) $index)
            );
        }

        $customers[] = new Customer($row['id'], $name);
    }

    return $customers;
}

$json = <<<'JSON'
[
    {"id": 101, "name": "Asha"},
    {"id": 102, "name": "Ravi"}
]
JSON;

try {
    $decoded = json_decode($json, true, 512, JSON_THROW_ON_ERROR);
    $customers = customersFrom($decoded);

    foreach ($customers as $customer) {
        printf("Customer #%d: %s\n", $customer->id, $customer->name);
    }
} catch (JsonException | InvalidArgumentException $exception) {
    echo 'Could not load customers: ' . $exception->getMessage() . "\n";
}
~~~

## Expected output

~~~text
Customer #101: Asha
Customer #102: Ravi
~~~

## Explanation

### 1. Native types describe what PHP can enforce

The constructor requires an integer ID and a string name. The function accepts **mixed** because JSON decoding and other external sources can produce values with different types. PHP enforces these native declarations when the relevant code runs.

The function's native return type is **array**. PHP can enforce that the result is an array, but that declaration alone cannot say that it is a sequential list of **Customer** objects.

### 2. PHPDoc adds collection detail for compatible tools

The **@return list<Customer>** annotation tells compatible IDEs and static-analysis tools that callers should receive a sequential list whose elements are **Customer** objects. PHP does not enforce this annotation at runtime. Keep the annotation truthful: this implementation builds the result with **$customers[]**, so it returns sequential numeric keys and only appends **Customer** instances.

PHPDoc features such as **list<T>** and array shapes depend on the project's analyzer and configuration. Use syntax supported by the tools the project actually runs.

### 3. External data still needs runtime validation

A PHPDoc comment cannot validate JSON, form submissions, database rows, or other untrusted values. The function checks that:

- The decoded top-level value is an array.
- Each row is an array.
- Every row has an integer **id**.
- Every row has a string **name**.
- The name is not empty after trimming.

Only after those checks does the code construct a **Customer**. If a row is invalid, the function throws an **InvalidArgumentException** rather than returning an object that violates its contract.

### 4. The exception is part of the API contract

The **@throws** annotation documents the exception callers may need to handle. It does not make PHP throw it automatically. The implementation explicitly throws it when validation fails. The **try/catch** block also handles invalid JSON through **JsonException**.

### 5. Test the failure path by changing the input

For example, change the first row to give **id** a string value:

~~~json
{"id": "101", "name": "Asha"}
~~~

The program reports an error similar to:

~~~text
Could not load customers: Customer row 0 must have an integer id.
~~~

The PHPDoc return promise still describes only successful results. Invalid input is rejected before a **Customer** is created.

## Key takeaways

- Native declarations are runtime contracts for the types PHP supports.
- PHPDoc can add precise collection information for compatible tools.
- PHPDoc is documentation for tools and people; it is not runtime validation.
- Validate untrusted data before constructing trusted, typed objects.
- Document exceptions and behavior, and keep every documented promise consistent with the implementation.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [145 - PHP version compatibility and migration guides](../lessons/145-php-version-compatibility-and-migration-guides.md) | [Lesson](../lessons/146-phpdoc-api-documentation-and-type-annotations.md) | [Q&A](../qnas/146-phpdoc-api-documentation-and-type-annotations.md) | [147 - Code review, maintenance, and upgrades](../lessons/147-code-review-maintenance-and-upgrades.md) |
