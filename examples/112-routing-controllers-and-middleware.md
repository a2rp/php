# 112 - Routing, controllers, and middleware: Example

## Goal

Create a small PHP web application that routes requests by method and path, sends matched requests to a controller, and wraps dispatch in middleware.

## Question

How do a front controller, router, controller, request/response objects, and middleware work together to handle a request and return a consistent HTTP response?

## Assumptions

- PHP 8.1 or newer and Composer are available.
- Commands are run from the project root.
- This example uses a tiny custom router and message classes to make the flow visible. They are for learning, not a replacement for a maintained production framework or a complete PSR-7/PSR-15 implementation.
- The PHP built-in server is used only for a local demonstration.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
routing-demo/
├── public/
│   └── index.php
├── src/
│   ├── Controller/
│   │   └── GreetingController.php
│   ├── Http/
│   │   ├── Middleware/
│   │   │   ├── ExceptionMiddleware.php
│   │   │   ├── Middleware.php
│   │   │   └── RequestIdMiddleware.php
│   │   ├── Request.php
│   │   ├── Response.php
│   │   └── Router.php
│   └── Infrastructure/
│       └── (other adapters can be added when needed)
├── .gitignore
└── composer.json
~~~

This example keeps its small route handlers in one controller. A larger application can call application use cases from controllers and place business rules in the domain layer, as shown in the previous chapter.

## 2. Configure Composer autoloading

Create `composer.json`:

~~~json
{
    "name": "example/routing-demo",
    "type": "project",
    "description": "A small PHP example of routing, controllers, and middleware",
    "require": {
        "php": "^8.1"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/"
        }
    }
}
~~~

Create `.gitignore`:

~~~gitignore
/vendor/
~~~

From the project root, generate Composer's autoloader:

~~~sh
composer install
~~~

## 3. Represent the incoming request

Create `src/Http/Request.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http;

final class Request
{
    public function __construct(
        private readonly string $method,
        private readonly string $path,
    ) {
    }

    public static function fromGlobals(): self
    {
        $method = strtoupper($_SERVER['REQUEST_METHOD'] ?? 'GET');
        $requestUri = $_SERVER['REQUEST_URI'] ?? '/';
        $path = parse_url($requestUri, PHP_URL_PATH);

        if (!is_string($path) || $path === '') {
            $path = '/';
        }

        return new self($method, $path);
    }

    public function method(): string
    {
        return $this->method;
    }

    public function path(): string
    {
        return $this->path;
    }
}
~~~

This small request object extracts only the method and path needed by this example. A real application also needs deliberate handling for headers, query parameters, body parsing, cookies, uploaded files, and server configuration.

## 4. Represent and send a response

Create `src/Http/Response.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http;

final class Response
{
    /** @param array<string, string> $headers */
    public function __construct(
        private readonly int $status,
        private readonly string $body,
        private readonly array $headers = [],
    ) {
    }

    public function withHeader(string $name, string $value): self
    {
        $headers = $this->headers;
        $headers[$name] = $value;

        return new self($this->status, $this->body, $headers);
    }

    public function send(): void
    {
        http_response_code($this->status);

        foreach ($this->headers as $name => $value) {
            header($name . ': ' . $value, true);
        }

        echo $this->body;
    }
}
~~~

The response sends the status and headers before the body. `withHeader()` returns a new response instance, so middleware must keep the returned response.

## 5. Define the middleware contract

Create `src/Http/Middleware/Middleware.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use App\Http\Request;
use App\Http\Response;

interface Middleware
{
    /** @param callable(Request): Response $next */
    public function process(Request $request, callable $next): Response;
}
~~~

This small project-specific contract receives a request and a callable for the next step. It demonstrates the pipeline shape without implementing the PSR-15 interfaces.

## 6. Add request ID middleware

Create `src/Http/Middleware/RequestIdMiddleware.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use App\Http\Request;
use App\Http\Response;

final class RequestIdMiddleware implements Middleware
{
    /** @param callable(Request): Response $next */
    public function process(Request $request, callable $next): Response
    {
        $requestId = bin2hex(random_bytes(8));
        $response = $next($request);

        return $response->withHeader('X-Request-Id', $requestId);
    }
}
~~~

The middleware calls the next handler and then adds a response header. A production request identifier may also need to be attached to the request context and included in structured logs.

## 7. Convert unexpected exceptions to a safe response

Create `src/Http/Middleware/ExceptionMiddleware.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http\Middleware;

use App\Http\Request;
use App\Http\Response;
use Throwable;

final class ExceptionMiddleware implements Middleware
{
    /** @param callable(Request): Response $next */
    public function process(Request $request, callable $next): Response
    {
        try {
            return $next($request);
        } catch (Throwable $exception) {
            error_log('Unhandled request exception: ' . $exception::class);

            return new Response(
                500,
                '<!doctype html><html lang="en"><title>Server error</title>'
                    . '<h1>Internal Server Error</h1>',
                ['Content-Type' => 'text/html; charset=utf-8'],
            );
        }
    }
}
~~~

The client receives a generic response; the server log records a limited diagnostic. Production logging should use protected logs and include only appropriate, redacted context.

## 8. Match routes by method and path

Create `src/Http/Router.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Http;

use LogicException;

final class Router
{
    /** @var list<array{method: string, path: string, handler: callable}> */
    private array $routes = [];

    public function add(string $method, string $path, callable $handler): void
    {
        $this->routes[] = [
            'method' => strtoupper($method),
            'path' => $path,
            'handler' => $handler,
        ];
    }

    public function dispatch(Request $request): Response
    {
        $allowedMethods = [];

        foreach ($this->routes as $route) {
            $parameters = $this->matchPath($route['path'], $request->path());

            if ($parameters === null) {
                continue;
            }

            $allowedMethods[] = $route['method'];

            if ($route['method'] === $request->method()) {
                $response = ($route['handler'])($request, $parameters);

                if (!$response instanceof Response) {
                    throw new LogicException('A route handler must return a Response.');
                }

                return $response;
            }
        }

        if ($allowedMethods !== []) {
            $allowedMethods = array_values(array_unique($allowedMethods));
            sort($allowedMethods);

            return new Response(
                405,
                'Method Not Allowed',
                [
                    'Allow' => implode(', ', $allowedMethods),
                    'Content-Type' => 'text/plain; charset=utf-8',
                ],
            );
        }

        return new Response(
            404,
            'Not Found',
            ['Content-Type' => 'text/plain; charset=utf-8'],
        );
    }

    /** @return array<string, string>|null */
    private function matchPath(string $pattern, string $path): ?array
    {
        $patternParts = $this->segments($pattern);
        $pathParts = $this->segments($path);

        if (count($patternParts) !== count($pathParts)) {
            return null;
        }

        $parameters = [];

        foreach ($patternParts as $index => $patternPart) {
            $pathPart = rawurldecode($pathParts[$index]);

            if (preg_match('/^\{([A-Za-z][A-Za-z0-9_]*)\}$/', $patternPart, $matches) === 1) {
                if (str_contains($pathPart, '/') || str_contains($pathPart, "\0")) {
                    return null;
                }

                $parameters[$matches[1]] = $pathPart;
                continue;
            }

            if (rawurldecode($patternPart) !== $pathPart) {
                return null;
            }
        }

        return $parameters;
    }

    /** @return list<string> */
    private function segments(string $path): array
    {
        if ($path === '/') {
            return [];
        }

        return explode('/', trim($path, '/'));
    }
}
~~~

The router checks both the method and path. It returns 404 when no path matches and 405 when a path matches but the method does not. A 405 includes an `Allow` header. The tiny matcher treats leading and trailing slashes consistently and accepts one path segment for each `{parameter}`. A production router should use a maintained implementation with carefully defined URL and method behavior.

## 9. Create the controller

Create `src/Controller/GreetingController.php`:

~~~php
<?php

declare(strict_types=1);

namespace App\Controller;

use App\Http\Request;
use App\Http\Response;

final class GreetingController
{
    /** @param array<string, string> $parameters */
    public function home(Request $request, array $parameters): Response
    {
        return new Response(
            200,
            '<!doctype html><html lang="en"><meta charset="utf-8">'
                . '<title>Routing demo</title><h1>Routing demo</h1>'
                . '<p>Try <a href="/hello/Ashish">/hello/Ashish</a>.</p>',
            ['Content-Type' => 'text/html; charset=utf-8'],
        );
    }

    /** @param array<string, string> $parameters */
    public function greet(Request $request, array $parameters): Response
    {
        $name = htmlspecialchars(
            $parameters['name'] ?? 'guest',
            ENT_QUOTES | ENT_SUBSTITUTE,
            'UTF-8',
        );

        return new Response(
            200,
            '<!doctype html><html lang="en"><meta charset="utf-8">'
                . '<title>Greeting</title><h1>Hello, ' . $name . '!</h1>',
            ['Content-Type' => 'text/html; charset=utf-8'],
        );
    }
}
~~~

The controller receives the request and matched parameters, then creates a response. It escapes the name for HTML output because it came from the URL and is controlled by the client.

## 10. Wire the front controller and middleware pipeline

Create `public/index.php`:

~~~php
<?php

declare(strict_types=1);

use App\Controller\GreetingController;
use App\Http\Middleware\ExceptionMiddleware;
use App\Http\Middleware\RequestIdMiddleware;
use App\Http\Request;
use App\Http\Response;
use App\Http\Router;

require dirname(__DIR__) . '/vendor/autoload.php';

$router = new Router();
$controller = new GreetingController();

$router->add('GET', '/', [$controller, 'home']);
$router->add('GET', '/hello/{name}', [$controller, 'greet']);

$request = Request::fromGlobals();
$middleware = [
    new ExceptionMiddleware(),
    new RequestIdMiddleware(),
];

$handler = static fn (Request $request): Response => $router->dispatch($request);

foreach (array_reverse($middleware) as $component) {
    $next = $handler;
    $handler = static fn (Request $request): Response => $component->process($request, $next);
}

$response = $handler($request);
$response->send();
~~~

The front controller loads Composer, registers known routes, creates the request, wraps dispatch in middleware, and sends the response. The exception middleware is outermost in execution order so it can catch errors from the request ID middleware and router.

## 11. Run the local demonstration

From the project root, start PHP's built-in server with the public entry point as its router script:

~~~sh
php -S localhost:8000 -t public public/index.php
~~~

Open these paths in a browser or request them with an HTTP client:

| Request | Expected result |
| --- | --- |
| `GET /` | `200 OK` with the routing demo page |
| `GET /hello/Ashish` | `200 OK` with `Hello, Ashish!` |
| `GET /missing` | `404 Not Found` |
| `POST /hello/Ashish` | `405 Method Not Allowed` with `Allow: GET` |

Successful responses also include an `X-Request-Id` response header. The generated value changes on every request.

## How the request flows

~~~text
public/index.php
    -> ExceptionMiddleware
    -> RequestIdMiddleware
    -> Router
    -> GreetingController
    -> Response
~~~

The router selects a handler, but it does not implement greeting behavior. The controller translates route data to an HTML response. Middleware adds behavior around dispatch, and the response object sends its status and headers before the body.

## Important notes

- The custom router is deliberately small so its behavior is visible; use a maintained router or framework for production requirements.
- The request object does not parse a request body, cookies, or headers. Add a proper HTTP message implementation when the application needs those features.
- The demo's response and middleware contracts are educational and are not complete PSR-7 or PSR-15 implementations.
- In a real application, controllers should call application use cases rather than accumulate business rules.
- The built-in web server is for local development and demonstrations, not production hosting.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [111 - Project structure, namespaces, and application layers](../lessons/111-project-structure-namespaces-and-application-layers.md) | [Q&A](../qnas/112-routing-controllers-and-middleware.md) | [Lesson](../lessons/112-routing-controllers-and-middleware.md) | [113 - Dependency injection and service design](../lessons/113-dependency-injection-and-service-design.md) |
