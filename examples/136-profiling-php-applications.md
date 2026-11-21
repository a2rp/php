# Profiling PHP applications — example

This example profiles a small, repeatable PHP workload from the command line, then shows how to collect a function-level profile with Xdebug in a controlled environment.

The CLI benchmark is a teaching example. It is not a substitute for profiling a representative endpoint or measuring production behavior.

## 1. Measure repeated work with hrtime()

Save the following as profiling-demo.php and run it with PHP 7.3 or newer. It generates a predictable in-memory dataset, warms up the operation, takes multiple elapsed-time samples with the monotonic hrtime() clock, then reports the median and a sample p95.

~~~php
<?php
declare(strict_types=1);

function normalizeNames(array $names): array
{
    $normalized = [];

    foreach ($names as $name) {
        $normalized[] = strtolower(trim($name));
    }

    return $normalized;
}

$names = [];

for ($index = 0; $index < 1000; $index++) {
    $names[] = '  USER-' . $index . '  ';
}

$warmupRuns = 3;
$sampleCount = 21;
$checksum = 0;

for ($run = 0; $run < $warmupRuns; $run++) {
    $warmupResult = normalizeNames($names);
    $checksum += count($warmupResult);
}

$elapsedMilliseconds = [];

for ($run = 0; $run < $sampleCount; $run++) {
    $startedAt = hrtime(true);
    $result = normalizeNames($names);
    $finishedAt = hrtime(true);

    $elapsedMilliseconds[] = ($finishedAt - $startedAt) / 1000000;
    $checksum += count($result);
}

sort($elapsedMilliseconds, SORT_NUMERIC);

$middleIndex = intdiv(count($elapsedMilliseconds), 2);
$medianMilliseconds = $elapsedMilliseconds[$middleIndex];

$p95Index = (int) ceil(count($elapsedMilliseconds) * 0.95) - 1;
$p95Milliseconds = $elapsedMilliseconds[$p95Index];

$peakPhpMiB = memory_get_peak_usage(true) / 1024 / 1024;

$report = [
    'dataset_records' => count($names),
    'warmup_runs' => $warmupRuns,
    'measured_samples' => $sampleCount,
    'median_ms' => round($medianMilliseconds, 3),
    'sample_p95_ms' => round($p95Milliseconds, 3),
    'peak_php_memory_mib' => round($peakPhpMiB, 2),
    'checksum' => $checksum,
];

echo json_encode(
    $report,
    JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
), PHP_EOL;
~~~

### Example output

Timing and memory values vary by machine, PHP build, extensions, and current system load. A run may produce output like this:

~~~json
{
  "dataset_records": 1000,
  "warmup_runs": 3,
  "measured_samples": 21,
  "median_ms": 0.184,
  "sample_p95_ms": 0.231,
  "peak_php_memory_mib": 2,
  "checksum": 24000
}
~~~

### Understand the measurements

- The timer measures elapsed wall-clock time for each call to normalizeNames().
- The median summarizes the middle of these 21 samples. The sample p95 is only a rough illustration at this small sample count; do not treat it as a reliable production tail-latency estimate.
- The checksum consumes a result from every run and helps confirm that the measured operation produced output.
- peak_php_memory_mib is the peak PHP-managed memory for the script, including setup and warm-up. It is not isolated memory for one call and is not total process or PHP-FPM pool memory.
- This benchmark excludes network and database work. It does not represent the latency of a real web request.

For an application change, use representative data and compare the same workload before and after. Preserve the output and correctness checks, collect enough samples for the question you are answering, and measure on the target deployment class.

## 2. Capture a function profile with Xdebug

Use a local or access-controlled staging environment. Configure Xdebug 3 with profiling mode and trigger-based startup:

~~~ini
xdebug.mode = profile
xdebug.start_with_request = trigger
xdebug.output_dir = /var/tmp/php-profiles
xdebug.profiler_output_name = cachegrind.out.%p.%r
~~~

Create the output directory before profiling and make it writable by the PHP process. Keep it outside the public document root and restrict who can read it. Profiler files can be large and should be removed after the investigation.

With trigger mode, Xdebug starts when the XDEBUG_TRIGGER environment value, request parameter, or cookie is present. Trigger only the request you intend to profile. Do not make an unrestricted public trigger available on a production service; an attacker could create profiling overhead and fill the output disk.

Open the generated Cachegrind-compatible file in a compatible profile viewer. Follow the call graph from the request entry point and inspect:

- calls per function;
- self cost versus inclusive cost;
- repeated database and service calls;
- functions responsible for large allocations, where reported;
- whether the cost matches the slow operation you observed.

A function with high inclusive cost may be spending most of its time in a child call. Follow the call graph before rewriting the parent. If the request mostly waits on an external service, use dependency timing and monitoring to investigate that wait.

## 3. Compare a change

Use this sequence when testing an optimization:

1. Save the current implementation and record a baseline under a fixed environment and input.
2. Record the response or output, elapsed-time samples, memory use, query count, and relevant dependency timings.
3. Make one focused change based on the profile.
4. Repeat the same workload with the same setup and cache conditions.
5. Compare medians and relevant tail measurements, plus resource use and error rate.
6. Confirm the result remains correct and secure before keeping the change.
7. Check production monitoring after deployment to see whether real user behavior improved.

Do not compare one warm-cache run with one cold-cache run, or different input sizes and PHP configurations. Those differences can dominate the change being measured.

## Safety notes

- Xdebug profiling adds overhead; use profiler data to locate work, not as ordinary production latency.
- Keep profiler triggers restricted and temporary.
- Keep profile files outside the public web root, access-controlled, and cleaned up.
- Do not copy real credentials or unnecessary personal data into a profiling dataset.
- Do not expose profiler output through a public download route.
- Use process or container monitoring when you need total worker memory.
- Keep production monitoring enabled so you can validate improvements after release.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [135 - Safe error handling and private information](../lessons/135-safe-error-handling-and-private-information.md) | [Lesson](../lessons/136-profiling-php-applications.md) | [Q&A](../qnas/136-profiling-php-applications.md) | [137 - OPcache and PHP-FPM](../lessons/137-opcache-and-php-fpm.md) |