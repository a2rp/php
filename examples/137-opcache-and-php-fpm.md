# OPcache and PHP-FPM — example

This example shows how to configure OPcache, size a PHP-FPM pool from measured memory, protect operational status, and deploy code without accidentally serving stale bytecode. All values are illustrative; adapt them to the operating system, web server, PHP version, workload, and resource limits.

## 1. Configure OPcache

Add settings to the INI file loaded by the PHP-FPM service. Example values:

~~~ini
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 20000
opcache.validate_timestamps = 1
opcache.revalidate_freq = 2
~~~

The memory and script limits above are starting examples, not universal recommendations. Check OPcache status and host memory before changing them. Keep timestamp validation enabled unless the deployment has an explicit cache invalidation or FPM restart step.

CLI and FPM can load different INI files and use separate OPcache instances. Verify settings in the FPM environment that serves requests; a CLI inspection is not proof of FPM state.

## 2. Estimate the PHP-FPM worker limit

Suppose measurements show:

- the PHP-FPM pool has a 900 MiB budget after reserving headroom for the host and other services;
- a conservative busy-worker estimate is 75 MiB.

The memory-only upper bound is:

~~~text
900 MiB / 75 MiB per worker = 12 workers
~~~

Choose a lower initial limit if measurements are uncertain or the database and upstream services cannot serve 12 concurrent requests. For this example, the pool uses 10 children and is monitored under representative load.

## 3. Configure an FPM pool

A sample pool configuration could look like this:

~~~ini
[www]
user = app
group = app

listen = /run/php/php-fpm.sock
listen.owner = nginx
listen.group = nginx
listen.mode = 0660

pm = dynamic
pm.max_children = 10
pm.start_servers = 3
pm.min_spare_servers = 2
pm.max_spare_servers = 4
pm.max_requests = 500

request_slowlog_timeout = 5s
slowlog = /var/log/php-fpm/www-slow.log

pm.status_path = /fpm-status
ping.path = /fpm-ping
~~~

Replace app, nginx, socket paths, limits, and log paths with values for the actual deployment. The socket owner and group must allow the web server to connect while preventing unrelated local users from connecting.

The process-manager settings must be internally consistent. For dynamic mode, the spare-worker and startup values must fit within pm.max_children. pm.max_requests = 500 is an example recycling limit, not a fix for a memory leak.

The status and ping paths must be restricted at the web-server or network layer to trusted monitoring. Do not publish them as general public routes. Keep the slow log protected because it can contain stack traces and internal paths.

## 4. Review status from the FPM context

A restricted diagnostic route in the application can call opcache_get_status(false) to return a small allowlisted summary. Place it behind existing administrator authentication and a web-server access rule; do not create a publicly accessible status script.

Example summary fields to inspect:

~~~php
$status = opcache_get_status(false);

if (is_array($status)) {
    $summary = [
        'enabled' => $status['opcache_enabled'] ?? false,
        'cache_full' => $status['cache_full'] ?? null,
        'memory_usage' => $status['memory_usage'] ?? [],
        'statistics' => $status['opcache_statistics'] ?? [],
    ];
}
~~~

Return only operational fields needed by trusted monitoring. Do not return the complete status object or cached script list to an unauthenticated caller.

For FPM, review active and idle workers, listen queue, maximum active processes, maximum children reached, and slow requests. For OPcache, review used and free memory, wasted memory, cached script count, hits and misses, and restart counters.

## 5. Deploy a release with cache freshness handled

If timestamp checking is disabled for an immutable production release, use a deployment sequence that deliberately replaces or restarts the FPM workers after the new files are in place. A platform-specific service command may look like this:

~~~sh
sudo systemctl restart php8.3-fpm
~~~

The service name and restart behavior vary. Use the platform's graceful rollout procedure where available so existing requests finish safely. Do not try to clear the FPM cache by running opcache_reset() from CLI; that is a separate SAPI/cache instance.

Before applying configuration, validate the FPM configuration with the command provided by the installed package, commonly:

~~~sh
php-fpm -t
~~~

The binary name and configuration path vary by platform. Then deploy, reload or restart the correct FPM service as planned, and verify the active FPM status and application behavior. The CLI php --ini output only describes the CLI SAPI.

If timestamp validation remains enabled, test how quickly the serving SAPI detects a changed file under the configured revalidation frequency. Avoid changing source files in place during live requests when an immutable release strategy is available.

## 6. Check behavior after rollout

After applying the configuration, observe the application under a representative load and confirm:

- FPM workers stay within the intended memory budget;
- listen queues and maximum-children counters do not indicate sustained saturation;
- the web server can connect to the configured FPM socket;
- slow requests are recorded in the protected slow log;
- OPcache is enabled and not repeatedly restarting from capacity pressure;
- the new code is running in FPM after the release;
- latency, error rates, database load, and upstream health remain acceptable;
- status endpoints are reachable only by trusted monitoring or administrators.

Increase worker or cache limits only when measurements identify that specific limit and the host and downstream services have capacity for the change.

## Security reminders

- Never expose the FastCGI listener to the public internet.
- Keep FPM status, OPcache summaries, and phpinfo() private.
- FPM pools are not complete security isolation; use separate OS or container boundaries when stronger isolation is required.
- Pass only required environment variables into workers, and keep secrets out of diagnostic output.
- Keep slow logs and status data access-controlled.
- Make cache freshness an explicit part of the deployment and rollback process.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [136 - Profiling PHP applications](../lessons/136-profiling-php-applications.md) | [Lesson](../lessons/137-opcache-and-php-fpm.md) | [Q&A](../qnas/137-opcache-and-php-fpm.md) | [138 - Memory use and performance optimization](../lessons/138-memory-use-and-performance-optimization.md) |