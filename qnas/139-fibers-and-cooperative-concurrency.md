# Fibers and cooperative concurrency — Q&A

These questions review [Chapter 139](../lessons/139-fibers-and-cooperative-concurrency.md). Every answer follows its question, with an explanation where useful.

## Multiple-choice questions

### 1. In which PHP version were native Fibers introduced?

- A. PHP 7.0
- B. PHP 7.4
- C. PHP 8.1
- D. PHP 8.4

**Answer: C.**

**Explanation:** The native Fiber API is available from PHP 8.1.

### 2. What happens when a running Fiber calls Fiber::suspend('ready')?

- A. It creates an operating-system thread.
- B. It pauses and returns the value to the call that started or resumed it.
- C. It terminates permanently.
- D. It automatically waits for a network response.

**Answer: B.**

**Explanation:** The caller receives the value passed to suspend(). The Fiber can later continue when the owner resumes it.

### 3. What does Fiber::resume('continue') provide to the suspended Fiber?

- A. The return value of the entire Fiber.
- B. The value returned from its suspended Fiber::suspend() call.
- C. A new PHP process.
- D. The next HTTP request.

**Answer: B.**

**Explanation:** The value passed to resume() becomes the result of the suspension expression inside the Fiber.

### 4. What is required for Fibers to overlap independent network operations?

- A. A non-blocking I/O implementation and a scheduler or event loop.
- B. Only Fiber::suspend() inside a normal blocking HTTP request.
- C. More PHP syntax.
- D. A second call to start() on each Fiber.

**Answer: A.**

**Explanation:** Fibers provide a way to pause and resume call stacks. An event loop or library must manage readiness and resume tasks when I/O completes.

### 5. What happens if a Fiber calls a normal blocking sleep() function?

- A. The current PHP thread waits; other Fibers on that thread cannot run meanwhile.
- B. PHP automatically converts sleep() to a non-blocking timer.
- C. A new operating-system thread starts.
- D. The Fiber is cancelled.

**Answer: A.**

**Explanation:** A Fiber only yields when it suspends or calls a fiber-aware abstraction. A blocking call still blocks the thread.

### 6. Which statement about Fibers and CPU parallelism is correct?

- A. Fibers automatically execute on separate CPU cores.
- B. Fibers interleave cooperatively but do not provide CPU parallelism by themselves.
- C. Every Fiber is a separate PHP-FPM worker.
- D. A Fiber always executes after the current request ends.

**Answer: B.**

**Explanation:** Native Fibers are a low-level cooperative primitive within a PHP thread. Use processes or another parallel-execution mechanism for CPU parallelism.

## Short-answer questions

### 7. How is a Fiber different from a Generator?

**Answer:** A Generator yields a sequence of values at yield expressions. A Fiber can suspend from within a call stack and later resume that stack.

**Explanation:** Generators are a natural fit for iteration. Fibers help schedulers and async libraries suspend ordinary call chains.

### 8. What does Fiber::start() return?

**Answer:** It returns the value from the first Fiber::suspend() call, or null if the Fiber completes without suspending.

**Explanation:** A Fiber's final callback return value is retrieved with getReturn() after the Fiber terminates normally.

### 9. What value does resume() return?

**Answer:** It returns the value from the next suspension point, or null if the Fiber completes.

**Explanation:** The argument passed to resume() is delivered back into the suspended Fiber::suspend() call.

### 10. When can getReturn() be called successfully?

**Answer:** After the Fiber has terminated normally and returned a value.

**Explanation:** Calling it before termination is invalid. If the Fiber terminated by throwing an uncaught exception, getReturn() does not provide a normal result.

### 11. What is the role of an event loop in Fiber-based I/O?

**Answer:** The event loop watches non-blocking operations and resumes suspended Fibers when an operation is ready, times out, or fails.

**Explanation:** PHP's Fiber API alone does not monitor sockets or schedule tasks. A scheduler or library must provide that behavior.

### 12. Why should code treat a suspension point as a shared-state boundary?

**Answer:** Other Fibers can run while the current Fiber is suspended and may change shared objects before it resumes.

**Explanation:** Fibers do not execute simultaneously in the same thread, but their operations can interleave. Recheck assumptions and keep shared invariants safe.

## True or false

### 13. Fibers automatically make every database or HTTP client asynchronous.

**Answer: False.**

**Explanation:** A blocking client still blocks the current PHP thread. Use an asynchronous or Fiber-aware client integrated with the application's scheduler.

### 14. PHP automatically switches between Fibers at arbitrary points in a loop.

**Answer: False.**

**Explanation:** Fibers are cooperative. They switch at explicit suspension points or through abstractions that suspend them.

### 15. Calling resume() on a Fiber that is not suspended is valid and starts it again.

**Answer: False.**

**Explanation:** start() can start a Fiber only once, and resume() requires a suspended Fiber. Invalid lifecycle operations throw FiberError.

### 16. Multiple Fibers on one PHP thread can interleave updates to shared state.

**Answer: True.**

**Explanation:** They are not executing simultaneously, but another Fiber may modify shared state during a suspension. Keep critical updates and invariants deliberate.

### 17. A Fiber that is never resumed cannot retain captured values or resources.

**Answer: False.**

**Explanation:** A suspended Fiber can remain alive while references to it are held, retaining its call stack, captured values, and resources. Bound task lifetimes and define cancellation and shutdown behavior.

## Applied scenarios

### 18. A Fiber calls a synchronous database query that takes two seconds. What happens to other Fibers on the same thread?

**Answer:** They cannot make progress during that blocking query call.

**Explanation:** The query must use a non-blocking driver integrated with the scheduler to overlap I/O. Merely wrapping the call in a Fiber does not change its behavior.

### 19. A Fiber is waiting on a network operation that times out. How can the scheduler report the failure to it?

**Answer:** A scheduler can resume the Fiber with an exception using Fiber::throw(), or deliver the failure through its higher-level async abstraction.

**Explanation:** The exception is raised at the suspended Fiber::suspend() point. Fiber code should handle expected failures and use finally blocks for cleanup.

### 20. A task uses 200 Fibers for outbound requests, but memory and service load spike. What should be changed?

**Answer:** Bound the number of concurrent tasks, apply backpressure, and measure memory and downstream capacity.

**Explanation:** More concurrency can increase pending state, open connections, and load on dependencies. It does not guarantee lower latency.

### 21. A CPU-heavy loop runs inside a Fiber and never suspends. Will a second Fiber run before it finishes?

**Answer: No.**

**Explanation:** Cooperative switching needs a suspension point or scheduler-aware operation. A CPU-bound task that never yields continues to occupy the current thread.

## Code-reading questions

### 22. What is wrong with this code?

~~~php
$fiber = new Fiber(function (): void {
    sleep(2);
    Fiber::suspend('done');
});

$fiber->start();
~~~

**Answer:** sleep() blocks the current PHP thread for two seconds. The Fiber does not yield until after the blocking call.

**Explanation:** Use a scheduler-aware timer or non-blocking wait if other tasks should progress during the delay.

### 23. What does this code print, and what does resume() return?

~~~php
$fiber = new Fiber(function (): void {
    Fiber::suspend('paused');
});

echo $fiber->start(), PHP_EOL;
var_dump($fiber->resume());
~~~

**Answer:** It prints paused, then bool(false) is not printed; var_dump() prints NULL.

**Explanation:** start() receives the value passed to suspend(). On resume, the Fiber continues and returns normally, so resume() returns null. The bool(false) wording in the first sentence is intentionally clarified here: the actual second output is NULL.

## Practice exercise

### 24. Describe the pieces needed to issue several HTTP requests concurrently with Fibers.

**Answer:** Use a scheduler or event loop, a non-blocking Fiber-aware HTTP client, bounded concurrency, and explicit timeout, cancellation, error, and cleanup behavior.

**Explanation:** Start independent requests, suspend tasks while sockets are pending, then let the event loop resume each Fiber when data is ready. A synchronous HTTP client inside a Fiber still blocks the thread.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [138 - Memory use and performance optimization](../lessons/138-memory-use-and-performance-optimization.md) | [Lesson](../lessons/139-fibers-and-cooperative-concurrency.md) | [Example](../examples/139-fibers-and-cooperative-concurrency.md) | [140 - Background jobs, queues, and workers](../lessons/140-background-jobs-queues-and-workers.md) |