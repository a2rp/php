# 084 - HTTP requests, responses, and the PHP request lifecycle

## Learning goals

By the end of this lesson, you will be able to:

- Describe the main parts of an HTTP request and response.
- Trace how a web request reaches a PHP script and returns to the client.
- Read request methods and common input sources from PHP superglobals.
- Distinguish query-string form data from JSON request bodies.
- Set a response status and headers before sending the response body.
- Return appropriate statuses for common outcomes.

## 1. What happens during an HTTP exchange?

HTTP is a request-and-response protocol. A client, usually a browser, sends a request to a server. The server returns a response containing a status, headers, and usually a body.

A request commonly includes:

| Part | Example | Purpose |
| --- | --- | --- |
| Method | `GET`, `POST` | Describes the requested operation |
| Target | `/products?id=42` | Identifies the path and may include query parameters |
| Headers | `Accept: text/html` | Provides metadata about the request |
| Body | Form fields or JSON | Carries submitted data, often with `POST` or `PUT` |

A response commonly includes:

| Part | Example | Purpose |
| --- | --- | --- |
| Status | `200 OK`, `404 Not Found` | Describes the result of the request |
| Headers | `Content-Type: text/html; charset=utf-8` | Describes the response and how to handle it |
| Body | HTML, JSON, or no content | Carries the returned representation when one is needed |

A `GET` request typically retrieves a representation. Methods such as `POST`, `PUT`, and `PATCH` are commonly used to submit or change data. The method itself does not make a request safe or trustworthy; applications must still validate and authorize operations.

## 2. The PHP request lifecycle

A simplified web request usually follows these steps:

```text
Client
  -> sends HTTP request
Web server
  -> selects a route or PHP entry point
PHP runtime through its server interface
  -> starts request execution and exposes request/server data
PHP application
  -> validates input, performs work, prepares status/headers/body
Web server
  -> sends the HTTP response
Client
  -> receives and interprets the response
```

The exact setup depends on the web server and PHP Server API (SAPI), such as PHP-FPM or a server module. The application normally handles one request execution at a time even when a long-running server process can serve multiple requests over its lifetime.

The PHP script does not normally send its response directly to the browser as each line is executed. PHP and the server gather the response status, headers, and body, then send them as the HTTP response. Output buffering can affect when body output is flushed, but application code should still set headers before producing output.

## 3. Read request information

PHP makes request and environment information available through superglobals. These arrays are available in every scope.

### Request method and path

```php
<?php

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$uri = $_SERVER['REQUEST_URI'] ?? '/';

printf("Method: %s\n", $method);
printf("Request URI: %s\n", $uri);
```

Server variables depend on the web server and SAPI; some entries may be unavailable. Use `??` or another check when an entry is not guaranteed.

### Query-string values: `$_GET`

For a URL such as `/products?id=42`, PHP makes the query parameter available in `$_GET`:

```php
$productId = $_GET['id'] ?? null;
```

`$_GET` is populated from the query string whenever one is present, regardless of the request method. The query string is not private: it can appear in browser history, server logs, and shared URLs. Avoid placing secrets in it.

### Form values: `$_POST`

For an HTML form submitted using `application/x-www-form-urlencoded` or `multipart/form-data`, PHP makes the submitted fields available in `$_POST`:

```php
$email = $_POST['email'] ?? '';
```

Request values are external input. Check that expected values exist, have the expected shape, and satisfy the application's rules. A field named with square brackets can produce an array, so do not assume every value is a string.

`$_POST` is not a universal reader for all POST bodies. For JSON or other content types, read the raw request body through `php://input` and decode it according to the declared content type.

### JSON request bodies

A JSON endpoint can read and decode the body like this:

```php
<?php

$rawBody = file_get_contents('php://input');
$data = json_decode($rawBody, true, flags: JSON_THROW_ON_ERROR);
```

`JSON_THROW_ON_ERROR` makes invalid JSON throw a `JsonException`, which application code should catch and turn into a suitable client error response. After decoding, validate the resulting structure and each value before using it.

### Other request data

- `$_SERVER` contains request, server, and execution-environment details. Availability varies by environment.
- `$_COOKIE` contains cookie values sent by the client.
- `$_FILES` contains upload metadata for supported multipart form uploads; uploaded files still need validation and safe handling.
- `$_REQUEST` may combine GET, POST, and COOKIE values according to configuration. Prefer the specific source (`$_GET`, `$_POST`, or `$_COOKIE`) so the origin is clear.

## 4. Choose a response status

The status communicates the outcome. Common examples include:

| Status | Typical meaning |
| --- | --- |
| `200 OK` | Request succeeded and a response body is returned |
| `201 Created` | A resource was created successfully |
| `204 No Content` | Request succeeded and there is no response body |
| `302 Found` or `303 See Other` | Client is being redirected |
| `400 Bad Request` | Request is malformed or invalid |
| `401 Unauthorized` | Authentication is required or failed |
| `403 Forbidden` | Request is understood but not allowed |
| `404 Not Found` | Requested resource does not exist |
| `405 Method Not Allowed` | Resource exists but does not support this method |
| `422 Unprocessable Content` | Request syntax is valid but the submitted data fails application rules |
| `500 Internal Server Error` | Unexpected server-side failure |

A status number should match the actual result. For example, return `405` when the route exists but rejects the method, and include an `Allow` header listing supported methods.

## 5. Set status and headers before the body

Use `http_response_code()` to set a status and `header()` to send a response header:

```php
<?php

http_response_code(201);
header('Content-Type: application/json; charset=utf-8');

echo json_encode(['id' => 42], JSON_THROW_ON_ERROR);
```

Headers must be sent before body output begins. A leading blank line, HTML, an `echo`, or accidental output from an included file may cause a “headers already sent” warning and prevent later header changes. Keep response setup before rendering output and avoid whitespace before a PHP opening tag in a pure PHP entry point.

### Redirects

Set a redirect response header and stop the current request's script after sending it:

```php
<?php

header('Location: /account', true, 303);
exit;
```

Use a trusted, application-controlled redirect destination. Do not put an unvalidated user-provided URL directly into a `Location` header.

### No-content responses

A `204 No Content` response must not include a response body:

```php
<?php

http_response_code(204);
exit;
```

## 6. Handle methods deliberately

A route should define which methods it supports and respond clearly to others:

```php
<?php

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($method !== 'GET') {
    header('Allow: GET');
    http_response_code(405);
    echo 'Method Not Allowed';
    exit;
}

header('Content-Type: text/plain; charset=utf-8');
echo 'This endpoint supports GET.';
```

For a real application, method dispatch belongs in a router or controller. This small example shows the essential sequence: inspect the request method, set the status and headers, then produce a body and exit when necessary.

## 7. Input and response are different concerns

Keep these steps clear in an endpoint:

1. Read request metadata such as the method and content type.
2. Parse data from the appropriate source.
3. Validate shape, format, length, and allowed values.
4. Authenticate and authorize operations that require it.
5. Perform the application work.
6. Set the status and response headers.
7. Serialize or render the response body.

Do not treat `POST` as an access-control mechanism or assume that values are safe because they came from a form. Input validation, output escaping, authentication, authorization, and CSRF defenses address different concerns and are covered in their relevant chapters.

## 8. Common mistakes

- **Assuming `$_GET` only applies to GET requests:** it reflects the URL query string, whatever the request method.
- **Expecting JSON fields in `$_POST`:** `$_POST` handles specific form content types; JSON bodies need to be read and decoded.
- **Trusting a request value:** all client-provided values must be validated for the expected type and rules.
- **Setting headers after output:** set the status and response headers before rendering or echoing body content.
- **Forgetting to stop after a redirect:** call `exit` so the current script does not continue producing the old response.
- **Returning a body with `204`:** a no-content response has no response body.
- **Using `$_REQUEST` when the source matters:** read the specific superglobal to make the origin of data clear.
- **Assuming every `$_SERVER` entry exists:** server variables depend on the web server and runtime configuration.
- **Putting a secret in a query string:** URLs can be stored in history, logs, and referrer information.

## Key takeaways

- An HTTP request carries a method, target, headers, and sometimes a body; a response carries a status, headers, and sometimes a body.
- A PHP application receives request data through superglobals and returns a response through the server interface.
- Use the correct input source: `$_GET` for query parameters, `$_POST` for supported form bodies, and `php://input` for raw bodies such as JSON.
- Validate and authorize request data; client input is never trusted by default.
- Set status codes and headers before output, and use an appropriate status for each result.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [083 - Reflection API](083-reflection-api.md) | [Q&A](../qnas/084-http-requests-responses-and-the-php-request-lifecycle.md) | [Example](../examples/084-http-requests-responses-and-the-php-request-lifecycle.md) | [085 - Generating HTML safely with templates](085-generating-html-safely-with-templates.md) |
