# Composition and object collaboration - Example

## Goal

Build a small report workflow in which each object has one focused responsibility:

- <code>TextFormatter</code> formats report text.
- <code>Report</code> builds report content and delegates formatting.
- <code>MemoryPublisher</code> stores the finished content for this example.
- <code>ReportWorkflow</code> coordinates the report and publisher.

The report **uses** a formatter. It is not a kind of formatter, so this is a composition relationship.

## Complete PHP example

This example uses constructor property promotion and requires PHP 8.0 or later.

~~~php
<?php
declare(strict_types=1);

class TextFormatter
{
    public function formatTitle(string $title): string
    {
        return strtoupper(trim($title));
    }

    public function formatItem(string $item): string
    {
        return trim($item);
    }
}

class Report
{
    public function __construct(
        private TextFormatter $formatter
    ) {
    }

    /**
     * @param array<int, string> $items
     */
    public function render(string $title, array $items): string
    {
        if (trim($title) === '') {
            throw new InvalidArgumentException('The report title cannot be empty.');
        }

        $lines = [$this->formatter->formatTitle($title)];

        foreach ($items as $item) {
            if (!is_string($item)) {
                throw new InvalidArgumentException('Every report item must be a string.');
            }

            $lines[] = '- ' . $this->formatter->formatItem($item);
        }

        return implode(PHP_EOL, $lines);
    }
}

class MemoryPublisher
{
    private string $content = '';

    public function publish(string $content): void
    {
        $this->content = $content;
    }

    public function getContent(): string
    {
        return $this->content;
    }
}

class ReportWorkflow
{
    public function __construct(
        private Report $report,
        private MemoryPublisher $publisher
    ) {
    }

    /**
     * @param array<int, string> $items
     */
    public function deliver(string $title, array $items): void
    {
        $content = $this->report->render($title, $items);
        $this->publisher->publish($content);
    }
}

$formatter = new TextFormatter();
$report = new Report($formatter);
$publisher = new MemoryPublisher();
$workflow = new ReportWorkflow($report, $publisher);

$workflow->deliver(
    ' monthly results ',
    [
        'Revenue: 1200',
        'New customers: 18',
    ]
);

echo $publisher->getContent();
~~~

## Expected output

~~~text
MONTHLY RESULTS
- Revenue: 1200
- New customers: 18
~~~

## How the objects collaborate

The caller at the bottom of the file creates and connects the objects. That code is the composition root:

~~~text
ReportWorkflow
  |-- Report
  |     |-- TextFormatter
  |
  |-- MemoryPublisher
~~~

When <code>deliver()</code> runs:

1. <code>ReportWorkflow</code> asks <code>Report</code> to render the title and items.
2. <code>Report</code> asks <code>TextFormatter</code> to format the title and each item.
3. <code>ReportWorkflow</code> gives the finished string to <code>MemoryPublisher</code>.
4. The example reads the stored content and prints it.

## Why use these separate objects?

- <code>TextFormatter</code> owns formatting rules.
- <code>Report</code> owns the report's structure and validates its input.
- <code>MemoryPublisher</code> owns storage for this demonstration.
- <code>ReportWorkflow</code> coordinates the steps without implementing each task itself.

The publisher stores the result in memory; it does not write a file or send anything over a network. In a larger application, publishing can be replaced with a different collaborator that follows the same expected behavior.

## Try changing the collaborator

A different formatter can specialize the formatting behavior:

~~~php
class UppercaseItemFormatter extends TextFormatter
{
    public function formatItem(string $item): string
    {
        return strtoupper(parent::formatItem($item));
    }
}

$report = new Report(new UppercaseItemFormatter());
~~~

The report can receive this object because it extends <code>TextFormatter</code>. The caller selects the implementation while assembling the objects; <code>Report</code> does not need to know how that formatter is implemented.

A later lesson introduces interfaces, which allow collaborators to share a contract without sharing a parent class.

## What to notice

- Required collaborators arrive through constructors.
- Typed properties show which collaborator each object expects.
- Each client calls a public method instead of changing another object's private state.
- <code>Report</code> delegates formatting rather than extending the formatter.
- The caller controls how the object graph is assembled.
- The objects can be changed independently as long as their collaboration stays compatible.

## Practice

1. Add a <code>formatItem()</code> rule that surrounds every item with brackets.
2. Add a validation rule that rejects an empty item.
3. Create a second publisher that prints a heading before the report content.
4. Draw the object graph after adding that publisher.
5. Decide whether a <code>TextFormatter</code> should be shared by two reports. Explain what could happen if it stores mutable state.

## Key takeaways

- Composition models collaboration between objects.
- Constructor injection makes dependencies explicit and replaceable.
- Delegation lets each class focus on its own responsibility.
- A composition root creates the collaborators and connects them.
- Replacing a collaborator should preserve the behavior the client expects.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [067 - Inheritance and method overriding](../lessons/067-inheritance-and-method-overriding.md) | [Lesson](../lessons/068-composition-and-object-collaboration.md) | [Q&A](../qnas/068-composition-and-object-collaboration.md) | [069 - Abstract classes and final declarations](../lessons/069-abstract-classes-and-final-declarations.md) |
