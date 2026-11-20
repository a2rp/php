# 061 - Debugging, Diagnostics, and Application Logging: Example

## Question

How can you debug a calculation locally, record structured application events, and show users a safe failure message without exposing internal exception details?

## Complete example

This standalone example works with PHP 8.0 and later. Save it as `application-logging.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

error_reporting(E_ALL);

$requestId = bin2hex(random_bytes(8));
$logRecords = [];

// This demo logger stores JSON records in memory so the output is repeatable.
// Replace the array append with error_log($json) or the application's logger
// when connecting the example to a real application.
$logger = static function (
    string $level,
    string $event,
    array $context = []
) use (&$logRecords): void {
    $record = [
        'time' => gmdate(DATE_ATOM),
        'level' => $level,
        'event' => $event,
        'context' => $context,
    ];

    $json = json_encode(
        $record,
        JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
    );

    $logRecords[] = $json;
};

function calculateDiscount(mixed $subtotal, mixed $rate): float
{
    if (
        (!is_int($subtotal) && !is_float($subtotal))
        || $subtotal < 0
    ) {
        throw new InvalidArgumentException(
            'Subtotal must be a non-negative number.'
        );
    }

    if (
        (!is_int($rate) && !is_float($rate))
        || $rate < 0
        || $rate > 1
    ) {
        throw new InvalidArgumentException(
            'Discount rate must be between zero and one.'
        );
    }

    return round($subtotal * $rate, 2);
}

// In a local investigation, inspect only the values needed to reproduce
// the issue. Remove temporary dumps before deploying.
$subtotal = 120.00;
$rate = 0.15;
// var_dump($subtotal, $rate);

$logger('info', 'discount.calculation.started', [
    'request_id' => $requestId,
]);

try {
    $discount = calculateDiscount($subtotal, $rate);

    $logger('info', 'discount.calculation.completed', [
        'request_id' => $requestId,
        'discount' => $discount,
    ]);

    echo 'Calculated discount: '
        . number_format($discount, 2, '.', '')
        . PHP_EOL;

    // This invalid value is caught at the operation boundary. The log keeps
    // the exception type but omits its message and any potentially sensitive
    // data.
    try {
        calculateDiscount($subtotal, 1.5);
    } catch (InvalidArgumentException $exception) {
        $logger('warning', 'discount.input.rejected', [
            'request_id' => $requestId,
            'exception' => $exception::class,
        ]);

        echo 'Rejected invalid discount rate safely.' . PHP_EOL;
    }
} catch (Throwable $exception) {
    // A final application boundary records internal diagnostics but avoids
    // returning the exception message or trace to the user.
    $logger('error', 'discount.calculation.failed', [
        'request_id' => $requestId,
        'exception' => $exception::class,
        'file' => $exception->getFile(),
        'line' => $exception->getLine(),
    ]);

    echo 'The calculation could not be completed. Reference: '
        . $requestId
        . PHP_EOL;
}

echo 'Log records collected: ' . count($logRecords) . PHP_EOL;

$firstRecord = json_decode(
    $logRecords[0],
    true,
    512,
    JSON_THROW_ON_ERROR
);

echo 'First log event: ' . $firstRecord['event'] . PHP_EOL;
~~~

## Expected output

~~~text
Calculated discount: 18.00
Rejected invalid discount rate safely.
Log records collected: 3
First log event: discount.calculation.started
~~~

The JSON records contain a generated UTC timestamp and request ID, so those values are intentionally not printed in this deterministic output.

## Explanation

### 1. Reproduce and inspect locally

The values are assigned before the calculation so they can be inspected during a local investigation. Uncomment the `var_dump()` line when debugging in a controlled CLI run, then remove it when finished.

For syntax-only checking, run:

~~~sh
php -l application-logging.php
~~~

This checks syntax without executing the example. If the application behaves differently on a server, compare that runtime's PHP version, SAPI, loaded configuration, and extensions too.

### 2. Structured records are easier to search

The logger stores named fields for time, level, event, and context. The context contains a request ID and only the data needed to understand the operation.

In a real application, replace the in-memory sink:

~~~php
$logRecords[] = $json;
~~~

with the configured PHP logging destination:

~~~php
error_log($json);
~~~

The destination comes from the PHP and hosting configuration. Keep production logs protected and use the application's shared logging layer when one exists.

### 3. Invalid input is handled explicitly

`calculateDiscount()` checks its inputs with ordinary conditions and throws `InvalidArgumentException` when a value is outside the allowed range. These checks always run and do not depend on assertion settings or debug output.

The expected invalid-rate error is caught close to the operation. The log records the event, request ID, and exception class; the public output gives a safe explanation without revealing implementation details.

### 4. Unexpected failures are recorded at a boundary

The outer `catch (Throwable)` block illustrates a final boundary. It records a request ID, exception class, source file, and line. It avoids returning the exception message or stack trace to the user.

In a web application, use the framework's response mechanism instead of writing directly with `echo`. Depending on policy, file and line details may also be considered too sensitive for logs accessible beyond the operations team.

### 5. Keep log context safe

The example does not record the whole input array or exception message. In real systems, avoid logging passwords, tokens, session IDs, raw authorization headers, payment information, or entire request bodies. Validate and sanitize user-controlled values before including them in log context.

One request ID is generated at the start and reused so related log records can be found together. A client-provided reference should be checked for acceptable length and characters before it is trusted.

## Additional debugging snippets

### Inspect a value's type and contents

~~~php
var_dump($value);
~~~

Use only in local or otherwise controlled debugging. Do not send this output to ordinary visitors.

### Get a short backtrace without function arguments

~~~php
$trace = debug_backtrace(DEBUG_BACKTRACE_IGNORE_ARGS, 8);
~~~

Function arguments can contain secrets. Keep traces in protected diagnostics and avoid exposing them in responses.

### Inspect the most recent diagnostic

~~~php
$lastError = error_get_last();

if ($lastError !== null) {
    error_log(sprintf(
        'Diagnostic type=%d file=%s line=%d',
        $lastError['type'],
        $lastError['file'],
        $lastError['line']
    ));
}
~~~

This reports only the most recent PHP diagnostic. It is not a complete event history or a way to resume a terminated script.

## Try it yourself

1. Change the valid rate to `0.25` and confirm the result is `30.00`.
2. Pass a string as the subtotal and observe the validation path.
3. Add a safe internal order ID to the context without adding a customer email or payment data.
4. Replace the in-memory sink with `error_log($json)` in a local environment and inspect the configured log destination.
5. Add a second event for the rejected input and correlate both entries with the same request ID.

## Key takeaways

- Reproduce a failure and inspect only the values needed to understand it.
- Structured records make application events easier to search and correlate.
- Use one request ID to connect events from the same operation.
- Catch expected exceptions near the operation and unexpected failures at a final boundary.
- Keep secrets, raw request data, and detailed traces out of public responses and unnecessary log context.
- Use `error_log()` or an established logging service with a protected destination in real applications.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [060 - Error reporting, handlers, and assertions](../lessons/060-error-reporting-handlers-and-assertions.md) | [Q&A](../qnas/061-debugging-diagnostics-and-application-logging.md) | [Lesson](../lessons/061-debugging-diagnostics-and-application-logging.md) | [062 - Classes, objects, and object-oriented PHP](../lessons/062-classes-objects-and-object-oriented-php.md) |
