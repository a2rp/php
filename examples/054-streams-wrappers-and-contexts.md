# 054 - Streams, Wrappers, and Contexts: Complete Example

This example writes to and reads from a temporary stream, inspects stream metadata and available wrappers, configures an HTTP context without making a request, and transforms data with a write filter.

## Goal

See how a stream handle, wrapper, context, and filter work together while keeping the program self-contained and predictable.

## Requirements

- PHP 8.1 or later.
- The built-in **php://** and **file://** wrappers.
- The built-in **string.toupper** stream filter.
- Run the example from the command line.

The script does not contact a remote server.

## Complete code

Save this as **index.php**:

~~~php
<?php

declare(strict_types=1);

// A temporary stream behaves like a seekable file but does not create
// a permanent application file.
$buffer = fopen('php://temp/maxmemory:1024', 'w+b');

if ($buffer === false) {
    throw new RuntimeException('Could not open a temporary stream.');
}

try {
    $written = fwrite($buffer, "alpha\nbeta\n");

    if ($written === false) {
        throw new RuntimeException('Could not write to the temporary stream.');
    }

    rewind($buffer);

    $firstLine = fgets($buffer);

    if ($firstLine === false) {
        throw new RuntimeException('Could not read the first line.');
    }

    $remaining = stream_get_contents($buffer);

    if ($remaining === false) {
        throw new RuntimeException('Could not read the rest of the stream.');
    }

    $metadata = stream_get_meta_data($buffer);

    echo 'First chunk: ' . rtrim($firstLine, "\r\n") . "\n";
    echo 'Remaining: ' . rtrim($remaining, "\r\n") . "\n";
    echo 'Temporary stream seekable: '
        . ($metadata['seekable'] ? 'yes' : 'no')
        . "\n";
} finally {
    fclose($buffer);
}

// Wrapper and filter availability is determined by this PHP installation.
$wrappers = stream_get_wrappers();
$filters = stream_get_filters();

echo 'Wrapper availability: '
    . 'file=' . (in_array('file', $wrappers, true) ? 'yes' : 'no')
    . ', php=' . (in_array('php', $wrappers, true) ? 'yes' : 'no')
    . "\n";

if (!in_array('string.toupper', $filters, true)) {
    throw new RuntimeException('The string.toupper filter is unavailable.');
}

// Contexts store wrapper-specific options. Creating one does not send a request.
$httpContext = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => "Accept: text/plain\r\n",
        'timeout' => 5.0,
    ],
]);

$httpOptions = stream_context_get_options($httpContext);

echo 'HTTP method in context: ' . $httpOptions['http']['method'] . "\n";
echo 'HTTP timeout in context: '
    . (string) $httpOptions['http']['timeout']
    . "\n";

// A write filter transforms data before it is stored in this temporary stream.
$filtered = fopen('php://temp', 'w+b');

if ($filtered === false) {
    throw new RuntimeException('Could not open the filtered stream.');
}

try {
    $filter = stream_filter_append(
        $filtered,
        'string.toupper',
        STREAM_FILTER_WRITE
    );

    if ($filter === false) {
        throw new RuntimeException('Could not attach the uppercase filter.');
    }

    $written = fwrite($filtered, 'Stream filters work.');

    if ($written === false) {
        throw new RuntimeException('Could not write filtered data.');
    }

    if (!fflush($filtered)) {
        throw new RuntimeException('Could not flush the filtered stream.');
    }

    rewind($filtered);

    $transformed = stream_get_contents($filtered);

    if ($transformed === false) {
        throw new RuntimeException('Could not read transformed data.');
    }

    echo 'Filtered stream: ' . $transformed . "\n";
} finally {
    fclose($filtered);
}
~~~

## Expected output

~~~text
First chunk: alpha
Remaining: beta
Temporary stream seekable: yes
Wrapper availability: file=yes, php=yes
HTTP method in context: GET
HTTP timeout in context: 5
Filtered stream: STREAM FILTERS WORK.
~~~

## How it works

- **php://temp/maxmemory:1024** opens a temporary read/write stream with a 1 KB memory threshold. It can use temporary-file storage if its data grows past that threshold.
- **fwrite()** puts two lines in the stream. **rewind()** resets the position, **fgets()** reads the first line, and **stream_get_contents()** reads from the new current position.
- **stream_get_meta_data()** reports properties of the open stream, including whether it can be seeked.
- **stream_get_wrappers()** and **stream_get_filters()** report what this PHP process currently provides.
- **stream_context_create()** stores HTTP wrapper options. The program inspects them but never opens an HTTP URL, so no network request is sent.
- **stream_filter_append()** attaches **string.toupper** to the write chain. Data is uppercased before it is stored.
- **fflush()** sends pending output through the filter. After rewinding, the script reads the transformed bytes back from the temporary stream.
- Both handles are closed in **finally** blocks.

## Try it yourself

1. Change **alpha** and **beta** and observe the first-line and remaining-content split.
2. Change the temporary stream's **maxmemory** threshold and write a larger payload.
3. Compare the metadata of **php://temp** and a local file stream.
4. Add an HTTP header to the context and inspect it with **stream_context_get_options()**.
5. Attach **string.toupper** to the read chain instead of the write chain and compare the result.
6. Check for **string.rot13** with **stream_get_filters()** and try it as a read filter.
7. Use **fwrite(STDOUT, $text)** after attaching a filter to **STDOUT**; compare that with **echo $text**.
8. Inspect wrappers and filters on another PHP installation; the available set can differ.

## Common mistakes

- Assuming a stream context opens a resource or sends a request by itself.
- Assuming all streams are seekable like regular files.
- Forgetting that **stream_get_contents()** starts at the current position.
- Attaching a filter after the data has already passed through the stream.
- Assuming a filter affects **echo** or **print** when attached to **STDOUT**.
- Assuming every PHP build has the same wrappers and filters.
- Treating wrapper names or URLs from users as trusted input.
- Forgetting to close stream handles after an exception.
- Assuming **php://temp** always stores all content in memory.

## Key takeaways

- Streams provide a common interface for different data sources.
- Wrappers identify the source or protocol; contexts supply wrapper-specific options.
- Filters transform data as it passes through read or write chains.
- Check stream operations, inspect capabilities when needed, and close handles.
- Validate remote resource identifiers and configure timeouts before making network requests.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [053 - Files, Directories, and Filesystem Operations](../lessons/053-files-directories-and-filesystem-operations.md) | [Read lesson](../lessons/054-streams-wrappers-and-contexts.md) | [Open Q&A](../qnas/054-streams-wrappers-and-contexts.md) | [055 - Encoding and Decoding JSON](../lessons/055-encoding-and-decoding-json.md) |
