# 089 - Response headers, status codes, and redirects

## Learning goals

By the end of this lesson, you will be able to:

- Explain the status line, response headers, and body in an HTTP response.
- Set response headers and status codes in PHP before output begins.
- Choose common status codes for success, client errors, and redirects.
- Return a content type that matches the response body.
- Use redirects safely and stop execution after sending one.
- Diagnose “headers already sent” problems and avoid unsafe redirect destinations.

## 1. Parts of an HTTP response

A server response contains a status code, headers, and usually a body:

```text
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8

<h1>Welcome</h1>
```

- The **status code** tells the client the broad result of the request.
- **Headers** carry metadata such as content type, cache policy, and redirect destination.
- The **body** contains the page, JSON, file, or other representation. Some responses, such as `204 No Content`, have no body.

PHP scripts normally set response metadata before producing the body.

## 2. Set the status code

Use `http_response_code()` to set the response status:

```php
http_response_code(404);
header('Content-Type: text/plain; charset=UTF-8');
echo 'Page not found.';
```

The status should match what happened. Returning an error message with status `200 OK` can mislead browsers, clients, caches, and monitoring systems into treating a failure as success.

Common status codes include:

| Status | Meaning | Typical use |
| --- | --- | --- |
| `200 OK` | Request succeeded | Return a page or data. |
| `201 Created` | A new resource was created | Confirm creation; often identify the resource with a `Location` header. |
| `204 No Content` | Request succeeded without a response body | A successful operation that has no representation to return. |
| `301 Moved Permanently` | Resource has a permanent new URL | Permanent URL changes when clients may update saved links. |
| `302 Found` | Temporary redirect | Temporary navigation; the method handling can vary in clients. |
| `303 See Other` | Follow up by retrieving another URL | Post/Redirect/Get after a successful form submission. |
| `307 Temporary Redirect` | Temporary redirect that preserves the method | Temporarily route a request while preserving its method and body. |
| `308 Permanent Redirect` | Permanent redirect that preserves the method | Permanently move a URL while preserving its method and body. |
| `400 Bad Request` | Request is malformed or invalid | Reject malformed request data. |
| `401 Unauthorized` | Authentication credentials are missing or invalid | Include the appropriate `WWW-Authenticate` challenge. |
| `403 Forbidden` | Request is understood but not allowed | Deny access when the caller is not permitted. |
| `404 Not Found` | No matching resource is available | Missing route or resource. |
| `405 Method Not Allowed` | The resource does not support this method | Include an `Allow` header listing supported methods. |
| `422 Unprocessable Content` | Request syntax is understood but instructions cannot be processed | Field validation errors for a well-formed request. |
| `500 Internal Server Error` | Unexpected server failure | Generic failure response; do not reveal internal details. |

Choose status codes that describe the actual result. Do not use `401` when the user is authenticated but lacks permission; that is usually a `403` case.

## 3. Send response headers before output

The `header()` function sends an HTTP header. It must run before any body output has been sent:

```php
<?php

declare(strict_types=1);

http_response_code(200);
header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Welcome</title>
</head>
<body>
    <h1>Welcome</h1>
</body>
</html>
```

Output before `header()` can be easy to miss. It may come from:

- HTML or whitespace before the opening PHP tag.
- A UTF-8 byte order mark at the start of a file.
- An included file that prints content.
- A warning, notice, or debug `echo` that is displayed.

Keep response handling before templates and output. `headers_sent($file, $line)` can help identify where output began, but it does not make a late header possible. Output buffering can change when output is sent; do not rely on it to hide a response-ordering bug.

## 4. Choose the correct content type

Tell the client how to interpret the response body:

```php
header('Content-Type: text/html; charset=UTF-8');
```

For a plain text response:

```php
header('Content-Type: text/plain; charset=UTF-8');
echo "Request could not be completed.\n";
```

For a JSON response:

```php
header('Content-Type: application/json; charset=UTF-8');

echo json_encode(
    ['ok' => true, 'count' => 3],
    JSON_THROW_ON_ERROR
);
```

The header should match the actual body. Setting `application/json` does not convert HTML into JSON; encode a structured PHP value and return that JSON body.

## 5. Return useful errors

Set the error status and a response body that is safe to expose:

```php
http_response_code(404);
header('Content-Type: text/html; charset=UTF-8');
require __DIR__ . '/../templates/not-found.php';
```

A `405 Method Not Allowed` response should say which methods are accepted:

```php
http_response_code(405);
header('Allow: GET, POST');
header('Content-Type: text/plain; charset=UTF-8');
echo 'Use GET or POST for this endpoint.';
```

For an unexpected server failure, log diagnostic details privately and return a generic message. Do not send stack traces, database credentials, filesystem paths, or internal exception messages to the client.

For a successful operation with no body, do not render a page afterward:

```php
http_response_code(204);
exit;
```

A `204` response means there is no response content. If the client needs a confirmation message or resource representation, choose a response that includes an appropriate body instead.

## 6. Redirect a browser

A redirect uses a `Location` header with a redirect status. For Post/Redirect/Get, a successful POST can respond with `303 See Other`:

```php
// Validate, authorize, and process the POST before this point.
header('Location: /orders/confirmation.php', true, 303);
exit;
```

`exit` matters: it stops the current script from continuing to output the old page or perform more work after the redirect response.

The redirect destination should be controlled by the application. Do not insert an untrusted query value directly into `Location`; otherwise a route can become an open redirect to an attacker-controlled site. Use fixed local paths or map a validated identifier to an application-controlled destination.

### Redirect status behavior

- `303` tells the client to retrieve the destination as a separate request, commonly a GET after POST.
- `307` and `308` preserve the original method and body when following the redirect.
- `301` and `308` represent permanent moves; `302` and `307` represent temporary ones.

Choose the status based on the intended follow-up behavior instead of assuming every redirect is interchangeable.

## 7. Send a created resource location

When a request creates a resource, `201 Created` can identify the new resource with a `Location` header and optionally return a representation:

```php
http_response_code(201);
header('Location: /api/items/42');
header('Content-Type: application/json; charset=UTF-8');

echo json_encode(
    ['id' => 42, 'name' => 'Notebook'],
    JSON_THROW_ON_ERROR
);
```

Use a `Location` value that points to the resource the application actually created. Do not claim creation before the operation succeeds.

## 8. Avoid unsafe or confusing response handling

### Do not trust a redirect target from the request

```php
// Unsafe: the request can name another site.
header('Location: ' . $_GET['next']);
```

Prefer a fixed destination, or validate a key against an allowlist of known application paths.

### Do not send a success status for an error

If validation fails, set an appropriate client-error status before rendering the form with errors. If a route does not exist, return `404` rather than a normal `200` page that merely says “not found.”

### Do not expose internal error details

Return a brief, useful message and keep technical details in controlled server logs. Error display settings should be suitable for the environment.

### Do not keep executing after a redirect or terminal response

Call `exit` after a redirect and after responses that should have no further output. This prevents accidental double responses or side effects.

## Key takeaways

- A response has a status, headers, and possibly a body; make all three agree.
- Set status codes and headers before output starts.
- Use `Content-Type` that matches the body and include the intended character encoding for text.
- Use `http_response_code()` for statuses; use `header()` for response headers such as `Location` and `Allow`.
- A `303` is useful for Post/Redirect/Get; `307` and `308` preserve the request method.
- Keep redirect targets application-controlled and call `exit` after redirecting.
- Return generic client errors and log sensitive diagnostics privately.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [088 - Input validation and normalization](088-input-validation-and-normalization.md) | [Q&A](../qnas/089-response-headers-status-codes-and-redirects.md) | [Example](../examples/089-response-headers-status-codes-and-redirects.md) | [090 - Cookies and browser state](090-cookies-and-browser-state.md) |