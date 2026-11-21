# Background jobs, queues, and workers — Example

This example models a small PHP CLI worker. It demonstrates a temporary failure, a retry, duplicate delivery for the same logical action, acknowledgment after handling, and a permanently invalid job sent to a dead-letter list.

The queue and idempotency store below are in memory. They exist only to make the processing flow easy to see in one file. They disappear when the process exits, do not support multiple workers, and do not provide real retry delays or durable acknowledgments. Use a maintained framework component or queue client with a durable broker or job store in a real application.

## Question

A web application needs to send an order receipt in the background. How can a worker process the job, retry a temporary failure, and avoid sending the same receipt twice if the logical job is delivered more than once?

## Complete PHP example

Save this as **background-jobs.php**:

~~~php
<?php

declare(strict_types=1);

final class TemporaryJobFailure extends RuntimeException
{
}

final class PermanentJobFailure extends RuntimeException
{
}

/**
 * Simulate the job handler.
 *
 * The completed-actions array stands in for persistent idempotency state.
 * The fail-first flag simulates one temporary mail-service failure.
 */
function handleJob(
    array $job,
    array &$completedActions,
    bool &$failFirstDelivery
): string {
    if ($job['type'] !== 'send_receipt') {
        throw new PermanentJobFailure(
            'Unsupported job type: ' . $job['type']
        );
    }

    $key = $job['idempotency_key'];

    if (isset($completedActions[$key])) {
        return 'duplicate';
    }

    if ($job['delivery_id'] === 'delivery-a' && $failFirstDelivery) {
        $failFirstDelivery = false;

        throw new TemporaryJobFailure(
            'Mail service is temporarily unavailable'
        );
    }

    // In this demo, recording the key represents one successful logical
    // receipt operation. A real system needs durable idempotency support.
    $completedActions[$key] = true;

    return 'processed';
}

$queue = [
    [
        'delivery_id' => 'delivery-a',
        'type' => 'send_receipt',
        'order_id' => 'order-8472',
        'idempotency_key' => 'receipt:order-8472',
        'attempt' => 0,
        'max_attempts' => 3,
    ],
    [
        // A second delivery for the same logical receipt.
        'delivery_id' => 'delivery-b',
        'type' => 'send_receipt',
        'order_id' => 'order-8472',
        'idempotency_key' => 'receipt:order-8472',
        'attempt' => 0,
        'max_attempts' => 3,
    ],
    [
        'delivery_id' => 'delivery-bad',
        'type' => 'unknown_task',
        'order_id' => 'order-9999',
        'idempotency_key' => 'unknown:order-9999',
        'attempt' => 0,
        'max_attempts' => 2,
    ],
];

$completedActions = [];
$deadLetters = [];
$failFirstDelivery = true;

while ($queue !== []) {
    $job = array_shift($queue);
    $job['attempt']++;

    echo 'Received ' . $job['delivery_id']
        . ' (attempt ' . $job['attempt']
        . '/' . $job['max_attempts'] . ')' . PHP_EOL;

    try {
        $result = handleJob($job, $completedActions, $failFirstDelivery);

        if ($result === 'duplicate') {
            echo 'Skipped duplicate receipt for ' . $job['order_id'] . PHP_EOL;
        } else {
            echo 'Sent receipt for ' . $job['order_id'] . PHP_EOL;
        }

        // Acknowledge only after the handler reports success.
        echo 'Acknowledged ' . $job['delivery_id'] . PHP_EOL;
    } catch (TemporaryJobFailure $failure) {
        echo 'Temporary failure: ' . $failure->getMessage() . PHP_EOL;

        if ($job['attempt'] < $job['max_attempts']) {
            $nextAttempt = $job['attempt'] + 1;

            echo 'Retry scheduled for ' . $job['delivery_id']
                . ' (next attempt ' . $nextAttempt
                . '/' . $job['max_attempts'] . ')' . PHP_EOL;

            // Immediate requeue keeps this local demonstration short.
            // A real queue should schedule a delayed retry with backoff.
            $queue[] = $job;
        } else {
            $deadLetters[] = $job;
            echo 'Moved ' . $job['delivery_id']
                . ' to dead-letter list after retries' . PHP_EOL;
        }
    } catch (PermanentJobFailure $failure) {
        // Invalid or unsupported work should not be retried unchanged.
        $deadLetters[] = $job;

        echo 'Permanent failure: ' . $failure->getMessage() . PHP_EOL;
        echo 'Moved ' . $job['delivery_id']
            . ' to dead-letter list' . PHP_EOL;
    }
}

echo 'Completed logical actions: ' . count($completedActions) . PHP_EOL;
echo 'Dead-letter messages: ' . count($deadLetters) . PHP_EOL;
~~~

## Run the example

Run it with PHP CLI:

~~~text
php background-jobs.php
~~~

## Expected output

~~~text
Received delivery-a (attempt 1/3)
Temporary failure: Mail service is temporarily unavailable
Retry scheduled for delivery-a (next attempt 2/3)
Received delivery-b (attempt 1/3)
Sent receipt for order-8472
Acknowledged delivery-b
Received delivery-bad (attempt 1/2)
Permanent failure: Unsupported job type: unknown_task
Moved delivery-bad to dead-letter list
Received delivery-a (attempt 2/3)
Skipped duplicate receipt for order-8472
Acknowledged delivery-a
Completed logical actions: 1
Dead-letter messages: 1
~~~

## How it works

1. The first delivery fails before performing the simulated receipt operation. The worker does not acknowledge it; it schedules another attempt.
2. The second delivery represents the same logical receipt. It succeeds and is acknowledged.
3. The malformed job has an unsupported type. It is treated as permanent and moved aside instead of being retried unchanged.
4. The retry of the first delivery finds the same idempotency key already completed. It skips the duplicate action and is acknowledged.
5. The final count shows one completed logical action even though the receipt had two delivery records.

## Applying this pattern to a real queue

Replace the local queue array with the queue library's receive or reserve operation. Use the broker's acknowledgment, lease, or visibility-timeout API. If processing succeeds, commit any required application database transaction and then acknowledge. If the worker exits before acknowledgment, the queue should make the message available again according to its documented behavior.

Replace the completed-actions array with durable idempotency state. For database-only work, a unique constraint or conditional transaction can protect the logical operation. For an external provider, use its idempotency-key support when available and reconcile uncertain outcomes. An in-memory array cannot protect against process restarts or concurrent workers.

The demo requeues a temporary failure immediately. Production retries should have a maximum attempt count, exponential backoff with jitter, and a dead-letter destination for exhausted jobs. Configure worker concurrency and job timeouts for the limits of the database and external services.

A production worker should also validate the message, enforce permissions, set network timeouts, log a correlation or job ID without logging private payload data, respond to shutdown signals where supported, and run under a process supervisor or container platform.

## What this example does not provide

- durable storage or recovery after a process crash;
- actual email delivery or a transaction with an external provider;
- parallel workers, leases, or a visibility timeout;
- delayed retry scheduling, backoff, or jitter;
- persistent dead-letter storage or an operator replay interface.

Those behaviors depend on the queue system and application architecture. The example focuses on the worker's decision flow and the need for idempotency.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [139 - Fibers and cooperative concurrency](../lessons/139-fibers-and-cooperative-concurrency.md) | [Lesson](../lessons/140-background-jobs-queues-and-workers.md) | [Q&A](../qnas/140-background-jobs-queues-and-workers.md) | [141 - Deployment environments and release workflows](../lessons/141-deployment-environments-and-release-workflows.md) |