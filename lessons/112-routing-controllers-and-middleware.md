# 112 - Routing, controllers, and middleware

## Goals

After this lesson, you should be able to:

- Explain how an HTTP request reaches a PHP application and becomes a response.
- Describe how a router matches an HTTP method and URL path to an action.
- Distinguish path parameters from query parameters and request body data.
- Keep controllers focused on translating between HTTP and application use cases.
- Explain how middleware can inspect, change, stop, or continue request processing.
- Choose middleware order deliberately and handle failures without exposing private details.
- Recognize when a framework router and HTTP stack are safer than a custom implementation.

## From an HTTP request to a PHP response

A browser or API client sends an HTTP request to a web server. The server serves a matching static file directly or forwards the request to the application's public entry point. That entry point loads dependencies and starts request processing. The application returns an HTTP response containing a status, headers, and possibly a body.

~~~text
client
  -> web server and public document root
  -> public entry point
  -> middleware pipeline
  -> router selects a route
  -> controller translates the request
  -> application use case and domain rules
  -> controller creates a response
  -> middleware finishes response processing
  -> web server sends the response to the client
~~~

This is a common shape, not a mandate that every framework uses exactly this order or these class names. Some stacks place routing before certain middleware, and static files often never enter PHP at all.

## What a route does

A route connects an HTTP method and a path pattern to a handler. For example:

| Method | Path pattern | Typical purpose |
| --- | --- | --- |
| `GET` | `/orders` | Show a list of orders |
| `GET` | `/orders/{orderNumber}` | Show one order |
| `POST` | `/orders` | Submit data to create an order |
| `PATCH` | `/orders/{orderNumber}` | Change selected order fields |
| `DELETE` | `/orders/{orderNumber}` | Request removal of an order |

A router typically considers both the method and the path. `GET /orders` and `POST /orders` can be different actions even though they share a path. A path parameter such as `{orderNumber}` captures part of the path for the selected handler.

Routes describe how an HTTP operation reaches application behavior. They should not contain a full database implementation or all the business rules themselves.

### HTTP method meaning

Use methods consistently with the operation being requested:

- `GET` retrieves a representation and should not perform a business-changing action.
- `POST` commonly submits data to create or process a resource.
- `PUT` commonly replaces a resource representation.
- `PATCH` commonly applies a partial update.
- `DELETE` requests that a resource be removed.

Method names are not access control. A route still needs authentication and authorization where appropriate. Do not make a state-changing action available only through `GET`; clients, crawlers, caches, or previews may issue `GET` requests without the user intending a change.

### Path, query, and body data

These parts of a request serve different purposes:

- **Path:** identifies a resource or route, such as `/orders/ORD-17`.
- **Query string:** supplies optional filters or representation choices, such as `/orders?status=open&page=2`.
- **Request body:** carries submitted data, commonly for `POST`, `PUT`, or `PATCH`.
- **Headers:** carry metadata such as content type, accepted formats, and authentication credentials.

Treat all client-supplied values as untrusted. Validate path parameters, query values, and body fields for the specific operation. A route parameter being present does not mean it has the right format or refers to a record the current user may access.

## Route matching and response outcomes

A router should have a clear policy for matching paths, trailing slashes, case, and encoded characters. Frameworks provide established behavior for these details. A hand-built router should keep its matching rules explicit and test the boundaries.

A request can fail to match in two different ways:

- **404 Not Found:** no route matches the requested path.
- **405 Method Not Allowed:** the path is known, but it does not accept the requested method. A response can include an `Allow` header listing supported methods.

Do not turn an unknown URL into a PHP filename and include it. Building file paths from a request path can expose source files or create path traversal and code execution risks. Use a route table or a maintained router that maps known patterns to known handlers.

Route parameters should be passed as data to the selected handler. They should not be concatenated into a SQL query, filesystem path, or PHP include statement without the relevant validation and safe API.

## Controllers translate HTTP and application behavior

A controller is an HTTP-facing adapter for one or more routes. A controller action commonly:

1. Reads the route parameter or parsed request data it needs.
2. Performs boundary-level validation and chooses an application use case.
3. Calls the use case with application data, not a raw web server global if avoidable.
4. Maps the result or expected failure to an HTTP status, headers, and body.

For example, `OrderController::show()` could receive an order number, ask an application service to retrieve the order, and return either an order page or a not-found response. The controller should not own SQL statements, order pricing rules, or a large collection of unrelated decisions.

A controller can be a class or a callable, depending on the framework and project. The word describes a responsibility, not a required PHP syntax.

### Keep controllers thin and clear

“Thin controller” does not mean a controller can contain no logic. It may perform HTTP-specific translation, such as choosing a response status or mapping form fields. Keep reusable business rules and multi-step application behavior in the appropriate application or domain responsibility.

A controller that performs every database query, authorization rule, business calculation, email send, and HTML rendering becomes difficult to reuse and reason about. Split work by responsibility when that improves clarity, not merely to meet a class-count target.

### HTTP responses

A response includes:

- A status code describing the result.
- Headers describing metadata or behavior, such as `Content-Type` or `Location`.
- An optional body, such as HTML, JSON, or an empty body.

Common examples include:

| Status | Typical use |
| --- | --- |
| `200 OK` | A request succeeded and a representation is returned |
| `201 Created` | A request created a resource; a `Location` header can identify it |
| `204 No Content` | A request succeeded and no response body is needed |
| `302 Found` or `303 See Other` | Redirect a client, often after form submission |
| `400 Bad Request` | The request is malformed or cannot be understood |
| `401 Unauthorized` | Authentication is required or invalid; HTTP authentication schemes have additional header requirements |
| `403 Forbidden` | The request is understood but the caller is not allowed to perform it |
| `404 Not Found` | The resource or route is not available |
| `405 Method Not Allowed` | The path exists but does not support this method |
| `422 Unprocessable Content` | The request format is understood but its submitted values fail application validation |
| `500 Internal Server Error` | An unexpected server failure occurred |

Use status codes consistently with the application's API or framework conventions. Do not return `200` for every outcome and rely on a message in the body to communicate failure.

When emitting a response in plain PHP, send headers before writing the body. Framework response objects usually make this order explicit. If using immutable HTTP message interfaces, methods such as `withHeader()` return a new response object; keep the returned object.

## Middleware handles shared request and response concerns

Middleware is a component that participates in request processing around a later handler. It may inspect or modify the request, return a response immediately, pass the request to the next handler, and inspect or modify the response afterward.

~~~text
request -> middleware A -> middleware B -> route/controller
response <- middleware A <- middleware B <- route/controller
~~~

That two-way flow makes middleware a useful home for cross-cutting HTTP behavior that should not be copied into every controller. Common examples include:

- Request identifiers and access logging.
- Exception handling and conversion to a safe error response.
- Session setup.
- Authentication checks and route-level authorization integration.
- CSRF protection for applicable browser form submissions.
- Content negotiation or request-body parsing.
- Security headers and response timing.

Whether a concern belongs in middleware depends on the framework and application. A rule specific to placing an order belongs in the order use case or domain, not in generic middleware.

### Middleware can short-circuit

Middleware does not always have to call the next handler. It can return a response directly when it has a complete answer, for example:

- A health-check handler returns a status response.
- An authentication guard rejects an unauthenticated request.
- A cache returns a stored response.
- A request-size limit rejects a body before the controller runs.

When middleware delegates, it can continue processing after the handler returns, such as adding a request ID header or recording response duration.

### Order is part of behavior

Middleware order changes what the application does. An exception-handling component is commonly placed outside the rest of the pipeline so it can catch failures from later middleware and handlers. Session setup may need to happen before middleware that reads the session. Authentication should run before protected controller actions. Response logging may need to wrap downstream processing to record the final status.

Frameworks define how middleware is registered and ordered. Read that framework's rules instead of assuming that every stack uses the same convention.

## Error handling and safe responses

Expected outcomes should become appropriate responses: not found, validation errors, authentication failures, or forbidden access. Unexpected exceptions should be logged with useful diagnostic context and converted to a generic server error for the client.

Do not send stack traces, SQL errors, local file paths, environment values, or secret-bearing request data to a public response. Detailed errors belong in access-controlled logs, with sensitive values omitted or redacted. A request identifier can help connect a safe client-facing error to a server-side log entry.

Avoid catching every exception in every controller just to return an empty success response. Handle expected failures where they can be translated meaningfully, and use an outer exception boundary for unexpected failures.

## Security checks at the HTTP boundary

Routing does not make a request trusted. Before an operation changes data, consider:

- Is the caller authenticated when authentication is required?
- Is the caller authorized to act on this specific resource?
- Is the submitted data valid for this use case?
- If browser cookies authenticate the request, is CSRF protection required?
- Is user-controlled output encoded for the context where it is rendered?
- Are redirects restricted to appropriate destinations?

Authentication identifies a caller; authorization decides what that caller may do. A valid route parameter or an unguessable-looking identifier is not a substitute for an authorization check.

CORS controls which browser origins may make certain cross-origin requests. It is not authentication, authorization, or CSRF protection.

## Standards and framework interfaces

PSR-7 defines common PHP interfaces for HTTP request and response messages. PSR-15 defines interfaces for server request handlers and middleware that process those messages. A framework or library can implement these contracts so different components interoperate.

A project may also use its framework's own request, response, controller, and middleware APIs. Do not add PSR packages solely because they exist; use the interfaces required by the chosen stack or needed at a real integration boundary.

## Development server and production server

PHP includes a built-in web server that is useful for local demonstrations and development. It is not a production web server. For a local application whose public document root is `public/`, a development command can specify that root explicitly:

~~~sh
php -S localhost:8000 -t public
~~~

Use a production web server and deployment configuration appropriate for the application. Keep the document root pointed at the public directory and ensure non-public files are not served.

## Common mistakes

### Matching only the path and ignoring the method

`GET /orders` and `POST /orders` usually represent different actions. Include the method in route selection and return a clear response for unsupported methods.

### Putting application logic in the router

A router should select a handler. It should not become a large switch statement containing all order, billing, or account rules.

### Making a controller responsible for every layer

Controllers translate HTTP and call application behavior. Put persistence in an infrastructure adapter and reusable business rules in domain or application responsibilities.

### Treating middleware as a place for every rule

Middleware is best for behavior that applies across a request pipeline. Feature-specific decisions usually belong in a use case or domain service.

### Using `GET` to change state

GET requests may be triggered by links, crawlers, previews, and caches. Require an appropriate state-changing method and applicable protections for operations that modify data.

### Returning a success status for failures

Use a status that reflects the outcome. A client should not need to inspect a custom body string to discover that the operation failed.

### Exposing exception details to clients

Return a safe error response and keep diagnostics in protected logs. Detailed server internals can reveal secrets and make attacks easier.

### Building a custom production router without a need

A minimal router is useful for learning, but production routing needs reliable method handling, parameter parsing, error behavior, and security review. Prefer a maintained framework or router when the application needs those features.

## A practical request-handling checklist

1. Configure the web server to serve only the public document root.
2. Let static assets be served directly when appropriate.
3. Load the application through one public entry point.
4. Run shared middleware in a documented order.
5. Match both the HTTP method and route pattern.
6. Validate boundary data and check authentication and authorization.
7. Keep the controller focused on HTTP-to-application translation.
8. Run business behavior through the application and domain layers.
9. Return a status, headers, and body that match the outcome.
10. Log unexpected failures privately and return safe error messages.

## Key takeaways

- A route maps an HTTP method and path pattern to a handler.
- Path, query, body, and header values are all client input and must be validated for their use.
- A controller translates HTTP details to application calls and maps results to responses.
- Middleware wraps request processing and can continue, modify, or short-circuit it.
- Middleware order affects behavior; exception handling is commonly placed around the rest of the pipeline.
- Use HTTP status codes to describe outcomes instead of returning success for every request.
- Authentication, authorization, CSRF protection, CORS, and input validation solve different problems.
- A custom router is useful for learning; mature applications should use a maintained router or framework where appropriate.
- PHP's built-in server is for development and demonstrations, not production hosting.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [111 - Project structure, namespaces, and application layers](111-project-structure-namespaces-and-application-layers.md) | [Q&A](../qnas/112-routing-controllers-and-middleware.md) | [Example](../examples/112-routing-controllers-and-middleware.md) | [113 - Dependency injection and service design](113-dependency-injection-and-service-design.md) |
