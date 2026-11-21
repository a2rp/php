# 131 - Command injection and unsafe deserialization

Command injection happens when data supplied by a user is interpreted as part of an operating-system command. Unsafe deserialization happens when an application turns untrusted bytes into PHP values or objects in a way that lets the input control object creation or trigger application behavior. Both problems arise when data crosses a trust boundary and is given more meaning or authority than it should have.

The safest defenses are architectural: avoid starting a shell command when PHP can do the work directly, pass arguments separately when a process is truly needed, and do not use PHP object deserialization for untrusted input.

## Learning goals

By the end of this chapter, you should be able to:

- Distinguish shell command injection from argument or option injection.
- Explain why blacklists and shell escaping are fragile defenses.
- Prefer native PHP APIs over operating-system commands.
- Use `proc_open()` with an argument array on PHP 7.4 and newer.
- Validate an external program's arguments according to their meaning.
- Explain the risks of calling `unserialize()` on untrusted input.
- Use JSON plus schema validation for ordinary data exchange.
- Test rejection paths without running destructive commands.

## Command injection

A shell command is a string in a language of its own. It can include commands, pipelines, redirections, substitutions, quoting, and other syntax. If an application builds that string by joining trusted command text with user input, the shell may interpret some of the input as new instructions.

For example, this is unsafe:

~~~php
$filename = $_GET['file'] ?? '';
$command = 'pdfinfo ' . $filename;
exec($command);
~~~

The application intended to pass one filename to `pdfinfo`. Instead, it gives a shell a complete string to parse. An attacker may be able to change what the shell runs. The exact syntax varies between shells and operating systems, so trying to block a few punctuation characters is not a dependable defense.

Never build a command by concatenating request data, form values, cookies, database fields, uploaded filenames, or other untrusted values into a shell command string.

## Shell injection and argument injection

Avoiding a shell prevents shell metacharacters from being interpreted as shell syntax, but it does not automatically make every argument safe.

Consider a program that accepts an option such as `--output`. If an untrusted value becomes an argument beginning with a dash, the external program may interpret it as an option rather than as a filename. This is argument injection: the shell is not required for input to change the meaning of a command.

For every external process:

1. Fix the executable and the operation in server-controlled code.
2. Validate each dynamic argument for its intended meaning and allowed range.
3. Use an argument array so values remain individual arguments.
4. Use the external tool's end-of-options marker, commonly `--`, when that tool supports it.
5. Keep the process least-privileged and constrain its working directory, environment, time, and output.

An end-of-options marker is an extra safeguard, not a replacement for validation. Programs differ, and not every program supports it.

## Prefer native PHP operations

Before invoking an external tool, check whether PHP already provides the needed capability. Native file, image, archive, hashing, and process APIs often avoid the shell boundary entirely.

For example, use PHP's file APIs to read an application-controlled path rather than constructing a command such as `cat $path`. Keep paths rooted in a server-controlled directory, map user-facing IDs to known files, and apply the file-size and authorization limits required by the feature.

A native API still needs validation and authorization. Avoiding the shell removes one interpreter; it does not make arbitrary paths or operations safe.

## Use an argument array when a process is necessary

PHP 7.4 and newer let `proc_open()` accept the command and its arguments as an array. PHP starts the process directly instead of passing a command string through a shell. Keep the executable and program code fixed, and pass validated data as a separate argument.

This small CLI demonstration starts PHP itself with fixed code and supplies one bounded label as a separate argument:

~~~php
<?php

declare(strict_types=1);

$label = $argv[1] ?? null;

if (!is_string($label) || preg_match('/\A[A-Za-z0-9 _-]{1,40}\z/', $label) !== 1) {
    fwrite(STDERR, "Provide a label of 1 to 40 letters, numbers, spaces, underscores, or hyphens.`n");
    exit(2);
}

$phpCode = 'echo strtoupper($argv[1]), PHP_EOL;';
$command = [PHP_BINARY, '-r', $phpCode, $label];

$descriptors = [
    0 => ['pipe', 'r'],
    1 => ['pipe', 'w'],
    2 => ['pipe', 'w'],
];

$pipes = [];
$process = proc_open($command, $descriptors, $pipes);

if (!is_resource($process)) {
    fwrite(STDERR, "Could not start the worker.`n");
    exit(1);
}

fclose($pipes[0]);
$stdout = stream_get_contents($pipes[1]);
$stderr = stream_get_contents($pipes[2]);
fclose($pipes[1]);
fclose($pipes[2]);

$exitCode = proc_close($process);

if ($exitCode !== 0) {
    fwrite(STDERR, $stderr ?: "Worker failed.`n");
    exit($exitCode);
}

echo $stdout;
~~~

Save it as `safe-process.php` and run:

~~~sh
php safe-process.php "weekly report"
~~~

Expected output:

~~~text
WEEKLY REPORT
~~~

The code uses a fixed executable (the current PHP binary), fixed worker code, and a separate argument. The allowlist also limits what the label can mean to this operation. The process example demonstrates safe argument separation; it is intentionally small and is not a complete production process supervisor. A real worker that can run for a long time or produce substantial output also needs enforced time limits, bounded output handling, safe environment and working-directory settings, and cleanup when it exceeds its budget.

Do not select the executable or the worker code from request input. Avoid relying on an unexpected `PATH` entry to locate a program; use a server-controlled executable path where the deployment permits it.

## Why escaping and blacklists are not the main solution

- A blacklist such as “remove semicolons” misses other shell syntax and varies by shell.
- `escapeshellcmd()` does not turn an arbitrary command into an approved operation.
- `escapeshellarg()` can quote one argument for a shell command, but it does not prevent the called program from interpreting that argument as an option.
- Quoting rules vary across platforms and command interpreters.
- Escaping cannot decide whether the caller is authorized to request the operation.

If a legacy integration truly requires a shell string, build the command from a fixed command template, validate every value for its semantic purpose, escape each argument with the platform-appropriate mechanism, and review the exact deployed shell and executable. Treat this as a constrained compatibility measure. Prefer an API that does not invoke a shell.

## Unsafe PHP deserialization

`unserialize()` converts PHP's serialized representation back into a value. When the input represents an object, PHP may instantiate that class and invoke its `__unserialize()` or `__wakeup()` method. Other magic methods can run later as the object is used or destroyed. If attacker-controlled data can select object classes or property values, existing application or dependency code may combine into a dangerous chain of side effects.

Never call `unserialize()` on untrusted user input. The PHP option `allowed_classes => false` prevents normal classes from being instantiated, but it does not make untrusted serialized input a recommended or generally safe data format. It is not a substitute for avoiding deserialization.

Treat data as untrusted when it comes from a request, cookie, uploaded file, queue, cache, database row, or another service unless you have a strong, verified reason to trust its origin and integrity. Storage does not automatically make data trusted.

## Prefer JSON for data interchange

For ordinary structured data, JSON avoids PHP-specific object instantiation. Decode with an explicit depth limit and strict error handling, then validate the expected shape and every field before using it.

~~~php
<?php

declare(strict_types=1);

function decodeProfile(string $json): array
{
    if (strlen($json) > 16_384) {
        throw new LengthException('Profile data is too large.');
    }

    $value = json_decode(
        $json,
        true,
        32,
        JSON_THROW_ON_ERROR
    );

    if (
        !is_array($value)
        || !isset($value['displayName'])
        || !is_string($value['displayName'])
        || strlen($value['displayName']) > 80
    ) {
        throw new UnexpectedValueException('Invalid profile data.');
    }

    return [
        'displayName' => $value['displayName'],
    ];
}

$profile = decodeProfile('{"displayName":"Asha"}');
~~~

JSON is a data format, not a complete security policy. Limit input size and nesting, validate types and ranges, authorize actions separately, and escape values for the context where they will be displayed or used. Do not trust a decoded field merely because JSON parsing succeeded.

## If legacy serialized data must be retained

The first choice should still be to migrate to a safe data format. If a legacy system must read PHP serialized data, restrict the source to data the application itself produced and protect its integrity with a keyed message authentication code. Verify the MAC with `hash_equals()` before deserializing; do not treat a plain hash as proof of authenticity. Restrict classes and depth as additional controls, and plan to remove the dependency.

These controls reduce risk for authenticated, application-owned stored data. They do not justify passing attacker-controlled serialized input to `unserialize()`.

## Secure process and data checklist

- Keep command names, executable paths, and operation code under server control.
- Prefer native PHP APIs and fixed operation IDs over arbitrary command strings.
- Use `proc_open()` array form on PHP 7.4+ when a child process is unavoidable.
- Validate arguments by type, format, range, and authorization; consider option injection as well as shell injection.
- Do not rely on punctuation blacklists or a single escaping function.
- Run child processes with minimal permissions and controlled environment, directory, time, and output.
- Never deserialize untrusted data; prefer JSON and validate its schema.
- Limit JSON size and depth, then validate every field before use.
- Keep serialization secrets outside source control and rotate them if exposed.
- Log rejected operation IDs and validation failures safely; do not log secrets or entire attacker-controlled payloads.

## Testing checklist

Test that the application:

- Rejects command text, unknown operation IDs, and unexpected argument types.
- Rejects empty, oversized, malformed, and out-of-range argument values.
- Treats values beginning with a dash as data only when the selected tool and validation rules allow them.
- Keeps shell metacharacters inside a single data value when using array-form process invocation.
- Cannot choose an executable, working directory, environment variable, or arbitrary output path.
- Handles process startup failure, nonzero exit status, large output, and timeouts.
- Rejects malformed JSON, excessive nesting, oversized input, wrong field types, and unknown fields where the schema disallows them.
- Does not unserialize request, cookie, upload, or other untrusted bytes.
- Verifies a keyed integrity check before reading any retained legacy serialized data.
- Does not expose command lines, serialized payloads, secrets, or private process output in error messages.

## Key takeaways

- A shell command string is executable syntax; never concatenate untrusted input into one.
- Avoid a process when native PHP can perform the operation.
- Array-form `proc_open()` avoids shell parsing, but arguments still need semantic validation.
- Shell escaping and `--` do not replace a fixed operation and allowlist.
- Do not use `unserialize()` with untrusted input, even when `allowed_classes` is disabled.
- Prefer bounded, validated JSON for ordinary data exchange.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [130 - SSRF, URL validation, and redirect safety](130-ssrf-url-validation-and-redirect-safety.md) | [Q&A](../qnas/131-command-injection-and-unsafe-deserialization.md) | [Example](../examples/131-command-injection-and-unsafe-deserialization.md) | [132 - Managing secrets and production configuration](132-managing-secrets-and-production-configuration.md) |
