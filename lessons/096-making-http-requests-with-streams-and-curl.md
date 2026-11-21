# 096 - Making HTTP requests with streams and cURL

## Learning goals

By the end of this lesson, you will be able to:

- Choose between PHP's HTTP stream wrapper and the cURL extension.
- Configure an HTTP method, headers, body, timeout, and redirect policy.
- Distinguish a network or transfer failure from an HTTP error response.
- Read the response body, headers, and status code safely.
- Send JSON to an HTTP endpoint and decode a JSON response.
- Recognize the security risks of requesting user-controlled URLs.

## 1. PHP as an HTTP client

A PHP application can send requests to another HTTP service. Common reasons include calling an API, checking an internal service, or sending data to a remote system.

Two built-in PHP options are:

- **HTTP streams:** The HTTP wrapper used by functions such as file_get_contents(). It can be enough for a small request with basic controls.
- **cURL:** The cURL extension gives more control over timeouts, headers, methods, transfers, redirects, and diagnostics. It must be enabled in the PHP installation.

In both cases, the application must handle transport errors, inspect the HTTP status, validate the response, and apply time limits. A completed request does not mean the server returned a successful status.

## 2. Use an HTTP stream context

A stream context lets a request configure HTTP options. This example asks a local service for JSON:

~~~php
$url = 'http://127.0.0.1:8000/mock-api.php';

$context = stream_context_create([
    'http' => [
        'method' => 'GET',
        'header' => [
            'Accept: application/json',
        ],
        'timeout' => 5.0,
        'ignore_errors' => true,
        'follow_location' => 0,
        'max_redirects' => 0,
        'protocol_version' => 1.1,
    ],
    'ssl' => [
        'verify_peer' => true,
        'verify_peer_name' => true,
    ],
]);

$body = file_get_contents($url, false, $context);
$responseHeaders = $http_response_header ?? [];
~~~

The HTTP context options configure the method, request headers, optional request content, timeout, and redirect behavior. The ignore_errors option lets PHP return the response body for HTTP error statuses such as 404 instead of treating the status itself as a read failure.

The $http_response_header variable contains the response headers for the request. It is created in the scope where the HTTP stream operation happens. Parse the status line rather than assuming that a non-false body means a successful response.

With HTTPS, peer and peer-name verification should remain enabled. Do not make a certificate error disappear by disabling verification.

## 3. Use cURL for more control

The cURL extension provides a handle-based interface for building a request and inspecting the result:

~~~php
$handle = curl_init('http://127.0.0.1:8000/mock-api.php');

curl_setopt_array($handle, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
        'Accept: application/json',
    ],
    CURLOPT_CONNECTTIMEOUT => 3,
    CURLOPT_TIMEOUT => 8,
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_SSL_VERIFYPEER => true,
    CURLOPT_SSL_VERIFYHOST => 2,
]);

$body = curl_exec($handle);
$status = curl_getinfo($handle, CURLINFO_RESPONSE_CODE);
$error = curl_error($handle);

curl_close($handle);
~~~

CURLOPT_RETURNTRANSFER returns the response body instead of printing it. Set both a connection timeout and a total timeout so a slow or unreachable service cannot hold a PHP worker indefinitely.

For an HTTPS request, certificate verification should stay enabled. CURLOPT_FOLLOWLOCATION is disabled above, so the client does not silently request a redirect target. If an application follows redirects, it must consider where each redirect can send the server.

## 4. Separate transport errors from HTTP statuses

A transport error means the request could not be completed as an HTTP exchange. Examples include DNS failure, a refused connection, a timeout, or a TLS verification failure.

An HTTP status is the remote server's response after it receives the request. A 404 or 500 response is still a completed HTTP exchange. In cURL, curl_exec() can return the response body for a 404; inspect the status separately with curl_getinfo().

A useful response-handling order is:

1. Check whether the transport operation failed.
2. Retrieve the final HTTP status and response headers.
3. Decide which status codes the application accepts.
4. Decode the body only after checking its expected format and size.
5. Handle invalid JSON, empty bodies, and unexpected responses without exposing internals.

Do not rely on CURLOPT_FAILONERROR as a substitute for status handling. Your application should make its expected status-code policy explicit.

## 5. Send JSON with a POST request

For JSON, encode the data and send a matching content type. A cURL request can be configured like this:

~~~php
$payload = json_encode(
    ['message' => 'Hello from PHP'],
    JSON_THROW_ON_ERROR
);

curl_setopt_array($handle, [
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => $payload,
    CURLOPT_HTTPHEADER => [
        'Accept: application/json',
        'Content-Type: application/json',
    ],
]);
~~~

For an HTTP stream, put the encoded string in the context's content option and send the same headers:

~~~php
$context = stream_context_create([
    'http' => [
        'method' => 'POST',
        'header' => [
            'Accept: application/json',
            'Content-Type: application/json',
        ],
        'content' => $payload,
        'timeout' => 5.0,
        'ignore_errors' => true,
        'follow_location' => 0,
    ],
]);
~~~

The next chapter focuses on building JSON and REST APIs. Here, the goal is to understand how a PHP client sends and receives an HTTP exchange.

## 6. Construct URLs and headers carefully

Encode query values with http_build_query() rather than joining untrusted strings by hand:

~~~php
$query = http_build_query([
    'page' => 2,
    'search' => 'blue shirts',
]);

$url = 'http://127.0.0.1:8000/mock-api.php?' . $query;
~~~

Use a fixed base URL for a known service and validate values inserted into its path or query. Send only the headers the service needs. Do not place passwords, API keys, or tokens in the URL, where they may appear in logs and monitoring systems.

## 7. Protect requests from SSRF

If a server makes a request to a URL supplied by a user, that user may try to reach local services, cloud metadata endpoints, or private network hosts from the PHP server. This is called server-side request forgery (SSRF).

Avoid arbitrary user-provided destinations. Prefer a fixed service URL or an explicit hostname allowlist. If a feature must accept URLs, validate the scheme and resolved addresses, reject loopback/private/link-local destinations, account for DNS changes, and apply outbound network restrictions. Redirects must be disabled or each target must be validated again.

A hostname check alone may not be enough because DNS can resolve to a different address later. Network-level egress controls provide an additional boundary.

## 8. Bound response size and time

A remote endpoint can be slow or return a very large response. Set connection and total timeouts, cap bytes read, and avoid loading unbounded response bodies into memory. For large downloads, stream to a controlled destination and enforce a maximum size as the data arrives.

Also consider retry behavior. Retrying a GET may be safe for many APIs, while retrying a payment or other state-changing POST can perform the action more than once. Use idempotency keys or an explicit service contract where retries are needed.

## 9. Handle credentials and sensitive data

Use HTTPS for credentials and private data. Keep certificate verification enabled, store API secrets outside source control, and send them only to the intended service. Do not log authorization headers or full sensitive request bodies.

Do not forward a user's session cookie to another service. Pass only the service credential required for that request, using the service's supported authentication method.

## Key takeaways

- Choose HTTP streams for simple requests and cURL when the application needs more transfer controls.
- Set finite timeouts, request headers, and an explicit redirect policy.
- A transport error and an HTTP 4xx/5xx response are different outcomes.
- Inspect the HTTP status and validate the response body before using it.
- Keep TLS certificate verification enabled.
- Treat user-controlled URLs as an SSRF risk and constrain outbound destinations.
- Bound response sizes and avoid unsafe retries of state-changing requests.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [095 - Validating uploads and storing files safely](095-validating-uploads-and-storing-files-safely.md) | [Q&A](../qnas/096-making-http-requests-with-streams-and-curl.md) | [Example](../examples/096-making-http-requests-with-streams-and-curl.md) | [097 - Building JSON and REST APIs](097-building-json-and-rest-apis.md) |