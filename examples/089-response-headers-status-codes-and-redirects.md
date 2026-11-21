# 089 - Response headers, status codes, and redirects: Example

## Goal

Build one small PHP endpoint that returns different HTTP responses. You will see how the status code, response headers, and body work together for HTML, JSON, errors, empty responses, and redirects.

## Question

How can one endpoint return the right response for each case, reject unsupported methods, and redirect safely without trusting a URL supplied by the visitor?

## Complete example: `public/response-demo.php`

```php
<?php

declare(strict_types=1);

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($method !== 'GET') {
    http_response_code(405);
    header('Allow: GET');
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Method not allowed. Use GET.\n");
}

$case = $_GET['case'] ?? 'html';

if (!is_string($case)) {
    http_response_code(400);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("The case parameter must be one value.\n");
}

switch ($case) {
    case 'html':
        http_response_code(200);
        header('Content-Type: text/html; charset=UTF-8');
        echo '<!doctype html>';
        echo '<html lang="en"><head><meta charset="utf-8"><title>HTML response</title></head>';
        echo '<body><h1>HTML response</h1><p>Status: 200 OK</p></body></html>';
        break;

    case 'json':
        http_response_code(200);
        header('Content-Type: application/json; charset=UTF-8');
        echo json_encode(
            [
                'status' => 'ok',
                'lesson' => 89,
                'topic' => 'response headers and status codes',
            ],
            JSON_THROW_ON_ERROR
        );
        break;

    case 'not-found':
        http_response_code(404);
        header('Content-Type: text/plain; charset=UTF-8');
        exit("The requested demo resource was not found.\n");

    case 'no-content':
        http_response_code(204);
        exit;

    case 'redirect':
        // This destination is fixed by the application, not supplied by the request.
        header('Location: /response-demo.php?case=html', true, 303);
        exit;

    default:
        http_response_code(404);
        header('Content-Type: text/plain; charset=UTF-8');
        exit("Unknown demo case.\n");
}
```

## How it works

1. **Method check:** The endpoint accepts only `GET`. Other methods receive `405 Method Not Allowed` and `Allow: GET`.
2. **Input shape check:** `case[]` produces an array. The endpoint rejects it with `400 Bad Request` instead of treating an unexpected shape as a route name.
3. **HTML response:** The `html` case returns status `200`, a `text/html` content type, and an HTML body.
4. **JSON response:** The `json` case returns status `200`, an `application/json` content type, and JSON encoded from a PHP array.
5. **Not found:** The `not-found` and unknown cases return status `404` with a plain-text message.
6. **No content:** The `no-content` case returns status `204` and exits without sending a body.
7. **Redirect:** The `redirect` case sends a fixed local destination with status `303`. `exit` prevents later code from adding a body or doing more work.

The status and content type are selected before body output. Each response body matches the declared content type.

## Run it

From the project root, start PHP's built-in development server:

```bash
php -S localhost:8000 -t public
```

Open these URLs in a browser, or request them from a terminal:

```text
http://localhost:8000/response-demo.php
http://localhost:8000/response-demo.php?case=json
http://localhost:8000/response-demo.php?case=not-found
http://localhost:8000/response-demo.php?case=no-content
http://localhost:8000/response-demo.php?case=redirect
```

Use `curl.exe` to inspect status lines and headers. Run these in a second terminal while the PHP server is running:

```bash
curl.exe -i "http://localhost:8000/response-demo.php?case=json"
curl.exe -i "http://localhost:8000/response-demo.php?case=not-found"
curl.exe -i "http://localhost:8000/response-demo.php?case=no-content"
curl.exe -i "http://localhost:8000/response-demo.php?case=redirect"
curl.exe -i -X POST "http://localhost:8000/response-demo.php"
curl.exe -i "http://localhost:8000/response-demo.php?case%5B%5D=json"
```

The encoded `case%5B%5D` request is equivalent to `case[]=json` and demonstrates the invalid array input.

## Expected results

| Request | Status | Important response header | Body or browser behavior |
| --- | --- | --- | --- |
| No `case` parameter | `200 OK` | `Content-Type: text/html; charset=UTF-8` | A small HTML page titled “HTML response” |
| `?case=json` | `200 OK` | `Content-Type: application/json; charset=UTF-8` | `{"status":"ok","lesson":89,"topic":"response headers and status codes"}` |
| `?case=not-found` | `404 Not Found` | `Content-Type: text/plain; charset=UTF-8` | A short not-found message |
| `?case=no-content` | `204 No Content` | No body | An empty successful response |
| `?case=redirect` | `303 See Other` | `Location: /response-demo.php?case=html` | A browser follows the redirect and displays the HTML case |
| POST request | `405 Method Not Allowed` | `Allow: GET` | A plain-text method message |
| `case[]=json` | `400 Bad Request` | `Content-Type: text/plain; charset=UTF-8` | A message that the parameter must be one value |
| Unknown case | `404 Not Found` | `Content-Type: text/plain; charset=UTF-8` | An unknown-case message |

The redirect response itself is a `303`; a browser normally follows it automatically. Use `curl.exe -i` without `-L` to inspect the redirect before it is followed.

## Optional: identify a newly created resource

When a server successfully creates a resource, it can return `201 Created` and identify the resource with a `Location` header. The URI should come from the application's successful creation result, never from an untrusted request value.

```php
<?php

http_response_code(201);
header('Location: /items/42');
header('Content-Type: application/json; charset=UTF-8');
echo json_encode(
    ['id' => 42, 'name' => 'Notebook'],
    JSON_THROW_ON_ERROR
);
```

In a real application, use the actual identifier returned after persistence succeeds. This snippet demonstrates response construction; it does not create a database record.

## Key takeaways

- Send the status and headers before writing the body.
- Declare a content type that matches the body.
- Include `Allow` when returning `405`; do not add a body to `204`.
- Use an application-controlled redirect destination and stop execution after sending it.
- Inspect responses with `curl.exe -i` to see the status and headers alongside the body.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [088 - Input validation and normalization](../lessons/088-input-validation-and-normalization.md) | [Q&A](../qnas/089-response-headers-status-codes-and-redirects.md) | Example | [090 - Cookies and browser state](../lessons/090-cookies-and-browser-state.md) |