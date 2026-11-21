# 089 - Response headers, status codes, and redirects: Q&A

## Multiple-choice questions

### 1. Which parts make up a typical HTTP response?

- **A.** Method, query string, and request body
- **B.** Status code, headers, and possibly a body
- **C.** Hostname, cookie, and PHP function
- **D.** Form fields, session, and template

**Answer: B.** A response communicates a status, metadata in headers, and usually a body. A `204 No Content` response is an example where there is no body.

### 2. When must PHP send a response header?

- **A.** After the entire body has been printed.
- **B.** Before actual output starts.
- **C.** Only after calling `exit`.
- **D.** Headers can be sent at any time.

**Answer: B.** `header()` must run before output begins. HTML, whitespace, a byte order mark, or output from an included file can make later headers fail.

### 3. Which PHP function sets a response status code?

- **A.** `http_response_code()`
- **B.** `response_status()`
- **C.** `set_status()`
- **D.** `status_header()`

**Answer: A.** `http_response_code(404)` sets the status to 404. `header()` is used to send headers such as `Content-Type`, `Location`, and `Allow`.

### 4. What header should accompany a 405 response?

- **A.** `Content-Length`
- **B.** `Allow`
- **C.** `Referer`
- **D.** `Accept-Encoding`

**Answer: B.** A `405 Method Not Allowed` response should identify supported methods in an `Allow` header, for example `Allow: GET, POST`.

### 5. What should a `204 No Content` response contain?

- **A.** A full HTML page.
- **B.** A JSON error object.
- **C.** No response body.
- **D.** A redirect URL in the body.

**Answer: C.** A 204 indicates success without response content. If a client needs a representation or message, use an appropriate status that permits a body.

### 6. Which redirect status is commonly used for Post/Redirect/Get?

- **A.** 303
- **B.** 204
- **C.** 401
- **D.** 422

**Answer: A.** A 303 directs the client to retrieve the destination as a separate request, commonly with GET after a POST submission.

### 7. Which redirect statuses preserve the original request method when followed?

- **A.** 301 and 302
- **B.** 303 and 302
- **C.** 307 and 308
- **D.** 201 and 204

**Answer: C.** 307 is a temporary redirect and 308 is a permanent redirect; both preserve the method and body when following the redirect.

## Short-answer questions

### 8. What is the role of a `Content-Type` response header?

**Answer:** It tells the client how to interpret the response body, for example as HTML, plain text, or JSON.

**Explanation:** The declared type must match the actual body. Setting `application/json` does not turn HTML into JSON; encode a data structure and return that JSON body.

### 9. What does `headers_sent($file, $line)` help diagnose?

**Answer:** It checks whether output has already caused headers to be sent. If so, it can provide the source file and line where output began.

**Explanation:** It is a diagnostic tool; it does not allow a response to send new headers after output has already started.

### 10. Why should code call `exit` after a redirect?

**Answer:** To stop the current script from continuing to produce a body or perform additional work after sending the redirect response.

### 11. Why is a redirect target from `$_GET['next']` risky?

**Answer:** An attacker may provide a URL to an external site, turning the application's redirect into an open redirect. Use a fixed local destination or validate a key against an application-controlled allowlist.

### 12. What is the difference between 401 and 403?

**Answer:** A 401 response indicates missing or invalid authentication credentials and typically includes a `WWW-Authenticate` challenge. A 403 response means the server understood the request but refuses access.

### 13. What should a production server return for an unexpected internal exception?

**Answer:** A generic error response with an appropriate 5xx status, while keeping stack traces and internal details in controlled server-side logs.

**Explanation:** Returning internal paths, SQL errors, credentials, or exception traces can expose implementation details to clients.

### 14. What information can a 201 response include when a new resource is created?

**Answer:** The response can identify the created resource with a `Location` header and may include a representation in the body.

## Code and behavior questions

### 15. Why can this redirect fail?

```php
echo '<p>Saved</p>';
header('Location: /account.php');
```

**Answer:** The script output the body before trying to send the `Location` header. Headers need to be sent before body output.

**Safer version:**

```php
header('Location: /account.php', true, 303);
exit;
```

### 16. What is missing from this method rejection?

```php
http_response_code(405);
echo 'Method not allowed';
```

**Answer:** The response should include an `Allow` header listing the accepted methods, such as `header('Allow: GET, POST');`. It should also set an appropriate content type before returning its body.

### 17. What response does this code intend to send?

```php
http_response_code(204);
exit;
```

**Answer:** It intends to return a successful response with no content. It should not emit an HTML or JSON body after setting the 204 status.

### 18. Does this code send JSON?

```php
header('Content-Type: application/json; charset=UTF-8');
echo '<h1>Done</h1>';
```

**Answer:** No. The header claims the body is JSON, but the body is HTML markup. Encode a PHP value with `json_encode()` and return the resulting JSON string, or change the content type to match an HTML body.

### 19. What happens if PHP sets `Location` without an explicit status code?

**Answer:** PHP normally uses a redirect status of 302 unless another status was already set. Specify the intended status explicitly when the redirect's semantics matter, such as `header('Location: /done.php', true, 303)` for PRG.

### 20. Is output buffering a good permanent fix for a late `header()` call?

**Answer:** Usually not. Output buffering can delay when output reaches the client, but the response code should be structured so status and headers are chosen before rendering begins. Use buffering only when it is an intentional part of the design.

## Key takeaways

- Set a status and headers before sending the response body.
- Match the `Content-Type` to the body you actually return.
- Use `Allow` with 405, and do not send a body with 204.
- Use 303 for Post/Redirect/Get; 307 and 308 preserve the request method.
- Keep redirect destinations controlled and stop execution after redirecting.
- Keep detailed diagnostics private and return appropriate status codes to clients.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [088 - Input validation and normalization](../lessons/088-input-validation-and-normalization.md) | [Lesson](../lessons/089-response-headers-status-codes-and-redirects.md) | [Example](../examples/089-response-headers-status-codes-and-redirects.md) | [090 - Cookies and browser state](../lessons/090-cookies-and-browser-state.md) |