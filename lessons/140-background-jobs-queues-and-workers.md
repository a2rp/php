# Background jobs, queues, and workers

A background job is work that an application accepts now and completes later, outside the request that started it. A queue stores or coordinates pending jobs, and a worker is a process that takes jobs from the queue and runs their handlers.

This pattern keeps slow or unreliable tasks from holding a web request open. It also introduces new responsibilities: jobs may run more than once, fail after partial work, wait in a backlog, or be interrupted during deployment. A reliable design plans for those cases.

## Learning objectives

After this lesson, you should be able to:

- distinguish request work, queued jobs, and scheduled tasks;
- describe how a producer, queue, and worker fit together;
- design small, validated job messages;
- explain acknowledgments, retries, dead-letter handling, and duplicate delivery;
- make handlers idempotent and safe to retry;
- monitor queue health and shut down workers cleanly;
- recognize when a framework or queue service should manage the implementation.

## When to use a background job

A web request should usually finish after the application has completed the work needed to answer the user. Some tasks take longer or depend on a remote service that may be slow. Examples include:

- sending an email or notification;
- generating a report or thumbnail;
- importing a large file;
- processing a video or image;
- calling a partner API;
- delivering a webhook;
- recalculating data across many records.

The request can validate and record the user's intent, enqueue a job, and return a job identifier. A separate worker does the slower work. The user can then see a status such as queued, processing, completed, or failed.

A response such as HTTP 202 Accepted can indicate that the request was accepted for later processing. It does not mean the job has finished. Provide a way to check the result when the user needs it.

Not every task belongs in a queue. Keep small, fast, required work in the request when the user needs its result immediately. Use a scheduler or cron-like system to start predictable work at a time or interval; that scheduled command can enqueue jobs if the work is large. A queue is useful when work should be processed asynchronously, buffered during bursts, or distributed across workers.

## The producer, queue, and worker

A common flow has three parts:

1. **Producer:** the web request, command, or another service validates the request and creates a job message.
2. **Queue:** a broker or durable job store holds pending messages and coordinates which worker may process them.
3. **Worker:** a separate process claims a message, runs the handler, and reports success or failure to the queue.

The worker should acknowledge or mark a job complete only after its work has succeeded. If it acknowledges first and then crashes, the queue may believe the work is done even though it was not. If the worker performs the work and crashes before acknowledging, the queue may deliver the job again. That is why duplicate-safe handlers matter.

A simplified lifecycle looks like this:

~~~text
request or command
    -> validate and save the business operation
    -> enqueue a small job message
    -> return an identifier or result

worker
    -> claim a message
    -> validate the message again
    -> perform the operation
    -> acknowledge success
    -> or retry a temporary failure
    -> or move a permanent/exhausted failure aside
~~~

Queue APIs differ. They may use acknowledgments, leases, visibility timeouts, reservations, or database status columns to represent a claim. Learn the chosen system's exact guarantees rather than assuming every queue behaves the same way.

## Keep messages small and stable

A job message should contain enough information to find and perform the work, not a copy of an entire database or uploaded file. A message might contain a job type, a record identifier, a schema version, and an idempotency key.

~~~json
{
  "type": "send_receipt",
  "version": 1,
  "order_id": "order-8472",
  "idempotency_key": "receipt:order-8472"
}
~~~

The worker should load the current record and verify its state before acting. This avoids processing stale copies of mutable data. Store large files in file or object storage and put a controlled reference in the message.

Treat queue content as untrusted input. Check the message shape, supported type and version, identifier format, authorization assumptions, and current business state. Do not put passwords, access tokens, or unnecessary personal data into messages. Protect queue access and avoid logging sensitive payloads.

## Delivery is commonly at least once

Many queue systems provide **at-least-once delivery**: a message should not be lost merely because a worker disappears, but it can be delivered more than once. For example, a worker may send a receipt and crash before acknowledging the message. The queue cannot know whether the email was sent, so it may retry.

Design the handler to be **idempotent**: processing the same logical job again should not repeat an externally visible effect or corrupt state. Common techniques include:

- use a stable idempotency key for the logical action;
- enforce uniqueness with a database constraint;
- record job completion in the same transaction as the database changes it protects;
- make state transitions conditional, such as updating an order only while it is still pending;
- pass an idempotency key to an external API when that API supports it.

A local “already processed” check followed by an unrelated external side effect is not automatically safe. The process can crash between those steps. For a payment, email, or remote API call, use provider idempotency support where available, or design reconciliation and duplicate handling explicitly.

Do not promise exactly-once execution unless the complete system can actually guarantee it. A queue acknowledgment and a database commit are usually separate operations. A worker can crash between them.

## Retries and permanent failures

Failures have different causes:

- A temporary network timeout or rate limit may succeed later.
- Invalid input, a missing required record, or an unsupported job type is usually permanent until the data or code changes.
- A bug may make every retry fail in the same way.

Retry only failures that may recover. Use a bounded number of attempts and a delay between attempts. Exponential backoff with jitter spreads retries out and avoids sending a whole backlog to a recovering service at once. Do not immediately requeue a failing job in a tight loop.

After retries are exhausted, move the message to a dead-letter queue or a visible failed state. Preserve enough context for investigation while protecting private data. Provide a safe way to inspect, correct, and replay a job. Fix the cause before replaying; otherwise the same message may fail repeatedly.

A dead-letter queue is not a substitute for monitoring. Alert when jobs start accumulating there or when the oldest pending job is too old.

## Database changes and queue publication

A common reliability problem happens when an application writes to its database and publishes a queue message as two separate operations:

1. the database transaction commits, then publishing fails; or
2. the message is published, then the database transaction rolls back.

The **transactional outbox** pattern addresses this gap. The request writes the business change and an outbox event in one database transaction. A separate publisher reads unsent outbox rows, publishes them, and records successful publication. Publishing may happen more than once, so the consumer still needs idempotency.

When a worker changes application data, commit the database transaction before acknowledging the queue message. If the worker crashes after the commit but before the acknowledgment, it may receive the message again; the handler's idempotency checks must recognize the already-completed change.

A broker's durability, publisher confirms, replicated storage, and backup options are implementation-specific. Configure and understand the guarantees your selected system provides. “Durable” does not remove the need for backups, recovery planning, and idempotent consumers.

## Worker processes in PHP

A worker is generally a PHP CLI process, separate from PHP-FPM web request processes. It can run a loop that waits for a job, calls a handler, and then acknowledges or reports failure through the queue library.

~~~text
php bin/worker.php
~~~

Use a framework queue component or maintained client library for the chosen broker. PHP itself does not provide one universal queue API. A production worker needs operational management outside its script: a process supervisor, container platform, or job runner should restart a process that exits unexpectedly and provide logs and deployment controls.

Workers should:

- use explicit timeouts for network and database calls;
- limit memory, execution time, concurrency, and batch size;
- avoid busy-spinning when the queue is empty;
- catch and classify handler failures at the job boundary;
- log a job identifier and outcome without exposing secrets;
- release resources and database transactions in a cleanup block when appropriate;
- stop accepting new work during shutdown and finish or safely release the current job.

On systems where signal handling is available, a worker can handle a termination signal such as SIGTERM and begin graceful shutdown. Signal support depends on the PHP build and operating system. Do not assume a process can finish indefinitely: configure a shutdown grace period, and ensure that a job interrupted after that period can be retried safely.

If the queue uses a lease or visibility timeout, make sure processing can finish before the lease expires, or extend it using the queue's supported mechanism. A lease that expires while the original worker is still running can allow a second worker to process the same message concurrently.

## Scaling and backpressure

More workers can reduce a backlog, but they also add database connections, memory use, and load on downstream services. Scale with the capacity of the slowest dependency in mind.

Monitor queue depth and age, job throughput, processing duration, retry rate, failure count, dead-letter count, and worker health. Queue age often tells you more about user impact than raw queue size: a large queue of quick jobs may be healthy, while a small queue of very old jobs may indicate a stuck worker.

Use concurrency limits and backpressure. If an email provider or database is rate-limited, increasing workers can make the outage worse. Separate job types into queues when they have different priority, resource, or retry needs. Ordering, priority, and fairness are queue-specific; parallel workers can complete messages in a different order from the order they were enqueued.

## Security and operations

Only trusted, authorized producers should be able to publish jobs. Workers should have only the database, storage, and service permissions their job types require. Validate message data even if the producer already validated it, because messages can be delayed, replayed, or created by another producer.

Make jobs observable without logging complete payloads. Include a correlation or job ID in logs and traces so the original request, queue wait, attempts, and final outcome can be connected. Retain failed messages according to a policy, and ensure replay tools cannot accidentally repeat irreversible operations.

A job may run after the user, record, or configuration has changed. Check current state at execution time and define what should happen if the task is no longer valid. For jobs that take a long time, consider checkpoints or splitting work into smaller jobs so one failure does not repeat a large amount of completed work.

## Common mistakes

- Performing long work in a web request and calling it “background” without a separate worker.
- Acknowledging before the operation has committed.
- Assuming a message can never be delivered twice.
- Retrying every exception forever or immediately.
- Putting secrets, whole records, or large files in the message.
- Starting workers without supervision, health checks, or graceful shutdown.
- Increasing concurrency without checking database and service capacity.
- Treating a successful enqueue response as proof that the job completed.
- Replaying a failed payment, email, or other irreversible operation without an idempotency plan.

## Key takeaways

- A producer enqueues a small job; a separate worker processes it.
- Return a job identifier when work continues after the request, and expose useful status when needed.
- Acknowledge only after successful work, and expect a retry if a worker fails before acknowledgment.
- Make handlers idempotent because duplicate delivery is a normal failure case.
- Use bounded retries with backoff; isolate permanent or exhausted failures for review.
- Use an outbox when a database change and message publication must stay consistent.
- Run PHP workers as managed CLI processes, with limits, observability, and graceful shutdown.
- Scale against queue age and downstream capacity, and follow the guarantees of the actual queue implementation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [139 - Fibers and cooperative concurrency](139-fibers-and-cooperative-concurrency.md) | [Q&A](../qnas/140-background-jobs-queues-and-workers.md) | [Example](../examples/140-background-jobs-queues-and-workers.md) | [141 - Deployment environments and release workflows](141-deployment-environments-and-release-workflows.md) |