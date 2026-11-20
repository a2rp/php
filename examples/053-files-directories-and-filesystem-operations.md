# 053 - Files, Directories, and Filesystem Operations: Complete Example

This example creates a temporary directory, writes and appends a text file, reads it as a whole and line by line, inspects metadata, lists entries, copies and renames a file, and removes only the paths it created.

## Goal

Practice common filesystem operations while checking return values and keeping paths anchored to a known directory.

## Requirements

- PHP 8.1 or later.
- Permission to create and remove a temporary directory under **sys_get_temp_dir()**.
- Run the example from the command line.
- The script creates a uniquely named temporary directory and removes its own files and directories when it finishes.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

function requireWrite(string $path, string $contents, int $flags = 0): int
{
    $bytes = file_put_contents($path, $contents, $flags);

    if ($bytes === false) {
        throw new RuntimeException("Could not write: {$path}");
    }

    return $bytes;
}

$temporaryRoot = sys_get_temp_dir()
    . DIRECTORY_SEPARATOR
    . 'php-filesystem-example-'
    . bin2hex(random_bytes(6));

$dataDirectory = $temporaryRoot . DIRECTORY_SEPARATOR . 'data';
$notesPath = $dataDirectory . DIRECTORY_SEPARATOR . 'notes.txt';
$copyPath = $dataDirectory . DIRECTORY_SEPARATOR . 'notes-copy.txt';
$archivePath = $dataDirectory . DIRECTORY_SEPARATOR . 'archive.txt';

try {
    // Recursive creation makes both the temporary root and data directory.
    if (
        !mkdir($dataDirectory, 0700, true)
        && !is_dir($dataDirectory)
    ) {
        throw new RuntimeException('Could not create the temporary data directory.');
    }

    echo 'Directories created: ' . (is_dir($dataDirectory) ? 'yes' : 'no') . "\n";

    // The first write creates the file; the second write appends to it.
    $firstWrite = requireWrite($notesPath, "First line\n");
    $appendWrite = requireWrite(
        $notesPath,
        "Second line\n",
        FILE_APPEND | LOCK_EX
    );

    echo "Wrote bytes: {$firstWrite}\n";
    echo "Appended bytes: {$appendWrite}\n";

    // Read a small file as a single string and distinguish failure from empty content.
    $contents = file_get_contents($notesPath);

    if ($contents === false) {
        throw new RuntimeException('Could not read the notes file.');
    }

    echo "Complete file contents:\n";
    echo $contents;

    echo 'File is regular: ' . (is_file($notesPath) ? 'yes' : 'no') . "\n";

    clearstatcache(true, $notesPath);
    $size = filesize($notesPath);

    if ($size === false) {
        throw new RuntimeException('Could not get the file size.');
    }

    echo "File size: {$size} bytes\n";

    // Process lines through a handle instead of loading the file again.
    $handle = fopen($notesPath, 'rb');

    if ($handle === false) {
        throw new RuntimeException('Could not open the notes file for streaming.');
    }

    try {
        while (($line = fgets($handle)) !== false) {
            echo 'Read line: ' . rtrim($line, "\r\n") . "\n";
        }

        if (!feof($handle)) {
            throw new RuntimeException('An error occurred while reading a line.');
        }
    } finally {
        fclose($handle);
    }

    // Copy the file, then rename the copy. Each operation can fail.
    if (!copy($notesPath, $copyPath)) {
        throw new RuntimeException('Could not copy the notes file.');
    }

    if (!rename($copyPath, $archivePath)) {
        throw new RuntimeException('Could not rename the copied file.');
    }

    echo 'Copied and renamed: '
        . (is_file($archivePath) && !file_exists($copyPath) ? 'yes' : 'no')
        . "\n";

    // scandir() includes dot entries; remove them before displaying names.
    $entries = scandir($dataDirectory);

    if ($entries === false) {
        throw new RuntimeException('Could not list the data directory.');
    }

    $visibleEntries = array_values(array_filter(
        $entries,
        static fn (string $entry): bool => $entry !== '.' && $entry !== '..'
    ));

    echo 'Directory entries: ' . implode(', ', $visibleEntries) . "\n";
} finally {
    // Remove only the exact files created by this script.
    foreach ([$archivePath, $copyPath, $notesPath] as $file) {
        if (is_file($file) && !unlink($file)) {
            throw new RuntimeException("Could not remove temporary file: {$file}");
        }
    }

    // Remove child directories before their parent; rmdir() requires emptiness.
    foreach ([$dataDirectory, $temporaryRoot] as $directory) {
        if (is_dir($directory) && !rmdir($directory)) {
            throw new RuntimeException("Could not remove temporary directory: {$directory}");
        }
    }
}

echo "Cleanup: complete\n";
~~~

## Expected output

~~~text
Directories created: yes
Wrote bytes: 11
Appended bytes: 12
Complete file contents:
First line
Second line
File is regular: yes
File size: 23 bytes
Read line: First line
Read line: Second line
Copied and renamed: yes
Directory entries: archive.txt, notes.txt
Cleanup: complete
~~~

## How it works

- **sys_get_temp_dir()** supplies the platform's temporary directory. **random_bytes()** makes the example's directory name unlikely to collide with an existing one.
- **mkdir($path, 0700, true)** creates the nested path. The mode is subject to the operating system and process umask.
- **requireWrite()** checks for **false** and returns the number of bytes written. It accepts zero as a successful byte count.
- The first **file_put_contents()** call creates or replaces the file. **FILE_APPEND** adds the second line; **LOCK_EX** requests a cooperative lock during that write.
- **file_get_contents()** reads the entire small file. A strict **=== false** check allows an empty file to remain a valid result.
- **fgets()** processes the same file line by line. The handle is closed in **finally**, even if line processing fails.
- **filesize()** reads metadata after **clearstatcache()**. Filesystem operations can fail even after an earlier check.
- **copy()** leaves the original in place; **rename()** moves the copy to a new name.
- **scandir()** returns the dot entries too. The example filters those names before printing the sorted file list.
- **finally** removes only the exact temporary files and directories created here. It removes files first and empty directories afterward.

## Try it yourself

1. Change the appended text and update the expected byte count.
2. Create an empty file and confirm that a successful read returns an empty string and its size is zero.
3. Replace the whole-file read with a line counter that does not store all lines.
4. Add another file and observe the sorted order returned by **scandir()**.
5. Try opening a new path with **fopen($path, 'x')** and handle a second open attempt failing.
6. Remove the copy operation and compare the directory listing.
7. Add a check for **is_readable()**, but still handle a later read failure.
8. Keep a file open while attempting to remove it and observe platform-specific behavior.

## Common mistakes

- Using the process working directory when the path should be based on the PHP source file.
- Assuming **file_put_contents()** appends by default.
- Treating a zero-byte result as failure instead of checking for **false**.
- Using **fopen()** mode **w** without realizing it truncates an existing file.
- Reading a large file wholly into memory when line-by-line processing is enough.
- Forgetting to close a stream after an error.
- Removing a directory before its child files or subdirectories.
- Assuming a file lock coordinates with programs that ignore the same lock protocol.
- Constructing deletion paths directly from untrusted input.

## Key takeaways

- Build paths deliberately and check each filesystem operation's result.
- Use whole-file helpers for small files and streams for incremental processing.
- Know whether an operation overwrites, appends, creates exclusively, copies, renames, or removes.
- Clean up resources and temporary files in a controlled order.
- Restrict destructive operations to paths the application itself has selected.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [052 - Text Encoding, Localization, and Internationalization](../lessons/052-text-encoding-localization-and-internationalization.md) | [Read lesson](../lessons/053-files-directories-and-filesystem-operations.md) | [Open Q&A](../qnas/053-files-directories-and-filesystem-operations.md) | [054 - Streams, Wrappers, and Contexts](../lessons/054-streams-wrappers-and-contexts.md) |
