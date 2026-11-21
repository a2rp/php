# 084 - HTTP requests, responses, and the PHP request lifecycle: Q&A

Each answer follows its question and includes an explanation.

## Multiple-choice questions

### 1. What are the three main parts of a typical HTTP response?

- A. Method, path, and query string.
- B. Status, headers, and body.
- C. Function, class, and namespace.
- D. Cookie, session, and database.

**Answer:** B. Status, headers, and body.

**Explanation:** The status communicates the result, headers carry response metadata, and the body contains the returned representation when one is needed.

### 2. What does an HTTP request method such as `GET` or `POST` describe?

- A. The requested operation or intent.
- B. The server's PHP version.
- C. The response's encoding only.
- D. The file extension of the PHP script.

**Answer:** A. The requested operation or intent.

**Explanation:** The method is part of the request. The application should decide which methods each route supports.

### 3. Which PHP superglobal contains URL query-string values?

- A. `$_POST`
- B. `$_GET`
- C. `$_FILES`
- D. `$_SESSION`

**Answer:** B. `$_GET`.

**Explanation:** PHP parses query-string values into `$_GET` whenever a query string is present, regardless of the request method.

### 4. Which server variable commonly contains the request method?

- A. `$_SERVER['REQUEST_METHOD']`
- B. `$_GET['REQUEST_METHOD']`
- C. `$_POST['HTTP_METHOD']`
- D. `$_COOKIE['METHOD']`

**Answer:** A. `$_SERVER['REQUEST_METHOD']`.

**Explanation:** `$_SERVER` contains request and execution environment information supplied by the server; its exact entries can vary.

### 5. For which form content types does PHP populate `$_POST`?

- A. Only `application/json`.
- B. `application/x-www-form-urlencoded` and `multipart/form-data`.
- C. Every possible request content type.
- D. Only requests without a body.

**Answer:** B. `application/x-www-form-urlencoded` and `multipart/form-data`.

**Explanation:** JSON request bodies are not automatically converted into `$_POST` fields.

### 6. How can PHP read a raw JSON request body?

- A. `file_get_contents('php://input')`
- B. `file_get_contents($_POST)`
- C. `$_GET['body']`
- D. `json_encode($_SERVER)`

**Answer:** A. `file_get_contents('php://input')`.

**Explanation:** Decode the returned string with `json_decode()` and handle invalid JSON.

### 7. When should code call `header()` to set response headers?

- A. After the full response body is printed.
- B. Before body output begins.
- C. Only after `exit`.
- D. Only from an included template.

**Answer:** B. Before body output begins.

**Explanation:** Once output has been sent, PHP may no longer be able to send or change response headers.

### 8. Which PHP function sets the HTTP status code directly?

- A. `set_status()`
- B. `http_response_code()`
- C. `response_header()`
- D. `status_code()`

**Answer:** B. `http_response_code()`.

**Explanation:** It can set the status and can also return the current status when called without an argument.

### 9. What does a `204 No Content` response mean for the response body?

- A. It must contain a JSON object.
- B. It must contain an HTML page.
- C. It has no response body.
- D. It redirects the browser.

**Answer:** C. It has no response body.

**Explanation:** Use `204` only when the operation succeeded and there is no content to return.

### 10. What should a route generally send when it exists but does not support the incoming method?

- A. `405 Method Not Allowed` and an `Allow` header.
- B. `200 OK` with no details.
- C. `404 Not Found` in every case.
- D. `204 No Content` with an HTML body.

**Answer:** A. `405 Method Not Allowed` and an `Allow` header.

**Explanation:** The `Allow` response header can list the methods that the route accepts.

### 11. Which status is commonly used after successfully creating a resource?

- A. `201 Created`
- B. `301 Moved Permanently`
- C. `401 Unauthorized`
- D. `500 Internal Server Error`

**Answer:** A. `201 Created`.

**Explanation:** Use a status that accurately describes the result of the operation.

### 12. Why should application code usually prefer `$_GET` or `$_POST` over `$_REQUEST`?

- A. They make the source of the input clear.
- B. They automatically validate every value.
- C. `$_REQUEST` is only available in CLI mode.
- D. They encrypt the request.

**Answer:** A. They make the source of the input clear.

**Explanation:** `$_REQUEST` may combine sources according to configuration, making it harder to tell where a value came from.

## Short-answer questions

### 13. What information can an HTTP request contain?

**Answer:** A method, a target path and optional query string, headers, and sometimes a body.

**Explanation:** The exact contents depend on the request and the operation being performed.

### 14. What is a PHP SAPI?

**Answer:** The Server API through which PHP integrates with its execution environment, such as PHP-FPM or a server module.

**Explanation:** It affects how requests reach PHP and how PHP returns output to the web server.

### 15. What is the difference between `$_GET` and `$_POST`?

**Answer:** `$_GET` contains query-string values; `$_POST` contains supported form-encoded request-body values.

**Explanation:** They represent different input sources. Use the one that matches how the client sent the data.

### 16. Why is JSON body data usually read from `php://input`?

**Answer:** PHP does not automatically put JSON fields into `$_POST`.

**Explanation:** The script reads the raw body, decodes it, then validates the decoded structure.

### 17. Why should an application check `$_SERVER['REQUEST_METHOD']` with a fallback or presence check?

**Answer:** Server variables depend on the web server and SAPI and are not guaranteed in every environment.

**Explanation:** The same code may run in CLI, tests, or a differently configured server where request variables are absent.

### 18. What is the purpose of the `Content-Type` response header?

**Answer:** It describes the media type and possibly character encoding of the response body.

**Explanation:** For example, an HTML response can use `text/html; charset=utf-8`, while JSON can use `application/json`.

### 19. Why does accidental output interfere with later calls to `header()`?

**Answer:** Headers are sent before the response body, and body output may cause PHP to send them first.

**Explanation:** Once headers are sent, the script cannot reliably change the status or add response headers.

### 20. Why should a redirect usually be followed by `exit`?

**Answer:** To stop the current script from continuing to produce the original response.

**Explanation:** Sending a `Location` header alone does not terminate PHP execution.

### 21. What does `$_REQUEST` commonly combine?

**Answer:** GET, POST, and COOKIE values according to PHP configuration.

**Explanation:** The specific sources and their precedence may vary, so a specific superglobal is clearer when the input origin matters.

### 22. Why is a query string a poor place for a secret?

**Answer:** URLs may be stored in browser history, logs, and other places where they can be exposed.

**Explanation:** Query strings are useful for non-sensitive parameters, not credentials or secret tokens.

## Scenario questions

### 23. A client sends `Content-Type: application/json` with `{"name":"Maya"}`. Why is `$_POST['name']` likely missing?

**Answer:** `$_POST` does not automatically parse JSON request bodies.

**Explanation:** Read the raw body with `php://input`, decode it, then check that the expected `name` value exists and has the right type.

### 24. A response sets `Content-Type` after an included template has printed HTML. What may happen?

**Answer:** PHP may report that headers have already been sent and fail to send the new header.

**Explanation:** Set response headers before rendering or echoing the body.

### 25. A route supports `GET`, but receives `POST`. What response is appropriate?

**Answer:** Return `405 Method Not Allowed` and an `Allow: GET` header.

**Explanation:** The route exists, but the requested method is unsupported.

### 26. A valid create operation stores a new record. Which status is a good choice?

**Answer:** `201 Created`.

**Explanation:** It communicates that the operation created a resource successfully.

### 27. A delete operation succeeds and the endpoint has no body to return. Which status may be appropriate?

**Answer:** `204 No Content`.

**Explanation:** Do not emit a body with a `204` response.

### 28. A script redirects to a destination supplied directly by a query parameter. What is the concern?

**Answer:** The destination may be untrusted and could redirect the user to an unsafe external site.

**Explanation:** Use a fixed application path or validate destinations against an explicit allowlist.

### 29. A form sends `age[]=12`. Why should code not assume `$_POST['age']` is a string?

**Answer:** Bracket notation can make PHP parse the value as an array.

**Explanation:** Validate both the existence and type or shape of request values before using them.

### 30. A handler returns `500` when a user submits an invalid email address. What should it do instead?

**Answer:** Return an appropriate client-error response, such as `400` or `422`, depending on the API's validation design.

**Explanation:** `500` indicates an unexpected server-side failure, not ordinary invalid client input.

## Code-reading questions

For questions 31–35, assume this endpoint code:

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
echo 'Product: ' . ($_GET['id'] ?? 'missing');
```

### 31. What is the fallback request method if `REQUEST_METHOD` is unavailable?

**Answer:** `GET`.

**Explanation:** The null-coalescing operator chooses the fallback when the array key is missing or `null`.

### 32. What status is sent when the request method is not `GET`?

**Answer:** `405 Method Not Allowed`.

**Explanation:** The route explicitly sets status `405` and indicates the supported method with the `Allow` header.

### 33. Why does the script call `exit` in the unsupported-method branch?

**Answer:** To stop execution before it sends the normal response body.

**Explanation:** Without `exit`, the code could continue and append the product response after the error message.

### 34. If the request URL is `/products?id=42` and the method is `GET`, what body does the code produce?

**Answer:** `Product: 42`.

**Explanation:** The query parameter is available as `$_GET['id']`.

### 35. If the `id` query parameter is absent, what body does the code produce?

**Answer:** `Product: missing`.

**Explanation:** The null-coalescing operator supplies the fallback string when `$_GET['id']` is not set.

## Key takeaways

- Answers follow each question so the material can be reviewed immediately.
- Choose input sources based on where the client sent the data.
- JSON request bodies need explicit reading and decoding.
- Set an accurate response status and headers before producing the body.
- Validate client input and use method checks, redirects, and no-content responses deliberately.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [083 - Reflection API](../lessons/083-reflection-api.md) | [Lesson](../lessons/084-http-requests-responses-and-the-php-request-lifecycle.md) | [Example](../examples/084-http-requests-responses-and-the-php-request-lifecycle.md) | [085 - Generating HTML safely with templates](../lessons/085-generating-html-safely-with-templates.md) |
