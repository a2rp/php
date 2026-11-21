# Magic methods - Example

## Goal

Build a small task object that demonstrates property overloading, method overloading, string conversion, callable objects, debug information, and custom serialization.

## Requirements

This example uses PHP 8.0 or later. The custom <code>__serialize()</code> and <code>__unserialize()</code> methods require PHP 7.4 or later; the example's <code>mixed</code> type declaration requires PHP 8.0.

## Complete PHP example

~~~php
<?php
declare(strict_types=1);

final class Task implements Stringable
{
    private array $data = [];

    public function __construct(string $title)
    {
        $this->__set('title', $title);
        $this->data['done'] = false;
    }

    public function __get(string $name): mixed
    {
        if (!array_key_exists($name, $this->data)) {
            throw new OutOfBoundsException(
                'Unknown task property: ' . $name
            );
        }

        return $this->data[$name];
    }

    public function __set(string $name, mixed $value): void
    {
        if ($name === 'title') {
            if (!is_string($value)) {
                throw new InvalidArgumentException(
                    'Task title must be a string.'
                );
            }

            $title = trim($value);

            if ($title === '') {
                throw new InvalidArgumentException(
                    'Task title cannot be empty.'
                );
            }

            $this->data['title'] = $title;
            return;
        }

        if ($name === 'done') {
            if (!is_bool($value)) {
                throw new InvalidArgumentException(
                    'Task status must be a boolean.'
                );
            }

            $this->data['done'] = $value;
            return;
        }

        if ($name === 'note') {
            if (!is_string($value)) {
                throw new InvalidArgumentException(
                    'Task note must be a string.'
                );
            }

            $this->data['note'] = trim($value);
            return;
        }

        throw new OutOfBoundsException(
            'Unsupported task property: ' . $name
        );
    }

    public function __isset(string $name): bool
    {
        return isset($this->data[$name]);
    }

    public function __unset(string $name): void
    {
        unset($this->data[$name]);
    }

    public function __call(string $name, array $arguments): mixed
    {
        if ($name === 'rename') {
            if (count($arguments) !== 1 || !is_string($arguments[0])) {
                throw new BadMethodCallException(
                    'rename() expects one string argument.'
                );
            }

            $this->__set('title', $arguments[0]);
            return $this;
        }

        if ($name === 'complete') {
            if ($arguments !== []) {
                throw new BadMethodCallException(
                    'complete() does not accept arguments.'
                );
            }

            $this->__set('done', true);
            return $this;
        }

        throw new BadMethodCallException(
            'Unknown task operation: ' . $name
        );
    }

    public function __toString(): string
    {
        $title = $this->data['title'] ?? 'Untitled';
        $status = ($this->data['done'] ?? false)
            ? 'done'
            : 'pending';

        return $title . ' [' . $status . ']';
    }

    public function __invoke(): string
    {
        $this->__call('complete', []);

        return (string) $this;
    }

    public function __debugInfo(): array
    {
        return [
            'title' => $this->data['title'] ?? 'Untitled',
            'done' => $this->data['done'] ?? false,
            'hasNote' => isset($this->data['note']),
        ];
    }

    public function __serialize(): array
    {
        return [
            'title' => $this->data['title'] ?? 'Untitled',
            'done' => $this->data['done'] ?? false,
        ];
    }

    public function __unserialize(array $data): void
    {
        if (!isset($data['title'])
            || !is_string($data['title'])
            || !array_key_exists('done', $data)
            || !is_bool($data['done'])) {
            throw new UnexpectedValueException(
                'Invalid serialized task data.'
            );
        }

        $title = trim($data['title']);

        if ($title === '') {
            throw new UnexpectedValueException(
                'Serialized task title cannot be empty.'
            );
        }

        $this->data = [
            'title' => $title,
            'done' => $data['done'],
        ];
    }
}

$task = new Task('Review PHP magic methods');

echo 'Initial task: ' . $task . PHP_EOL;
echo 'Has note initially: '
    . (isset($task->note) ? 'yes' : 'no')
    . PHP_EOL;

$task->note = 'Check property overloading';
echo 'Has note after setting: '
    . (isset($task->note) ? 'yes' : 'no')
    . PHP_EOL;

unset($task->note);
echo 'Has note after unsetting: '
    . (isset($task->note) ? 'yes' : 'no')
    . PHP_EOL;

$task->rename('  Review object hooks  ');
$task->complete();

echo 'Updated task: ' . $task . PHP_EOL;

$debug = $task->__debugInfo();
echo 'Debug title: ' . $debug['title'] . PHP_EOL;
echo 'Debug complete: ' . ($debug['done'] ? 'yes' : 'no') . PHP_EOL;

// var_dump($task); // PHP calls __debugInfo() for this output.

$serialized = serialize($task);
$restored = unserialize(
    $serialized,
    ['allowed_classes' => [Task::class]],
);

echo 'Restored task: ' . $restored . PHP_EOL;

$callableTask = new Task('Run as a callable');
echo 'Invoked task: ' . $callableTask() . PHP_EOL;
~~~

## Expected output

~~~text
Initial task: Review PHP magic methods [pending]
Has note initially: no
Has note after setting: yes
Has note after unsetting: no
Updated task: Review object hooks [done]
Debug title: Review object hooks
Debug complete: yes
Restored task: Review object hooks [done]
Invoked task: Run as a callable [done]
~~~

## Explanation

### Property overloading uses a private backing array

The class does not declare public <code>title</code>, <code>done</code>, or <code>note</code> properties. Reads and writes to these names call <code>__get()</code> and <code>__set()</code>.

The setter accepts only the names and types the class supports. Unknown property names throw an exception instead of being silently stored. <code>__isset()</code> and <code>__unset()</code> define the behavior for checks and removal.

### Method overloading supports only selected operations

The class has no declared <code>rename()</code> or <code>complete()</code> methods. Those calls are routed to <code>__call()</code>, which validates the method name and its arguments.

This is deliberately a small allowlist. Unsupported calls throw <code>BadMethodCallException</code>, making misspellings visible instead of silently succeeding.

### String conversion and invocation have clear results

<code>__toString()</code> creates a short label for display. <code>__invoke()</code> marks the task complete and returns the same string representation.

The class also implements <code>Stringable</code> explicitly to make its string-conversion contract visible to readers and type declarations.

### Debug output is selected deliberately

<code>__debugInfo()</code> returns only the fields intended for inspection. PHP calls it automatically when <code>var_dump($task)</code> is used.

In a real class, omit credentials, tokens, and other sensitive values from this returned array.

### Serialization includes only the intended state

<code>__serialize()</code> returns the title and completion status. The restoration method verifies the data types and title before rebuilding the backing array.

The example serializes and immediately restores an object created by the same program. Do not use <code>unserialize()</code> on untrusted input; it can instantiate classes and invoke object behavior. The <code>allowed_classes</code> option is not a substitute for trusting and validating the source.

## Try these changes

1. Call <code>$task->rename()</code> with no argument. Which exception is thrown?
2. Assign an empty title. Which validation rejects it?
3. Try <code>$task->unknown = 'value'</code>. Why does the object reject it?
4. Add an allowed <code>priority</code> property with an integer value.
5. Add a <code>__callStatic()</code> handler for one explicitly supported static operation.
6. Add another field to <code>__debugInfo()</code>. Which fields should remain private?
7. Change the serialized data shape and update <code>__unserialize()</code> validation to match.

## Key takeaways

- Property overloading handles undefined or inaccessible properties.
- Method overloading handles undefined or inaccessible methods.
- Validate names and arguments so overloading does not hide mistakes.
- <code>__toString()</code> and <code>__invoke()</code> give an object useful string and callable behavior.
- <code>__debugInfo()</code> controls what <code>var_dump()</code> reveals.
- Custom serialization should store only the intended state and validate it when restoring.
- Never unserialize untrusted input.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [075 - Property hooks](../lessons/075-property-hooks.md) | [Lesson](../lessons/076-magic-methods.md) | [Q&A](../qnas/076-magic-methods.md) | [077 - Cloning and comparing objects](../lessons/077-cloning-and-comparing-objects.md) |
