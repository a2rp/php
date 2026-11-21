# Fibers and cooperative concurrency — example

## Question

Can two PHP Fibers take turns while preserving ordinary function calls, and does that mean their work runs in parallel?

This PHP 8.1+ command-line example starts two Fibers and uses a small round-robin loop to resume each suspended task. It demonstrates cooperative interleaving. It does not use an event loop or perform asynchronous I/O.

## Complete example

Save this as fibers-demo.php:

~~~php
<?php
declare(strict_types=1);

$fibers = [];

$definitions = [
    ['A', 3],
    ['B', 3],
];

foreach ($definitions as [$name, $steps]) {
    $fibers[$name] = new Fiber(
        static function (string $taskName, int $taskSteps): string {
            for ($step = 1; $step <= $taskSteps; $step++) {
                echo $taskName . ' step ' . $step . PHP_EOL;

                if ($step < $taskSteps) {
                    Fiber::suspend('yielded');
                }
            }

            return $taskName . ' complete';
        }
    );

    $value = $fibers[$name]->start($name, $steps);
    echo 'Main received from ' . $name . ': ' . $value . PHP_EOL;
}

do {
    $hasSuspendedFiber = false;

    foreach ($fibers as $fiber) {
        if ($fiber->isSuspended()) {
            $hasSuspendedFiber = true;
            $fiber->resume();
        }
    }
} while ($hasSuspendedFiber);

foreach ($fibers as $fiber) {
    if (!$fiber->isTerminated()) {
        throw new LogicException('A task did not finish');
    }

    echo $fiber->getReturn() . PHP_EOL;
}
~~~

Run it from a terminal with PHP 8.1 or newer:

~~~sh
php fibers-demo.php
~~~

## Expected output

~~~text
A step 1
Main received from A: yielded
B step 1
Main received from B: yielded
A step 2
B step 2
A step 3
B step 3
A complete
B complete
~~~

## Explanation

- Each callback runs until it calls Fiber::suspend(). The first start() returns the value yielded by that suspension.
- The main loop checks each Fiber and resumes only those currently suspended.
- The tasks take turns at explicit suspension points. Their output is interleaved, but they do not execute at the same time.
- The callback return value becomes available through getReturn() after normal termination.
- The scheduler is intentionally small. It has no I/O readiness, timers, cancellation, error queue, or shutdown policy.

## Add a value at a suspension point

The value passed to resume() becomes the result of Fiber::suspend() inside the paused Fiber. For example:

~~~php
$fiber = new Fiber(function (): void {
    $message = Fiber::suspend('waiting');

    echo 'Resumed with: ' . $message . PHP_EOL;
});

echo $fiber->start() . PHP_EOL;
$fiber->resume('continue');
~~~

Expected output:

~~~text
waiting
Resumed with: continue
~~~

This is how a scheduler can pass a result back to a paused task. A real async library usually wraps the mechanism in a higher-level API.

## What this example does not do

There is no automatic asynchronous work in the round-robin demo. The task only yields because the code explicitly calls Fiber::suspend(). If a task calls sleep(), a blocking database method, or a normal blocking HTTP client, that call blocks the current PHP thread.

For useful I/O concurrency, use a maintained event loop or async library with non-blocking clients. It must suspend tasks while operations are pending and resume them when the socket, timer, or other event is ready.

## Production considerations

- Bound the number of active Fibers and pending operations.
- Define how the scheduler reports exceptions, timeouts, cancellation, and shutdown.
- Use try/finally for resources that must be released.
- Treat every suspension point as a place where shared state may change.
- Do not hold database transactions or scarce locks across long waits without a deliberate design.
- Use a library whose I/O clients integrate with the same event loop.
- Measure memory, latency, throughput, and downstream service load.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [138 - Memory use and performance optimization](../lessons/138-memory-use-and-performance-optimization.md) | [Lesson](../lessons/139-fibers-and-cooperative-concurrency.md) | [Q&A](../qnas/139-fibers-and-cooperative-concurrency.md) | [140 - Background jobs, queues, and workers](../lessons/140-background-jobs-queues-and-workers.md) |