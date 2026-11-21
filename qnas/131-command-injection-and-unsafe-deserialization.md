# 131 - Command injection and unsafe deserialization: Questions and answers

Use these questions to review command injection, argument handling, external processes, and the risks of PHP object deserialization.

## Multiple-choice questions

### 1. What causes command injection?

- A. A PHP file contains too many functions.
- B. Untrusted data is combined with a command string and interpreted as instructions by a shell.
- C. JSON contains a Unicode character.
- D. A process returns a nonzero exit code.

**Answer:** B. **Explanation:** A shell parses command strings as a language. If untrusted input becomes part of that string, it may alter the operation rather than remain data.

### 2. Which is the best first choice when PHP can perform the operation itself?

- A. Build a shell command and escape the input.
- B. Use a blacklist of shell punctuation.
- C. Use an appropriate native PHP API.
- D. Run the command as an administrator.

**Answer:** C. **Explanation:** A native API avoids the shell boundary. It still needs validation and authorization, but removes an entire interpreter from the design.

### 3. What does array-form `proc_open()` provide starting with PHP 7.4?

- A. It automatically authorizes the caller.
- B. It passes arguments separately and starts the process without a shell.
- C. It makes every external program ignore all options.
- D. It guarantees the process cannot hang.

**Answer:** B. **Explanation:** PHP 7.4 added an array form for `command`. The process is opened directly without going through a shell. This does not validate arguments or supervise the process lifetime.

### 4. Can an argument array still be vulnerable to attacker-controlled input?

- A. No, because arrays make every argument harmless.
- B. Yes; the external program may interpret an argument as an option or path with unintended meaning.
- C. Only if the array contains an integer.
- D. Only on Windows.

**Answer:** B. **Explanation:** Separating arguments prevents shell parsing, but the called program still has its own argument syntax. Validate arguments according to what that program does.

### 5. Why is `escapeshellcmd()` not an adequate defense for arbitrary command input?

- A. It converts every command to JSON.
- B. It does not turn a user-selected command into a fixed, authorized operation.
- C. It works only with arrays.
- D. It disables process output.

**Answer:** B. **Explanation:** Escaping command characters does not establish which executable or operation the application should permit. Keep those fixed in trusted code.

### 6. What is the safe policy for `unserialize()` when data comes from a request or cookie?

- A. Use `allowed_classes => false` and accept the input.
- B. Call it only after removing semicolons.
- C. Do not deserialize it; use a data format such as JSON and validate the result.
- D. Store it in a database first.

**Answer:** C. **Explanation:** The PHP documentation warns against passing untrusted input to `unserialize()` regardless of `allowed_classes`. JSON avoids PHP object instantiation, but its decoded fields still need validation.

### 7. What may happen when PHP unserializes an object?

- A. PHP may instantiate the object and call `__unserialize()` or `__wakeup()`.
- B. PHP always converts it to a string.
- C. PHP ignores every class name.
- D. PHP automatically checks authorization.

**Answer:** A. **Explanation:** Object reconstruction can invoke magic methods. Those methods or other behavior in available classes can cause side effects.

### 8. What does the `--` marker do when supported by an external program?

- A. It ends PHP execution.
- B. It can tell the program that following values are operands rather than options.
- C. It escapes a shell string for all operating systems.
- D. It validates the user's permissions.

**Answer:** B. **Explanation:** It helps prevent a value beginning with a dash from being parsed as an option by that specific program. Support and behavior vary, so input validation is still required.

## Short-answer questions

### 9. Explain the difference between shell injection and argument injection.

**Answer:** Shell injection changes the command language that a shell interprets, often because application code concatenates data into a shell command string. Argument injection changes how the target program interprets one or more arguments, for example when a supplied filename is mistaken for an option. **Explanation:** Avoiding the shell addresses shell injection, but the target executable still processes its own options and operands.

### 10. Why is a punctuation blacklist fragile?

**Answer:** Shells support many forms of syntax, and the syntax differs across operating systems and interpreters. A blacklist can miss an equivalent form or break legitimate input. **Explanation:** Define the allowed operation and validate data for its meaning instead of trying to enumerate every dangerous character.

### 11. What values should remain under server control when starting a process?

**Answer:** The executable, the operation or script, and preferably the working directory and environment should be selected by trusted application configuration. **Explanation:** If a caller can choose these values, they can often choose what code runs or what resources it can access.

### 12. Why does `allowed_classes => false` not make untrusted serialization a good input format?

**Answer:** It is a mitigation for object instantiation, not a guarantee that arbitrary serialized input is safe. PHP explicitly advises not to pass untrusted input to `unserialize()` regardless of that option. **Explanation:** The safer design is to avoid PHP serialization for untrusted data and decode a data interchange format such as JSON with validation.

### 13. Is valid JSON automatically safe to use?

**Answer:** No. JSON parsing only establishes that the bytes follow JSON syntax. The application must still enforce size and depth limits, check the expected data shape and types, authorize requested actions, and safely encode values when displaying or using them. **Explanation:** Parsing and validation are distinct steps.

### 14. Why might serialized data read from a database still be untrusted?

**Answer:** The database may contain values originally written from user input, imported from another system, modified by a compromised account, or altered through another bug. **Explanation:** Trust depends on provenance and integrity, not on whether data is stored persistently.

### 15. Name four controls for a necessary external process beyond using an argument array.

**Answer:** Examples include a fixed executable path, semantic argument validation, least-privilege permissions, a controlled working directory and environment, an enforced timeout, bounded output, and cleanup after failure. **Explanation:** Argument arrays address shell parsing; they do not limit authority, execution time, or resource consumption.

## Code review questions

### 16. What is unsafe about this code?

~~~php
$format = $_GET['format'] ?? 'txt';
$command = 'render-report --format ' . $format;
exec($command);
~~~

**Answer:** The code creates a shell command from request data. The input may alter the shell command, and even if shell parsing were avoided, an arbitrary format value could change the target program's behavior. **Explanation:** Map a small set of supported format IDs to fixed server-controlled arguments, and prefer a library API if one can render the report.

### 17. Is this safe just because it calls `escapeshellarg()`?

~~~php
$file = $_GET['file'] ?? '';
exec('viewer ' . escapeshellarg($file));
~~~

**Answer:** No. Shell quoting can keep the value within one shell argument in a compatible context, but the viewer may still treat a leading-dash value as an option, and the application has not shown that the file is allowed or authorized. **Explanation:** Map an authorized file ID to a server-controlled path, avoid invoking a shell, and validate according to the specific program's argument rules.

### 18. Identify one strength and two remaining limitations in this code.

~~~php
$process = proc_open(
    ['/usr/local/bin/report-tool', '--', $filename],
    $descriptors,
    $pipes
);
~~~

**Answer:** A strength is that array-form invocation avoids shell parsing on PHP 7.4 and later. Limitations include that `$filename` still needs validation and authorization, and the example does not show timeout, output-size, environment, working-directory, or child-permission controls. **Explanation:** Shell separation is one layer, not a complete process security policy.

### 19. What is wrong with relying on the following code for untrusted payloads?

~~~php
$value = unserialize($payload, ['allowed_classes' => false]);
~~~

**Answer:** The payload remains untrusted serialized input. Disabling normal class instantiation does not make `unserialize()` an acceptable general-purpose parser for attacker-controlled bytes. **Explanation:** Prefer JSON, set input and depth limits, and validate each expected field.

## Applied scenarios

### 20. A report feature supports only PDF and CSV. How should the request select the format?

**Answer:** Accept a small format ID, such as `pdf` or `csv`, validate it against a server-controlled mapping, and map it to fixed program arguments or a native PHP implementation. Reject unknown values. **Explanation:** The caller chooses from explicitly exposed capabilities rather than supplying an executable, option string, or command.

### 21. An external tool must read a user-selected file. What checks should happen before invoking it?

**Answer:** First authorize the user to access the file. Resolve a server-controlled file ID to a path within the intended directory, verify the file type and size required by the feature, and pass the path as a separate argument. Use the target tool's end-of-options marker if supported and apply process restrictions. **Explanation:** A syntactically valid path is not necessarily authorized, contained, or safe for that operation.

### 22. The application stores a PHP-serialized shopping-cart object in a cookie. What should change?

**Answer:** Stop accepting PHP serialized objects from the cookie. Store a random, opaque session identifier in a secure cookie and keep the cart server-side, or use a bounded JSON representation with strict validation and server-side price and authorization checks. **Explanation:** Client-controlled object serialization creates an unsafe deserialization boundary; client-side values must not be trusted for prices or permissions.

### 23. A legacy database column contains PHP-serialized objects written only by the application. What is a sensible migration plan?

**Answer:** Stop writing new serialized values, introduce a versioned JSON representation, and migrate trusted records in controlled batches. For any temporary legacy reader, verify a keyed integrity code before deserialization, restrict accepted classes and depth, and remove the reader after migration. **Explanation:** Integrity protection can help establish that stored legacy bytes have not been altered by an attacker who lacks the key, but it does not make arbitrary user-supplied serialized data safe.

## Coding exercise

### 24. Write a PHP function that accepts a report format ID and returns fixed arguments for an approved tool. What should happen for an unknown format?

**Answer:**

~~~php
<?php

declare(strict_types=1);

function reportArguments(mixed $format): array
{
    $formats = [
        'pdf' => ['--format', 'pdf'],
        'csv' => ['--format', 'csv'],
    ];

    if (!is_string($format) || !isset($formats[$format])) {
        throw new InvalidArgumentException('Unsupported report format.');
    }

    return $formats[$format];
}

$args = reportArguments($_GET['format'] ?? null);
$command = [
    '/usr/local/bin/report-tool',
    ...$args,
    '--',
    '/srv/app/reports/weekly-report.json',
];
~~~

**Explanation:** The request selects a key from a fixed map; it cannot supply an arbitrary option string or executable. The report path is also fixed in this example. In a real application, choose the executable path for the deployment, confirm the tool supports `--`, check the user's authorization, and supervise the process with time, output, and permission limits.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [130 - SSRF, URL validation, and redirect safety](../lessons/130-ssrf-url-validation-and-redirect-safety.md) | [Lesson](../lessons/131-command-injection-and-unsafe-deserialization.md) | [Example](../examples/131-command-injection-and-unsafe-deserialization.md) | [132 - Managing secrets and production configuration](../lessons/132-managing-secrets-and-production-configuration.md) |