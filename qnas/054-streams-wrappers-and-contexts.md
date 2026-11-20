# 054 - Streams, Wrappers, and Contexts: Q&A

Review stream handles, wrapper schemes, contexts, filters, temporary streams, and remote-resource safety. Try each question before reading its answer. See the [chapter lesson](../lessons/054-streams-wrappers-and-contexts.md).

## Multiple-choice questions

### 1. What does a PHP stream provide?

A. A common interface for reading and writing data from different sources
B. A database table definition
C. A Unicode normalization form
D. A way to disable file permissions

**Answer:** A. A common interface for reading and writing data from different sources.

**Explanation:** Streams can represent files, temporary buffers, network resources, and other sources behind a consistent handle-based interface.

### 2. What does a stream wrapper identify?

A. The PHP variable type
B. The resource type or protocol handled by a stream
C. The user's language
D. A file's permissions only

**Answer:** B. The resource type or protocol handled by a stream.

**Explanation:** Schemes such as **file://**, **php://**, and **https://** select the wrapper that handles the resource.

### 3. Which function lists wrappers registered in the current PHP process?

A. **stream_get_filters()**
B. **stream_get_wrappers()**
C. **stream_context_get_options()**
D. **stream_get_contents()**

**Answer:** B. **stream_get_wrappers()**

**Explanation:** The set of available wrappers can depend on the PHP build and loaded extensions.

### 4. What is the main difference between **php://memory** and **php://temp**?

A. **php://memory** always stores data in memory; **php://temp** can spill to a temporary file after a threshold
B. **php://temp** is read-only
C. **php://memory** can only be used by web requests
D. They are both permanent application files

**Answer:** A. **php://memory** always stores data in memory; **php://temp** can spill to a temporary file after a threshold.

**Explanation:** **php://temp** is useful when file-like temporary storage may grow beyond a chosen memory budget.

### 5. Which wrapper exposes the raw HTTP request body in supported request contexts?

A. **php://output**
B. **php://input**
C. **php://temp**
D. **file://input**

**Answer:** B. **php://input**

**Explanation:** It provides access to raw request-body data, subject to PHP's request parsing behavior and limitations.

### 6. What are **STDIN**, **STDOUT**, and **STDERR**?

A. Predefined command-line stream constants
B. HTTP response headers
C. File mode strings
D. Stream filters

**Answer:** A. Predefined command-line stream constants.

**Explanation:** They represent standard input, standard output, and standard error in the CLI SAPI.

### 7. How are stream context options grouped?

A. By PHP variable name
B. By wrapper name, then option name
C. By file extension
D. By directory depth

**Answer:** B. By wrapper name, then option name.

**Explanation:** For example, HTTP request settings are nested under the **http** key.

### 8. What does **stream_context_create()** return?

A. A stream context resource
B. An open file handle
C. A network response body
D. A registered wrapper name

**Answer:** A. A stream context resource.

**Explanation:** The context holds options and parameters to pass to stream operations that support it. It does not itself open a resource.

### 9. What does a stream filter do?

A. Transforms data while it is read from or written to a stream
B. Changes a file's permissions
C. Creates a new wrapper automatically
D. Guarantees that input is trusted

**Answer:** A. Transforms data while it is read from or written to a stream.

**Explanation:** Filters operate on data as it moves through the stream. They do not replace validation or destination-specific escaping.

### 10. Which mode constant selects the write filter chain?

A. **STREAM_FILTER_READ**
B. **STREAM_FILTER_WRITE**
C. **LOCK_EX**
D. **FILE_APPEND**

**Answer:** B. **STREAM_FILTER_WRITE**

**Explanation:** It applies the filter to data written through the stream.

### 11. Which function lists filters available in the current PHP process?

A. **stream_get_filters()**
B. **stream_get_wrappers()**
C. **stream_context_get_options()**
D. **filter_list()**

**Answer:** A. **stream_get_filters()**

**Explanation:** Available filters depend on the PHP installation and registered extensions.

### 12. Why might a stream not support **fseek()**?

A. Some streams are not seekable, such as many network streams
B. PHP streams never have a position
C. **fseek()** works only on directories
D. All wrappers disable seeking by default

**Answer:** A. Some streams are not seekable, such as many network streams.

**Explanation:** Stream capabilities vary by resource. Inspect metadata or handle the operation's failure instead of assuming a disk-file behavior.

### 13. What does the HTTP context's **timeout** option configure?

A. The maximum number of bytes in the response
B. The time limit for a network operation
C. The file permission mode
D. The PHP script's total execution time in every case

**Answer:** B. The time limit for a network operation.

**Explanation:** A wrapper-specific timeout helps prevent a request from waiting indefinitely. It is distinct from the script's execution-time setting.

### 14. What is **php://filter**?

A. A meta-wrapper that can apply filters while opening another stream
B. A directory iterator
C. A permanent cache
D. A context option for the local filesystem

**Answer:** A. A meta-wrapper that can apply filters while opening another stream.

**Explanation:** It can be useful with functions that open and consume a resource in one operation. Construct its target carefully.

### 15. What is the safest general approach when the application expects a local file?

A. Accept any user-provided URL and pass it to **fopen()**
B. Validate or map input to an approved local resource
C. Assume every string without a slash is a local path
D. Disable return-value checks

**Answer:** B. Validate or map input to an approved local resource.

**Explanation:** A scheme can select a network or special wrapper. Treat untrusted resource identifiers as a security boundary.

## Short-answer questions

### 16. What should code do after **fopen()** returns a stream handle?

**Answer:** Use the handle for the intended operations and close it when finished, typically in a **finally** block.

**Explanation:** This releases the underlying resource even if processing throws an exception.

### 17. Explain the difference between a wrapper and a stream context.

**Answer:** A wrapper handles a resource type or protocol; a context supplies options and parameters that modify a compatible operation using that wrapper.

**Explanation:** An HTTP context does not change how the local-file wrapper behaves.

### 18. Why does filter order matter?

**Answer:** Each filter receives data transformed by earlier filters, so changing the order can change the final result.

**Explanation:** For example, encoding and compression transforms generally need to occur in a compatible order.

### 19. What is the default wrapper for an ordinary local filesystem path?

**Answer:** The **file://** wrapper.

**Explanation:** A normal path such as **storage/data.txt** is handled by the local filesystem wrapper.

### 20. Why should network stream operations use an explicit timeout?

**Answer:** To bound how long the operation may wait for a remote resource.

**Explanation:** Network services can be slow or unavailable. Also handle failures and validate response status and content.

### 21. Name one limitation of **php://input**.

**Answer:** With PHP's normal multipart form parsing enabled, it is not available for multipart POST bodies.

**Explanation:** Multipart file uploads are parsed into **$_FILES** and related request data; raw-body handling has different behavior.

### 22. What does **stream_get_contents($stream)** read?

**Answer:** It reads the remaining contents from the stream's current position.

**Explanation:** It does not automatically rewind the stream. The optional length can limit how many bytes are read.

### 23. Why might **stream_get_wrappers()** differ across two servers?

**Answer:** PHP builds and loaded extensions can register different wrappers.

**Explanation:** Check the runtime's available wrappers rather than assuming every deployment has the same set.

### 24. When should a filter be attached to a stream?

**Answer:** Before the relevant data is read or written through the stream.

**Explanation:** Data already processed may not be transformed retroactively. Buffered data can also affect whether a newly attached read filter sees prior bytes.

### 25. What does **fread()** return at end of a stream?

**Answer:** It returns an empty string when there are no more bytes, and **false** on failure.

**Explanation:** Use strict checks and consider the stream's EOF state when reading chunks.

### 26. Why is a successful HTTP body read not sufficient to decide whether a request succeeded?

**Answer:** The HTTP response may have an error status even when a body was read successfully.

**Explanation:** Inspect response status and validate the body for the operation's expected format and meaning.

### 27. What risk comes from passing a user-controlled scheme or URL to a stream function?

**Answer:** The input could select an unintended wrapper or remote destination, potentially creating a server-side request forgery or data-access risk.

**Explanation:** Validate the scheme and destination with an allowlist, and keep local path access constrained.

### 28. Why might a filter attached to **STDOUT** not transform calls to **echo**?

**Answer:** **echo** and **print** use PHP's output mechanism and do not write through the **STDOUT** stream handle.

**Explanation:** A stream filter applies to writes that pass through the filtered handle, such as **fwrite(STDOUT, $text)**.

### 29. When is **php://temp** useful?

**Answer:** When code needs temporary file-like storage that can remain in memory for small data and spill to a temporary file after a threshold.

**Explanation:** It can avoid a permanent application file while supporting stream operations such as writing, rewinding, and reading.

## True or false

### 30. Every PHP stream supports seeking and rewinding.

**Answer:** False.

**Explanation:** Some streams, especially network streams, may not be seekable.

### 31. Creating a stream context automatically opens a stream.

**Answer:** False.

**Explanation:** The context only stores options and parameters. It must be passed to a compatible operation that opens or uses a stream.

### 32. **php://temp** always stores all data in memory.

**Answer:** False.

**Explanation:** It can spill to a temporary file after its memory threshold is reached.

### 33. Stream filters can be attached to read and write operations.

**Answer:** True.

**Explanation:** Use modes such as **STREAM_FILTER_READ** and **STREAM_FILTER_WRITE** to select the chain.

### 34. A stream filter proves that input is valid and safe for HTML.

**Answer:** False.

**Explanation:** A filter transforms bytes or text; validation and context-specific output escaping remain separate tasks.

### 35. **STDIN**, **STDOUT**, and **STDERR** are intended as standard stream constants for CLI PHP.

**Answer:** True.

**Explanation:** Web requests use HTTP input/output mechanisms instead of the CLI's standard streams.

### 36. **stream_get_filters()** lists stream wrappers.

**Answer:** False.

**Explanation:** It lists filters. Use **stream_get_wrappers()** for wrappers.

### 37. The **allow_url_fopen** configuration setting can affect network URL wrappers.

**Answer:** True.

**Explanation:** URL wrapper access can be restricted by PHP configuration; this setting does not control every local filesystem operation.

## Predict the output

### 38. What does this print?

~~~php
<?php

$stream = fopen('php://temp', 'w+b');
fwrite($stream, 'abc');
rewind($stream);

echo stream_get_contents($stream);
fclose($stream);
~~~

**Answer:**

~~~text
abc
~~~

**Explanation:** The script rewinds the temporary stream before reading, so the read starts at the beginning.

### 39. What does this print if the filter is available?

~~~php
<?php

$stream = fopen('php://temp', 'w+b');
stream_filter_append($stream, 'string.toupper', STREAM_FILTER_WRITE);
fwrite($stream, 'hello');
fflush($stream);
rewind($stream);

echo stream_get_contents($stream);
fclose($stream);
~~~

**Answer:**

~~~text
HELLO
~~~

**Explanation:** The write filter uppercases data before it is stored in the temporary stream.

### 40. What does this print?

~~~php
<?php

$stream = fopen('php://temp', 'w+b');
fwrite($stream, 'abcdef');
rewind($stream);

echo fread($stream, 2) . '|';
echo stream_get_contents($stream);

fclose($stream);
~~~

**Answer:**

~~~text
ab|cdef
~~~

**Explanation:** The first read advances the stream position by two bytes. **stream_get_contents()** then reads the remaining bytes.

### 41. What does this print?

~~~php
<?php

$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'timeout' => 5.0,
    ],
]);

$options = stream_context_get_options($context);

echo $options['http']['method'];
~~~

**Answer:**

~~~text
GET
~~~

**Explanation:** The context stores the wrapper-specific options; retrieving them does not perform an HTTP request.

### 42. What does this print if **string.rot13** is an available read filter?

~~~php
<?php

$stream = fopen('php://temp', 'w+b');
fwrite($stream, 'hello');
rewind($stream);
stream_filter_append($stream, 'string.rot13', STREAM_FILTER_READ);

echo stream_get_contents($stream);
fclose($stream);
~~~

**Answer:**

~~~text
uryyb
~~~

**Explanation:** The read filter transforms the stored bytes as the script reads them.

## Coding questions

### 43. Create an HTTP context with a method, header, and timeout, then show how to pass it to a request.

**Answer:**

~~~php
<?php

$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => "Accept: application/json\r\n",
        'timeout' => 5.0,
    ],
]);

$response = file_get_contents($trustedUrl, false, $context);

if ($response === false) {
    throw new RuntimeException('The HTTP stream request failed.');
}
~~~

**Explanation:** The URL should come from a trusted or validated source. In a real application, also inspect the HTTP status and validate the response body; a readable body alone does not guarantee application-level success.

### 44. Read text through a ROT13 filter attached to a temporary stream.

**Answer:**

~~~php
<?php

$stream = fopen('php://temp', 'w+b');

if ($stream === false) {
    throw new RuntimeException('Could not open the temporary stream.');
}

try {
    if (fwrite($stream, 'hello') === false) {
        throw new RuntimeException('Could not write to the stream.');
    }

    rewind($stream);

    $filter = stream_filter_append(
        $stream,
        'string.rot13',
        STREAM_FILTER_READ
    );

    if ($filter === false) {
        throw new RuntimeException('The ROT13 filter is unavailable.');
    }

    $result = stream_get_contents($stream);

    if ($result === false) {
        throw new RuntimeException('Could not read the stream.');
    }

    echo $result;
} finally {
    fclose($stream);
}
~~~

**Expected output:**

~~~text
uryyb
~~~

**Explanation:** The filter is attached before the content is read, so it transforms the bytes as they pass through the read chain.

### 45. Copy a stream into a temporary stream, rewind it, and read the result safely.

**Answer:**

~~~php
<?php

$temporary = fopen('php://temp', 'w+b');

if ($temporary === false) {
    throw new RuntimeException('Could not open a temporary stream.');
}

try {
    $bytes = fwrite($temporary, "one\ntwo\n");

    if ($bytes === false) {
        throw new RuntimeException('Could not write to the temporary stream.');
    }

    rewind($temporary);

    $contents = stream_get_contents($temporary);

    if ($contents === false) {
        throw new RuntimeException('Could not read the temporary stream.');
    }

    echo $contents;
} finally {
    fclose($temporary);
}
~~~

**Expected output:**

~~~text
one
two
~~~

**Explanation:** The script checks the handle and I/O return values, rewinds before reading, and closes the resource even if an operation throws.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [053 - Files, Directories, and Filesystem Operations](../lessons/053-files-directories-and-filesystem-operations.md) | [Read lesson](../lessons/054-streams-wrappers-and-contexts.md) | [Open example](../examples/054-streams-wrappers-and-contexts.md) | [055 - Encoding and Decoding JSON](../lessons/055-encoding-and-decoding-json.md) |
