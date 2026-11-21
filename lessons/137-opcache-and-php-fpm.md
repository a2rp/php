# OPcache and PHP-FPM

A PHP web application usually parses and runs PHP code for incoming requests. **OPcache** stores compiled PHP bytecode in shared memory so PHP can reuse it instead of parsing the same files on each request. **PHP-FPM** manages persistent PHP worker processes and serves requests through FastCGI.

They address related but different costs: OPcache reduces repeated compilation work, while PHP-FPM controls request workers and concurrency. Neither one automatically fixes slow database queries, inefficient algorithms, or overloaded dependencies.

## Learning objectives

After this lesson, you should be able to:

- explain how OPcache and PHP-FPM work together;
- choose OPcache settings appropriate to a deployment and release strategy;
- understand FPM pools and process-manager modes;
- size worker concurrency from measured memory and capacity limits;
- inspect cache and worker health without exposing sensitive status endpoints;
- deploy configuration changes and code releases without serving stale code.

## How requests reach PHP

A common web request path is:

1. A browser sends an HTTP request to a web server or reverse proxy.
2. The web server serves static files itself or forwards a PHP request over FastCGI.
3. PHP-FPM assigns that request to an available worker.
4. The worker runs the application and returns a response through the web server.

FPM workers are persistent processes, but ordinary PHP request data is generally initialized and cleaned up between requests. Extensions, native libraries, global process state, memory fragmentation, or bugs can still cause long-lived worker behavior to matter. Monitor workers over time.

## What OPcache does

Without opcode caching, PHP must read and compile source files into executable bytecode as needed. OPcache stores compiled bytecode in shared memory for reuse. This avoids repeated file parsing and compilation, but it does not cache every database result or eliminate application work.

OPcache is normally configured in PHP's INI configuration. An illustrative configuration is:

~~~ini
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 20000
opcache.validate_timestamps = 1
opcache.revalidate_freq = 2
~~~

These values are examples, not universal recommendations. Choose cache memory and script capacity from application size and observed status. Check available memory and cache restarts instead of copying values blindly.

### File change detection and deployments

When timestamp validation is enabled, OPcache can check whether a cached file has changed. The revalidation frequency controls how often timestamps are checked. A longer interval can delay detection of a deployment, depending on configuration and version.

When timestamp validation is disabled, PHP does not automatically detect changed source files in the normal way. Deployment must invalidate or reset the cache or restart the relevant PHP process after releasing code. A release should not depend on an operator remembering to clear a stale cache later.

Use one explicit release strategy:

- keep timestamp checking enabled with a suitable deployment configuration; or
- deploy immutable release directories and deliberately reset/restart the FPM service as part of the release.

Test the strategy in staging. Be especially careful when deploying by modifying source files in place while requests are being served.

### CLI and web OPcache are different

The CLI SAPI and PHP-FPM are separate processes with separate configuration and cache instances. OPcache is commonly disabled for CLI by default. A CLI script that prints OPcache status therefore does not necessarily show what the web workers are using.

Inspect the same SAPI and runtime configuration that serves the application. Avoid treating php -i or a CLI status result as proof that FPM has the same settings.

### Memory, scripts, and preloading

OPcache uses shared memory for compiled scripts and interned strings. If the configured memory or script table is too small, cache restarts or missed caching can occur. Monitor status and tune based on measurements.

Preloading, available in PHP 7.4 and newer, loads selected code when the persistent PHP process starts. It can increase baseline memory use and requires the preload process to be restarted when the preloaded code changes. It is an advanced production choice, not a default optimization to enable without a measured reason.

JIT is separate from ordinary opcode caching. It can help selected CPU-bound workloads, but it is not a general remedy for slow I/O, database queries, or excessive request work. Benchmark the real workload before considering JIT settings.

## PHP-FPM process managers

FPM manages a pool of worker processes. The process-manager mode determines how the pool grows and shrinks:

- **static** keeps a fixed number of children;
- **dynamic** maintains configured minimum and maximum idle workers and grows up to its limit;
- **ondemand** starts workers when requests arrive and retires idle workers after a timeout.

The main capacity setting is pm.max_children: it limits the number of simultaneous requests a pool can serve. If every child is busy, new work waits in the listen queue or at the web server. Too few workers can create queueing; too many can exhaust memory and reduce overall service health.

Other useful settings include:

- pm.start_servers, pm.min_spare_servers, and pm.max_spare_servers for dynamic pools;
- pm.process_idle_timeout for ondemand pools;
- pm.max_requests to recycle workers after a bounded number of requests;
- request_slowlog_timeout and slowlog to capture stack traces for requests that exceed a threshold;
- pm.status_path and ping.path for operational health checks.

Worker recycling can limit damage from gradual memory growth, but it does not fix the cause of a leak. Use the slow log to investigate long requests; do not increase worker counts blindly when requests are slow.

## Estimate a safe worker limit

Estimate the memory budget available to the PHP-FPM pool after reserving headroom for the operating system, web server, database or other co-located services, and short-term spikes.

Then use representative measurements to estimate worker memory:

~~~text
safe child count <= PHP-FPM memory budget / conservative per-worker memory estimate
~~~

For example, if the pool budget is 768 MiB and measured busy workers require about 80 MiB each, 9 workers is an upper bound before further safety margin. Do not use the average of a tiny sample as the only input. Measure realistic requests and use a conservative high-water estimate.

The final maximum may be lower because of CPU, database connection limits, upstream capacity, or latency goals. A PHP-FPM pool with more children than the database can serve may increase contention instead of throughput.

## Monitor both layers

For OPcache, inspect enabled state, used and free memory, wasted memory, cached script counts, hits, misses, and restart counters. Cache hit rate is useful context, but it is not a complete measure of application speed.

For PHP-FPM, monitor active and idle processes, listen queue, maximum active processes, maximum children reached, slow requests, and worker exits. These help distinguish a worker-capacity problem from a slow dependency or an application bottleneck.

A status endpoint can expose internal topology and capacity details. Restrict it to trusted monitoring or administrators at the web-server or network layer. Do not make FPM status, OPcache status, phpinfo(), or diagnostic endpoints public.

## Pools and security boundaries

FPM pools let an operator configure different users, sockets, limits, and PHP settings for groups of requests. Pools are useful for configuration and process management, but they are not complete security isolation. In particular, pools can share one OPcache instance.

Use separate operating-system or container boundaries when applications require stronger isolation. Keep FastCGI bound to a Unix socket or a trusted private network, configure socket permissions, and do not expose an FPM port to the public internet. A party that can connect directly to FastCGI may be able to influence request configuration and execute code.

Review environment variables deliberately. FPM clears the worker environment by default and only passes explicitly configured values. Add only the environment needed by the application, and keep secrets out of diagnostic status pages and logs.

## Practical operations

- Check which INI files the web SAPI loads; CLI settings can differ.
- Validate FPM configuration before reloading it.
- Apply FPM and OPcache changes through the deployment process and verify they took effect.
- Restart or invalidate the correct cache when a deployment strategy requires it.
- Watch worker queues, memory, slow logs, cache restarts, and application latency after a change.
- Keep status interfaces private and remove temporary diagnostic scripts.
- Keep logs and slow traces access-controlled because they can reveal paths or request behavior.

## Key takeaways

- OPcache reuses compiled bytecode; PHP-FPM manages persistent workers and request concurrency.
- Tune cache and pool limits using observed application and deployment behavior.
- pm.max_children is constrained by memory, CPU, database limits, and upstream capacity.
- CLI and FPM use separate runtime contexts and cache instances.
- A deployment must deliberately handle OPcache code freshness.
- Protect FastCGI, status endpoints, profiler output, and slow logs from public access.
- Preloading and JIT are specialized choices; measure before enabling or tuning them.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [136 - Profiling PHP applications](136-profiling-php-applications.md) | [Q&A](../qnas/137-opcache-and-php-fpm.md) | [Example](../examples/137-opcache-and-php-fpm.md) | [138 - Memory use and performance optimization](138-memory-use-and-performance-optimization.md) |