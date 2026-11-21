# 096 - Making HTTP requests with streams and cURL: Example

## Goal

Run a local JSON endpoint and call it from a PHP CLI client using either HTTP streams or cURL. Compare transport failures with HTTP error statuses, send a JSON POST body, and keep response handling bounded.

## Question

How can a PHP client make a timed HTTP request, send JSON, read the response status and body, and handle failures without confusing a server error with a network error?

## Complete example

Create the following two files in a small project:

~~~text
project/
├── client.php
└── public/
    └── mock-api.php
~~~

### 1. Local endpoint: public/mock-api.php

This endpoint is a local learning target. It has no database and accepts only a small JSON request.

~~~php
<?php

declare(strict_types=1);

header('Content-Type: application/json; charset=UTF-8');
header('Cache-Control: no-store');
header('X-Content-Type-Options: nosniff');

function sendJson(int $status, array $payload): void
{
    http_response_code($status);
    echo json_encode(
        $payload,
        JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES
    );
    exit;
}

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($method === 'GET') {
    $mode = $_GET['mode'] ?? 'ok';

    if ($mode === 'missing') {
        sendJson(404, ['error' => 'item_not_found']);
    }

    sendJson(200, [
        'ok' => true,
        'message' => 'Local endpoint is ready.',
    ]);
}

if ($method === 'POST') {
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    $mediaType = strtolower(trim(explode(';', $contentType, 2)[0]));

    if ($mediaType !== 'application/json') {
        sendJson(415, ['error' => 'json_required']);
    }

    $rawBody = file_get_contents('php://input');

    if (!is_string($rawBody)) {
        sendJson(400, ['error' => 'request_body_unavailable']);
    }

    try {
        $data = json_decode($rawBody, true, 512, JSON_THROW_ON_ERROR);
    } catch (JsonException) {
        sendJson(400, ['error' => 'invalid_json']);
    }

    if (!is_array($data)) {
        sendJson(400, ['error' => 'json_object_required']);
    }

    sendJson(201, [
        'created' => true,
        'received' => $data,
    ]);
}

header('Allow: GET, POST');
sendJson(405, ['error' => 'method_not_allowed']);
~~~

### 2. Client: client.php

Choose streams or cURL as the first command-line argument. The client only requests the local mock endpoint; it does not accept a user-supplied URL.

~~~php
<?php

declare(strict_types=1);

const MAX_RESPONSE_BYTES = 65536;

function findHttpStatus(array $headers): ?int
{
    $status = null;

    foreach ($headers as $header) {
        if (preg_match('/^HTTP\/\S+\s+([0-9]{3})(?:\s|$)/', $header, $matches) === 1) {
            $status = (int) $matches[1];
        }
    }

    return $status;
}

/**
 * @return array{status: int, body: string}
 */
function requestWithStreams(string $url, string $method, ?string $jsonBody): array
{
    $headers = ['Accept: application/json'];

    $httpOptions = [
        'method' => $method,
        'header' => $headers,
        'timeout' => 5.0,
        'ignore_errors' => true,
        'follow_location' => 0,
        'max_redirects' => 0,
        'protocol_version' => 1.1,
    ];

    if ($jsonBody !== null) {
        $httpOptions['header'][] = 'Content-Type: application/json';
        $httpOptions['content'] = $jsonBody;
    }

    $context = stream_context_create([
        'http' => $httpOptions,
        'ssl' => [
            'verify_peer' => true,
            'verify_peer_name' => true,
        ],
    ]);

    $stream = @fopen($url, 'r', false, $context);
    $responseHeaders = $http_response_header ?? [];

    if ($stream === false) {
        throw new RuntimeException('The HTTP stream request could not be opened.');
    }

    $body = stream_get_contents($stream, MAX_RESPONSE_BYTES + 1);
    fclose($stream);

    if (!is_string($body)) {
        throw new RuntimeException('The HTTP response body could not be read.');
    }

    if (strlen($body) > MAX_RESPONSE_BYTES) {
        throw new RuntimeException('The HTTP response exceeded the size limit.');
    }

    $status = findHttpStatus($responseHeaders);

    if ($status === null) {
        throw new RuntimeException('The HTTP response did not include a status line.');
    }

    return ['status' => $status, 'body' => $body];
}

/**
 * @return array{status: int, body: string}
 */
function requestWithCurl(string $url, string $method, ?string $jsonBody): array
{
    if (!extension_loaded('curl')) {
        throw new RuntimeException('The PHP cURL extension is not enabled.');
    }

    $handle = curl_init($url);

    if ($handle === false) {
        throw new RuntimeException('The cURL request could not be initialized.');
    }

    $responseBody = '';
    $responseTooLarge = false;
    $headers = ['Accept: application/json'];

    $options = [
        CURLOPT_CONNECTTIMEOUT => 3,
        CURLOPT_TIMEOUT => 8,
        CURLOPT_FOLLOWLOCATION => false,
        CURLOPT_SSL_VERIFYPEER => true,
        CURLOPT_SSL_VERIFYHOST => 2,
        CURLOPT_HTTPHEADER => $headers,
        CURLOPT_WRITEFUNCTION => static function ($curlHandle, string $chunk) use (&$responseBody, &$responseTooLarge): int {
            $chunkLength = strlen($chunk);

            if (strlen($responseBody) + $chunkLength > MAX_RESPONSE_BYTES) {
                $responseTooLarge = true;
                return 0;
            }

            $responseBody .= $chunk;
            return $chunkLength;
        },
    ];

    if ($jsonBody !== null) {
        $options[CURLOPT_POST] = true;
        $options[CURLOPT_POSTFIELDS] = $jsonBody;
        $options[CURLOPT_HTTPHEADER][] = 'Content-Type: application/json';
    }

    if (!curl_setopt_array($handle, $options)) {
        curl_close($handle);
        throw new RuntimeException('The cURL options could not be applied.');
    }

    $result = curl_exec($handle);
    $status = curl_getinfo($handle, CURLINFO_RESPONSE_CODE);
    $error = curl_error($handle);
    curl_close($handle);

    if ($responseTooLarge) {
        throw new RuntimeException('The HTTP response exceeded the size limit.');
    }

    if ($result === false) {
        throw new RuntimeException('The cURL transfer failed: ' . $error);
    }

    if (!is_int($status)) {
        throw new RuntimeException('The HTTP response status could not be read.');
    }

    return ['status' => $status, 'body' => $responseBody];
}

if (PHP_SAPI !== 'cli') {
    http_response_code(403);
    exit("Run this client from the command line.\n");
}

$transport = $argv[1] ?? 'streams';
$mode = $argv[2] ?? 'ok';

if (!in_array($transport, ['streams', 'curl'], true)) {
    fwrite(STDERR, "Transport must be streams or curl.\n");
    exit(2);
}

if (!in_array($mode, ['ok', 'missing', 'post'], true)) {
    fwrite(STDERR, "Mode must be ok, missing, or post.\n");
    exit(2);
}

$url = 'http://127.0.0.1:8000/mock-api.php';
$method = $mode === 'post' ? 'POST' : 'GET';
$jsonBody = $mode === 'post'
    ? json_encode(['message' => 'Hello from PHP'], JSON_THROW_ON_ERROR)
    : null;

if ($mode === 'missing') {
    $url .= '?' . http_build_query(['mode' => 'missing']);
}

try {
    $response = $transport === 'curl'
        ? requestWithCurl($url, $method, $jsonBody)
        : requestWithStreams($url, $method, $jsonBody);

    $decoded = json_decode($response['body'], true, 512, JSON_THROW_ON_ERROR);

    echo 'HTTP status: ' . $response['status'] . PHP_EOL;
    echo json_encode(
        $decoded,
        JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES | JSON_THROW_ON_ERROR
    ) . PHP_EOL;

    if ($response['status'] < 200 || $response['status'] >= 300) {
        exit(1);
    }
} catch (Throwable $error) {
    fwrite(STDERR, 'Request failed: ' . $error->getMessage() . PHP_EOL);
    exit(1);
}
~~~

## Run it

Open a terminal in the project root and start the local endpoint:

~~~bash
php -S 127.0.0.1:8000 -t public
~~~

Open another terminal in the same project root and try the stream and cURL transports:

~~~bash
php client.php streams ok
php client.php curl ok
~~~

Send a JSON POST with cURL:

~~~bash
php client.php curl post
~~~

Request the mock 404 response. This is a completed HTTP exchange, so the client prints the status and JSON body, then exits with a non-zero status because 404 is outside its success policy:

~~~bash
php client.php streams missing
~~~

The cURL examples require the PHP cURL extension. The streams examples use PHP's HTTP wrapper.

## Expected behavior

| Command | Expected result |
| --- | --- |
| streams ok | Status 200 and a JSON message from the local endpoint |
| curl ok | Status 200 and the same JSON message |
| curl post | Status 201 and the JSON value received by the endpoint |
| streams missing | Status 404 and an item-not-found body; client exits with status 1 |
| curl missing | Status 404 and an item-not-found body; client exits with status 1 |
| Endpoint is stopped or unavailable | A transport error is reported |
| The PHP cURL extension is disabled | The streams commands still work; cURL reports that its extension is unavailable |
| A response exceeds 64 KiB | The client stops reading and reports a response-size error |

## How it works

1. **Local destination:** The URL is fixed to 127.0.0.1. Do not turn this into a fetch-any-URL feature without an SSRF defense.
2. **Stream request:** The context sets method, headers, a five-second timeout, and disables redirects. The response is read with a 64 KiB cap.
3. **cURL request:** The client sets connection and total timeouts, captures the body with a write callback, and refuses to follow redirects.
4. **TLS:** Both clients keep peer verification enabled. The local example uses HTTP only to avoid requiring a local certificate; use HTTPS for real credentials or private data.
5. **HTTP status:** The streams client parses the status line; the cURL client reads CURLINFO_RESPONSE_CODE. Both keep status handling separate from transport errors.
6. **Error response bodies:** The stream context uses ignore_errors so the mock 404 body remains readable. cURL returns HTTP error bodies because the example does not enable CURLOPT_FAILONERROR.
7. **JSON:** The client checks that the body decodes as JSON. The mock endpoint returns JSON for both success and error responses.
8. **POST body:** The client uses JSON_THROW_ON_ERROR, sends Content-Type: application/json, and the endpoint returns status 201 for a valid JSON body.

## Important notes

- This mock server is for local learning only and does not represent an authenticated production API.
- Never disable TLS peer or hostname verification to silence a certificate problem.
- Do not accept arbitrary user URLs. Use a fixed service destination or a strict allowlist, and validate every redirect target.
- A real service client should validate the response content type and schema, redact credentials from logs, and define safe retry behavior.
- A response-size cap is included here; production limits should match the data expected from the service.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [095 - Validating uploads and storing files safely](../lessons/095-validating-uploads-and-storing-files-safely.md) | [Q&A](../qnas/096-making-http-requests-with-streams-and-curl.md) | Example | [097 - Building JSON and REST APIs](../lessons/097-building-json-and-rest-apis.md) |