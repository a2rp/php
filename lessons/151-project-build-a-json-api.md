# Project: build a JSON API

A JSON API lets another program request data and send changes over HTTP. PHP receives the request, validates it, performs the requested operation, and sends a JSON response with an appropriate HTTP status.

This chapter builds a small task API. It uses the same database-backed task ideas from the previous project, but returns machine-readable JSON instead of HTML pages.

## Project goals

The API will let a client:

- List tasks with GET.
- Create a task with POST.
- Read one task with GET.
- Partially update a task with PATCH.
- Delete a task with DELETE.
- Receive predictable JSON success and error responses.
- Understand failures from HTTP status codes.
- Store data in SQLite through PDO prepared statements.

The sample is a local, single-user learning project. It has no authentication and must not be exposed as a public service. A real deployment needs authentication and authorization for each task, plus operational protections such as rate limiting and request logging.

## What makes an API

An API is a contract between a server and its clients. The contract describes:

- Which URLs represent resources.
- Which HTTP methods can be used at each URL.
- Which request headers and body formats are accepted.
- Which response statuses and JSON shapes are returned.
- Which fields are required, optional, or read-only.
- What a client should do when a request fails.

For this project, a task is a resource. The collection URL represents all tasks, and a URL containing an ID represents one task.

| Request | Meaning |
| --- | --- |
| GET /api/tasks | Return the task collection |
| POST /api/tasks | Create a task |
| GET /api/tasks/42 | Return task 42 |
| PATCH /api/tasks/42 | Change selected fields of task 42 |
| DELETE /api/tasks/42 | Delete task 42 |

A URL identifies a resource. The method describes the operation. Do not create a separate URL for every verb, such as /api/create-task or /api/delete-task.

## Choose methods by their effect

Use GET to read data. A GET request should not create, update, or delete records. Browsers, proxies, and clients may retry or prefetch GET requests.

Use POST when submitting data that creates a resource or triggers an operation. Creating the same task twice with two POST requests normally creates two tasks, so POST is not automatically safe to retry.

Use PUT when replacing the complete representation of a resource. Use PATCH when applying a partial change. This project uses PATCH so a client can update a task title without resending every field.

Use DELETE to remove a resource. A repeat DELETE should leave the resource deleted, even though the second request may receive a 404 response.

These method expectations help clients understand the API and retry requests safely. They do not replace authorization or validation.

## Agree on JSON request and response formats

A client should send a request body as JSON and identify it with the Content-Type header:

~~~http
POST /api/tasks HTTP/1.1
Content-Type: application/json
Accept: application/json

{"title":"Read the PHP notes","description":"Review the API chapter","status":"open"}
~~~

The server should send JSON with a matching response header:

~~~http
HTTP/1.1 201 Created
Content-Type: application/json; charset=utf-8
Location: /api/tasks/42

{"data":{"id":42,"title":"Read the PHP notes","description":"Review the API chapter","status":"open"}}
~~~

The Location header tells the client where the newly created resource can be read. The response body uses a data envelope so the API can add metadata later without changing the resource shape.

A consistent error response is easier for clients to handle:

~~~json
{
  "error": {
    "code": "validation_failed",
    "message": "The request contains invalid fields.",
    "fields": {
      "title": "Enter a title no longer than 200 characters."
    }
  }
}
~~~

Keep error codes stable. The message can be written for a person, while the code gives a client a predictable value to branch on. Never return SQL, stack traces, filesystem paths, or secrets in an API error.

## Decode request JSON carefully

Request data arrives as bytes. Read it from the request body, reject an oversized body, and then decode it. A small endpoint can use a body limit such as 64 KiB; the limit should match the application’s actual needs.

~~~php
try {
    $payload = json_decode($rawBody, false, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    // Return a 400 Bad Request response with a safe error body.
}

if (!$payload instanceof stdClass) {
    // This endpoint expects one JSON object, not an array or scalar.
}
~~~

JSON can represent objects, arrays, strings, numbers, booleans, and null. A syntactically valid JSON array is still the wrong shape if an endpoint expects one task object. Check the decoded type before reading fields.

Check the Content-Type header before decoding a body that must be JSON. Return 415 Unsupported Media Type when the client sends a different format. If the server supports content negotiation, inspect Accept and return 406 Not Acceptable when it cannot produce a representation the client accepts.

Do not trust a Content-Type header as proof that the body is valid. Decode the bytes and validate the result too.

## Validate fields on the server

Client-side checks are useful for feedback, but clients can send requests without using your page. The server must validate every value before using it.

For a task, define rules such as:

- title is required, is a string, and is at most 200 characters.
- description is a string and is at most 2,000 characters.
- status is exactly open or done.
- IDs are positive integers.
- PATCH contains at least one supported field.
- Unknown fields are rejected or deliberately ignored according to the documented contract.

Validate the type before applying string functions. Trim surrounding whitespace where appropriate, then validate the resulting value. Do not silently turn an array, object, or boolean into a string.

Use an allowlist for accepted field names and status values. Never let a request choose arbitrary SQL column names, sort expressions, or table names. SQL parameters represent values, not SQL structure; for dynamic sort fields, map a small set of approved names to fixed SQL fragments.

Validation errors can identify individual fields. A well-formed JSON object with invalid field values is commonly answered with 422 Unprocessable Content. A malformed JSON document is a 400 Bad Request.

## Return useful HTTP status codes

Status codes are part of the API contract.

| Status | Typical use |
| --- | --- |
| 200 OK | A successful read or update that returns a body |
| 201 Created | A resource was created; include Location when useful |
| 204 No Content | A successful operation has no response body, such as DELETE |
| 400 Bad Request | The request cannot be parsed or contains an invalid identifier |
| 401 Unauthorized | Authentication is missing or invalid |
| 403 Forbidden | The authenticated client lacks permission |
| 404 Not Found | The route or requested resource does not exist |
| 405 Method Not Allowed | The route exists but does not support this method; send Allow |
| 409 Conflict | The request conflicts with current resource state |
| 413 Content Too Large | The request body exceeds the configured limit |
| 415 Unsupported Media Type | The request uses an unsupported Content-Type |
| 422 Unprocessable Content | JSON is valid, but its values fail validation |
| 429 Too Many Requests | A rate limit has been reached |
| 500 Internal Server Error | An unexpected server-side failure occurred |

Do not send a JSON body with 204 No Content. For 405, include an Allow header listing accepted methods, such as GET, POST for the collection or GET, PATCH, DELETE for one task.

Return the same general error shape for expected failures. Log unexpected details on the server and send clients a generic 500 response.

## Keep the database boundary safe

Use PDO and prepared statements for every user-supplied value. Keep SQL structure fixed, pass values through parameters, and select only the columns the API needs.

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, description, status FROM tasks WHERE id = :id'
);
$statement->execute(['id' => $taskId]);
$task = $statement->fetch();
~~~

A task ID from the URL is still user input. Parse it as a positive integer, then use a prepared statement. A missing database row should become a 404 response.

When an operation requires several database writes that must succeed together, use a transaction. Commit only after all related writes succeed, and roll back when an operation fails. Do not catch a database exception and claim success.

## Handle routes and methods explicitly

For each route, decide what methods it allows. A useful order is:

1. Parse the URL path and identify the collection or item route.
2. Reject unknown routes with 404.
3. Check the HTTP method and return 405 plus Allow if it is unsupported.
4. Validate the item ID, if present.
5. Check authentication and authorization in a real application.
6. Check request headers and decode the body when required.
7. Validate fields.
8. Read or change the database.
9. Return the documented status, headers, and JSON body.

Avoid using loose substring checks to match routes. Match the full path and handle trailing slashes intentionally. Keep route matching separate from the database operations so each part can be understood.

## Protect the API

This learning API has no login. If deployed publicly, any visitor could list, change, or delete every task. Add authentication and check authorization for every resource before returning or changing it.

CORS controls whether browser JavaScript from another origin may read a response. It is not authentication, and non-browser clients are not restricted by it. If the API uses cookie-based authentication, account for CSRF protections as well. Keep credentials out of source control, use HTTPS in production, and avoid logging request secrets.

Also consider request-size limits, rate limiting, timeouts, backups, and a safe policy for unexpected errors. Return only fields that the client is allowed to see.

## Make collection endpoints practical

A growing collection should support pagination. Accept a bounded page size and a cursor or offset, and return metadata that lets the client request the next page. Place reasonable maximums on page size.

Filtering and sorting should use documented allowlists. Do not concatenate raw query parameters into SQL. If the API supports search, define what it searches and how special characters are handled.

Avoid changing a published response shape without considering existing clients. Add optional fields in a backward-compatible way, document breaking changes, and use an explicit API versioning strategy when needed.

## Retries and duplicate requests

Networks fail, and a client may not know whether a server completed a request before the connection ended. GET requests are usually safe to retry. A repeated create request may make duplicate records. For important operations, design an idempotency strategy, such as a client-supplied request key stored with the result.

Do not claim an operation is idempotent merely because the PHP code looks simple. Consider the database effect and what happens when a request is repeated.

## Project completion checklist

| Area | Complete when |
| --- | --- |
| Routes | Collection and item paths match the documented contract |
| Methods | Each route allows only its documented methods |
| JSON input | Body size, media type, syntax, and object shape are checked |
| Validation | Types, lengths, allowed values, and IDs are validated |
| Responses | Success and error bodies have stable shapes and correct statuses |
| Persistence | PDO prepared statements are used for all supplied values |
| Errors | Client responses are safe and unexpected details are logged privately |
| Access control | Public use requires authentication and per-resource authorization |
| Operations | Rate limits, backups, HTTPS, and deployment routing are planned |
| Documentation | Request examples, response examples, and error cases are recorded |

## Key takeaways

- An API contract includes URLs, methods, headers, response bodies, and status codes.
- Decode JSON with explicit error handling, then validate its shape and every field.
- GET reads; POST commonly creates; PATCH changes part; DELETE removes.
- Prepared statements protect database values, while authorization controls access.
- CORS does not authenticate a client.
- Return safe, consistent errors and never expose internal diagnostics.
- A local learning project needs authentication and operational protections before public deployment.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [150 - Project: build a database-backed CRUD application](150-project-build-a-database-backed-crud-application.md) | [Q&A](../qnas/151-project-build-a-json-api.md) | [Example](../examples/151-project-build-a-json-api.md) | [152 - Capstone: plan, build, test, and document an application](152-capstone-project-plan-build-test-and-document-an-application.md) |
