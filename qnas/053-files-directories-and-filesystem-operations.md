# 053 - Files, Directories, and Filesystem Operations: Q&A

Review path construction, file reads and writes, stream modes, directories, metadata, permissions, locking, and safe path handling. Try each question before reading its answer. See the [chapter lesson](../lessons/053-files-directories-and-filesystem-operations.md).

## Multiple-choice questions

### 1. What is the base for resolving a relative filesystem path?

A. Always the directory containing the PHP file
B. PHP's current working directory
C. The operating system's root directory
D. The directory containing the browser

**Answer:** B. PHP's current working directory.

**Explanation:** The working directory can vary by execution environment. Use **__DIR__** when a path should be based on the current source file.

### 2. What does **__DIR__** represent?

A. The current working directory at runtime
B. The directory containing the current PHP file
C. The server document root in every application
D. The user's home directory

**Answer:** B. The directory containing the current PHP file.

**Explanation:** **__DIR__** is useful for stable application-relative paths. It does not automatically identify the public document root.

### 3. Which statement about **file_exists()** is correct?

A. It checks only regular files
B. It checks whether a file or directory exists
C. It creates the path if missing
D. It guarantees the next operation will succeed

**Answer:** B. It checks whether a file or directory exists.

**Explanation:** Use **is_file()** or **is_dir()** when the type matters. The filesystem can change after a check, so still handle operation failures.

### 4. Which function checks whether a path refers to a regular file?

A. **is_dir()**
B. **is_link()**
C. **is_file()**
D. **basename()**

**Answer:** C. **is_file()**

**Explanation:** **is_file()** checks for a regular file. It follows a symbolic link to determine whether its target is a regular file; use **is_link()** to inspect the link itself.

### 5. What does **file_get_contents()** return when it cannot read the file?

A. An empty string in every case
B. **false**
C. An empty array
D. A file handle

**Answer:** B. **false**

**Explanation:** A successful read of an empty file returns an empty string. Compare the result with **=== false** to distinguish failure from valid empty content.

### 6. What happens when **file_put_contents()** writes to an existing file without special flags?

A. The new data is appended
B. The file is overwritten with the new data
C. The write is rejected
D. The file is renamed

**Answer:** B. The file is overwritten with the new data.

**Explanation:** By default, **file_put_contents()** replaces the existing contents. Use **FILE_APPEND** when the intent is to append.

### 7. Which flag appends data with **file_put_contents()**?

A. **FILE_APPEND**
B. **LOCK_SH**
C. **SCANDIR_SORT_NONE**
D. **DIRECTORY_SEPARATOR**

**Answer:** A. **FILE_APPEND**

**Explanation:** **FILE_APPEND** writes the provided data at the end of the existing file, creating the file if needed.

### 8. What does **LOCK_EX** request in **file_put_contents()**?

A. Permanent exclusive ownership of the file
B. An exclusive advisory lock while the write is performed
C. Encryption of the file
D. A file-system-wide transaction

**Answer:** B. An exclusive advisory lock while the write is performed.

**Explanation:** Locking coordinates only with processes that use compatible locks. It does not create a transaction or prevent unrelated programs from writing.

### 9. Which behavior belongs to **fopen($path, 'w')**?

A. Opens only an existing file for reading
B. Opens for writing and truncates an existing file
C. Appends without changing the current contents
D. Creates a file only if it does not exist and otherwise fails

**Answer:** B. Opens for writing and truncates an existing file.

**Explanation:** The **w** mode creates a missing file or truncates an existing one as soon as it opens. Choose the mode carefully to avoid losing data.

### 10. Which **fopen()** mode requests exclusive creation?

A. **r**
B. **a**
C. **x**
D. **w**

**Answer:** C. **x**

**Explanation:** The **x** mode creates a new file and fails if that path already exists. It is useful when replacing an existing file must be avoided.

### 11. What is the primary behavior of the **a** mode?

A. Read-only access from the beginning
B. Append writes to the end and create the file if needed
C. Truncate the file before every write
D. Fail unless the file already exists

**Answer:** B. Append writes to the end and create the file if needed.

**Explanation:** With **a**, writes go to the end of the file. The **a+** mode also enables reading.

### 12. What does **fgets()** normally return when reading a text stream?

A. One line, or **false** when it cannot read another line
B. The entire file as one string
C. The current file size
D. A list of directory names

**Answer:** A. One line, or **false** when it cannot read another line.

**Explanation:** A loop such as **while (($line = fgets($handle)) !== false)** can process the file incrementally without loading the whole file into memory.

### 13. Which special entries does **scandir()** include in its result?

A. Only hidden files
B. **.** and **..**
C. Only regular files
D. No entries unless sorted manually

**Answer:** B. **.** and **..**

**Explanation:** They represent the current directory and its parent. Filter them when displaying or processing ordinary entries.

### 14. What does **rmdir()** do?

A. Removes any directory and all of its contents recursively
B. Removes an empty directory
C. Removes a regular file
D. Renames a directory

**Answer:** B. Removes an empty directory.

**Explanation:** A non-empty directory must be handled deliberately before it can be removed. Recursive deletion needs a carefully constrained policy.

### 15. Why should code check the return value of **mkdir()** even after checking **is_dir()**?

A. Because another process can create the directory between the check and the call
B. Because **mkdir()** only works on files
C. Because **is_dir()** always creates the directory
D. Because directory paths cannot be checked

**Answer:** A. Because another process can create the directory between the check and the call.

**Explanation:** Filesystem state can change between separate operations. A common pattern checks **mkdir()** and then checks **is_dir()** again to account for a concurrent creation.

## Short-answer questions

### 16. Why should a failed **file_get_contents()** call be checked with **=== false**?

**Answer:** Because an empty file is read successfully as an empty string, while a failed read returns **false**.

**Explanation:** A loose or truthiness check can confuse valid empty content with failure.

### 17. Can **file_put_contents()** return zero on a successful write?

**Answer:** Yes. Writing an empty string can successfully write zero bytes.

**Explanation:** The return value is a byte count or **false**. Check **$result === false** rather than treating every false-like value as failure.

### 18. What is a time-of-check to time-of-use race in filesystem code?

**Answer:** It occurs when code checks a path and then another process changes that path before the operation uses it.

**Explanation:** For example, a file can be replaced after **is_writable()** returns true. The subsequent write still needs error handling.

### 19. Why is **basename()** not a complete defense against path traversal?

**Answer:** It only extracts a trailing path component; it does not establish that the resolved path is inside an approved directory or account for symbolic links.

**Explanation:** Prefer validated application IDs mapped to server-controlled filenames. If accepting paths is required, canonicalize existing paths and verify containment within a trusted base.

### 20. What does **realpath()** do, and what limitation matters for new files?

**Answer:** It resolves an existing path to a canonical absolute path, but returns **false** when the target does not exist.

**Explanation:** For a new file, validate the existing parent directory separately, then combine it with a validated filename.

### 21. When might **clearstatcache()** be useful?

**Answer:** After changing a file when code needs fresh cached metadata such as its size or modification time.

**Explanation:** Filesystem status results can be cached during a request. **clearstatcache(true, $path)** can clear the cache for the specified path.

### 22. What can affect the permissions created by **mkdir($path, $mode)**?

**Answer:** The process umask, operating system, and server configuration can affect the resulting permissions.

**Explanation:** Do not assume numeric mode values have identical effects on every platform. Inspect and configure the deployed environment appropriately.

### 23. How should code manage a stream handle if an exception may happen while processing it?

**Answer:** Check whether **fopen()** returned a handle, then close it in a **finally** block.

**Explanation:** The cleanup runs even if processing throws, preventing the handle from being left open longer than necessary.

### 24. What does appending **b** to an **fopen()** mode request?

**Answer:** Binary mode, such as **rb** for reading or **wb** for writing.

**Explanation:** This matters on systems that distinguish text and binary streams. It is especially important when processing binary data.

### 25. What is the difference between a local filesystem path and a URL?

**Answer:** A local path names a filesystem location, while a URL identifies a resource through a scheme such as **https://**.

**Explanation:** PHP stream wrappers let some filesystem functions work with URLs, but wrapper support varies. Do not treat arbitrary URL or wrapper input as a trusted local path.

### 26. How can a symbolic link affect path-safety checks?

**Answer:** A path that appears to be inside a directory can resolve through a symbolic link to a target elsewhere.

**Explanation:** Consider symlinks when enforcing directory boundaries. Canonicalize existing paths and use permissions that prevent untrusted users from replacing trusted path components.

### 27. How do **copy()** and **rename()** differ?

**Answer:** **copy()** creates a copy at another path; **rename()** changes the name or location of the original entry when the platform and filesystem permit it.

**Explanation:** Cross-filesystem moves, permissions, destination replacement, and platform behavior can affect **rename()**. Handle either function's failure.

### 28. Does **LOCK_EX** on **file_put_contents()** prevent every other process from writing?

**Answer:** No. It coordinates only with processes that honor a compatible advisory-locking protocol.

**Explanation:** Unrelated programs can ignore the lock. A lock is not a transaction or a universal access-control mechanism.

### 29. When is a stream preferable to **file_get_contents()**?

**Answer:** When the file is large or its content can be processed incrementally, such as line by line.

**Explanation:** **file_get_contents()** reads the whole file into memory. Streams can reduce memory use by processing chunks as they arrive.

## True or false

### 30. A relative path is always resolved from the directory containing the current PHP source file.

**Answer:** False.

**Explanation:** A relative path is resolved from PHP's current working directory. Use **__DIR__** to anchor a path to the source file's directory.

### 31. Opening an existing file with **fopen($path, 'w')** truncates it.

**Answer:** True.

**Explanation:** The **w** mode truncates an existing file immediately, before later writes.

### 32. **scandir()** omits the entries **.** and **..**.

**Answer:** False.

**Explanation:** They are included and should usually be skipped when iterating over ordinary entries.

### 33. **chmod()** mode values have exactly the same effect on every operating system and server.

**Answer:** False.

**Explanation:** Platform behavior, server configuration, and umask can affect permissions.

### 34. **rmdir()** removes only an empty directory.

**Answer:** True.

**Explanation:** It does not recursively delete the directory's contents.

### 35. If **is_writable($path)** returns true, a later write to that path is guaranteed to succeed.

**Answer:** False.

**Explanation:** The filesystem can change after the check, and later operations can still fail. Check the result of the actual write.

### 36. A successful read of an empty file returns **false** from **file_get_contents()**.

**Answer:** False.

**Explanation:** An empty file returns an empty string. **false** indicates that the read failed.

### 37. An advisory file lock can coordinate with a process that does not use the same locking convention.

**Answer:** False.

**Explanation:** Other readers and writers must follow a compatible locking protocol for the lock to coordinate their access.

## Predict the output

### 38. What does this print if **empty.txt** is a readable zero-byte file?

~~~php
<?php

$contents = file_get_contents('empty.txt');

echo $contents === false ? 'failed' : strlen($contents);
~~~

**Answer:**

~~~text
0
~~~

**Explanation:** The read succeeds and returns an empty string. Its byte length is zero, so the strict failure check is false.

### 39. What is the final file content and return value if the first write succeeds?

~~~php
<?php

file_put_contents('log.txt', 'A');
$bytes = file_put_contents('log.txt', 'B', FILE_APPEND);

echo $bytes . ':' . file_get_contents('log.txt');
~~~

**Answer:**

~~~text
1:AB
~~~

**Explanation:** The second call appends one byte. The return value is the number of bytes written by that call.

### 40. What names are printed if the directory contains **b.txt** and **a.txt**, and the code skips the dot entries?

~~~php
<?php

$entries = scandir('storage');

foreach ($entries as $entry) {
    if ($entry !== '.' && $entry !== '..') {
        echo $entry . "\n";
    }
}
~~~

**Answer:**

~~~text
a.txt
b.txt
~~~

**Explanation:** By default, **scandir()** sorts entries in ascending order. The code filters out **.** and **..**.

### 41. What does the loop print for a file with two lines, **red** and **blue**, each ending in a newline?

~~~php
<?php

$handle = fopen('colors.txt', 'rb');

while (($line = fgets($handle)) !== false) {
    echo strtoupper($line);
}

fclose($handle);
~~~

**Answer:**

~~~text
RED
BLUE
~~~

**Explanation:** **fgets()** returns each line including its line ending, and **strtoupper()** changes the letters while preserving the newline.

### 42. What happens when **notes.txt** already exists and this open is attempted?

~~~php
<?php

$handle = fopen('notes.txt', 'x');

var_dump($handle);
~~~

**Answer:** **fopen()** fails and returns **false**; PHP also emits a warning unless the error is handled or suppressed.

**Explanation:** The **x** mode is for exclusive creation and fails when the path already exists. Production code should check the result and report the failure.

## Coding questions

### 43. Write a safe small-file read that reports a clear failure.

**Answer:**

~~~php
<?php

$path = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'message.txt';
$contents = file_get_contents($path);

if ($contents === false) {
    throw new RuntimeException('Could not read the message file.');
}

echo $contents;
~~~

**Explanation:** **__DIR__** makes the path independent of the current working directory. The strict comparison distinguishes read failure from an empty file.

### 44. Create a nested directory safely when another process might create it at the same time.

**Answer:**

~~~php
<?php

$directory = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'reports';

if (
    !is_dir($directory)
    && !mkdir($directory, 0770, true)
    && !is_dir($directory)
) {
    throw new RuntimeException('Could not create the reports directory.');
}
~~~

**Explanation:** Recursive **mkdir()** creates missing parent directories. The final **is_dir()** check accepts the case where a concurrent process created the target after the first check.

### 45. Append a line to a file and handle write failure.

**Answer:**

~~~php
<?php

$path = __DIR__ . DIRECTORY_SEPARATOR . 'storage' . DIRECTORY_SEPARATOR . 'events.log';
$bytesWritten = file_put_contents(
    $path,
    "Report created.\n",
    FILE_APPEND | LOCK_EX
);

if ($bytesWritten === false) {
    throw new RuntimeException('Could not append to the event log.');
}
~~~

**Explanation:** **FILE_APPEND** preserves existing content and adds the new line. **LOCK_EX** requests a cooperative exclusive lock during the write; it is not a transaction or a guarantee against programs that ignore file locks.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [052 - Text Encoding, Localization, and Internationalization](../lessons/052-text-encoding-localization-and-internationalization.md) | [Read lesson](../lessons/053-files-directories-and-filesystem-operations.md) | [Open example](../examples/053-files-directories-and-filesystem-operations.md) | [054 - Streams, Wrappers, and Contexts](../lessons/054-streams-wrappers-and-contexts.md) |
