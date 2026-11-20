# 064 - Constructors and Destructors: Example

## Question

How can a constructor validate and initialize an object, and when does its destructor run if multiple variables refer to that object?

## Complete example

This standalone example works with PHP 8.0 and later because it uses constructor property promotion. Save it as `constructors-and-destructors.php` and run it from the command line.

~~~php
<?php

declare(strict_types=1);

class BaseJob
{
    protected string $jobId;

    public function __construct(string $jobId)
    {
        $this->jobId = $jobId;
    }
}

class ReportJob extends BaseJob
{
    private string $status = 'pending';

    public function __construct(
        string $jobId,
        private string $title,
    ) {
        // A child constructor must call the parent constructor explicitly
        // when the parent has required initialization.
        parent::__construct($jobId);

        if (trim($this->title) === '') {
            throw new InvalidArgumentException(
                'Report title cannot be empty.'
            );
        }

        $this->title = trim($this->title);
    }

    public function run(): void
    {
        $this->status = 'complete';
    }

    public function summary(): string
    {
        return sprintf(
            'Report: %s - %s [%s]',
            $this->jobId,
            $this->title,
            $this->status
        );
    }

    public function __destruct()
    {
        // Printed only to demonstrate object destruction.
        // Production destructors should not write user-facing output.
        echo 'Destroying report job ' . $this->jobId . PHP_EOL;
    }
}

$job = new ReportJob('job-42', ' Monthly report ');
$job->run();

echo $job->summary() . PHP_EOL;

$anotherReference = $job;

unset($job);
echo 'First reference removed.' . PHP_EOL;

unset($anotherReference);
echo 'Last reference removed.' . PHP_EOL;
~~~

## Expected output

~~~text
Report: job-42 - Monthly report [complete]
First reference removed.
Destroying report job job-42
Last reference removed.
~~~

The destructor runs when the last reference to this ordinary, non-cyclic object is removed. During script shutdown, destructor timing and ordering are not predictable enough to coordinate essential work.

## Explanation

### 1. Construction establishes object state

The `new ReportJob(...)` expression creates the object and calls its constructor. The constructor receives the job ID and title, calls the required parent constructor, validates the title, and stores a normalized value.

If the title is empty, the constructor throws `InvalidArgumentException`. The caller should catch that exception around the `new` expression if invalid construction is an expected outcome.

### 2. Constructor property promotion

The constructor parameter `private string $title` declares a private property and initializes it from the argument. This syntax is constructor property promotion, available in PHP 8.0 and later.

The constructor can still validate or change the promoted property's value after PHP assigns the incoming argument.

### 3. Parent initialization is explicit

`ReportJob` defines its own constructor, so PHP does not automatically run `BaseJob::__construct()`. The call to `parent::__construct($jobId)` initializes the inherited `jobId` property.

### 4. Removing one variable may not destroy an object

After `$anotherReference = $job`, both variables identify the same instance. Removing `$job` leaves another reference alive, so the destructor does not run yet.

Removing `$anotherReference` removes the last reference, so PHP destroys the object and calls `__destruct()`.

### 5. Destructor output is for demonstration only

The destructor prints a line so the example makes its timing visible. Do not use destructors to write required responses, commit transactions, or perform important work at a precise time.

Use explicit methods and exception handling for essential operations. Use `finally` when cleanup must occur as control leaves a block:

~~~php
$handle = fopen('report.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the report.');
}

try {
    processReport($handle);
} finally {
    fclose($handle);
}
~~~

## Try it yourself

1. Create a second `ReportJob` and compare its summary with the first.
2. Remove the call to `$job->run()` and observe the initial status.
3. Pass an empty title inside a `try/catch` and observe that construction throws.
4. Remove `parent::__construct($jobId)` and see why parent state is not initialized.
5. Add a third reference to the object, remove two references, and observe when the destructor line appears.

## Key takeaways

- A constructor runs during object creation and should establish valid initial state.
- Constructor property promotion creates and initializes a property from a constructor parameter.
- A child constructor must explicitly invoke a parent constructor when parent setup is needed.
- An object is not destroyed while references to it remain.
- A destructor is a fallback cleanup hook; avoid relying on exact shutdown timing.
- Use explicit cleanup and `finally` for work that must happen predictably.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| :--- | :--- | :--- | :--- |
| [063 - Properties, methods, and object creation](../lessons/063-properties-methods-and-object-creation.md) | [Q&A](../qnas/064-constructors-and-destructors.md) | [Lesson](../lessons/064-constructors-and-destructors.md) | [065 - Visibility and encapsulation](../lessons/065-visibility-and-encapsulation.md) |
