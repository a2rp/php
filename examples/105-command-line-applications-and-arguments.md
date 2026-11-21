# 105 - Command-line applications and arguments: Example

## Goal

Build a PHP CLI command that reads a JSON task list, validates its options and input, prints a table or JSON, and returns useful exit statuses.

## Question

How can a command accept an optional input file, --format and --limit options, keep diagnostics out of the result, and reject invalid input instead of silently continuing?

## Requirements

- PHP 8.0 or newer.
- Run the command from the directory containing the JSON file, or pass a path to another file.
- Provide one JSON array containing task objects with a nonempty title and a Boolean done value.
- Support --format=table or --format=json.
- Support --limit=N, where N is from 1 through 1000.
- Accept at most one input filename after --.
- Send help and results to STDOUT.
- Send errors to STDERR.
- Return 0 on success, 1 for an input or runtime failure, and 2 for invalid command usage.

## Project layout

~~~text
task-tool/
├── task-report.php
└── tasks.json
~~~

## Sample input

Create tasks.json:

~~~json
[
  {
    "title": "Review database schema",
    "done": true
  },
  {
    "title": "Write migration",
    "done": false
  },
  {
    "title": "Update deployment notes",
    "done": false
  }
]
~~~

## Complete command

Create task-report.php:

~~~php
<?php

declare(strict_types=1);

function printUsage($stream): void
{
    $lines = [
        'Usage: php task-report.php [options] [--] [file]',
        '',
        'Options:',
        '  --format=table|json   Select the output format. Default: table.',
        '  --limit=N             Show at most N tasks. Default: 100.',
        '  --help                Display this help message.',
        '',
        'Exit status:',
        '  0  Command completed.',
        '  1  Input or runtime failure.',
        '  2  Invalid command usage.',
        '',
        'When supplied, the input file must follow --.',
    ];

    fwrite($stream, implode(PHP_EOL, $lines) . PHP_EOL);
}

function parseArguments(array $argv): array
{
    $options = [
        'format' => 'table',
        'limit' => 100,
        'help' => false,
        'file' => 'tasks.json',
    ];

    $seen = [];
    $positionalArguments = [];
    $argumentCount = count($argv);

    for ($index = 1; $index < $argumentCount; $index++) {
        $token = $argv[$index];

        if ($token === '--') {
            $positionalArguments = array_slice($argv, $index + 1);
            break;
        }

        if (!str_starts_with($token, '--')) {
            throw new InvalidArgumentException(
                'Positional arguments must follow --.'
            );
        }

        $optionText = substr($token, 2);
        $equalsPosition = strpos($optionText, '=');

        if ($equalsPosition === false) {
            $name = $optionText;
            $value = null;
        } else {
            $name = substr($optionText, 0, $equalsPosition);
            $value = substr($optionText, $equalsPosition + 1);
        }

        if ($name === 'help') {
            if ($value !== null || isset($seen[$name])) {
                throw new InvalidArgumentException(
                    '--help does not accept a value and must not be repeated.'
                );
            }

            $options['help'] = true;
            $seen[$name] = true;
            continue;
        }

        if ($name !== 'format' && $name !== 'limit') {
            throw new InvalidArgumentException('Unknown option.');
        }

        if (isset($seen[$name])) {
            throw new InvalidArgumentException('An option was repeated.');
        }

        if ($value === null) {
            $index++;

            if (
                !isset($argv[$index])
                || str_starts_with($argv[$index], '--')
            ) {
                throw new InvalidArgumentException(
                    'An option is missing its value.'
                );
            }

            $value = $argv[$index];
        }

        if ($value === '') {
            throw new InvalidArgumentException(
                'An option value must not be empty.'
            );
        }

        $options[$name] = $value;
        $seen[$name] = true;
    }

    if (count($positionalArguments) > 1) {
        throw new InvalidArgumentException(
            'Provide no more than one input file.'
        );
    }

    if ($positionalArguments !== []) {
        $options['file'] = $positionalArguments[0];
    }

    if (!in_array($options['format'], ['table', 'json'], true)) {
        throw new InvalidArgumentException(
            '--format must be table or json.'
        );
    }

    $limit = filter_var($options['limit'], FILTER_VALIDATE_INT);

    if ($limit === false || $limit < 1 || $limit > 1000) {
        throw new InvalidArgumentException(
            '--limit must be an integer from 1 through 1000.'
        );
    }

    $options['limit'] = $limit;

    return $options;
}

function loadTasks(string $path): array
{
    $contents = @file_get_contents($path);

    if ($contents === false) {
        throw new RuntimeException('Could not read the input file.');
    }

    try {
        $decoded = json_decode(
            $contents,
            false,
            512,
            JSON_THROW_ON_ERROR
        );
    } catch (JsonException $exception) {
        throw new RuntimeException(
            'The input file does not contain valid JSON.',
            0,
            $exception
        );
    }

    if (!is_array($decoded)) {
        throw new RuntimeException(
            'The JSON document must contain an array of tasks.'
        );
    }

    $tasks = [];

    foreach ($decoded as $index => $taskObject) {
        if (!$taskObject instanceof stdClass) {
            throw new RuntimeException(
                'Each task must be a JSON object.'
            );
        }

        $task = get_object_vars($taskObject);

        if (
            !isset($task['title'])
            || !is_string($task['title'])
            || trim($task['title']) === ''
            || !array_key_exists('done', $task)
            || !is_bool($task['done'])
        ) {
            $rowNumber = $index + 1;

            throw new RuntimeException(
                'Task ' . $rowNumber
                . ' must have a nonempty title and a Boolean done value.'
            );
        }

        $tasks[] = [
            'title' => $task['title'],
            'done' => $task['done'],
        ];
    }

    return $tasks;
}

function writeTable(array $tasks, int $total): void
{
    fwrite(STDOUT, sprintf("%-6s %s\n", 'STATE', 'TASK'));

    foreach ($tasks as $task) {
        $state = $task['done'] ? 'Done' : 'Todo';
        $title = preg_replace(
            '/[\x00-\x1F\x7F]/',
            ' ',
            $task['title']
        );

        if ($title === null) {
            throw new RuntimeException('Could not format a task title.');
        }

        fwrite(STDOUT, sprintf("%-6s %s\n", $state, $title));
    }

    fwrite(
        STDOUT,
        sprintf(
            'Showing %d of %d tasks.%s',
            count($tasks),
            $total,
            PHP_EOL
        )
    );
}

function writeJson(array $tasks, int $total): void
{
    $result = [
        'total' => $total,
        'shown' => count($tasks),
        'tasks' => array_map(
            static fn (array $task): array => [
                'title' => $task['title'],
                'done' => $task['done'],
            ],
            $tasks
        ),
    ];

    try {
        $json = json_encode(
            $result,
            JSON_PRETTY_PRINT
                | JSON_UNESCAPED_SLASHES
                | JSON_THROW_ON_ERROR
        );
    } catch (JsonException $exception) {
        throw new RuntimeException(
            'Could not encode the report as JSON.',
            0,
            $exception
        );
    }

    fwrite(STDOUT, $json . PHP_EOL);
}

function main(array $argv): int
{
    try {
        $options = parseArguments($argv);

        if ($options['help']) {
            printUsage(STDOUT);

            return 0;
        }

        $allTasks = loadTasks($options['file']);
        $visibleTasks = array_slice($allTasks, 0, $options['limit']);

        if ($options['format'] === 'json') {
            writeJson($visibleTasks, count($allTasks));
        } else {
            writeTable($visibleTasks, count($allTasks));
        }

        return 0;
    } catch (InvalidArgumentException $exception) {
        fwrite(STDERR, 'Error: ' . $exception->getMessage() . PHP_EOL);
        printUsage(STDERR);

        return 2;
    } catch (RuntimeException $exception) {
        fwrite(STDERR, 'Error: ' . $exception->getMessage() . PHP_EOL);

        return 1;
    } catch (Throwable $exception) {
        fwrite(STDERR, "Unexpected command failure.\n");

        return 1;
    }
}

exit(main($argv));
~~~

## Run the table report

From the task-tool directory:

~~~sh
php task-report.php --format=table --limit=2 -- tasks.json
~~~

Expected output:

~~~text
STATE  TASK
Done   Review database schema
Todo   Write migration
Showing 2 of 3 tasks.
~~~

The -- token separates options from the input filename. The filename is optional; when it is omitted, the command looks for tasks.json in the current working directory.

## Run the JSON report

~~~sh
php task-report.php --format=json --limit=2 -- tasks.json
~~~

Expected output:

~~~json
{
    "total": 3,
    "shown": 2,
    "tasks": [
        {
            "title": "Review database schema",
            "done": true
        },
        {
            "title": "Write migration",
            "done": false
        }
    ]
}
~~~

The JSON goes to STDOUT without a human-readable summary line, so a pipeline can parse it.

## Display help

~~~sh
php task-report.php --help
~~~

Expected output:

~~~text
Usage: php task-report.php [options] [--] [file]

Options:
  --format=table|json   Select the output format. Default: table.
  --limit=N             Show at most N tasks. Default: 100.
  --help                Display this help message.

Exit status:
  0  Command completed.
  1  Input or runtime failure.
  2  Invalid command usage.

When supplied, the input file must follow --.
~~~

The help request is handled before the script reads the input file. It succeeds even if tasks.json is absent.

## Observe an invalid option

~~~sh
php task-report.php --format=xml -- tasks.json
~~~

Expected STDERR:

~~~text
Error: --format must be table or json.
~~~

The process returns status 2. In PowerShell, inspect the native process status with:

~~~powershell
$LASTEXITCODE
~~~

In a POSIX shell, inspect it immediately with:

~~~sh
echo $?
~~~

Avoid printing shell-specific status commands as part of a data pipeline; they are diagnostic commands for the caller.

## How the parser works

1. It skips argv[0], which contains the script name.
2. It accepts --format and --limit before the -- separator.
3. It supports both --format=json and --format json forms.
4. It rejects unknown options, repeated options, empty values, and missing values.
5. It validates the format against an allowlist.
6. It converts the limit to an integer and enforces a range.
7. It accepts at most one positional filename after --.
8. It defaults to tasks.json in the process's current working directory.

A positional value before -- is rejected, so options cannot be accidentally ignored due to parsing order.

## How the input validation works

- file_get_contents() failure becomes a controlled runtime error.
- json_decode() uses JSON_THROW_ON_ERROR so invalid JSON is not confused with a valid null result.
- The document must be a JSON list.
- Each task must have a nonempty string title and a Boolean done value.
- Invalid input returns status 1 and sends the error to STDERR.
- A title's control characters are replaced with spaces in table output so they cannot break the row layout or insert terminal control sequences.

This is a small-file example: it reads and decodes the whole JSON document in memory. A very large data source should be processed as a stream or through a database query.

## Why output streams and exit statuses matter

- The table or JSON result goes only to STDOUT.
- Usage and runtime errors go only to STDERR.
- Success returns 0.
- Invalid options return 2.
- Invalid input or runtime failures return 1.

A shell script can redirect the JSON result without collecting error messages in the same file:

~~~sh
php task-report.php --format=json -- tasks.json > report.json
~~~

If the command fails, its diagnostic remains on STDERR, and the caller can inspect the process exit status.

## Important notes

- CLI arguments and input files are user-controlled data. Validate them even when the command runs locally.
- This example does not execute a shell command and does not concatenate arguments into SQL.
- The input filename is relative to the current working directory unless an absolute path is supplied.
- Do not use untrusted file paths for destructive operations without a deliberate allowed-directory policy.
- The PHP CLI and the web-server PHP runtime may use different versions, configuration, and extensions.
- The script requires PHP 8.0 or newer for str_starts_with().
- This source is displayed for study and was not executed in a PHP runtime here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [104 - Schema design and database migrations](../lessons/104-schema-design-and-database-migrations.md) | [Q&A](../qnas/105-command-line-applications-and-arguments.md) | [Lesson](../lessons/105-command-line-applications-and-arguments.md) | [106 - Environment configuration and secrets](../lessons/106-environment-configuration-and-secrets.md) |
