# Profiling PHP applications

Profiling measures where an application spends time and memory so you can improve the part that matters. It replaces guesses such as “this loop looks slow” with evidence from a representative workload.

The goal is not to make every line faster. The goal is to reduce user-visible latency, resource use, or capacity pressure without changing correct behavior.

## Learning objectives

After this lesson, you should be able to:

- distinguish profiling, benchmarking, and production monitoring;
- choose a representative request or workload to investigate;
- interpret elapsed time, call counts, memory use, and profiler cost summaries;
- use PHP timing and memory measurements carefully;
- run Xdebug profiling in a controlled environment;
- compare changes fairly and verify that an optimization preserves behavior.

## Profiling, benchmarking, and monitoring

These practices answer different questions:

- **Profiling** identifies where a particular run spends CPU time, wall-clock time, or memory. It helps locate a bottleneck.
- **Benchmarking** compares the performance of a defined operation or version under controlled, repeatable conditions.
- **Monitoring** tracks production behavior over time, such as request latency, error rate, traffic, database latency, and resource saturation.

A profiler can explain why a request is slow, but it does not by itself show how often the issue affects real users. Monitoring reveals the impact and helps you choose which route or job to profile.

Start with an observable problem: a slow endpoint, long-running job, high memory use, increased database load, or rising p95 latency. Choose a representative workload and record a baseline before changing code.

## Measure the right kind of time

A request can spend time in different places:

- **CPU time** is time spent executing instructions.
- **Wall-clock time** is the elapsed time visible to the caller. It includes waiting for a database, filesystem, network, lock, or other process.
- **Memory use** describes memory allocated by PHP or by the wider process, depending on the tool and measurement.

A request can have low CPU time and high wall time because it is waiting on an external service. Optimizing PHP instructions will not fix a slow query or a network timeout. Use traces, database metrics, and service timing to tell these cases apart.

For elapsed-time measurements inside PHP 7.3 or newer, use **hrtime()**. It provides a monotonic high-resolution clock: unlike a wall-clock timestamp, it is not adjusted when system time changes. Its numeric form is in nanoseconds, so convert the difference to milliseconds or seconds for readable output.

~~~php
$startedAt = hrtime(true);

// Run the operation being measured.

$elapsedMilliseconds = (hrtime(true) - $startedAt) / 1_000_000;
~~~

The absolute value returned by hrtime() has no useful calendar meaning. Measure a difference between two readings. For older runtimes, microtime(true) can be used for elapsed-time measurements, but remember that it is based on the system clock.

PHP's memory_get_usage() and memory_get_peak_usage() report memory tracked by PHP's memory manager. They do not necessarily account for all memory used by the operating-system process or native extensions. The real_usage parameter reports memory reserved by PHP's memory manager, not total machine or process memory.

~~~php
$currentPhpBytes = memory_get_usage(true);
$peakPhpBytes = memory_get_peak_usage(true);

$currentPhpMiB = $currentPhpBytes / 1024 / 1024;
$peakPhpMiB = $peakPhpBytes / 1024 / 1024;
~~~

Use a process monitor or container metrics when you need total process or container memory. Do not confuse one PHP request's allocator measurements with the memory of the whole PHP-FPM worker pool.

## Read profiler summaries correctly

An instrumenting profiler records function calls and their costs while the program runs. A call-graph view can show:

- **Calls**: how often a function ran;
- **self or exclusive cost**: time attributed to the function body, excluding its children;
- **inclusive cost**: time spent in the function and the functions it called;
- **call relationships**: which callers lead to expensive work;
- **memory information**: where supported, allocations and peak use.

A function with high inclusive cost may simply call an expensive child. Follow the call graph until you find the operation doing the work. A function with high self cost may contain a costly loop, repeated transformation, or inefficient algorithm.

A high call count can be as important as a slow individual call. Thousands of fast database queries may reveal an N+1 query pattern. A slow database query can dominate the response even when PHP spends little CPU time.

Profiler cost is an estimate of the observed run, not a promise that the same proportion will hold under every traffic pattern. Use the call graph to form a hypothesis, then compare the user-visible behavior before and after the change.

## Use Xdebug profiling deliberately

Xdebug can write Cachegrind-compatible profiler files that a compatible viewer can use to inspect function costs and call relationships. Profiling can add substantial overhead and produce large files. Use it for a controlled local or staging investigation, not as an always-on production setting.

A sample Xdebug 3 configuration is:

~~~ini
xdebug.mode = profile
xdebug.start_with_request = trigger
xdebug.output_dir = /var/tmp/php-profiles
xdebug.profiler_output_name = cachegrind.out.%p.%r
~~~

The directory must exist and be writable by the PHP process. Keep profiling files outside the public document root, restrict access, and delete them when they are no longer needed. Profile artifacts can reveal application structure and operational details.

With trigger mode, Xdebug starts profiling only when the configured trigger is present. Use a temporary, access-controlled trigger in a local or staging environment. Do not expose an unrestricted public URL or cookie that lets anyone generate large profiling files.

Xdebug changes runtime cost. Compare code using a consistent measurement setup, and do not treat profiler-instrumented latency as ordinary production latency. Use production monitoring or a production-suitable APM to measure real user impact.

## Profile representative work

A useful investigation follows a repeatable process:

1. **Choose the symptom.** Identify the endpoint, job, or operation and the affected metric.
2. **Record the workload.** Use representative input sizes, database shape, and dependency behavior without copying production secrets into a development environment.
3. **Establish a baseline.** Record response time, throughput where relevant, error rate, and memory under the same setup.
4. **Capture a profile.** Use a local or staging profiler and reproduce the workload enough times to understand ordinary variation.
5. **Find the dominant cost.** Follow call relationships and inspect query counts, external waits, loops, serialization, and allocations.
6. **Change one cause.** Keep the code change focused so the result can be attributed.
7. **Compare and verify.** Repeat the same workload, compare distributions and resource use, and check behavior and correctness.
8. **Keep or revert.** Retain the change only when the evidence shows a useful improvement without an unacceptable trade-off.

Avoid profiling a single unusually fast or slow request and treating it as a general result. Caches, background load, filesystem state, database plans, and network variation can change measurements.

## Benchmark fairly

A microbenchmark is useful for a small, isolated operation, but it can mislead when it does not resemble application work.

For more reliable comparisons:

- use the same PHP version, extensions, configuration, machine, and input;
- separate setup and cleanup from the operation when appropriate;
- run warm-up iterations where runtime or caches need warming;
- collect multiple samples and compare the median and tail values, not only the fastest run;
- avoid unrelated concurrent load, or measure its effect deliberately;
- measure enough work that timer and loop overhead do not dominate;
- change one implementation at a time;
- include memory and output correctness, not just elapsed time;
- repeat the comparison on the actual deployment class before claiming a production improvement.

Do not benchmark a function whose work the runtime can eliminate, or whose results are not consumed. Do not subtract timer overhead mechanically unless you understand the measurement setup and can show that it matters.

For endpoints, compare latency distributions such as p50 and p95 under a defined load. A lower average can still hide a poor tail or increased error rate. Keep production monitoring in place after deployment to confirm that the improvement holds under real traffic.

## Common bottlenecks to investigate

### Repeated database queries

Count queries per request and inspect their duration. A query in a loop may turn one page render into many round trips. Consider batching, eager loading, or a better query, and verify result correctness and memory impact.

### External services and file I/O

Measure each dependency separately with timeouts. A request blocked on a slow service will not become faster from PHP loop micro-optimizations. Consider caching, bounded concurrency, retries with limits, or moving non-critical work to a queue.

### Inefficient algorithms or repeated work

Check whether nested loops, repeated parsing, or repeated construction make work grow rapidly with input size. Improve the algorithm or reuse safe intermediate results. Do not introduce shared mutable caches without considering lifetime and invalidation.

### Excessive memory use

Look for large arrays, duplicated data, unbounded result sets, and buffering entire files. Paginate or stream where appropriate, and verify peak memory at realistic input sizes.

### Serialization and rendering

Large JSON payloads, repeated template work, or expensive transformations can dominate a request. Measure before reducing fields or changing formats; preserve API contracts and output correctness.

## Optimization and trade-offs

Optimize the bottleneck that measurements identify. A small local speed-up is not useful if it increases memory dramatically, changes behavior, weakens security, or makes code much harder to maintain.

Prefer changes with clear causes and measurable effects: reduce redundant queries, bound data loaded into memory, remove repeated work, or improve an algorithm. Preserve caching semantics, validation, authorization, and error handling.

Do not use a profiler to justify premature micro-optimization. Readable and correct code is the baseline; optimize after a measured problem appears and re-measure after the change.

## Profiling safety and operations

- Do not enable expensive profiling globally on a public production service.
- Keep profiler output out of the web root and access-controlled.
- Limit the duration and scope of profiling, and clean up generated artifacts.
- Avoid copying sensitive production inputs, credentials, or personal data into profiling environments.
- Ensure profiling triggers cannot be used by arbitrary visitors.
- Monitor resource impact while profiling; the profiler itself can affect latency, memory, and disk usage.
- Treat profiler reports as operational data and handle them accordingly.

## Key takeaways

- Monitoring tells you where users are affected; profiling helps explain why a selected workload is expensive.
- Measure wall time, CPU cost, calls, and memory as separate signals.
- Use hrtime() for monotonic elapsed-time measurements and interpret PHP memory metrics within their scope.
- Profile representative workloads, follow the dominant cost, and change one cause at a time.
- Xdebug profiling is useful in controlled environments but adds overhead and writes potentially sensitive artifacts.
- Compare repeatable baselines and verify both performance and correctness after every optimization.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [135 - Safe error handling and private information](135-safe-error-handling-and-private-information.md) | [Q&A](../qnas/136-profiling-php-applications.md) | [Example](../examples/136-profiling-php-applications.md) | [137 - OPcache and PHP-FPM](137-opcache-and-php-fpm.md) |