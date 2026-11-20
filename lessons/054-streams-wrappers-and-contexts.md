# 054 - Streams, Wrappers, and Contexts

PHP uses streams as a common way to read and write data. A stream may connect to a local file, a temporary buffer, a network resource, or another registered source. Wrappers identify the resource type, contexts provide wrapper-specific options, and filters can transform data as it passes through a stream.

## What you will learn

- Understand a stream handle as an open data source with a position and metadata.
- Recognize common PHP stream wrappers and their schemes.
- Use **php://temp** and standard input/output streams.
- Configure stream behavior with a context.
- Apply filters while reading or writing.
- Check stream operations and close resources reliably.
- Avoid unsafe wrapper and remote-resource input.

## What is a stream?

A stream is a resource that PHP can open and process through a consistent interface. A stream handle has an underlying source or destination, an access mode, and often a current position.

For a local file, **fopen()** creates a stream handle:

~~~php
<?php

$handle = fopen(__DIR__ . '/message.txt', 'rb');

if ($handle === false) {
    throw new RuntimeException('Could not open the file.');
}

try {
    $contents = stream_get_contents($handle);

    if ($contents === false) {
        throw new RuntimeException('Could not read the stream.');
    }

    echo $contents;
} finally {
    fclose($handle);
}
~~~

After reading, the stream position advances. Functions such as **fread()**, **fgets()**, **fwrite()**, **fseek()**, and **rewind()** work with a handle. Not every kind of stream supports seeking, so check the stream's capabilities rather than assuming every resource behaves like a regular disk file.

A stream function can return **false**, return a byte count, or emit a warning depending on the operation. Check its documented return value and close handles when finished.

## Wrappers identify a stream source

A wrapper handles a resource identified by a scheme such as **file://**, **php://**, or **https://**. Many PHP functions use streams behind the scenes, including **fopen()**, **file_get_contents()**, and **copy()**.

| Wrapper | Typical use |
| --- | --- |
| **file://** | Local filesystem access; this is the default for ordinary paths |
| **php://input** | Raw HTTP request body, subject to PHP request parsing behavior |
| **php://output** | Write to PHP's output mechanism |
| **php://stdin**, **php://stdout**, **php://stderr** | Standard streams, especially in command-line programs |
| **php://memory** | A temporary stream stored entirely in memory |
| **php://temp** | A temporary stream that can spill from memory to a temporary file |
| **php://filter** | Apply filters while opening another stream |
| **http://**, **https://**, **ftp://** | Network resources when the relevant wrapper and configuration permit them |
| **phar://** | Read files from a PHP archive |

The available wrappers depend on the PHP installation and registered extensions. **stream_get_wrappers()** lists the wrappers available in the current process.

A scheme is not proof that a resource is safe. A value that looks like a filename may instead select a URL or another wrapper. Do not pass untrusted strings directly to stream functions when the application expects a local file. Validate the intended resource type and keep paths within the application's trusted boundaries.

## Use temporary and standard streams

**php://memory** keeps all stream data in memory. **php://temp** keeps data in memory up to a threshold, then can use a temporary file. The default threshold for **php://temp** is 2 MB; a maximum memory size can be supplied in the path.

~~~php
<?php

$stream = fopen('php://temp/maxmemory:1048576', 'w+b');

if ($stream === false) {
    throw new RuntimeException('Could not open the temporary stream.');
}

try {
    fwrite($stream, "first line\nsecond line\n");
    rewind($stream);

    while (($line = fgets($stream)) !== false) {
        echo $line;
    }
} finally {
    fclose($stream);
}
~~~

Temporary streams are useful when code needs a file-like handle without choosing a persistent application path. Closing the handle releases its temporary storage.

In command-line PHP, **STDIN**, **STDOUT**, and **STDERR** are predefined stream constants:

~~~php
<?php

fwrite(STDOUT, "Enter a name: ");
$name = fgets(STDIN);

if ($name === false) {
    fwrite(STDERR, "No input was received.\n");
    exit(1);
}

fwrite(STDOUT, 'Hello, ' . trim($name) . "\n");
~~~

These constants are intended for the CLI SAPI; web requests use the HTTP request and response mechanisms instead.

## Configure a stream context

A stream context is a set of options and parameters passed to a compatible stream operation. The options are grouped by wrapper name. A context does not itself open or validate a resource; the wrapper interprets the options when an operation uses that context.

For example, an HTTP context can describe a request method, headers, and timeout:

~~~php
<?php

$options = [
    'http' => [
        'method' => 'GET',
        'header' => "Accept: application/json\r\n",
        'timeout' => 5.0,
    ],
];

$context = stream_context_create($options);

// Pass the context to a compatible stream operation:
// $response = file_get_contents($trustedUrl, false, $context);
~~~

The HTTP options only apply to an HTTP stream. Options for another wrapper belong under that wrapper's key. Check the supported options for the wrapper you use.

Use explicit timeouts for network operations and handle transport failures and HTTP status codes. A successful stream read does not necessarily mean the HTTP response represents a successful application-level operation. Network access can also be restricted by PHP configuration such as **allow_url_fopen**.

**stream_context_get_options()** retrieves configured options. **stream_context_set_option()** can set an option after creating the context. Keep the context associated with the operation that needs it instead of silently changing process-wide defaults.

## Transform stream data with filters

A stream filter transforms data as it is read from or written to a stream. Filters can be attached to the read chain, the write chain, or both. The filter must be attached before the relevant data passes through the stream.

This example uppercases text as it is written to a temporary stream:

~~~php
<?php

$stream = fopen('php://temp', 'w+b');

if ($stream === false) {
    throw new RuntimeException('Could not open the temporary stream.');
}

try {
    $filter = stream_filter_append(
        $stream,
        'string.toupper',
        STREAM_FILTER_WRITE
    );

    if ($filter === false) {
        throw new RuntimeException('The uppercase filter is unavailable.');
    }

    $bytes = fwrite($stream, 'Stream filters transform data.');

    if ($bytes === false) {
        throw new RuntimeException('Could not write to the stream.');
    }

    fflush($stream);
    rewind($stream);

    $result = stream_get_contents($stream);

    if ($result === false) {
        throw new RuntimeException('Could not read the filtered data.');
    }

    echo $result;
} finally {
    fclose($stream);
}
~~~

The built-in filters available depend on the PHP installation. **stream_get_filters()** lists the registered filters. Filters can be chained, so order matters: changing data before another filter can produce a different result than applying the operations in the opposite order.

The **php://filter** wrapper can apply filters to a resource as it is opened. This is useful with functions that open and consume a stream in one operation. When using filter paths, construct the resource portion carefully and do not let untrusted input choose arbitrary filters or resources.

A filter is a data transformation, not a substitute for validating or escaping data for its final use.

## Stream errors and cleanup

A stream operation can fail because the resource is missing, permissions deny access, a network connection times out, a wrapper rejects an option, or a filter is unavailable.

Use return checks and **try/finally** for stream handles:

~~~php
<?php

$stream = fopen($path, 'rb');

if ($stream === false) {
    throw new RuntimeException('Could not open the stream.');
}

try {
    $chunk = fread($stream, 4096);

    if ($chunk === false) {
        throw new RuntimeException('Could not read from the stream.');
    }

    echo $chunk;
} finally {
    fclose($stream);
}
~~~

Do not assume every stream can be rewound or that an error will always throw an exception. The exact behavior depends on the function, wrapper, context, and PHP configuration.

## Remote wrappers and security

Network wrappers can make an HTTP request look like a file read, but the operation still has network behavior and security consequences.

- Do not pass a user-controlled URL directly to **file_get_contents()** or **fopen()**.
- Validate schemes and destinations against a clear allowlist.
- Consider server-side request forgery risks if a remote URL can reach internal services.
- Set timeouts and enforce response-size limits where appropriate.
- Check the HTTP response status and validate the response body.
- Use an HTTP client library when the application needs richer features such as retries, authentication, or detailed response handling.
- Do not use **php://filter** or another wrapper to bypass path or content validation.
- Keep raw request-body handling separate from parsed form uploads; **php://input** has request-type limitations.

## Common mistakes

- Treating every stream as a regular seekable local file.
- Forgetting to close a stream handle after an exception.
- Assuming wrappers and filters are identical on every PHP installation.
- Passing user-controlled paths, schemes, or URLs to stream functions.
- Expecting HTTP contexts to affect a local file wrapper.
- Treating a successful read as proof that an HTTP request succeeded semantically.
- Attaching a filter after data has already passed through the stream.
- Relying on **echo** or **print** to write through a filter attached to **STDOUT**; those constructs do not write through the standard output stream handle.
- Assuming **php://temp** always stays entirely in memory.
- Using a transformation filter as a substitute for input validation or output escaping.

## Try it yourself

1. Open **php://temp**, write multiple lines, rewind it, and read the lines back.
2. Compare **php://memory** and **php://temp** for a larger temporary payload.
3. Attach **string.toupper** to a write stream and inspect the stored bytes.
4. List the available wrappers with **stream_get_wrappers()**.
5. List available filters with **stream_get_filters()** and check whether a requested filter is installed.
6. Create an HTTP context with a method, header, and timeout, then inspect its options with **stream_context_get_options()** without making a network request.
7. Read command-line input from **STDIN** and send an error message to **STDERR**.
8. Try reading from a seekable file and a non-seekable network stream; compare which position operations are supported.

## Key takeaways

- A stream is the common interface for reading and writing data from different sources.
- A wrapper identifies and handles the resource type.
- A context passes wrapper-specific options to a stream operation.
- A filter transforms data as it moves through a stream.
- Stream capabilities, wrappers, filters, and error behavior can vary by resource and configuration.
- Validate resource identifiers, check results, use timeouts for network access, and close handles.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [053 - Files, Directories, and Filesystem Operations](053-files-directories-and-filesystem-operations.md) | [Open Q&A](../qnas/054-streams-wrappers-and-contexts.md) | [Open example](../examples/054-streams-wrappers-and-contexts.md) | [055 - Encoding and Decoding JSON](055-encoding-and-decoding-json.md) |
