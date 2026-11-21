# 112 - Routing, controllers, and middleware: Q&A

## Multiple-choice questions

### 1. What does a route commonly map to a handler?

- **A.** An HTTP method and a URL path pattern
- **B.** A database password and a PHP filename
- **C.** A response body and a CSS selector
- **D.** A Composer package and a log file

**Answer: A.** Routing selects a handler using request information such as the HTTP method and path. **Explanation:** `GET /orders` and `POST /orders` may select different operations even though they share a path.

### 2. Why can `GET /orders` and `POST /orders` represent different actions?

- **A.** Routes commonly match both method and path
- **B.** PHP changes the path automatically for POST
- **C.** GET requests cannot contain headers
- **D.** POST requests skip the web server

**Answer: A.** The method is part of route selection. **Explanation:** A GET route might retrieve a list, while a POST route submits data to create or process an order.

### 3. Where is `?status=open&page=2` in `/orders?status=open&page=2`?

- **A.** The query string
- **B.** The path parameter
- **C.** The request body
- **D.** The response header

**Answer: A.** The values after `?` form the query string. **Explanation:** The path identifies a route or resource, while query parameters commonly provide filters or representation options.

### 4. What is the typical distinction between 404 and 405?

- **A.** 404 means no route/resource was found; 405 means the path exists but not for that method
- **B.** 404 means the password is wrong; 405 means the server is offline
- **C.** 404 means a database failed; 405 means a template is missing
- **D.** They always mean the same thing

**Answer: A.** A missing path and an unsupported method are different outcomes. **Explanation:** A 405 response can include an `Allow` header listing methods supported at that path.

### 5. What is a controller's main responsibility?

- **A.** Translate HTTP input to application calls and map results to responses
- **B.** Implement every database query and business rule in the project
- **C.** Replace the web server
- **D.** Generate Composer's autoloader

**Answer: A.** A controller adapts HTTP details to application behavior. **Explanation:** Reusable business rules and persistence details belong in their appropriate application, domain, and infrastructure responsibilities.

### 6. What can middleware do when it receives a request?

- **A.** Inspect or change it, return a response, or pass it to the next handler
- **B.** Only render HTML templates
- **C.** Only change the URL path
- **D.** Replace authentication and authorization as one feature

**Answer: A.** Middleware participates in request processing and may continue or short-circuit the pipeline. **Explanation:** It can also inspect or modify the response after downstream processing completes.

### 7. Why is exception-handling middleware often placed around the rest of a pipeline?

- **A.** So it can catch failures from downstream middleware and handlers
- **B.** So it can skip every controller permanently
- **C.** So all exceptions become successful responses
- **D.** So the router does not need routes

**Answer: A.** An outer exception boundary can observe failures from the processing it wraps. **Explanation:** It can log diagnostics safely and convert an unexpected error into a generic server response.

### 8. Why should a state-changing operation not be exposed only through GET?

- **A.** Clients, crawlers, previews, or caches may issue GET without intending a change
- **B.** GET requests cannot be routed
- **C.** GET requests cannot return a response body
- **D.** POST requests are automatically authenticated

**Answer: A.** GET is intended for retrieval and may be triggered in ways that do not represent deliberate submission. **Explanation:** Use an appropriate state-changing method and apply the relevant protections.

### 9. What does authentication answer?

- **A.** Who the caller is
- **B.** Which route pattern matched
- **C.** Whether a response has a body
- **D.** Which database stores the record

**Answer: A.** Authentication establishes or verifies the caller's identity. **Explanation:** Authorization is a separate decision about what that caller may do.

### 10. What does CORS not provide by itself?

- **A.** Authentication or authorization
- **B.** A browser cross-origin policy mechanism
- **C.** Response headers
- **D.** A way for a browser to decide whether a response is accessible to a page

**Answer: A.** CORS is not an identity or access-control system. **Explanation:** It does not replace authentication, authorization, or CSRF protections.

### 11. What is PHP's built-in web server intended for?

- **A.** Local development and controlled demonstrations
- **B.** Hosting public production applications
- **C.** Replacing every production web server
- **D.** Serving only database backups

**Answer: A.** The built-in server is useful for development and demonstrations. **Explanation:** It is not intended to be a production web server.

### 12. Which statements describe PSR-7 and PSR-15?

- **A.** PSR-7 defines HTTP message interfaces; PSR-15 defines server handler and middleware interfaces
- **B.** PSR-7 defines SQL syntax; PSR-15 defines HTML tags
- **C.** PSR-7 defines routes; PSR-15 defines Composer packages
- **D.** Both require every PHP application to use the same framework

**Answer: A.** The standards define interoperable contracts for HTTP messages and server request processing. **Explanation:** A project may instead use a framework's own APIs when that is the chosen stack.

## Short-answer questions

### 13. What are the main parts of an HTTP response?

**Answer:** A status code, headers, and an optional body. **Explanation:** The status communicates the result, headers describe response metadata or behavior, and the body may contain HTML, JSON, text, or no content.

### 14. Distinguish a path parameter from a query parameter.

**Answer:** A path parameter is part of the route path, such as the order number in `/orders/ORD-17`; a query parameter follows `?`, such as `page=2`. **Explanation:** Both come from the client and must be validated before use.

### 15. What does middleware mean by “wrapping” a handler?

**Answer:** Middleware can process the request before delegation and process the returned response afterward. **Explanation:** This supports shared behavior without copying it into each controller.

### 16. What header may accompany a 405 response?

**Answer:** The `Allow` header can list the methods supported for the requested path. **Explanation:** It helps the client understand which methods are accepted there.

### 17. Why might a browser form that changes data need CSRF protection?

**Answer:** Cookie-authenticated browsers may attach credentials to a request initiated by another site. **Explanation:** CSRF protection helps distinguish an intentional form submission from a forged cross-site request.

### 18. Distinguish authentication from authorization.

**Answer:** Authentication verifies who the caller is; authorization decides whether that caller may perform a specific action. **Explanation:** An authenticated user can still be forbidden from viewing or changing a particular resource.

### 19. How should an application respond to an unexpected exception in production?

**Answer:** Log useful diagnostic details in a protected place and return a generic server error to the client. **Explanation:** Stack traces, file paths, SQL errors, and secrets should not be exposed in public responses.

### 20. When is 201 Created commonly appropriate?

**Answer:** When a request successfully creates a resource. **Explanation:** A `Location` header can identify the new resource when that is useful.

### 21. What happens to a static asset that the web server can serve directly?

**Answer:** The server can return it without running the PHP application. **Explanation:** Static files often bypass the front controller and middleware pipeline entirely.

### 22. Why have an outer exception boundary instead of repeating broad catches in every controller?

**Answer:** It provides one consistent place to catch unexpected failures and convert them to safe responses. **Explanation:** Controllers can then translate expected outcomes without hiding unrelated programming errors.

### 23. Why must a route parameter be treated as untrusted input?

**Answer:** The client controls the request path and can submit unexpected or malicious values. **Explanation:** Validate its format and check whether the caller may access the referenced resource.

### 24. What kind of behavior is a good fit for middleware?

**Answer:** Shared request-pipeline behavior such as logging, exception conversion, or security headers. **Explanation:** A feature-specific business rule usually belongs in an application use case or domain responsibility.

## Practical and scenario questions

### 25. An application has both `GET /orders` and `POST /orders`. Should the router treat them as the same route?

**Answer:** No. Match both method and path so each request reaches the intended handler. **Explanation:** GET may list orders, while POST may submit data to create one.

### 26. No route matches `/unknown-page`, regardless of its method. What response is normally appropriate?

**Answer:** 404 Not Found. **Explanation:** There is no matching route or resource for that path.

### 27. `/orders` exists for GET, but the client sends DELETE and the application does not support it. What response is normally appropriate?

**Answer:** 405 Method Not Allowed, with an `Allow` header when the application can provide supported methods. **Explanation:** The path exists, but the requested method is not accepted there.

### 28. A browser form changes account settings using a cookie-based session. What should the application consider beyond validating the submitted fields?

**Answer:** CSRF protection, authentication, authorization for that account, and appropriate response handling. **Explanation:** Valid data alone does not prove that the request was intentionally submitted by an authorized user.

### 29. A controller contains SQL, order pricing, email delivery, and HTML rendering. How should the code be reorganized?

**Answer:** Keep HTTP translation in the controller, move the use case and business rules into application/domain responsibilities, and isolate SQL and email behind infrastructure adapters. **Explanation:** This separates different reasons for code to change and makes behavior easier to reuse.

### 30. An exception page returns the SQL connection string and full stack trace to every visitor. What should change?

**Answer:** Return a generic error response and write redacted diagnostic details to protected logs. **Explanation:** Public error output can reveal credentials, file paths, and implementation details.

### 31. A cache middleware finds a valid cached response. Must it call the next handler?

**Answer:** No. It can return the cached response directly. **Explanation:** Middleware can short-circuit when it already has a complete response.

### 32. A middleware needs to log the final response status and add a response header. At what point can it do that?

**Answer:** It can delegate to the next handler, then inspect or modify the returned response before returning it. **Explanation:** Middleware commonly participates both before and after downstream processing.

### 33. A router converts `/pages/../../config/secrets.php` into a PHP include path. What is the risk and a safer design?

**Answer:** The request may escape the intended directory and expose or execute private files. **Explanation:** Map known method/path patterns to known handlers instead of including files named from client-controlled paths.

### 34. A developer starts PHP's built-in server on a public production host because it works locally. What should they do?

**Answer:** Use a production web server and deployment setup; keep the built-in server for local development or controlled demonstrations. **Explanation:** PHP documents the built-in server as not intended for production use.

### 35. A team enables permissive CORS and assumes that private account routes are now protected. What is missing?

**Answer:** Actual authentication and resource-level authorization, plus CSRF protection where cookie-authenticated browser requests require it. **Explanation:** CORS controls browser cross-origin access behavior; it does not establish identity or permission.

### 36. Plain PHP emits the response body and only afterward calls `header('Content-Type: application/json')`. What is wrong?

**Answer:** Headers should be sent before output begins. **Explanation:** Once output is sent, PHP may no longer be able to modify the HTTP headers reliably.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [111 - Project structure, namespaces, and application layers](../lessons/111-project-structure-namespaces-and-application-layers.md) | [Lesson](../lessons/112-routing-controllers-and-middleware.md) | [Example](../examples/112-routing-controllers-and-middleware.md) | [113 - Dependency injection and service design](../lessons/113-dependency-injection-and-service-design.md) |
