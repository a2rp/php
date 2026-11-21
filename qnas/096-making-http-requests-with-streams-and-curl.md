# 096 - Making HTTP requests with streams and cURL: Q&A

## Multiple-choice questions

### 1. Which PHP option is often enough for a small HTTP request with basic controls?

- **A.** HTTP streams
- **B.** PHP sessions
- **C.** File uploads
- **D.** Output buffering

**Answer: A.** PHP's HTTP stream wrapper can configure a method, headers, body, timeout, and redirect behavior for straightforward requests.

### 2. What does the cURL extension provide?

- **A.** A way to store database records
- **B.** A handle-based interface with detailed transfer controls
- **C.** A way to render HTML templates
- **D.** Automatic API authorization

**Answer: B.** cURL offers controls and diagnostics for HTTP transfers, but the extension must be available in the PHP runtime.

### 3. What does CURLOPT_RETURNTRANSFER do?

- **A.** It prints the response body immediately
- **B.** It returns the response body from curl_exec()
- **C.** It turns off TLS checks
- **D.** It follows redirects

**Answer: B.** With CURLOPT_RETURNTRANSFER enabled, curl_exec() returns the response body for the application to inspect instead of sending it directly to output.

### 4. In cURL, does an HTTP 404 necessarily make curl_exec() return false?

- **A.** Yes, every non-2xx status is a transport failure
- **B.** No, curl_exec() can return a body; inspect the HTTP status separately
- **C.** Yes, unless redirects are enabled
- **D.** No, because cURL never returns false

**Answer: B.** A 404 is an HTTP response, not necessarily a transfer failure. Check the return from curl_exec() and retrieve the status with curl_getinfo().

### 5. What does ignore_errors in an HTTP stream context allow?

- **A.** It disables PHP error reporting for the whole script
- **B.** It permits reading a response body for HTTP error statuses
- **C.** It ignores TLS certificate errors
- **D.** It retries a failed request

**Answer: B.** It lets the stream wrapper return the response body even when the HTTP status indicates an error. The application must still inspect the status.

### 6. Which option limits the total time a cURL transfer may take?

- **A.** CURLOPT_TIMEOUT
- **B.** CURLOPT_RETURNTRANSFER
- **C.** CURLOPT_HTTPHEADER
- **D.** CURLOPT_FOLLOWLOCATION

**Answer: A.** CURLOPT_TIMEOUT places a total time bound on the transfer. CURLOPT_CONNECTTIMEOUT separately bounds the time allowed to establish a connection.

### 7. Which status should normally be returned by curl_getinfo() after a request?

- **A.** The HTTP response code
- **B.** The PHP process exit code
- **C.** The DNS server address
- **D.** The response body length only

**Answer: A.** Use CURLINFO_RESPONSE_CODE to obtain the last HTTP response status, then compare it with the status codes the application expects.

### 8. Why should TLS certificate verification remain enabled?

- **A.** It lets the client accept any certificate
- **B.** It helps verify the HTTPS server's certificate and hostname
- **C.** It makes HTTP use encryption automatically
- **D.** It replaces application authorization

**Answer: B.** Certificate and hostname verification help the client connect to the intended HTTPS server. Disabling them weakens protection against interception.

### 9. Why might an application disable automatic redirects?

- **A.** A redirect can point the server-side client to an unintended destination
- **B.** Redirects always contain malware
- **C.** HTTP does not support redirects
- **D.** It makes the response body larger

**Answer: A.** Redirects can change the destination. If the URL or service is user-controlled, following a redirect without revalidation can create SSRF risk.

### 10. What is server-side request forgery (SSRF)?

- **A.** A user causes the server to request destinations the user should not be able to reach
- **B.** A browser refuses to save a cookie
- **C.** A PHP function formats JSON incorrectly
- **D.** An API returns a 404 response

**Answer: A.** An attacker may use a server-side HTTP client to reach loopback services, private networks, or cloud metadata endpoints.

### 11. How should query parameters be assembled?

- **A.** Concatenate raw user values into a URL
- **B.** Use http_build_query() for parameter encoding
- **C.** Put every value in an Authorization header
- **D.** Use a session cookie as the query string

**Answer: B.** http_build_query() encodes query parameters consistently. Still validate values and use a trusted base URL.

### 12. Which header should accompany a JSON request body?

- **A.** Content-Type: application/json
- **B.** Accept: text/css
- **C.** Location: application/json
- **D.** Allow: multipart/form-data

**Answer: A.** Content-Type describes the format of the request body. Accept can separately indicate which response format the client wants.

## Short-answer questions

### 13. What is the difference between a transport failure and an HTTP error response?

**Answer:** A transport failure means PHP could not complete the exchange, such as because of DNS, connection, timeout, or TLS problems. An HTTP error status such as 404 means the remote server responded; the client should inspect that response and apply its own status policy.

### 14. Why set both a connection timeout and a total timeout in cURL?

**Answer:** The connection timeout bounds how long connection setup can take. The total timeout bounds the whole transfer, including sending and receiving data, so a slow service cannot occupy the PHP worker indefinitely.

### 15. Where can PHP streams expose the response headers?

**Answer:** The HTTP wrapper sets the $http_response_header variable in the scope where the stream request runs. The client can parse its status line and headers to understand the response.

### 16. Why should a client validate a response before using its body?

**Answer:** A remote service can return an unexpected status, content type, malformed JSON, or an oversized body. Check the transport result, status, response size, content type, and JSON decoding result before acting on the data.

### 17. Why should API credentials not be placed in the URL?

**Answer:** URLs often appear in server logs, browser history, monitoring systems, and error reports. Send secrets only to the intended service in its supported authorization header or credential mechanism, and avoid logging them.

### 18. What should the client do when it receives a redirect?

**Answer:** It should follow only if that behavior is needed and the destination is safe. For user-controlled or otherwise untrusted URLs, disable redirects or validate every redirect target before making the next request.

## Practical scenarios

### 19. curl_exec() returns a response body and curl_getinfo() reports 500. Did the HTTP transfer fail?

**Answer:** Not necessarily. The server completed the HTTP exchange and returned status 500. The application should treat it as an unsuccessful API result while distinguishing it from a network or TLS transfer failure.

### 20. A request hangs while waiting for a remote service. What should you configure?

**Answer:** Set a finite connection timeout and total timeout. For streams, set the HTTP context timeout. Also consider response-size limits and whether a retry is safe.

### 21. A user supplies a URL to fetch, and the PHP service follows redirects automatically. What can go wrong?

**Answer:** The user may direct the server to private or local services, including through a redirect. Prefer fixed destinations or a strict allowlist, disable redirects or revalidate each target, and apply outbound network restrictions.

### 22. Why is checking only that a URL begins with https:// insufficient?

**Answer:** HTTPS does not prove that the destination is authorized or public. The hostname may resolve to a private address or redirect elsewhere. Validate allowed destinations and enforce egress controls.

### 23. The API returns JSON with status 404, but file_get_contents() returns false. What should you inspect?

**Answer:** Check the stream context's ignore_errors option and inspect the local $http_response_header status line. With ignore_errors enabled, PHP can return the error response body; the application must still handle the 404 status.

### 24. Why is retrying a payment POST different from retrying a GET?

**Answer:** A GET is commonly intended to read data, while repeating a payment POST may perform the charge more than once. Retry state-changing requests only with an idempotency mechanism or a clear API contract.

### 25. What is wrong with this cURL request?

~~~php
curl_setopt($handle, CURLOPT_SSL_VERIFYPEER, false);
curl_setopt($handle, CURLOPT_TIMEOUT, 0);
curl_setopt($handle, CURLOPT_FOLLOWLOCATION, true);
~~~

**Answer:** It disables TLS peer verification, allows the transfer to run without a total timeout, and follows redirects without reviewing their destination. Keep verification enabled, use finite timeouts, and follow redirects only under a safe, explicit policy.

### 26. Which option should be used to send a JSON string using cURL?

- **A.** CURLOPT_POSTFIELDS with a JSON string and a matching Content-Type header
- **B.** CURLOPT_HTTPHEADER with the entire JSON document
- **C.** CURLOPT_URL with the JSON string appended as a fragment
- **D.** CURLOPT_TIMEOUT with the JSON string

**Answer: A.** Set the request body with CURLOPT_POSTFIELDS and send Content-Type: application/json. Also set Accept if the client expects a JSON response.

## Key takeaways

- HTTP streams suit straightforward requests; cURL provides more detailed transfer controls.
- A network failure differs from an HTTP 4xx or 5xx response.
- Set timeouts, check status codes, validate response bodies, and bound response size.
- Keep TLS verification enabled and handle redirects deliberately.
- Restrict outbound destinations when URLs can be influenced by users.
- Avoid logging secrets and take care when retrying state-changing requests.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [095 - Validating uploads and storing files safely](../lessons/095-validating-uploads-and-storing-files-safely.md) | [Lesson](../lessons/096-making-http-requests-with-streams-and-curl.md) | [Example](../examples/096-making-http-requests-with-streams-and-curl.md) | [097 - Building JSON and REST APIs](../lessons/097-building-json-and-rest-apis.md) |