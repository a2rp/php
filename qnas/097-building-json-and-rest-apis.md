# 097 - Building JSON and REST APIs: Q&A

## Multiple-choice questions

### 1. What should be defined before implementing an API endpoint?

- **A.** Only its PHP filename
- **B.** Its URL, methods, request and response shapes, validation rules, permissions, and errors
- **C.** Only the color of its documentation page
- **D.** A database table name, regardless of the endpoint

**Answer: B.** An endpoint is a contract. Clients need to know what they can send, what they may receive, which permissions apply, and how failures are reported.

### 2. Which HTTP method is intended for retrieving a resource without changing its state?

- **A.** GET
- **B.** DELETE
- **C.** PATCH
- **D.** POST

**Answer: A.** GET is used to retrieve representations. A state-changing operation should not be hidden behind a GET route because clients and intermediaries can make assumptions about GET requests.

### 3. Which statement best describes the difference between PUT and PATCH?

- **A.** PUT retrieves data; PATCH creates a collection
- **B.** PUT commonly replaces a resource representation; PATCH changes selected parts
- **C.** They are interchangeable in every API
- **D.** PATCH is only for retrieving files

**Answer: B.** The API should document exactly which fields are required for replacement and how omitted fields behave during a partial update.

### 4. Which PHP stream is commonly used to read a raw HTTP request body?

- **A.** php://input
- **B.** php://output
- **C.** php://stderr
- **D.** php://memory

**Answer: A.** php://input exposes the raw request body. For JSON endpoints, the handler can read it, enforce a byte limit, and then decode it.

### 5. Why should an API check Content-Type before parsing a body?

- **A.** To know whether the body uses a format the endpoint accepts
- **B.** To determine the caller's identity
- **C.** To prevent all malformed data automatically
- **D.** To enable CORS

**Answer: A.** Content-Type is part of the endpoint contract. An endpoint that accepts JSON should reject unsupported request media types, commonly with 415.

### 6. What does JSON_THROW_ON_ERROR do with an invalid JSON string passed to json_decode()?

- **A.** Converts every invalid value to an empty object
- **B.** Throws JsonException
- **C.** Changes the HTTP status automatically
- **D.** Logs the request body

**Answer: B.** Catch JsonException and convert it into the API's documented client error response. The flag does not set response headers or an HTTP status by itself.

### 7. Is syntactically valid JSON automatically valid input for an endpoint?

- **A.** Yes, JSON syntax validation proves every field is safe
- **B.** No, the decoded shape, types, values, limits, and permissions still need validation
- **C.** Yes, if the request uses HTTPS
- **D.** No, because PHP cannot decode JSON

**Answer: B.** Decoding checks JSON syntax. The endpoint must separately enforce its schema, business rules, and authorization policy.

### 8. Which response status is commonly used when a request creates a resource?

- **A.** 201 Created
- **B.** 204 No Content
- **C.** 401 Unauthorized
- **D.** 304 Not Modified

**Answer: A.** A successful creation commonly returns 201. The response can include the representation and a Location header identifying the new resource.

### 9. When is 204 No Content appropriate?

- **A.** When the request succeeded and the API intentionally returns no body
- **B.** When JSON parsing failed
- **C.** When authentication is missing
- **D.** For every successful GET

**Answer: A.** A 204 response has no response body. If the client needs the updated resource representation, return a status such as 200 with JSON instead.

### 10. What should an API normally return when a known route receives an unsupported method?

- **A.** 405 Method Not Allowed, with an Allow header listing accepted methods
- **B.** 200 OK with an HTML error page
- **C.** 401 for every method mismatch
- **D.** 301 Moved Permanently

**Answer: A.** A 405 indicates that the route exists but does not support that method. The Allow header helps the client discover the methods the route accepts.

### 11. What is the usual distinction between 401 and 403?

- **A.** 401 concerns missing or invalid authentication; 403 means an authenticated caller lacks permission
- **B.** 401 means the body is too large; 403 means invalid JSON
- **C.** They both mean the route does not exist
- **D.** 403 is only a redirect status

**Answer: A.** Authentication identifies the caller; authorization decides what that caller may do. Some APIs use 404 to conceal a protected resource's existence, but the policy should be intentional.

### 12. What does CORS provide?

- **A.** A browser policy for whether web pages from other origins can read responses
- **B.** Authentication for every API client
- **C.** Protection against SQL injection
- **D.** A server-side permission check for each record

**Answer: A.** CORS is enforced by browsers. It is not authentication or authorization, and it does not replace CSRF protection for cookie-authenticated state-changing requests.

### 13. Why should collection endpoints use pagination?

- **A.** To avoid returning an unbounded number of records in one response
- **B.** To make every response a 404
- **C.** To prevent clients from using JSON
- **D.** To skip authorization checks

**Answer: A.** Pagination limits memory, query work, response size, and bandwidth. Define a maximum page size so callers cannot request an unlimited result.

### 14. What problem can an idempotency key help prevent?

- **A.** Duplicate processing when a client retries a state-changing operation
- **B.** Malformed JSON syntax
- **C.** Browser CORS errors
- **D.** Invalid PHP function names

**Answer: A.** When a network failure leaves the client unsure whether an operation completed, a server-supported idempotency key can let the server recognize a retry and avoid repeating the action.

## Short-answer questions

### 15. What does it mean to say an API endpoint has a contract?

**Answer:** The endpoint has documented rules for its route, method, headers, input fields, response data, status codes, authentication, authorization, and errors. **Explanation:** Clients can integrate reliably when the rules are predictable and changes are managed deliberately.

### 16. Why check that decoded JSON is an object when the endpoint expects an object?

**Answer:** JSON syntax also permits arrays, strings, numbers, booleans, and null. **Explanation:** json_decode() can successfully decode those values, so successful decoding alone does not prove the body has the shape required by the endpoint.

### 17. Why should request bodies be read with a size limit?

**Answer:** A caller could submit an extremely large body and consume application memory or processing time. **Explanation:** Read only up to a configured maximum plus one byte to detect overflow, and also configure the PHP runtime and fronting web server with compatible limits.

### 18. Why might an API return 422 after successfully decoding JSON?

**Answer:** The JSON syntax is valid, but the fields violate the endpoint's validation rules. **Explanation:** For example, a title might be missing, too long, or the wrong type. A 400 is commonly used for malformed JSON, while 422 can distinguish valid syntax with invalid field content.

### 19. Why should an API use a consistent error response shape?

**Answer:** Clients can process errors in a predictable way, such as reading a stable code and safe message from the same fields. **Explanation:** Consistency avoids endpoint-specific parsing and makes client error handling easier to maintain.

### 20. Why is a client-supplied owner ID unsafe as the authority for ownership?

**Answer:** The caller can modify the JSON body and claim another user's ID. **Explanation:** Determine the authenticated user from trusted server-side authentication state, then apply authorization checks to the requested object.

### 21. What is object-level authorization?

**Answer:** It checks whether the current caller may perform a particular action on the specific resource named by the request. **Explanation:** Permission to use an endpoint in general does not necessarily grant access to every record whose ID can be placed in a URL.

### 22. Why does CORS not replace CSRF protection?

**Answer:** CORS controls whether browser scripts from another origin may read a response; it does not prove that a state-changing request was intentionally initiated by the authenticated user. **Explanation:** Cookie-authenticated state-changing endpoints still need an appropriate CSRF defense.

### 23. What should an API include in a public 500 response?

**Answer:** A generic error code and a safe message, without stack traces, database details, secrets, or filesystem paths. **Explanation:** Detailed diagnostics belong in access-controlled server logs, with sensitive information removed.

### 24. How can an API evolve without unnecessarily breaking clients?

**Answer:** Add compatible optional behavior when possible, preserve existing field meanings and types, document deprecations, and use a planned versioning strategy for breaking changes. **Explanation:** Existing clients may not be updated at the same time as the server.

## Practical scenarios

### 25. A route receives a POST request where only GET is supported. What should it do?

**Answer:** Return 405 Method Not Allowed and an Allow header listing GET. **Explanation:** The route exists, so 404 is not the clearest status. The method mismatch should be explicit and machine-readable.

### 26. The request body contains valid JSON, but title is an array instead of a string. Should the handler continue?

**Answer:** No. Reject the request with the endpoint's documented validation error, commonly 422. **Explanation:** JSON decoding validates syntax, not the expected application type for each field.

### 27. A POST request creates a record, but the response has no status set and defaults to 200. What should be improved?

**Answer:** Return 201 Created and, where useful, a Location header and the created resource representation. **Explanation:** Clients should not have to infer creation from the response body alone.

### 28. An API returns an error object but always uses HTTP 200. Is that a good response design?

**Answer:** No. Set an HTTP status that matches the outcome and keep the JSON error envelope consistent. **Explanation:** HTTP clients, caches, monitoring, and retry policies use the status as well as the body.

### 29. A collection endpoint currently returns 30 records, so the developer omits pagination. Is that safe?

**Answer:** Not as a durable design. Add pagination and enforce a maximum page size. **Explanation:** The collection can grow, and an unbounded endpoint can use excessive memory, query time, and bandwidth.

### 30. A browser application is allowed by CORS to call an endpoint that changes account settings. Does that mean the endpoint is authorized?

**Answer:** No. The server must authenticate the caller and check permission for the account setting. Cookie-authenticated changes also need CSRF protection. **Explanation:** CORS is a browser response-access policy, not server-side identity or permission enforcement.

### 31. What is wrong with returning all exceptions directly in the JSON response?

**Answer:** Exception messages may expose internal paths, SQL statements, service addresses, or other sensitive implementation details. **Explanation:** Return a generic error to the caller, log necessary diagnostics securely, and correlate the two with a request ID if useful.

### 32. A client sends a user ID in JSON and asks to update that user's profile. What must the server verify?

**Answer:** Authenticate the caller, validate the identifier and fields, then authorize that caller to update the specific target user's profile. **Explanation:** A syntactically valid ID is not proof that the caller owns or may modify that resource.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [096 - Making HTTP requests with streams and cURL](096-making-http-requests-with-streams-and-curl.md) | [Lesson](../lessons/097-building-json-and-rest-apis.md) | [Example](../examples/097-building-json-and-rest-apis.md) | [098 - Relational databases and SQL fundamentals](098-relational-databases-and-sql-fundamentals.md) |
