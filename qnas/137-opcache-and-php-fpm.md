# OPcache and PHP-FPM — Q&A

These questions review [Chapter 137](../lessons/137-opcache-and-php-fpm.md). Each answer follows its question, with an explanation where it helps.

## Multiple-choice questions

### 1. What does OPcache primarily reuse?

A. Database query results for every user.
B. Compiled PHP bytecode stored in shared memory.
C. HTTP responses from the web server.
D. All variables between requests.

**Answer: B.**

**Explanation:** OPcache reduces repeated PHP source parsing and compilation. It does not automatically cache application data or replace response caching.

### 2. What is the main role of PHP-FPM?

A. Manage PHP worker processes and handle requests through FastCGI.
B. Compile CSS files.
C. Store every database result in memory.
D. Replace the web server and database.

**Answer: A.**

**Explanation:** A web server commonly forwards PHP requests to FPM, which assigns them to worker processes and returns their responses.

### 3. What does pm.max_children limit?

A. The number of PHP files in OPcache.
B. The number of simultaneous requests a pool can serve.
C. The number of database tables.
D. The number of web server virtual hosts.

**Answer: B.**

**Explanation:** If every child is busy, new requests may queue. Increasing the limit without checking memory and downstream capacity can make an overloaded service worse.

### 4. Which process-manager mode maintains a configured number of idle workers and grows within bounds?

A. static
B. dynamic
C. ondemand
D. disabled

**Answer: B.**

**Explanation:** Dynamic mode uses startup, minimum spare, maximum spare, and maximum child settings. Static keeps a fixed child count; ondemand starts workers when needed and can retire idle workers.

### 5. If opcache.validate_timestamps is disabled, what must a deployment plan do when PHP files change?

A. Nothing; OPcache always detects the change.
B. Deliberately reset/invalidate OPcache or restart the relevant PHP process.
C. Increase pm.max_children.
D. Clear the browser cache only.

**Answer: B.**

**Explanation:** Without timestamp validation, OPcache does not normally detect source changes automatically. The release process must handle code freshness.

### 6. Why should pm.max_children not be chosen from CPU count alone?

A. Workers also use memory and downstream services have capacity limits.
B. CPU count determines PHP syntax.
C. OPcache cannot run with multiple workers.
D. FPM only handles one request at a time.

**Answer: A.**

**Explanation:** More workers can increase memory use, database connections, and contention. Size the pool with measured worker memory and the capacity of other services.

## Short-answer questions

### 7. How do OPcache and PHP-FPM complement each other?

**Answer:** OPcache reuses compiled bytecode, while PHP-FPM manages the worker processes that execute PHP requests.

**Explanation:** One reduces repeated compilation; the other controls worker concurrency and request handling. Neither fixes slow application logic by itself.

### 8. Why is a CLI OPcache status result not necessarily the FPM status?

**Answer:** CLI and FPM are separate SAPIs with separate configuration and cache instances. OPcache is commonly disabled for CLI by default.

**Explanation:** Inspect the PHP runtime that serves web requests. A CLI command may verify a different INI configuration and cache.

### 9. How should an operator estimate a safe FPM child limit?

**Answer:** Reserve memory for the rest of the system, measure realistic worker memory, then divide the PHP-FPM memory budget by a conservative per-worker estimate and keep additional headroom.

**Explanation:** Also consider CPU, database connection limits, upstream capacity, and latency targets. The memory calculation is an upper bound, not necessarily the final setting.

### 10. What does pm.max_requests do, and what does it not do?

**Answer:** It recycles a worker after it has served a configured number of requests. It can limit the impact of gradual memory growth, but it does not identify or fix the leak.

**Explanation:** Use it as an operational guard where appropriate, then investigate the underlying memory behavior.

### 11. What information can FPM's slow log provide?

**Answer:** It can capture a stack trace for a request that exceeds the configured slow-request threshold.

**Explanation:** This helps locate long-running application work. Protect the log because traces can expose internal paths and request behavior.

### 12. Why should the FPM status path be restricted?

**Answer:** It can disclose operational information such as active workers, queueing, and pool capacity.

**Explanation:** Limit it at the web server or network layer to trusted monitoring or administrators. A status path is not safe merely because its URL is uncommon.

## True or false

### 13. OPcache makes slow database queries fast.

**Answer: False.**

**Explanation:** OPcache avoids repeated PHP source compilation. Database performance needs query and database measurement and possibly a different optimization.

### 14. A PHP-FPM pool is complete security isolation from every other pool.

**Answer: False.**

**Explanation:** Pools provide useful process and configuration boundaries, but they may share one OPcache instance. Use separate OS or container boundaries when stronger isolation is required.

### 15. When timestamp validation is disabled, a code deployment should include an explicit cache freshness step.

**Answer: True.**

**Explanation:** The release process must reset/invalidate the cache or restart the relevant PHP workers so that they use the new code.

### 16. Increasing pm.max_children always increases throughput.

**Answer: False.**

**Explanation:** Too many workers can exhaust memory or overload the database and other dependencies, increasing queueing and latency.

### 17. A public FPM status endpoint or phpinfo page is harmless if it does not show passwords.

**Answer: False.**

**Explanation:** Operational details can help attackers map the service. Keep status and diagnostic endpoints private and remove temporary ones.

## Applied scenarios

### 18. A deployment changed PHP files, but visitors still see old behavior. Timestamp validation is off. What should the operator investigate?

**Answer:** Verify that the deployment performs the required OPcache invalidation or restarts the correct PHP-FPM service, and confirm that the FPM workers use the expected release and configuration.

**Explanation:** A CLI cache reset may not affect the FPM cache. Use the deployment's defined process and inspect the serving SAPI.

### 19. The FPM listen queue grows and maximum children reached is increasing. Should the team immediately double pm.max_children?

**Answer: No.** First inspect worker memory, request duration, CPU, database and upstream limits, and available system memory.

**Explanation:** A queue can result from slow requests or constrained dependencies. More children may just create more memory pressure and concurrent load downstream.

### 20. The OPcache cache restarts repeatedly due to capacity pressure. What should be checked?

**Answer:** Inspect OPcache status for memory usage, cached script count, wasted memory, and restart counters. Compare the configured memory and script-table capacity with the deployed application.

**Explanation:** Increase capacity only after identifying which limit is reached and considering the host's shared-memory budget.

### 21. A team enables preloading but deploys new classes without restarting PHP-FPM. Why may workers not see the new preloaded code?

**Answer:** Preloaded code is loaded when the persistent PHP process starts. The process must be restarted as part of the release to load updated preloaded files.

**Explanation:** Preloading trades startup behavior and baseline memory for keeping selected code available. It needs an explicit deployment and rollback plan.

## Code-reading questions

### 22. What risk does this setting create if the deployment does not reset OPcache?

~~~ini
opcache.validate_timestamps = 0
~~~

**Answer:** Workers may continue executing cached bytecode after PHP source files have changed.

**Explanation:** With timestamp validation disabled, the release must deliberately invalidate the cache or restart the relevant PHP process.

### 23. What is incomplete about this dynamic FPM pool configuration?

~~~ini
pm = dynamic
pm.max_children = 12
~~~

**Answer:** A dynamic pool also needs valid startup and spare-worker settings such as pm.start_servers, pm.min_spare_servers, and pm.max_spare_servers.

**Explanation:** FPM requires these settings for dynamic process management. Values must be chosen consistently and within the maximum child limit.

## Practice exercise

### 24. A PHP-FPM service has a pool memory budget of 900 MiB after system headroom is reserved. A conservative busy-worker estimate is 75 MiB. What is the memory-based upper bound for pm.max_children, and what else must be checked?

**Answer:** The arithmetic upper bound is 12 children because 900 divided by 75 equals 12. Treat that as a ceiling, then keep safety margin and check CPU, database connection capacity, upstream limits, request duration, and observed queueing.

**Explanation:** This calculation uses an estimate, not a guarantee. Measure representative busy workers and monitor the service after changing the limit.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [136 - Profiling PHP applications](../lessons/136-profiling-php-applications.md) | [Lesson](../lessons/137-opcache-and-php-fpm.md) | [Example](../examples/137-opcache-and-php-fpm.md) | [138 - Memory use and performance optimization](../lessons/138-memory-use-and-performance-optimization.md) |