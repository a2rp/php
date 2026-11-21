# 097 - Building JSON and REST APIs

## Learning goals

By the end of this lesson, you will be able to:

- Describe a JSON API endpoint as a contract between a client and a server.
- Read a bounded JSON request body and reject malformed or unsupported input.
- Return JSON with an intentional HTTP status and content type.
- Map resource operations to HTTP methods and status codes.
- Validate input and authorize access to each requested resource.
- Design consistent errors, pagination, and request limits.
- Recognize the security and compatibility responsibilities of a production API.

## 1. Start with the API contract

An API is an interface that another program can call. Before writing a handler, define what the endpoint accepts and returns:

- The resource and URL pattern.
- The supported HTTP methods.
- Required headers and request fields.
- Field types, formats, and size limits.
- Authentication and authorization requirements.
- Success status codes and response shapes.
- Error status codes and response shapes.
- Pagination, sorting, and filtering rules.

For example, an articles API might expose:

| Request | Purpose | Typical success response |
| --- | --- | --- |
| GET /api/v1/articles | List articles | 200 with a collection and pagination details |
| GET /api/v1/articles/42 | Read article 42 | 200 with one article |
| POST /api/v1/articles | Create an article | 201 with the created article and a Location header |
| PUT /api/v1/articles/42 | Replace article 42 | 200 or 204 |
| PATCH /api/v1/articles/42 | Change selected fields | 200 or 204 |
| DELETE /api/v1/articles/42 | Delete article 42 | 204 with no response body |

The exact contract can differ between APIs. Keep it documented and consistent. Returning JSON does not automatically make an endpoint RESTful; REST also concerns resource-oriented URLs, HTTP semantics, and a uniform interface.

## 2. Use resource URLs and HTTP methods consistently

Prefer URLs that identify resources, such as /api/v1/articles and /api/v1/articles/42. Use the HTTP method to express the operation instead of putting an action word in every URL.

A resource API commonly uses:

- **GET** to retrieve a collection or one resource. GET should not perform a state-changing action.
- **POST** to submit data, often to create a resource or start an operation.
- **PUT** to replace a resource representation. Define whether the request must include every replaceable field.
- **PATCH** to update selected fields. Document how omitted and null fields behave.
- **DELETE** to remove a resource or mark it deleted according to the API contract.

HTTP method semantics matter to clients, caches, proxies, and retry logic. Do not use GET for a payment, deletion, or other state change. A client may retry requests after a network problem, so document whether a request is safe to repeat. For operations that create or charge something, an idempotency key can help prevent duplicate work when the server supports it.

## 3. Parse a JSON request safely

JSON request bodies are sent as bytes. Read the raw body from the PHP input stream, verify the request media type, enforce a size limit, and decode with explicit error handling.

~~~php
const MAX_REQUEST_BYTES = 16_384;

$contentType = strtolower(trim(
    explode(';', $_SERVER['CONTENT_TYPE'] ?? '', 2)[0]
));

if ($contentType !== 'application/json') {
    respond(415, [
        'error' => [
            'code' => 'unsupported_media_type',
            'message' => 'Send a JSON request body.',
        ],
    ]);
}

$rawBody = file_get_contents(
    'php://input',
    false,
    null,
    0,
    MAX_REQUEST_BYTES + 1
);

if ($rawBody === false) {
    respond(400, [
        'error' => [
            'code' => 'unreadable_request_body',
            'message' => 'The request body could not be read.',
        ],
    ]);
}

if (strlen($rawBody) > MAX_REQUEST_BYTES) {
    respond(413, [
        'error' => [
            'code' => 'request_too_large',
            'message' => 'The request body exceeds the allowed size.',
        ],
    ]);
}

try {
    $payload = json_decode($rawBody, false, 32, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    respond(400, [
        'error' => [
            'code' => 'invalid_json',
            'message' => 'The request body must contain valid JSON.',
        ],
    ]);
}

if (!$payload instanceof stdClass) {
    respond(400, [
        'error' => [
            'code' => 'invalid_json_shape',
            'message' => 'The JSON request body must be an object.',
        ],
    ]);
}
~~~

The length argument limits how many bytes PHP reads into the string. Also configure request-size limits in PHP and the web server or reverse proxy; the application limit is one layer, not a replacement for server configuration.

A valid JSON value can be an object, array, string, number, boolean, or null. If the endpoint contract requires an object, check for that shape instead of accepting every valid JSON value.

The example catches JsonException so it can return a controlled client error. Do not send exception details or the raw request body to the client. Log only the information needed for diagnosis, with sensitive values removed.

## 4. Validate fields against the endpoint schema

Successfully decoding JSON proves only that the syntax is valid. It does not prove the fields are allowed or meaningful.

Validate each field for:

- Presence when required.
- Expected type.
- Length, range, and format.
- Allowed values.
- Relationships to other fields.
- Whether the current user may set or change it.

For example, an article creation endpoint might accept a non-empty title and body, but never accept an owner ID from the client if the owner must be the signed-in user. Set that value from trusted authentication state on the server.

~~~php
$title = $payload->title ?? null;
$body = $payload->body ?? null;

if (
    !is_string($title)
    || trim($title) === ''
    || strlen($title) > 160
) {
    respond(422, [
        'error' => [
            'code' => 'invalid_title',
            'message' => 'Title must be a non-empty string of at most 160 bytes.',
        ],
    ]);
}

if (!is_string($body) || trim($body) === '' || strlen($body) > 20_000) {
    respond(422, [
        'error' => [
            'code' => 'invalid_body',
            'message' => 'Body must be a non-empty string within the size limit.',
        ],
    ]);
}
~~~

The example uses strlen(), which measures bytes. If the limit is defined in visible Unicode characters, use a multibyte-aware approach such as mb_strlen() and define the encoding. Validation should follow the product rule and the representation being measured.

Do not treat client-side validation as a security boundary. Clients can send requests without using your form or interface. Validate on the server every time.

## 5. Return JSON with the correct status and headers

Send response headers before writing the body. Set the HTTP status and a JSON content type, then encode the response:

~~~php
function respond(int $status, array $data): void
{
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    header('X-Content-Type-Options: nosniff');

    echo json_encode(
        $data,
        JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES
    );

    exit;
}
~~~

This helper provides a consistent response path for JSON objects. Make sure no output, whitespace, warning, or debug message is emitted before the headers. In a production application, unexpected encoding or server errors should be handled by a top-level error handler so they do not reveal internals or turn into HTML output.

A successful response could look like:

~~~json
{
  "data": {
    "id": 42,
    "title": "A first article",
    "body": "A short introduction."
  }
}
~~~

An error response could use the same predictable envelope:

~~~json
{
  "error": {
    "code": "invalid_title",
    "message": "Title must be a non-empty string."
  }
}
~~~

Choose one response format and use it across endpoints. Clients should not have to guess whether one error is a string, another is an array, and another is an HTML page.

## 6. Choose status codes deliberately

The status code is part of the API contract. Common choices include:

| Status | Use |
| --- | --- |
| 200 OK | A request succeeded and the response contains a representation. |
| 201 Created | A request created a resource. Include its location when useful. |
| 202 Accepted | Work was accepted but has not completed yet. |
| 204 No Content | A request succeeded and intentionally has no response body. |
| 400 Bad Request | The request is malformed, such as invalid JSON syntax. |
| 401 Unauthorized | Authentication is missing or invalid. |
| 403 Forbidden | The caller is authenticated but is not allowed to perform the action. |
| 404 Not Found | The resource does not exist, or the API deliberately hides its existence. |
| 405 Method Not Allowed | The route exists but does not support this method. Include an Allow header. |
| 409 Conflict | The request conflicts with the current resource state. |
| 413 Content Too Large | The request body exceeds the accepted limit. |
| 415 Unsupported Media Type | The request uses a body format the endpoint does not accept. |
| 422 Unprocessable Content | The JSON is valid, but its fields fail the endpoint rules. |
| 429 Too Many Requests | The caller exceeded a rate or usage limit. |
| 500 Internal Server Error | An unexpected server failure occurred. |

Use the response body to give a stable machine-readable error code and a safe message. A client can branch on the code; avoid making clients parse human-readable prose.

## 7. Route requests and reject unsupported methods

A router maps a path and method to a handler. Framework routers are preferable for applications with multiple routes because they handle path parameters, middleware, and method matching more consistently.

For a small learning endpoint, the important behavior is still explicit:

1. Normalize and match the path using a router or a carefully defined route table.
2. Check the HTTP method allowed for that route.
3. Return 404 for an unknown route.
4. Return 405 for a known route with an unsupported method and list allowed methods.
5. Authenticate, validate, authorize, perform the operation, and return a documented response.

Do not build routing by including a PHP file named from untrusted URL input. Match user input against a fixed route table and pass validated route parameters to handlers.

## 8. Authenticate and authorize every operation

Authentication establishes who the caller is. Authorization determines whether that caller may perform this operation on this specific resource.

For every endpoint:

- Require the authentication method chosen by the application.
- Check permission for the requested resource and action.
- Do not trust a user ID, role, or ownership claim supplied in the JSON body.
- Apply object-level authorization when a request names a specific record.
- Return 401 for missing or invalid authentication and 403 when an authenticated caller lacks permission, unless the documented policy uses 404 to avoid disclosing resource existence.
- Apply CSRF protection to cookie-authenticated state-changing requests. CORS is not a replacement for CSRF protection.
- Use HTTPS for credentials and private data.

A user who can view one article must not automatically be able to view or edit every article by changing the ID in the URL.

## 9. Treat CORS as browser access policy

Cross-Origin Resource Sharing (CORS) tells browsers which origins may read responses from a server. It is not authentication and does not stop command-line clients or other servers from making requests.

If browser code on another origin needs the API:

- Allow only the origins the application intends to support.
- Allow only the methods and request headers that are needed.
- Handle OPTIONS preflight requests where applicable.
- Do not combine credentialed requests with a wildcard origin.
- Keep authentication and authorization checks on every request.

If no cross-origin browser access is needed, do not add permissive CORS headers just in case.

## 10. Bound work and keep responses predictable

An API can be abused or accidentally overloaded by large inputs and expensive queries. Set limits that match the endpoint:

- Maximum request body and field lengths.
- Maximum number of items returned per page.
- Allowlisted sort fields and filter operators.
- Query, execution, and downstream-service timeouts.
- Per-user or per-token rate limits where appropriate.
- Response-size and export limits.
- Appropriate caching rules for data that may be cached.

For collections, use stable pagination. Return the current items and enough metadata or a next-page cursor for the client to continue. Do not return an unbounded database result just because the table is currently small.

For filtering and sorting, map public field names to known server-side expressions. Never concatenate arbitrary client input into SQL, shell commands, or filesystem paths. Database-specific query safety is covered in later lessons.

## 11. Keep errors safe and logs useful

A production error response should not include stack traces, filesystem paths, SQL text, credentials, or internal service details. Return a generic message and stable error code to the caller, then record useful diagnostic details in protected server logs.

Logs should help correlate a request and investigate a failure without storing passwords, access tokens, full sensitive bodies, or unnecessary personal information. Consider a request ID that can appear in both the response and server logs.

Handle expected failures explicitly, such as validation errors, missing resources, and permission denials. Let a central error handler record unexpected failures and return a generic 500 response.

## 12. Preserve compatibility as the API grows

Once clients depend on an API, changing its shape can break deployed applications. Plan changes deliberately:

- Add optional fields when existing clients can safely ignore them.
- Do not silently change the meaning or type of a field.
- Document whether fields can be absent or null.
- Deprecate old behavior with a clear migration period.
- Version the API when a breaking change cannot be made compatibly.
- Keep examples and response schemas aligned with the implementation.

A version in the URL, such as /api/v1/articles, is one common convention. The versioning scheme matters less than applying it consistently and documenting compatibility expectations.

## Common mistakes

### Returning HTTP 200 for every outcome

A JSON body saying an error occurred does not change the HTTP status. Set a status that matches the outcome so clients, caches, and monitoring can interpret the response.

### Decoding without validating

Valid JSON can still contain missing fields, unexpected types, oversized strings, or values the caller is not allowed to set. Validate the schema and permissions after decoding.

### Returning raw exception details

Internal error messages can expose implementation details and sensitive data. Return a safe error contract and keep diagnostics in protected logs.

### Trusting the client to choose the owner

A request body can be changed. Derive identity and ownership from trusted server-side authentication state.

### Using CORS as access control

CORS controls browser response access. It does not identify a caller or authorize an API operation.

### Returning every row in a collection

Large unbounded responses consume memory and bandwidth. Require pagination and cap page size.

## Key takeaways

- Define the endpoint contract before implementing it.
- Use resource-oriented routes and meaningful HTTP methods.
- Enforce request size and content type before decoding JSON.
- Use JSON_THROW_ON_ERROR and validate the decoded shape and every field.
- Return a consistent JSON envelope with an intentional HTTP status.
- Authenticate callers and authorize each action on each resource.
- CORS is a browser policy; it is not authentication or CSRF protection.
- Bound request sizes, page sizes, time, and other resource costs.
- Keep error details safe and make compatible changes deliberate.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [096 - Making HTTP requests with streams and cURL](096-making-http-requests-with-streams-and-curl.md) | [Q&A](../qnas/097-building-json-and-rest-apis.md) | [Example](../examples/097-building-json-and-rest-apis.md) | [098 - Relational databases and SQL fundamentals](098-relational-databases-and-sql-fundamentals.md) |
