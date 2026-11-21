# Fibers and cooperative concurrency

A Fiber is a low-level PHP mechanism for pausing a function and resuming it later. Fibers let an application or library build cooperative concurrency while keeping a synchronous-looking call stack.

A Fiber does not create an operating-system thread, start an event loop, or make blocking I/O asynchronous by itself. A scheduler or event loop must decide when a suspended Fiber can run again, usually after a non-blocking operation becomes ready.

## Learning objectives

After this lesson, you should be able to:

- explain what a PHP Fiber does and how it differs from a thread or Generator;
- use Fiber construction, start, suspend, resume, throw, and getReturn;
- understand Fiber states and common FiberError cases;
- explain why Fibers alone do not make blocking I/O concurrent;
- recognize when a framework or asynchronous I/O library should manage Fibers;
- handle shared state, exceptions, cancellation, and resource cleanup around suspension points.

## What a Fiber is

Fibers were added in PHP 8.1. A Fiber has its own call stack and can suspend execution from within a deeply nested function call. The suspended stack remains paused until code resumes it.

A Fiber is **cooperative**: it gives up control only at an explicit suspension point or through an abstraction that suspends it. PHP does not interrupt a Fiber at arbitrary times to switch to another one.

Only one Fiber executes at a time in a given PHP thread. Fibers can interleave tasks, but they do not provide CPU parallelism by themselves. To use multiple CPU cores, use an appropriate process, worker, or parallel-execution mechanism.

## Fiber compared with Generator and thread

- A **Generator** produces a sequence of values through yield. It is useful for iteration and typically pauses at the yield expression.
- A **Fiber** can suspend from anywhere in its call stack, including a nested function, and later resume that call stack.
- A **thread** can execute at the same time as another thread. Native PHP Fibers are not threads.

A Generator is often the simpler tool for streaming values. A Fiber is useful when a scheduler or asynchronous library wants to suspend a task while preserving ordinary call structure.

## Start, suspend, and resume

Create a Fiber with a callable, then start it. When it calls Fiber::suspend(), execution returns to the code that called start() or resume().

~~~php
$fiber = new Fiber(function (): string {
    $message = Fiber::suspend('waiting for input');

    return 'received: ' . $message;
});

$firstValue = $fiber->start();
// $firstValue is "waiting for input".

$fiber->resume('hello');
// The Fiber continues after Fiber::suspend().

$result = $fiber->getReturn();
// $result is "received: hello".
~~~

The value passed to Fiber::suspend() is returned by the start(), resume(), or throw() call that switched into the Fiber. When the Fiber is resumed with a value, that value becomes the result of the suspended Fiber::suspend() call.

If a Fiber completes without suspending, start() returns null and the Fiber is terminated. If it suspends more than once, each resume() continues until the next suspension or completion.

## Fiber lifecycle and errors

A Fiber can be inspected with isStarted(), isSuspended(), isRunning(), and isTerminated(). The lifecycle matters because operations are only valid in the right state:

- start() can be called only once;
- resume() requires a suspended Fiber;
- Fiber::suspend() must be called from within a running Fiber;
- getReturn() is available after normal termination, and throws if the Fiber failed.

Invalid lifecycle operations throw FiberError. An exception that escapes the Fiber callback is propagated to the code that started or resumed it. Handle failures at the task or scheduler boundary; do not assume every Fiber succeeds.

Fiber::throw($exception) resumes a suspended Fiber by throwing the given exception from its suspended Fiber::suspend() call. This can be used by a scheduler to deliver cancellation, timeout, or I/O failure. The Fiber's code must still handle that exception and clean up its resources.

## Fibers do not make blocking calls asynchronous

A Fiber only pauses when it suspends. If it calls a blocking function such as sleep() or performs blocking network I/O, the current PHP thread waits in that call. Other Fibers on that thread cannot make progress during that wait.

Actual I/O concurrency requires non-blocking I/O plus a scheduler or event loop that:

1. starts or resumes Fibers;
2. registers readiness callbacks for non-blocking operations;
3. suspends a Fiber while an operation is pending;
4. resumes it when the operation completes, times out, or fails.

Frameworks and asynchronous libraries often provide these pieces and a higher-level API. Use the library's Fiber-aware clients and primitives consistently. Calling a normal blocking client from a Fiber does not make that client non-blocking.

## Suspension points and shared state

Fibers do not execute simultaneously on the same PHP thread, but their operations can interleave. Another Fiber may run while the first Fiber is suspended and may change shared objects, caches, or application state before the first Fiber resumes.

Treat each suspension as a point where assumptions about mutable shared state may no longer hold. Keep invariants clear, avoid partially updated shared data across a suspension, and use synchronization offered by the scheduler or library when multiple tasks coordinate access.

Do not hold a database transaction, file lock, or other scarce resource across a long wait unless the design requires it. A suspended task can keep resources open while unrelated work proceeds.

## Cleanup, errors, and cancellation

Use try/finally for resources that must be released even if a Fiber receives an exception or is abandoned. A scheduler may deliver cancellation using Fiber::throw(); task code should handle cancellation deliberately and allow cleanup to run.

~~~php
$fiber = new Fiber(function () use ($resource): void {
    try {
        performWork($resource);
        Fiber::suspend('waiting for next step');
    } finally {
        closeResource($resource);
    }
});
~~~

The example shows the cleanup structure only. In real code, ownership of the resource must be clear: do not let multiple Fibers close or mutate the same resource without coordination.

A Fiber that is never resumed can retain its captured values and resources. Bound the number of active Fibers, define timeouts and cancellation behavior, and ensure the scheduler can finish or clean up tasks during shutdown.

## Use Fibers through a suitable abstraction

Raw Fiber APIs are low-level. A production scheduler must manage task states, event-loop callbacks, exceptions, cancellation, timeouts, and shutdown. Reimplementing those parts casually can produce deadlocks, lost failures, leaked resources, or unbounded tasks.

Prefer a maintained framework or asynchronous library when the application needs concurrent I/O. Check that database, HTTP, filesystem, and other client libraries use non-blocking operations or integrate with the same scheduler. A Fiber-aware HTTP client does not make an unrelated blocking database call asynchronous.

Use native Fibers directly for small, controlled demonstrations or when building a runtime abstraction with clear lifecycle and error handling.

## Performance and resource costs

Concurrency can reduce total waiting time when independent I/O operations overlap, but it does not guarantee faster work. More tasks can increase memory use, open connections, server load, and contention. Each Fiber also has runtime state and stack storage.

Measure end-to-end latency, throughput, memory, number of active tasks, and downstream load. Bound concurrency and use backpressure so a burst of work cannot create an unbounded number of Fibers or pending requests.

For CPU-bound work, Fibers on one thread do not run computations in parallel. Use algorithmic improvements or a process/worker design when parallel CPU execution is required.

## Key takeaways

- Fibers pause and resume call stacks cooperatively; PHP 8.1 introduced the API.
- A Fiber is not a thread, and Fiber switching alone does not provide asynchronous I/O.
- Fiber::suspend() returns a value to its caller; resume() supplies the value received at the suspension point.
- A scheduler or event loop is responsible for deciding when a Fiber resumes.
- Blocking I/O blocks the current thread even when called from a Fiber.
- Handle lifecycle errors, task exceptions, cancellation, resource cleanup, and shared state deliberately.
- Use a Fiber-aware library for production asynchronous I/O, then measure and bound concurrency.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [138 - Memory use and performance optimization](138-memory-use-and-performance-optimization.md) | [Q&A](../qnas/139-fibers-and-cooperative-concurrency.md) | [Example](../examples/139-fibers-and-cooperative-concurrency.md) | [140 - Background jobs, queues, and workers](140-background-jobs-queues-and-workers.md) |