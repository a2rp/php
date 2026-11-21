# Background jobs, queues, and workers — Q&A

This chapter reviews asynchronous job design, queue delivery, retries, PHP worker operations, and common failure cases. Each answer follows its question so the reasoning is easy to check.

## Multiple-choice questions

### 1. What is the main purpose of moving slow work to a background job?

- A. To make the work finish before the request returns
- B. To let the request return while a separate worker completes the work
- C. To avoid validating the user's request
- D. To guarantee that the work can never fail

**Answer: B.**

**Explanation:** The request can record the accepted work and enqueue it, then return a job identifier. A worker processes it later. Queuing does not make failures impossible.

### 2. When should a worker acknowledge a message?

- A. Before calling the job handler
- B. As soon as the worker has received the message
- C. After the operation has succeeded and required database changes have committed
- D. Only after the user closes the page

**Answer: C.**

**Explanation:** Acknowledging too early can lose work if the worker crashes before completing it. If the worker completes the operation and crashes before the acknowledgment, the message may be delivered again, so the handler must also be duplicate-safe.

### 3. What does at-least-once delivery mean?

- A. The job handler runs exactly once
- B. The queue may deliver a message again if completion is uncertain
- C. The queue never retries failed messages
- D. Every message is processed in strict order by one worker

**Answer: B.**

**Explanation:** A worker may finish an external side effect and crash before acknowledging. The queue cannot know whether the work happened, so it may deliver the message again. The handler must tolerate duplicate delivery.

### 4. Which design best helps prevent a receipt from being sent twice after redelivery?

- A. Give each delivery a new random job identifier
- B. Use a stable idempotency key for the logical receipt and enforce it in the processing design
- C. Increase the worker count
- D. Acknowledge every message before processing

**Answer: B.**

**Explanation:** Redeliveries may have different delivery identifiers but refer to the same logical action. A stable key, often backed by a database uniqueness rule or a provider's idempotency support, lets the system recognize that action.

### 5. What is an appropriate response when a temporary downstream service outage causes a job to fail?

- A. Retry forever without a delay
- B. Retry a bounded number of times with backoff, then isolate the exhausted job for review
- C. Delete the message immediately
- D. Keep the web request open until the service returns

**Answer: B.**

**Explanation:** Bounded retries with backoff avoid tight loops and reduce pressure on a recovering service. A dead-letter queue or failed state makes exhausted jobs visible for investigation.

### 6. What problem does the transactional outbox pattern address?

- A. It makes PHP code execute on multiple CPU cores
- B. It keeps a database change and the intent to publish a message in the same database transaction
- C. It guarantees an external email provider never fails
- D. It removes the need for idempotent consumers

**Answer: B.**

**Explanation:** Writing the business update and an outbox row atomically prevents the application from committing one while losing the other. A separate publisher delivers the outbox event; publication may repeat, so consumers still need idempotency.

### 7. Which signal is often more useful than queue size alone when deciding whether users are experiencing a delay?

- A. Age of the oldest pending job
- B. Number of PHP files in the repository
- C. Number of browser tabs open
- D. The total number of completed jobs since launch

**Answer: A.**

**Explanation:** A queue can contain many quick jobs and still be healthy. A small number of old jobs can indicate a stuck worker or a blocked dependency and directly show how long work is waiting.

## Short-answer questions

### 8. What are the producer, queue, and worker?

**Answer:** The producer creates a job message, the queue stores or coordinates pending messages, and the worker claims a message and runs its handler.

**Explanation:** A producer can be a web request, command, or another service. The queue separates job creation from execution; the worker performs the slower operation independently.

### 9. Why should a queue message usually contain an identifier instead of a full copy of the record?

**Answer:** A small identifier keeps messages compact and lets the worker load current data at execution time.

**Explanation:** A copied record can become stale and may expose more sensitive data than necessary. Large files should live in controlled storage, with only a suitable reference carried by the message.

### 10. What does idempotent processing mean?

**Answer:** Repeating the same logical operation produces no additional harmful or externally visible effect.

**Explanation:** Queue delivery can repeat after a crash or timeout. A stable idempotency key, unique database constraint, conditional state transition, or downstream API idempotency feature can help make a handler safe to retry.

### 11. Why is checking a local “processed” flag and then calling an unrelated external service not always enough for idempotency?

**Answer:** The process can crash between the check or flag update and the external call, leaving the two systems inconsistent.

**Explanation:** A database transaction cannot usually include an email provider or remote API. Prefer provider-supported idempotency keys, a design that can reconcile uncertain outcomes, or a safe duplicate-handling strategy.

### 12. How should temporary and permanent failures be treated differently?

**Answer:** Retry temporary failures that may recover, using a bounded policy and delay. Send permanent or repeatedly failing jobs to a visible failed or dead-letter state.

**Explanation:** Retrying invalid input or a programming bug does not usually fix it. Retrying a network timeout may succeed later. Classifying errors prevents infinite retry loops and makes failures actionable.

### 13. Why should a queue consumer validate a message even if the producer already validated it?

**Answer:** Messages can be delayed, replayed, or published by another producer, and the application may have changed since enqueue time.

**Explanation:** The worker should check the message shape, supported job type and version, identifier, permissions, and current business state before performing an action.

### 14. Why is a PHP CLI worker normally run separately from PHP-FPM request handling?

**Answer:** A worker is a long-running process that continuously waits for and processes jobs; PHP-FPM is designed to handle web requests.

**Explanation:** Keeping worker lifecycle and capacity separate allows independent restarts, scaling, resource limits, and graceful deployment without tying queue work to an individual HTTP request.

### 15. What is the difference between a queue and a scheduler?

**Answer:** A queue holds asynchronous work to be processed by workers. A scheduler starts work at defined times or intervals.

**Explanation:** A scheduled task can enqueue many jobs when a periodic workload is large. The scheduler decides when work should begin; the queue distributes and tracks asynchronous execution.

### 16. What should a graceful worker shutdown do?

**Answer:** Stop claiming new jobs, finish the active job within the shutdown grace period or safely release it, and then exit.

**Explanation:** Deployments and process managers terminate workers. Jobs interrupted after the grace period must be safe to retry. Signal handling depends on the operating system and PHP build.

## Scenarios and code reasoning

### 17. A worker updates an order in the database, commits, and then crashes before acknowledging the message. What may happen, and how should the job be designed?

**Answer:** The queue may deliver the message again. The handler should recognize that the order transition already happened and avoid repeating the business effect.

**Explanation:** The database commit and queue acknowledgment are separate operations. A conditional update, unique operation key, or transactionally recorded completion state can make redelivery safe.

### 18. A worker calls a payment API successfully, but its network connection times out before it receives the response. Should it issue the payment again with a new request key?

**Answer:** No. It should reuse a stable idempotency key if the payment provider supports one, then check or reconcile the operation's status.

**Explanation:** A timeout does not prove the provider failed to process the request. A new key can create a second charge. The application must handle an uncertain outcome explicitly.

### 19. A queue is growing while workers are healthy. The downstream database is already close to its connection limit. Should the team immediately add more workers?

**Answer:** No. First check queue age, processing duration, database capacity, and job behavior. Adding workers may overload the database further.

**Explanation:** Worker count is not the only limit. Concurrency should be bounded by downstream capacity; backpressure or a slower intake rate may be safer during an outage.

### 20. A job with malformed input fails on every attempt. What should happen?

**Answer:** Classify it as a permanent failure and move it to a failed or dead-letter state with enough safe diagnostic information to investigate.

**Explanation:** Repeating a permanently invalid message wastes resources and can obscure the real failure. Fix or reject the data before any controlled replay.

### 21. Why can immediate requeueing create a retry storm?

**Answer:** A failing job can be retried continuously, consuming worker capacity and repeatedly hitting an unhealthy dependency.

**Explanation:** Bounded attempts, exponential backoff, and jitter give the dependency time to recover and spread retries over time.

### 22. A job runs for longer than the queue's visibility timeout. What risk does this create?

**Answer:** The queue may make the message available to another worker while the original worker is still processing it.

**Explanation:** Two workers may then execute the same logical job concurrently. Configure an adequate timeout or extend the lease using the queue's supported mechanism, and still keep the handler idempotent.

### 23. A user receives HTTP 202 after starting a report, then immediately asks whether the report is ready. What should the application provide?

**Answer:** Return a job identifier and provide a status endpoint or page that reports queued, processing, completed, or failed state.

**Explanation:** HTTP 202 means the request was accepted for processing, not that the report has completed. The status interface can provide a result link or a useful failure state when available.

### 24. A request saves an order successfully, but publishing its job message fails. What pattern can prevent this gap?

**Answer:** Write an outbox row in the same database transaction as the order, then publish unsent outbox rows from a separate process.

**Explanation:** The order and the durable intent to publish commit together. The publisher can retry publication; consumers should still tolerate duplicate messages.

## Practice exercise

### 25. Design a background job for generating an invoice PDF. Name the message fields, at least two idempotency protections, a retry rule, and two useful metrics.

**Answer:**

- Message fields: job type and version, invoice ID, and a stable key such as **invoice-pdf:invoice-8472**.
- Idempotency: enforce one generated artifact per invoice version, and write the artifact to a deterministic storage key or use a conditional database state transition.
- Retry rule: retry temporary storage or database errors a bounded number of times with exponential backoff and jitter; send invalid invoice data or exhausted attempts to a visible failed state.
- Metrics: age of the oldest queued job and job processing duration. Queue depth, retry count, failure count, and dead-letter count are also useful.

**Explanation:** The worker should load and validate the current invoice, generate the PDF, store it safely, commit the result, and only then acknowledge the job. If it crashes before acknowledgment, the idempotency protections should prevent duplicate artifacts or conflicting invoice state.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [139 - Fibers and cooperative concurrency](../lessons/139-fibers-and-cooperative-concurrency.md) | [Lesson](../lessons/140-background-jobs-queues-and-workers.md) | [Example](../examples/140-background-jobs-queues-and-workers.md) | [141 - Deployment environments and release workflows](../lessons/141-deployment-environments-and-release-workflows.md) |