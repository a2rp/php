# 053 - Files, Directories, and Filesystem Operations

PHP can read, create, update, inspect, move, and remove files and directories. Safe filesystem code depends on choosing the right operation, checking its result, and treating every path as a boundary that needs care.

## What you will learn

- Build paths relative to the application instead of relying on the process working directory.
- Inspect file and directory types and handle failed operations.
- Read and write complete files safely.
- Open files with modes that match the intended operation.
- Process larger files line by line.
- List, create, rename, copy, and remove filesystem entries.
- Understand metadata, permissions, locking, and race conditions.
- Keep user-controlled paths from escaping an intended directory.

## Filesystem paths

A path identifies a file or directory. Relative paths are resolved from PHP's current working directory, which can vary between a command-line invocation, a web server, a test runner, or a scheduled job.

Use **__DIR__** when a path should be based on the directory containing the current PHP file:

~~~php
<?php

$configPath = __DIR__ . DIRECTORY_SEPARATOR . 'config' . DIRECTORY_SEPARATOR . 'settings.json';
$dataDirectory = __DIR__ . DIRECTORY_SEPARATOR . 'storage';
~~~

**DIRECTORY_SEPARATOR** is the platform's directory separator. PHP accepts forward slashes for many local paths on Windows too, but building paths with **DIRECTORY_SEPARATOR** makes the intent explicit.

A local filesystem path is different from a URL. Some filesystem functions also accept stream-wrapper URLs such as **file://** or **https://** when a wrapper supports the operation. Do not accept an arbitrary user-provided URL or wrapper string where the application expects a local path.

Useful path helpers include:

- **basename($path)** for the final path component.
- **dirname($path)** for the parent path.
- **pathinfo($path)** for components such as extension and filename.
- **realpath($path)** to resolve an existing path to an absolute canonical path.

These helpers manipulate or describe paths; they do not make untrusted input safe by themselves. For example, **basename()** is not a complete path-traversal defense.

## Check what a path refers to

Several functions answer different questions:

| Function | What it checks |
| --- | --- |
| **file_exists()** | Whether a file or directory exists |
| **is_file()** | Whether a path refers to a regular file |
| **is_dir()** | Whether a path refers to a directory |
| **is_link()** | Whether a path is a symbolic link |
| **is_readable()** | Whether PHP can read the path |
| **is_writable()** | Whether PHP can write to the path |

For example:

~~~php
<?php

$path = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'notes.txt';

if (is_file($path) && is_readable($path)) {
    echo 'A readable file is available.';
} elseif (is_dir($path)) {
    echo 'The path is a directory.';
} else {
    echo 'The path is missing or not a readable regular file.';
}
~~~

A check can become stale before the next operation. Another process may change the filesystem between **is_writable()** and **file_put_contents()**. This is a time-of-check to time-of-use race. Treat checks as useful information, but still handle failure from the operation itself.

If a program must create a new file only when the name is unused, do not rely on **file_exists()** followed by a write. Open the file with exclusive creation mode **x** and handle a failed **fopen()** instead.

## Read an entire file

For a small text file, **file_get_contents()** is a convenient way to read the whole file:

~~~php
<?php

$path = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'message.txt';
$contents = file_get_contents($path);

if ($contents === false) {
    throw new RuntimeException('Could not read the file.');
}

echo $contents;
~~~

The result is **string|false**. Use a strict comparison with **false** because an empty file is a successful read that returns an empty string.

The function reads the whole content into memory. It is suitable for small configuration files and documents, but use a stream for a large file or when data should be processed incrementally. Reading a file does not automatically validate its text encoding or parse its format.

## Write or append a complete file

**file_put_contents()** creates a file if needed and overwrites an existing file by default:

~~~php
<?php

$path = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'message.txt';
$contents = "Hello from PHP.\n";

$bytesWritten = file_put_contents($path, $contents, LOCK_EX);

if ($bytesWritten === false) {
    throw new RuntimeException('Could not write the file.');
}

echo "Wrote {$bytesWritten} bytes.\n";
~~~

The return value is the number of bytes written or **false**. A return value of **0** can be valid when writing an empty string, so check with **=== false**.

Use **FILE_APPEND** to append instead of replacing existing content:

~~~php
<?php

$bytesWritten = file_put_contents(
    $path,
    "Another line.\n",
    FILE_APPEND | LOCK_EX
);

if ($bytesWritten === false) {
    throw new RuntimeException('Could not append to the file.');
}
~~~

**LOCK_EX** requests an exclusive advisory lock while **file_put_contents()** writes. A lock only coordinates processes that follow the same locking convention; it is not a transaction and does not prevent unrelated programs from changing the file. A reader that needs coordinated access must also acquire a compatible lock.

Do not store secrets or executable uploads in a web-accessible directory. Choose a storage location, access controls, and backup policy based on the data.

## Open a file with fopen()

Use **fopen()** when you need a file handle, controlled read/write positions, incremental processing, or explicit creation behavior. It returns a stream resource on success or **false** on failure.

Common modes:

| Mode | Behavior |
| --- | --- |
| **r** | Read an existing file from the beginning |
| **r+** | Read and write an existing file without truncating it |
| **w** | Write from the beginning; truncate an existing file or create it |
| **w+** | Read and write; truncate an existing file or create it |
| **a** | Write at the end; create the file if needed |
| **a+** | Read and write, with writes appended; create the file if needed |
| **x** | Create a new file for writing; fail if it already exists |
| **x+** | Create a new file for reading and writing; fail if it already exists |
| **c** | Open for writing without truncating; create it if needed |
| **c+** | Open for reading and writing without truncating; create it if needed |

Append **b** to a mode, such as **rb** or **wb**, to request binary mode. It matters on platforms that distinguish text and binary streams.

The **w** and **w+** modes truncate as soon as the file is opened. Do not open in **w** and then try to lock the handle if preserving old content until the lock is acquired matters. Use an appropriate non-truncating mode, acquire the lock, then truncate or update deliberately.

Always close a handle. **finally** is useful when an exception may occur while the file is open:

~~~php
<?php

$handle = fopen($path, 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the file.');
}

try {
    $contents = stream_get_contents($handle);

    if ($contents === false) {
        throw new RuntimeException('Could not read from the open file.');
    }

    echo $contents;
} finally {
    fclose($handle);
}
~~~

## Process a large file incrementally

Do not load a very large file into memory if it can be processed a line at a time. **fgets()** reads one line from an open stream:

~~~php
<?php

$handle = fopen($path, 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the file.');
}

try {
    while (($line = fgets($handle)) !== false) {
        // Process the line before reading the next one.
        echo $line;
    }

    if (!feof($handle)) {
        throw new RuntimeException('An error occurred while reading the file.');
    }
} finally {
    fclose($handle);
}
~~~

A line can be longer than expected, and line-ending conventions can differ. For binary data or fixed-size chunks, use **fread()** and handle its result according to the file format. Do not assume that a single **fwrite()** call always writes every byte requested when writing through a low-level stream.

## List and create directories

For a small directory, **scandir()** returns an array of entry names or **false** on failure. It includes **.** and **..**, which refer to the current and parent directories:

~~~php
<?php

$directory = __DIR__ . DIRECTORY_SEPARATOR . 'storage';
$entries = scandir($directory);

if ($entries === false) {
    throw new RuntimeException('Could not list the directory.');
}

foreach ($entries as $entry) {
    if ($entry === '.' || $entry === '..') {
        continue;
    }

    echo $entry . "\n";
}
~~~

For object-oriented iteration, **DirectoryIterator** provides entries as objects and a **isDot()** method for skipping those special names. Avoid recursively scanning an untrusted or unexpectedly large tree without limits.

Use **mkdir()** to create a directory. Its mode is subject to the process umask and platform permissions. With recursive mode enabled, missing parent directories are created too:

~~~php
<?php

$directory = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'reports';

if (
    !is_dir($directory)
    && !mkdir($directory, 0770, true)
    && !is_dir($directory)
) {
    throw new RuntimeException('Could not create the directory.');
}
~~~

The final **is_dir()** check handles the case where another process created the directory between the first check and **mkdir()**. Permissions differ by operating system and server configuration; do not assume that a numeric mode has identical effects everywhere.

## Copy, rename, and remove entries

Common filesystem operations include:

- **copy($source, $destination)** to copy a file.
- **rename($from, $to)** to rename a file or directory, or move it when the platform and filesystem allow.
- **unlink($path)** to remove a file or symbolic link.
- **rmdir($path)** to remove an empty directory.

They return **true** on success and **false** on failure, and may emit warnings. Check the result and report an actionable error.

~~~php
<?php

if (!copy($sourcePath, $backupPath)) {
    throw new RuntimeException('Could not copy the file.');
}

if (!rename($temporaryPath, $finalPath)) {
    throw new RuntimeException('Could not move the file into place.');
}

if (!unlink($obsoletePath)) {
    throw new RuntimeException('Could not remove the obsolete file.');
}
~~~

**rmdir()** removes only an empty directory. Recursive deletion needs a deliberate traversal policy. Never build a recursive delete path directly from untrusted input; a mistaken path can remove important application data.

Cross-filesystem moves, destination replacement, permissions, open handles, and platform behavior can affect **rename()**. Do not assume it is a universal atomic replacement operation.

## File metadata and caching

Functions such as **filesize()**, **filemtime()**, **fileperms()**, and **filetype()** inspect metadata. They may return **false** when the information cannot be obtained. Filesystem metadata can be cached during a request; **clearstatcache()** can refresh cached results after a change.

~~~php
<?php

$size = filesize($path);

if ($size === false) {
    throw new RuntimeException('Could not read file size.');
}

echo "Size: {$size} bytes\n";

clearstatcache(true, $path);
~~~

Metadata checks are not a substitute for handling a later operation's failure. Another process can change or replace the path after a check.

## Protect path boundaries

Never concatenate an arbitrary request value into a filesystem path and assume that **basename()** makes it safe. Inputs such as **../** can target parent directories, and symbolic links can redirect a path to another location.

Prefer mapping a validated application identifier to a server-controlled filename:

~~~php
<?php

$documents = [
    'welcome' => 'welcome.txt',
    'help' => 'help.txt',
];

$id = 'welcome'; // Validate external input before using it as a key.

if (!array_key_exists($id, $documents)) {
    throw new RuntimeException('Unknown document.');
}

$baseDirectory = __DIR__ . DIRECTORY_SEPARATOR . 'storage';
$path = $baseDirectory . DIRECTORY_SEPARATOR . $documents[$id];
~~~

If an application must accept a relative path, define a trusted base directory, canonicalize existing paths with **realpath()**, and verify the resolved path remains within that base. New files require validating the parent directory separately because **realpath()** cannot resolve a path that does not exist yet. Consider symlink behavior and permissions as well.

Treat file contents as untrusted input too. Validate the expected format and encoding before parsing or displaying it. Escape output for its destination context.

## Permissions and safe operation

- Run PHP with only the filesystem permissions the application needs.
- Keep private data outside the public document root where possible.
- Avoid broad permissions such as making sensitive files readable or writable by everyone.
- Do not trust **is_readable()** or **is_writable()** as guarantees that a later read or write will succeed.
- Use exclusive creation mode when a new filename must not replace an existing file.
- Use locks only when all participating readers and writers follow the same locking protocol.
- Keep paths fixed or validated and constrained to an intended directory.

## Common mistakes

- Assuming relative paths are based on the PHP source file rather than the current working directory.
- Treating an empty string or a zero-byte write as an error instead of checking strictly for **false**.
- Using **file_exists()** as a check-then-write safety guarantee.
- Opening with **w** before acquiring a lock and accidentally truncating existing data.
- Reading a large file entirely into memory when incremental processing is appropriate.
- Forgetting to close a stream after an exception.
- Assuming **mkdir()** mode values work identically across platforms.
- Assuming **rmdir()** removes a non-empty directory.
- Treating **basename()** as a path-traversal defense.
- Ignoring symbolic links, filesystem permissions, and operation failures.

## Try it yourself

1. Build a path using **__DIR__** and **DIRECTORY_SEPARATOR**, then read a small text file.
2. Compare **file_exists()**, **is_file()**, and **is_dir()** on a file and a directory.
3. Write a file, append a second line with **FILE_APPEND**, then read it back.
4. Try the **x** mode twice and handle the second attempt failing because the file already exists.
5. Read a larger text file line by line and count its lines without storing the whole file.
6. List a directory with **scandir()** and skip **.** and **..**.
7. Create nested directories with recursive **mkdir()**, then remove the empty directories in reverse order.
8. Observe how **filemtime()** behaves after changing a file and refresh metadata with **clearstatcache()**.
9. Replace a user-provided filename with a fixed allowlist of application IDs and server-controlled names.

## Key takeaways

- Anchor application paths deliberately, usually with **__DIR__**.
- Handle **false** returns from reads, writes, opens, metadata checks, and directory operations.
- Use whole-file helpers for small data and streams for larger or incremental work.
- Choose **fopen()** modes carefully; **w** truncates, **a** appends, and **x** creates exclusively.
- Directory and file operations are subject to permissions, races, and platform behavior.
- Keep untrusted paths inside an explicitly trusted directory and account for symbolic links.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [052 - Text Encoding, Localization, and Internationalization](052-text-encoding-localization-and-internationalization.md) | [Open Q&A](../qnas/053-files-directories-and-filesystem-operations.md) | [Open example](../examples/053-files-directories-and-filesystem-operations.md) | [054 - Streams, Wrappers, and Contexts](054-streams-wrappers-and-contexts.md) |
