# Profiling PHP applications — Q&A

These questions review [Chapter 136](../lessons/136-profiling-php-applications.md). Each answer appears immediately after its question, with an explanation where useful.

## Multiple-choice questions

### 1. What is the main purpose of profiling?

A. To prove every line of code can be made faster.
B. To locate where a representative run spends time or memory.
C. To replace production monitoring.
D. To increase traffic to a slow endpoint.

**Answer: B.**

**Explanation:** Profiling helps identify the cost centers in a selected workload. Monitoring shows production impact over time; profiling helps investigate why a chosen operation is expensive.

### 2. Which PHP function is preferred for measuring elapsed time on PHP 7.3 and newer?

A. date()
B. hrtime()
C. rand()
D. memory_get_usage()

**Answer: B.**

**Explanation:** hrtime() uses a monotonic high-resolution clock. Measure the difference between two readings; its absolute value is not a calendar timestamp.

### 3. A request has low CPU time but high wall-clock latency. What is a likely explanation?

A. The request is waiting on a database, network, filesystem, or another process.
B. The request has no latency.
C. PHP has automatically optimized every function.
D. The profiler has measured memory instead of time.

**Answer: A.**

**Explanation:** Wall time includes waiting. Investigate dependency timing and I/O instead of assuming PHP computation is the bottleneck.

### 4. What do memory_get_peak_usage(true) results represent?

A. Total memory used by the server.
B. Peak memory reserved by PHP's memory manager for the script.
C. Memory used by all PHP-FPM workers.
D. Database server memory.

**Answer: B.**

**Explanation:** This PHP measurement does not necessarily include all native extension or operating-system process memory. Use process or container metrics for total process memory.

### 5. How should Xdebug profiling usually be used?

A. Enabled for every public request forever.
B. Enabled selectively in a controlled local or staging environment.
C. Used as a substitute for correct application behavior.
D. Configured to write profiler files inside the public web root.

**Answer: B.**

**Explanation:** Profiling can add substantial overhead and generate large artifacts. Restrict the scope, protect the output directory, and clean up the files.

### 6. A function has high inclusive cost but low self cost. What should you inspect next?

A. Its child calls and the operations they perform.
B. The logo on the page.
C. Only its variable names.
D. The number of README sections.

**Answer: A.**

**Explanation:** Inclusive cost includes child work. The expensive operation may be deeper in the call graph.

## Short-answer questions

### 7. How are profiling, benchmarking, and monitoring different?

**Answer:** Profiling locates costs in a selected run; benchmarking compares a defined operation under controlled conditions; monitoring tracks production behavior over time.

**Explanation:** Together, they help identify user impact, explain a bottleneck, and check whether a change improves the measured workload.

### 8. Why should you record a baseline before optimizing?

**Answer:** A baseline provides a comparison point under a known workload and environment.

**Explanation:** Without it, you cannot reliably tell whether a change helped, whether normal variation explains the result, or whether the work moved its cost elsewhere.

### 9. Why is one benchmark run usually insufficient?

**Answer:** Individual measurements can vary because of caches, background work, database plans, network conditions, and other system activity.

**Explanation:** Repeat the same workload and compare a distribution, such as the median and tail values, instead of selecting the fastest sample.

### 10. What does a high function call count suggest?

**Answer:** It may suggest repeated work, such as a database query inside a loop or an N+1 query pattern.

**Explanation:** A single call can be cheap while thousands of calls dominate the request. Inspect counts and call relationships alongside per-call cost.

### 11. Why can Xdebug profiler results differ from ordinary production latency?

**Answer:** The profiler instruments execution and adds overhead; its output describes that instrumented run.

**Explanation:** Use it to find call relationships and costs, but use production monitoring or a suitable APM to assess real user latency.

### 12. Why do PHP allocator memory figures not necessarily equal process memory?

**Answer:** PHP memory functions report allocations tracked by PHP's memory manager. They may not include all memory used by native extensions or the full operating-system process.

**Explanation:** Use process or container metrics when the question concerns total worker or container memory.

## True or false

### 13. A slow endpoint is always caused by PHP executing CPU-heavy code.

**Answer: False.**

**Explanation:** The request may be waiting for a slow query, network service, filesystem, or lock. Compare wall time, CPU cost, and dependency timing.

### 14. The absolute value returned by hrtime() is a useful calendar timestamp.

**Answer: False.**

**Explanation:** It is measured from an arbitrary monotonic clock origin. Take a difference between readings to measure elapsed time.

### 15. Xdebug profiler artifacts should be written to a publicly served directory so they are easy to download.

**Answer: False.**

**Explanation:** Keep profiling output outside the web root, access-controlled, and temporary. Profile files can reveal operational details and consume disk space.

### 16. A code change that reduces average latency but increases tail latency and error rate is automatically a successful optimization.

**Answer: False.**

**Explanation:** Performance has multiple dimensions. Compare distributions, resource use, errors, and correctness before keeping the change.

## Applied scenarios

### 17. A page performs 400 fast database queries. Which profiler clues should you inspect?

**Answer:** Inspect query counts, call relationships, repeated query patterns, and the aggregate database time per request.

**Explanation:** Many individually quick queries can create substantial wall time. A query in a loop may indicate an N+1 pattern that can be batched or eagerly loaded.

### 18. A profile shows a route spending most of its time waiting for an external API. What is a sensible next step?

**Answer:** Measure that dependency separately and inspect its timeout, latency, and retry behavior.

**Explanation:** PHP micro-optimizations will not remove network wait. Depending on the requirement, consider bounded retries, caching, asynchronous work, or a better-defined timeout.

### 19. A developer reports a 30% speed improvement after one run on a different machine with a warm cache. Is the result conclusive?

**Answer: No.** Repeat the comparison on the same environment and representative workload, with consistent cache and setup conditions.

**Explanation:** The machine and cache state changed, so the observed difference cannot be attributed confidently to the code change alone.

### 20. A profile highlights a function with large inclusive cost. What should you do before rewriting it?

**Answer:** Follow its child calls to find the operation responsible for the cost and form a specific hypothesis.

**Explanation:** The function may be an expensive caller of another function rather than the source of the work itself.

### 21. A memory profile looks acceptable for one small request, but production workers are being killed for memory use. What else should be measured?

**Answer:** Measure realistic input sizes and total worker or container process memory, including concurrent workers and memory outside PHP's allocator.

**Explanation:** A single request's PHP memory metric does not represent the full worker pool or process. Observe the deployment-level memory limit and concurrency.

## Code-reading questions

### 22. What is wrong with this timing code?

~~~php
$start = time();

// Run a short operation.

$elapsed = time() - $start;
~~~

**Answer:** time() has coarse one-second resolution, so a short operation may appear to take zero or one second.

**Explanation:** On PHP 7.3 and newer, use hrtime(true) and calculate the difference in nanoseconds before converting to a readable unit.

### 23. Does this code report total PHP-FPM pool memory?

~~~php
$peakBytes = memory_get_peak_usage(true);
~~~

**Answer: No.**

**Explanation:** It reports peak memory reserved by PHP's memory manager for the current script. Use process, worker, or container metrics to assess the wider service.

## Practice exercise

### 24. Plan a fair comparison for an endpoint that was optimized to reduce repeated queries. What should be held constant and what should be compared?

**Answer:** Hold constant the PHP version and configuration, machine or container class, database fixture and state, endpoint input, cache conditions, and concurrency. Repeat the workload before and after, then compare latency distributions, query count and time, memory, errors, and response correctness.

**Explanation:** Change one cause at a time and use representative data. A lower query count is valuable only if the endpoint still returns correct results and the full request behavior improves.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [135 - Safe error handling and private information](../lessons/135-safe-error-handling-and-private-information.md) | [Lesson](../lessons/136-profiling-php-applications.md) | [Example](../examples/136-profiling-php-applications.md) | [137 - OPcache and PHP-FPM](../lessons/137-opcache-and-php-fpm.md) |