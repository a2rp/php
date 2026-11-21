# 131 - Command injection and unsafe deserialization: Example

## Question

Create a runnable PHP CLI example that:

1. Validates a short report label.
2. Passes it to a fixed worker process as a separate argument without invoking a shell.
3. Decodes a small JSON profile and validates its shape.
4. Rejects an invalid label and invalid profile data.

The example must not execute a user-supplied command or deserialize untrusted PHP objects.

## Complete example

Save this file as `safe-command-and-json.php`:

~~~php
<?php

declare(strict_types=1);

function isValidLabel(mixed $value): bool
{
    return is_string($value)
        && preg_match('/\A[A-Za-z0-9 _-]{1,40}\z/', $value) === 1;
}

function runLabelWorker(string $label): string
{
    if (!isValidLabel($label)) {
        throw new InvalidArgumentException('Invalid label format.');
    }

    if (!function_exists('proc_open')) {
        throw new RuntimeException('Process execution is unavailable.');
    }

    // Fixed PHP executable, this fixed script, and a separate data argument.
    $command = [PHP_BINARY, __FILE__, '--worker', $label];

    $descriptors = [
        0 => ['pipe', 'r'],
        1 => ['pipe', 'w'],
        2 => ['pipe', 'w'],
    ];

    $pipes = [];
    $process = proc_open($command, $descriptors, $pipes, __DIR__);

    if (!is_resource($process)) {
        throw new RuntimeException('Could not start the worker.');
    }

    fclose($pipes[0]);

    // This worker produces only one short line, so these bounded reads are small.
    $stdout = stream_get_contents($pipes[1]);
    $stderr = stream_get_contents($pipes[2]);

    fclose($pipes[1]);
    fclose($pipes[2]);

    $exitCode = proc_close($process);

    if ($exitCode !== 0) {
        throw new RuntimeException(
            $stderr !== '' ? trim($stderr) : 'The worker failed.'
        );
    }

    return trim($stdout);
}

function decodeProfile(string $json): array
{
    if (strlen($json) > 1024) {
        throw new LengthException('Profile data is too large.');
    }

    try {
        $value = json_decode($json, false, 16, JSON_THROW_ON_ERROR);
    } catch (JsonException $exception) {
        throw new InvalidArgumentException(
            'Profile data is not valid JSON.',
            previous: $exception
        );
    }

    if (
        !$value instanceof stdClass
        || !property_exists($value, 'displayName')
        || !is_string($value->displayName)
        || strlen($value->displayName) > 40
    ) {
        throw new InvalidArgumentException('Invalid profile data.');
    }

    foreach (array_keys(get_object_vars($value)) as $key) {
        if ($key !== 'displayName') {
            throw new InvalidArgumentException('Unexpected profile field.');
        }
    }

    return [
        'displayName' => $value->displayName,
    ];
}

// A child process runs only this fixed worker branch.
if (($argv[1] ?? null) === '--worker') {
    $label = $argv[2] ?? null;

    if (!isValidLabel($label)) {
        fwrite(STDERR, "Invalid worker label.`n");
        exit(2);
    }

    echo strtoupper($label), PHP_EOL;
    exit(0);
}

$requestedLabel = $argv[1] ?? 'weekly report';

try {
    $workerOutput = runLabelWorker($requestedLabel);
    echo "Worker output: {$workerOutput}", PHP_EOL;
} catch (InvalidArgumentException $exception) {
    echo 'Rejected label: ', $exception->getMessage(), PHP_EOL;
} catch (Throwable $exception) {
    fwrite(STDERR, $exception->getMessage() . PHP_EOL);
    exit(1);
}

try {
    $profile = decodeProfile('{"displayName":"Asha"}');
    echo 'Decoded profile: ', $profile['displayName'], PHP_EOL;
} catch (InvalidArgumentException $exception) {
    echo 'Rejected profile: ', $exception->getMessage(), PHP_EOL;
}

try {
    decodeProfile('{"displayName":42}');
} catch (InvalidArgumentException $exception) {
    echo 'Rejected profile: ', $exception->getMessage(), PHP_EOL;
}
~~~

## Run it

Run the safe example with PHP 8.0 or newer:

~~~sh
php safe-command-and-json.php
~~~

Expected output:

~~~text
Worker output: WEEKLY REPORT
Decoded profile: Asha
Rejected profile: Invalid profile data.
~~~

Pass an invalid label to see it rejected before a process is started:

~~~sh
php safe-command-and-json.php "weekly report; echo unexpected"
~~~

Expected output:

~~~text
Rejected label: Invalid label format.
Decoded profile: Asha
Rejected profile: Invalid profile data.
~~~

## Explanation

- The only executable is `PHP_BINARY`, selected by the server. The worker code is this fixed file and a fixed `--worker` operation.
- The command is an array. The label is one separate argument, so it is not parsed as shell syntax.
- The label allowlist accepts only a small set of characters and a bounded length. The worker validates it again at its own process boundary.
- The worker has no file, network, or administrative operation. It only transforms the label and writes one bounded line.
- The JSON decoder has a byte limit, a nesting-depth limit, and strict parse errors. The application then checks the object shape and field type before returning only the expected field.
- Invalid data is rejected with a controlled message; the application does not echo a raw command or payload.

The CLI demonstration is deliberately small. For a real external program, use a fixed executable path appropriate to the deployment, authorize the requested operation, validate each argument for the target program, and use least privilege. Enforce process timeouts and output limits; this example's short worker has bounded output, but the simple pipe handling is not a general-purpose process supervisor.

## Unsafe patterns to avoid

Do not do this with a request value:

~~~php
$command = 'report-tool ' . $userInput;
exec($command);
~~~

Do not treat shell escaping as permission to accept arbitrary files or options:

~~~php
$command = 'report-tool ' . escapeshellarg($filename);
exec($command);
~~~

Do not parse untrusted cookies, requests, or uploads as PHP objects:

~~~php
$value = unserialize($untrustedPayload);
~~~

Use a fixed operation, separate validated arguments, and a structured interchange format such as JSON. A successfully decoded JSON value still needs size, shape, type, and authorization checks.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [130 - SSRF, URL validation, and redirect safety](../lessons/130-ssrf-url-validation-and-redirect-safety.md) | [Lesson](../lessons/131-command-injection-and-unsafe-deserialization.md) | [Q&A](../qnas/131-command-injection-and-unsafe-deserialization.md) | [132 - Managing secrets and production configuration](../lessons/132-managing-secrets-and-production-configuration.md) |
