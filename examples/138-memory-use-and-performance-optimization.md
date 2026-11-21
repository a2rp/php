# Memory use and performance optimization — example

## Question

How can a PHP command-line program aggregate a large CSV without loading the complete file and all of its rows into memory?

The example below reads each CSV row as it is needed, validates it, and keeps only a few counters and a running total. It also enforces a maximum input-file size so streaming does not become permission to process unbounded work.

## Sample input

Save this as orders.csv:

~~~csv
order_id,amount_cents
A-100,1250
A-101,850
A-102,not-a-number
A-103,1900
~~~

The example treats amount_cents as a non-negative integer. Using integer cents avoids floating-point rounding for a monetary total.

## Complete PHP script

Save this as summarize-orders.php. It requires PHP 8.0 or newer for nullable fgetcsv length and JSON_THROW_ON_ERROR. The explicit empty CSV escape argument also avoids relying on the legacy default escape behavior.

~~~php
<?php
declare(strict_types=1);

/**
 * Yield valid amount values as integers and malformed records as null.
 *
 * @return Generator<int, int|null>
 */
function readAmountCents(string $path): Generator
{
    $handle = fopen($path, 'rb');

    if ($handle === false) {
        throw new RuntimeException('Unable to open the CSV input');
    }

    try {
        $header = fgetcsv($handle, null, ',', '"', '');

        if ($header !== ['order_id', 'amount_cents']) {
            throw new RuntimeException('The CSV header is invalid');
        }

        while (($row = fgetcsv($handle, null, ',', '"', '')) !== false) {
            if ($row === [null]) {
                continue;
            }

            if (
                count($row) !== 2
                || !is_string($row[0])
                || $row[0] === ''
                || strlen($row[0]) > 64
                || !is_string($row[1])
                || !preg_match('/^(0|[1-9][0-9]*)$/D', $row[1])
            ) {
                yield null;
                continue;
            }

            $amountCents = filter_var($row[1], FILTER_VALIDATE_INT);

            if ($amountCents === false || $amountCents < 0) {
                yield null;
                continue;
            }

            yield $amountCents;
        }
    } finally {
        fclose($handle);
    }
}

$inputPath = __DIR__ . '/orders.csv';
$maximumInputBytes = 64 * 1024 * 1024;
$fileSize = filesize($inputPath);

if ($fileSize === false || $fileSize > $maximumInputBytes) {
    fwrite(STDERR, "Input file is missing or exceeds the configured size limit." . PHP_EOL);
    exit(1);
}

$startedAt = hrtime(true);
$validRows = 0;
$invalidRows = 0;
$totalCents = 0;

try {
    foreach (readAmountCents($inputPath) as $amountCents) {
        if ($amountCents === null) {
            $invalidRows++;
            continue;
        }

        if ($amountCents > PHP_INT_MAX - $totalCents) {
            throw new OverflowException('The aggregate amount is too large');
        }

        $totalCents += $amountCents;
        $validRows++;
    }
} catch (Throwable $exception) {
    // Keep raw file data and exception details out of the command output.
    fwrite(STDERR, "Import failed while reading the configured input." . PHP_EOL);
    exit(1);
}

$elapsedMilliseconds = (hrtime(true) - $startedAt) / 1000000;
$peakPhpMiB = memory_get_peak_usage(true) / 1024 / 1024;

$summary = [
    'valid_rows' => $validRows,
    'invalid_rows' => $invalidRows,
    'total_amount_cents' => $totalCents,
    'elapsed_ms' => round($elapsedMilliseconds, 3),
    'peak_php_memory_mib' => round($peakPhpMiB, 2),
];

echo json_encode(
    $summary,
    JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
), PHP_EOL;
~~~

## Expected output

Timing and memory values depend on the PHP build and environment. For the sample input, the stable aggregate fields are:

~~~json
{
  "valid_rows": 3,
  "invalid_rows": 1,
  "total_amount_cents": 4000,
  "elapsed_ms": 0.8,
  "peak_php_memory_mib": 2
}
~~~

The elapsed time and memory values are illustrative and will vary. The script does not keep all parsed records; its aggregation state grows with the counters, not with the number of valid rows.

## Explanation

- readAmountCents() is a generator. Each parsed row is yielded to the caller instead of being accumulated into an array.
- fgetcsv() reads one record at a time and handles quoted CSV fields. The explicit escape argument is empty; do not depend on a legacy default escape character.
- The script validates the header, row shape, identifier length, and integer amount before adding a value.
- The file-size check bounds this example's input. In an upload pipeline, enforce limits at upload time too and process a finalized file so its size cannot change after the check.
- The running total uses integer cents, checks for integer overflow, and does not retain order IDs or complete rows.
- The finally block closes the file even if parsing or processing throws.
- The catch block returns a generic CLI error rather than printing raw input or exception details.
- memory_get_peak_usage(true) reports PHP-managed memory reserved for the script, including startup and setup. It does not report total operating-system process or container memory and does not isolate one CSV row.
- A single extremely large CSV record can still consume significant memory while fgetcsv() parses it. Apply record and field-size limits appropriate to the application and reject oversized input.

## Compare with a whole-file approach

This version retains multiple representations of the input:

~~~php
$contents = file_get_contents($inputPath);
$rows = explode(PHP_EOL, $contents);
$parsedRows = array_map('str_getcsv', $rows);
~~~

For a large file, the full string, row strings, parsed arrays, and later transformations can occupy much more memory than the file size. Stream records and keep only the state needed for the result.

## Adapt it for production

- Store uploaded files outside the public web root and use generated filenames.
- Enforce file-size, row-count, and field-size limits before and during processing.
- If writing to a database, use bounded batches and transactions sized for the database and memory budget.
- Decide whether malformed rows should stop the import, be skipped, or be written to a protected error report.
- Do not log raw rows if they can contain personal or sensitive information.
- For very large jobs, process asynchronously with bounded worker concurrency and report progress safely.
- Measure throughput and memory with representative data before changing batch sizes.
- Test output correctness and failure behavior before using the importer on important data.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [137 - OPcache and PHP-FPM](../lessons/137-opcache-and-php-fpm.md) | [Lesson](../lessons/138-memory-use-and-performance-optimization.md) | [Q&A](../qnas/138-memory-use-and-performance-optimization.md) | [139 - Fibers and cooperative concurrency](../lessons/139-fibers-and-cooperative-concurrency.md) |