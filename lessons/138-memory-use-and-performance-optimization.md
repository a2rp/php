# Memory use and performance optimization

A PHP script can use too much memory when it loads an entire file or result set, keeps unnecessary values alive, or creates repeated intermediate arrays. A slow script can also waste resources even when its memory use is small.

The safest approach is to measure a representative workload, identify the dominant cost, and reduce unnecessary work or retained data without changing correct behavior.

## Learning objectives

After this lesson, you should be able to:

- interpret PHP memory measurements and the scope of memory_limit;
- identify common sources of high memory use;
- process large datasets incrementally with streams, pagination, or generators;
- choose data structures and algorithms with their time and memory trade-offs in mind;
- use caching and worker limits without hiding resource problems;
- measure and verify an optimization instead of relying on guesses.

## Understand memory measurements

PHP provides several memory-related tools:

- memory_get_usage() reports memory currently tracked by PHP's memory manager for the script.
- memory_get_peak_usage() reports the peak amount tracked for the script.
- Passing true asks for the amount reserved from the system by PHP's memory manager, which is different from the memory currently allocated to PHP values.
- Process and container metrics show a wider view that can include native extensions, runtime overhead, and other processes.

These numbers answer different questions. A PHP script can have a modest allocator peak while its worker process or container uses more memory. Conversely, memory reserved by PHP may remain available for later allocations even after some values are no longer referenced.

memory_limit is a per-script guardrail configured by PHP. It is not a total server or PHP-FPM pool memory budget. Keep headroom for concurrent workers, the web server, extensions, and the operating system. Do not set memory_limit to unlimited just to hide an allocation problem.

## Find what keeps memory alive

Common causes of high memory use include:

- reading a complete file or request body into one string;
- fetching all database rows into an array when only a few fields are needed;
- creating multiple transformed copies of a large dataset;
- storing all results before writing or sending them;
- retaining references in long-lived objects, static properties, or caches;
- using a data structure whose entries have much more overhead than their raw values;
- processing an unbounded input without size or record limits.

PHP arrays are flexible maps and can have substantial per-entry overhead compared with packed binary or numeric data. Use a structure that fits the actual access pattern. Do not replace a readable array with a complicated representation without measuring the benefit and considering maintenance.

PHP uses copy-on-write for many array assignments: a simple assignment can initially share underlying values, while a later modification can cause a copy. Avoid assuming every assignment immediately duplicates all data, but watch for transformations that actually create new large arrays.

## Stream instead of loading everything

For a large file, process one record at a time instead of calling file_get_contents() and then splitting the entire contents into more arrays.

A generator can produce records as the caller iterates over them:

~~~php
function readLines(string $path): Generator
{
    $handle = fopen($path, 'rb');

    if ($handle === false) {
        throw new RuntimeException('Unable to open the input file');
    }

    try {
        while (($line = fgets($handle)) !== false) {
            yield $line;
        }
    } finally {
        fclose($handle);
    }
}
~~~

The caller can handle each line without retaining all previous lines:

~~~php
foreach (readLines($path) as $line) {
    processLine($line);
}
~~~

Generators are forward-only iterators. They avoid building a complete result array ahead of time, but the memory savings disappear if the caller collects every yielded value into another array. A single extremely large line or record can still use substantial memory, so limit input size and field size where data is untrusted.

For database work, request only required columns and use pagination or an unbuffered/streaming cursor supported by the driver. Some database drivers buffer results even when application code iterates row by row; verify the driver's behavior.

## Bound the workload

Use explicit limits for externally controlled and potentially large work:

- cap uploaded file and request sizes;
- validate the number and size of records;
- paginate large result sets;
- limit batch size and concurrent jobs;
- use timeouts for database and network operations;
- reject or queue work that exceeds the service's capacity.

A high memory_limit does not make unbounded input safe. The application should define the maximum amount of work it will accept and how it will report a limit being exceeded.

## Improve algorithms before micro-optimizing

An algorithm's growth rate can matter more than small language-level changes:

- a single pass through n items is commonly O(n);
- comparing every item with every other item may be O(n²);
- repeated searches through a list may be replaced by a lookup structure if the extra memory is justified.

For example, nested loops that compare two large lists can become extremely slow as both lists grow. Building a keyed lookup may improve lookup time, but the map itself consumes memory. Measure with representative input sizes and choose the trade-off that fits the workload.

Avoid repeated parsing, formatting, serialization, and database queries when the same work can be safely reused. Be careful with caching: a cache consumes memory or external storage and introduces invalidation, staleness, and privacy concerns.

## Release values and understand PHP memory behavior

When a value is no longer needed, let it go out of scope or remove references to it. unset() can release a reference, but it does not guarantee that the operating system immediately reclaims the memory page. PHP's allocator may keep reserved memory available for later allocations in the same process.

PHP normally releases request-scoped variables at the end of a request. Cyclic references are handled by PHP's cycle collector. Calling gc_collect_cycles() repeatedly is not a general performance fix; use it only when measurements show that unreachable cycles are a meaningful part of a long-running process's memory behavior.

Long-running CLI workers and daemons need extra care: unlike ordinary request lifetimes, they may keep application objects and caches alive across many jobs. Bound queue batch sizes, clear job-specific references, monitor worker memory, and recycle workers through an explicit operational policy if needed.

## Measure the right bottleneck

Memory and latency are related but separate dimensions. A change that lowers memory may increase CPU time; a cache may improve latency while increasing memory; a larger batch may improve throughput while causing a higher peak.

Use profiling and monitoring to identify whether the problem is:

- CPU work such as repeated transformations or inefficient algorithms;
- memory pressure from retained or duplicated data;
- database or network waiting;
- contention caused by concurrency;
- garbage collection or worker lifecycle behavior.

Change one cause at a time. Compare the same workload before and after, and check response correctness, memory peak, latency distribution, error rate, and downstream load.

## Optimization checklist

Before keeping a change, confirm that:

- the workload and baseline are representative;
- the optimization addresses a measured bottleneck;
- input and batch sizes are bounded;
- large files and result sets are streamed or paginated where appropriate;
- extra arrays, object graphs, and caches are accounted for;
- PHP memory metrics are not mistaken for total process or container memory;
- concurrent PHP-FPM workers fit within the service memory budget;
- correctness, validation, and security behavior remain intact;
- the new version was measured under the same conditions as the baseline.

## Key takeaways

- Measure PHP allocator memory and total process/container memory as separate signals.
- memory_limit is per script, not a server-wide memory budget.
- Stream, paginate, or yield large datasets instead of retaining every item at once.
- A generator saves memory only if downstream code does not collect all its values.
- Choose algorithms and caches using both time and memory costs.
- unset() and garbage collection do not guarantee that process memory immediately returns to the operating system.
- Keep workloads bounded, optimize measured bottlenecks, and verify correctness after changes.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [137 - OPcache and PHP-FPM](137-opcache-and-php-fpm.md) | [Q&A](../qnas/138-memory-use-and-performance-optimization.md) | [Example](../examples/138-memory-use-and-performance-optimization.md) | [139 - Fibers and cooperative concurrency](139-fibers-and-cooperative-concurrency.md) |