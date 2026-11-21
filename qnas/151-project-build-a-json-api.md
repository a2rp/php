# Project: build a JSON API — Q&A

These questions review the JSON task API project, including HTTP methods, request validation, status codes, persistence, and safe deployment. Each answer follows its question.

## Multiple-choice questions

### 1. Which HTTP method should normally read the task collection without changing it?

- A. GET
- B. POST
- C. PATCH
- D. DELETE

**Answer: A.** GET is used to retrieve a representation. A GET request should not create, update, or delete data.

### 2. A client sends a valid JSON object, but its title is an array instead of a string. Which response is most appropriate?

- A. 201 Created
- B. 204 No Content
- C. 422 Unprocessable Content
- D. 301 Moved Permanently

**Answer: C.** The JSON syntax is valid, but the field value violates the API contract. A 422 response can report field-level validation errors.

### 3. Which header tells the server that a request body is JSON?

- A. Accept
- B. Content-Type
- C. Location
- D. Allow

**Answer: B.** Content-Type describes the format of the request body. Accept describes which response formats the client can handle.

### 4. What should a create endpoint commonly return after successfully creating a task?

- A. 201 Created, often with a Location header
- B. 204 No Content with a JSON body
- C. 404 Not Found
- D. 405 Method Not Allowed

**Answer: A.** 201 indicates that a resource was created. Location can identify the URL of that resource.

### 5. What is the appropriate response when a task is deleted successfully and the server has no body to return?

- A. 200 OK with an empty JSON object
- B. 204 No Content with no response body
- C. 302 Found
- D. 415 Unsupported Media Type

**Answer: B.** A 204 response has no body. Do not append JSON or other content to it.

### 6. What does a prepared statement protect when a task ID is supplied by a client?

- A. It authenticates the client.
- B. It makes the supplied value data instead of SQL syntax.
- C. It encrypts the database file.
- D. It validates that the task exists.

**Answer: B.** A prepared statement separates a parameter value from SQL structure. You must still validate the ID and check whether the record exists.

### 7. Why might an endpoint decode JSON with associative mode disabled?

- A. To tell a JSON object apart from a JSON array.
- B. To encrypt the request.
- C. To make invalid JSON valid.
- D. To remove unknown fields automatically.

**Answer: A.** With associative mode disabled, JSON objects decode as objects such as stdClass, while JSON arrays decode as PHP arrays. This helps enforce an object-shaped request contract.

### 8. What does CORS provide?

- A. Authentication for every API client.
- B. Authorization to edit a task.
- C. A browser policy for cross-origin access to responses.
- D. SQL injection prevention.

**Answer: C.** CORS controls browser behavior for cross-origin requests. It does not authenticate or authorize clients, and it does not stop non-browser tools from sending requests.

## Short-answer questions

### 9. What is the difference between Content-Type and Accept?

**Answer:** Content-Type describes the format of the body being sent. Accept describes the response format the client wants. A JSON request commonly sends Content-Type: application/json and Accept: application/json.

### 10. Why should the server validate JSON fields if the browser already validates them?

**Answer:** Clients can be bypassed or modified, and API requests can be sent by scripts or command-line tools. Server-side validation is the authority that protects application rules and data integrity.

### 11. What is the difference between malformed JSON and valid JSON with invalid field values?

**Answer:** Malformed JSON cannot be decoded and is commonly answered with 400 Bad Request. Valid JSON whose values do not satisfy the endpoint rules can be answered with 422 Unprocessable Content.

### 12. Why should the server limit request-body size?

**Answer:** A body limit prevents a client from forcing the server to read and process an unnecessarily large request. The endpoint should choose a bound that fits its expected data and return 413 Content Too Large when exceeded.

### 13. Why should a 405 response include an Allow header?

**Answer:** It tells the client which methods are supported for that route. For example, the task collection may allow GET and POST, while an individual task may allow GET, PATCH, and DELETE.

### 14. What does the Location header communicate after creating a task?

**Answer:** It identifies the URL at which the new task can be retrieved, such as /api/tasks/42.

### 15. Why should API errors avoid returning exception messages or SQL details?

**Answer:** Internal diagnostics can reveal database structure, file paths, secrets, or implementation details. Log those details privately and return a stable, safe error response to the client.

### 16. What is the difference between authentication and authorization?

**Answer:** Authentication establishes who the client is. Authorization decides whether that client may perform a particular action on a particular resource. A public API needs both checks where appropriate.

### 17. Why should dynamic SQL sort fields use an allowlist instead of a bound parameter?

**Answer:** Placeholders bind values, not SQL identifiers or syntax. Map approved sort names to fixed SQL fragments and reject unsupported names instead of inserting raw request input into a query.

## Scenario questions

### 18. A client sends PATCH /api/tasks/42 with a valid title, but task 42 does not exist. What should the API return?

**Answer:** Return 404 Not Found with the API’s standard error envelope. A valid body cannot make a missing resource exist.

### 19. A client sends POST /api/tasks using Content-Type: text/plain. What should the API do?

**Answer:** If the endpoint accepts only JSON, return 415 Unsupported Media Type. The server should check the declared media type and still decode and validate the body when the media type is supported.

### 20. The API route exists, but the client tries PUT and the route supports only GET, PATCH, and DELETE. What should happen?

**Answer:** Return 405 Method Not Allowed and include Allow: GET, PATCH, DELETE. This communicates the supported methods without pretending the URL is missing.

### 21. The API returns a generic 500 response, but the database operation threw an exception. Where should diagnostic details go?

**Answer:** Write the exception details to protected server logs with enough context to investigate. The client should receive only a safe generic error, without SQL, stack traces, paths, or secrets.

### 22. A browser page on another origin can read an API response. Does that prove the API is secure?

**Answer:** No. CORS is a browser access policy, not authentication or authorization. A command-line client can make requests without browser CORS enforcement, so the API must enforce access on the server.

### 23. A client times out while creating a task and retries the same POST. What problem can occur?

**Answer:** The first request may have succeeded even though the client did not receive the response, so the retry can create a duplicate. Important create operations can use an idempotency key recorded with the result so the same logical request is not applied twice.

## Coding questions

### 24. Decode a JSON body into an object and handle invalid JSON. What is a suitable PHP pattern?

**Answer:**

~~~php
try {
    $payload = json_decode($rawBody, false, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    // Send a 400 response with a safe error body.
}

if (!$payload instanceof stdClass) {
    // Send a 400 response because this endpoint expects a JSON object.
}
~~~

Using JSON_THROW_ON_ERROR makes malformed input explicit. Checking the decoded type also rejects a valid JSON array, string, number, boolean, or null when the endpoint expects an object.

### 25. Show a prepared query that reads one task by its ID.

**Answer:**

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, description, status FROM tasks WHERE id = :id'
);
$statement->execute(['id' => $taskId]);
$task = $statement->fetch();
~~~

The SQL structure stays fixed, while the ID is bound as a value. Validate that the ID is a positive integer and return 404 if no task is found.

### 26. A validation error occurs for the title field. What could a useful JSON response look like?

**Answer:**

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

The stable code helps clients handle the error programmatically. The field message helps a person correct the request.

### 27. Which response should a delete endpoint send after it removes a task, and should it contain JSON?

**Answer:** It can return 204 No Content and an empty body. If the API chooses to return a representation or a message body instead, it should use a status that permits a body, such as 200 OK.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [150 - Project: build a database-backed CRUD application](../lessons/150-project-build-a-database-backed-crud-application.md) | [Lesson](../lessons/151-project-build-a-json-api.md) | [Example](../examples/151-project-build-a-json-api.md) | [152 - Capstone: plan, build, test, and document an application](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) |
