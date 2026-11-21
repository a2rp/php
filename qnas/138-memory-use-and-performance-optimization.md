# Memory use and performance optimization — Q&A

These questions review [Chapter 138](../lessons/138-memory-use-and-performance-optimization.md). Every answer appears immediately after its question, with an explanation where it helps.

## Multiple-choice questions

### 1. What does memory_limit primarily restrict?

A. Total memory used by the operating system.
B. Memory a PHP script is allowed to allocate.
C. The combined memory of every PHP-FPM worker.
D. The size of the OPcache.

**Answer: B.**

**Explanation:** memory_limit is a per-script guardrail. It does not set the total memory budget for the host, container, or FPM pool.

### 2. Which approach is usually best when processing a very large file sequentially?

A. Load the whole file, split it into lines, then create another array.
B. Read and process records incrementally.
C. Increase memory_limit without measuring.
D. Store all lines in a global variable.

**Answer: B.**

**Explanation:** Streaming limits how much input must stay in memory at once. Also bound file and record sizes because a single very large record can still use substantial memory.

### 3. When does a generator provide little memory benefit?

A. When it yields values one at a time.
B. When the caller stores every yielded value in a new array.
C. When it is iterated with foreach.
D. When it reads from a file handle.

**Answer: B.**

**Explanation:** A generator avoids constructing the complete result in advance, but collecting all results recreates the same large in-memory collection.

### 4. What can memory_get_peak_usage(true) tell you?

A. The peak amount reserved by PHP's memory manager for the script.
B. The exact memory used by every PHP-FPM worker and extension.
C. The amount of free RAM on the server.
D. The database server's peak memory.

**Answer: A.**

**Explanation:** The value describes PHP-managed memory for the script. Use process or container metrics for broader memory use.

### 5. Why might an algorithm that uses an associative lookup need more memory?

A. Lookup structures consume memory even when they reduce repeated searching time.
B. Associative arrays do not store keys.
C. PHP disables garbage collection for arrays.
D. Every lookup writes a file to disk.

**Answer: A.**

**Explanation:** Performance changes often trade memory for time. Measure the real workload and choose the balance that fits service limits.

### 6. Which is the best reason to call gc_collect_cycles()?

A. It always makes every PHP request faster.
B. Measurements in a long-running process show unreachable reference cycles are a meaningful memory issue.
C. A normal script contains an array.
D. memory_limit is enabled.

**Answer: B.**

**Explanation:** PHP already has reference counting and a cycle collector. Calling the collector repeatedly without evidence can add work and does not fix retained live references.

## Short-answer questions

### 7. How are PHP allocator memory and total process memory different?

**Answer:** PHP memory functions track allocations made through PHP's memory manager. Total process memory can also include native extensions, runtime overhead, and other memory not reflected in those PHP values.

**Explanation:** Use both per-script measurements and operating-system or container metrics when diagnosing a service-level memory problem.

### 8. Why should memory_limit not be set to unlimited as a quick fix?

**Answer:** It removes a per-script guardrail without correcting unbounded input, retained values, duplicated data, or a leak.

**Explanation:** A script can still exhaust the worker, container, or host. Bound the workload, profile it, and set a limit consistent with the service budget.

### 9. What does PHP copy-on-write mean for arrays?

**Answer:** Some array assignments can initially share underlying values. A later modification may cause data to be copied.

**Explanation:** Do not assume every assignment immediately duplicates an entire array, but measure transformations and mutations that create additional large structures.

### 10. Why might unset() fail to lower the process's resident memory immediately?

**Answer:** It removes a reference to a value, but PHP's allocator may retain freed memory for later reuse instead of returning memory pages to the operating system immediately.

**Explanation:** The value may become available for future allocations in the process even if an external process monitor still reports a similar resident size.

### 11. How can pagination reduce memory pressure?

**Answer:** It limits how many database rows the application retrieves and retains at one time.

**Explanation:** Select only required columns and process bounded pages. For very large datasets, check whether the database driver buffers the full result despite row-by-row iteration.

### 12. What is the risk of adding an application cache to fix repeated work?

**Answer:** A cache can increase memory use and introduce stale data, invalidation complexity, and privacy risks.

**Explanation:** Measure the repeated work first, choose an appropriate cache lifetime and size, and define how entries are invalidated.

## True or false

### 13. A generator guarantees constant memory use for every kind of input.

**Answer: False.**

**Explanation:** A generator can avoid keeping all yielded items, but the caller may collect them, the generator may retain large state, or a single input record may itself be huge.

### 14. A low PHP memory_get_peak_usage() value proves a PHP-FPM service uses little memory overall.

**Answer: False.**

**Explanation:** That measurement is per script and limited to PHP's allocator scope. The FPM pool has multiple workers, and process or container memory includes additional costs.

### 15. A nested loop is always wrong and should always be replaced.

**Answer: False.**

**Explanation:** Algorithmic complexity matters when input sizes make the cost significant. Profile with representative data and preserve clarity and correctness.

### 16. A change that lowers memory but greatly increases latency is automatically a successful optimization.

**Answer: False.**

**Explanation:** Optimization balances latency, throughput, memory, and correctness. Compare the relevant service goals and resource constraints.

## Applied scenarios

### 17. A CLI import reads a 2 GB CSV with file_get_contents() and then calls explode() to make rows. Why may memory use far exceed 2 GB?

**Answer:** The entire file is held in a string, and splitting it creates many additional strings and array entries. Further transformations may create still more copies.

**Explanation:** Stream rows and process them incrementally. Enforce file and record limits, and avoid retaining processed rows.

### 18. A web service increases memory_limit for every request and stops seeing fatal memory errors, but its container is frequently killed. What should the team investigate?

**Answer:** Inspect per-request peaks, total process/container memory, FPM concurrency, input and result sizes, retained values, and native extension use.

**Explanation:** Raising a per-script limit can allow more concurrent memory use. Container-level memory is a separate constraint and may terminate the service before a request completes.

### 19. A generator reads database rows, but memory still rises with every row. What should you inspect?

**Answer:** Check whether the caller retains each row, whether the driver buffers the complete result, and whether the generator or surrounding objects keep references alive.

**Explanation:** Yielding one row at a time only helps if the whole pipeline avoids accumulating previous rows.

### 20. A repeated search is slow for a large list. A keyed map makes it faster but raises peak memory. How should the change be evaluated?

**Answer:** Compare the original and new approaches using representative list sizes; measure latency, peak memory, and output correctness.

**Explanation:** A faster lookup can be a good trade when memory headroom exists. If memory is constrained, use pagination or a different storage/access strategy.

### 21. A long-running worker's memory grows over time. Is adding gc_collect_cycles() on every job enough?

**Answer: No.** First find whether the worker retains live references, has unbounded caches, accumulates job data, or creates unreachable cycles.

**Explanation:** The cycle collector cannot free values that are still referenced. Use profiling and bounded batches; add explicit collection only if measurements show cycle collection is the issue.

## Code-reading questions

### 22. Why can this code consume much more memory than a streaming loop?

~~~php
$contents = file_get_contents($path);
$lines = explode(PHP_EOL, $contents);
$normalized = array_map('trim', $lines);
~~~

**Answer:** It keeps the full file string, an array of line strings, and a transformed array of trimmed strings.

**Explanation:** Each complete representation can add substantial memory overhead. Process the file incrementally and write or aggregate each result as it is read.

### 23. Does this code guarantee that memory returns to the operating system immediately?

~~~php
unset($largeArray);
~~~

**Answer: No.**

**Explanation:** If no references remain, PHP can make the value's memory available to its allocator, but the allocator may retain reserved pages for reuse. External process metrics need not drop immediately.

## Practice exercise

### 24. Design a safe approach for importing a large CSV into a database without retaining every row. What should the program do?

**Answer:** Validate the file and expected header, read one CSV record at a time, validate each field, write records in bounded database batches or transactions, keep only small counters, and close the input in a finally block. Enforce upload, file, record, and field-size limits.

**Explanation:** Streaming bounds retained rows but does not make untrusted input harmless. Define how malformed rows are handled, avoid logging sensitive row contents, and measure memory and throughput with representative data.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [137 - OPcache and PHP-FPM](../lessons/137-opcache-and-php-fpm.md) | [Lesson](../lessons/138-memory-use-and-performance-optimization.md) | [Example](../examples/138-memory-use-and-performance-optimization.md) | [139 - Fibers and cooperative concurrency](../lessons/139-fibers-and-cooperative-concurrency.md) |