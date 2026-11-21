# 097 - Building JSON and REST APIs: Example

## Goal

Build and call a small local JSON API. The endpoint will list articles, retrieve one article, create one, partially update it, and delete it. It will validate JSON input, return intentional HTTP statuses, and keep its demo data outside the public web directory.

## Question

How can a PHP endpoint expose resource-oriented routes, validate a bounded JSON request body, return a predictable JSON response, and safely handle basic article operations?

## Complete example

Create this project structure:

~~~text
project/
├── public/
│   └── index.php
└── storage/
    └── articles.json
~~~

The built-in PHP server will use the public directory as its document root. The storage file stays outside that directory.

### 1. Seed data: storage/articles.json

~~~json
[
  {
    "id": 1,
    "title": "Welcome to the API",
    "body": "This is the first sample article."
  },
  {
    "id": 2,
    "title": "Working with JSON",
    "body": "JSON gives clients a structured response."
  }
]
~~~

### 2. API endpoint: public/index.php

~~~php
<?php

declare(strict_types=1);

const STORE_PATH = __DIR__ . '/../storage/articles.json';
const MAX_REQUEST_BYTES = 16_384;
const MAX_TITLE_BYTES = 160;
const MAX_BODY_BYTES = 20_000;

function sendJson(int $status, array $payload): void
{
    http_response_code($status);
    header('Content-Type: application/json; charset=utf-8');
    header('Cache-Control: no-store');
    header('X-Content-Type-Options: nosniff');

    if ($status !== 204) {
        echo json_encode(
            $payload,
            JSON_THROW_ON_ERROR | JSON_UNESCAPED_SLASHES
        );
    }

    exit;
}

function sendError(int $status, string $code, string $message): void
{
    sendJson($status, [
        'error' => [
            'code' => $code,
            'message' => $message,
        ],
    ]);
}

function readJsonObject(): stdClass
{
    $contentType = strtolower(trim(
        explode(';', $_SERVER['CONTENT_TYPE'] ?? '', 2)[0]
    ));

    if ($contentType !== 'application/json') {
        sendError(
            415,
            'unsupported_media_type',
            'Send a JSON request body.'
        );
    }

    $body = file_get_contents(
        'php://input',
        false,
        null,
        0,
        MAX_REQUEST_BYTES + 1
    );

    if ($body === false) {
        sendError(400, 'unreadable_body', 'The request body could not be read.');
    }

    if (strlen($body) > MAX_REQUEST_BYTES) {
        sendError(413, 'request_too_large', 'The request body is too large.');
    }

    try {
        $data = json_decode($body, false, 32, JSON_THROW_ON_ERROR);
    } catch (JsonException $exception) {
        sendError(400, 'invalid_json', 'The request body must contain valid JSON.');
    }

    if (!$data instanceof stdClass) {
        sendError(400, 'json_object_required', 'The JSON body must be an object.');
    }

    return $data;
}

/**
 * Run one operation while holding an exclusive lock on the JSON store.
 *
 * The callback may change the article array and returns:
 * status, data, save, and optionally location.
 */
function withArticles(callable $operation): array
{
    $handle = fopen(STORE_PATH, 'c+');

    if ($handle === false) {
        throw new RuntimeException('Could not open the article store.');
    }

    if (!flock($handle, LOCK_EX)) {
        fclose($handle);
        throw new RuntimeException('Could not lock the article store.');
    }

    try {
        rewind($handle);
        $json = stream_get_contents($handle);

        if ($json === false) {
            throw new RuntimeException('Could not read the article store.');
        }

        $articles = json_decode($json, true, 32, JSON_THROW_ON_ERROR);

        if (!is_array($articles)) {
            throw new RuntimeException('The article store has an invalid shape.');
        }

        $result = $operation($articles);

        if (($result['save'] ?? false) === true) {
            $updatedJson = json_encode(
                array_values($articles),
                JSON_THROW_ON_ERROR | JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES
            );

            rewind($handle);

            if (!ftruncate($handle, 0)) {
                throw new RuntimeException('Could not truncate the article store.');
            }

            $written = fwrite($handle, $updatedJson);

            if ($written !== strlen($updatedJson) || !fflush($handle)) {
                throw new RuntimeException('Could not save the article store.');
            }
        }

        return $result;
    } finally {
        flock($handle, LOCK_UN);
        fclose($handle);
    }
}

function findArticleIndex(array $articles, int $id): ?int
{
    foreach ($articles as $index => $article) {
        if (($article['id'] ?? null) === $id) {
            return $index;
        }
    }

    return null;
}

function isValidText(mixed $value, int $maxBytes): bool
{
    return is_string($value)
        && trim($value) !== ''
        && strlen($value) <= $maxBytes;
}

$path = parse_url($_SERVER['REQUEST_URI'] ?? '/', PHP_URL_PATH);
$collectionPath = '/api/v1/articles';

if (!is_string($path)) {
    sendError(400, 'invalid_path', 'The request path is invalid.');
}

if ($path === $collectionPath) {
    $id = null;
} elseif (preg_match('#^/api/v1/articles/([1-9][0-9]*)$#D', $path, $matches) === 1) {
    $id = filter_var(
        $matches[1],
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1]]
    );

    if ($id === false) {
        sendError(404, 'article_not_found', 'The requested article was not found.');
    }
} else {
    sendError(404, 'route_not_found', 'The requested API route was not found.');
}

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';

if ($id === null && !in_array($method, ['GET', 'POST'], true)) {
    header('Allow: GET, POST');
    sendError(405, 'method_not_allowed', 'This route does not support that method.');
}

if ($id !== null && !in_array($method, ['GET', 'PATCH', 'DELETE'], true)) {
    header('Allow: GET, PATCH, DELETE');
    sendError(405, 'method_not_allowed', 'This route does not support that method.');
}

$payload = null;

if (
    ($id === null && $method === 'POST')
    || ($id !== null && $method === 'PATCH')
) {
    $payload = readJsonObject();
}

if ($method === 'POST') {
    $title = $payload->title ?? null;
    $body = $payload->body ?? null;

    if (!isValidText($title, MAX_TITLE_BYTES)) {
        sendError(
            422,
            'invalid_title',
            'Title must be a non-empty string of at most 160 bytes.'
        );
    }

    if (!isValidText($body, MAX_BODY_BYTES)) {
        sendError(
            422,
            'invalid_body',
            'Body must be a non-empty string within the size limit.'
        );
    }
}

if ($method === 'PATCH') {
    $changes = get_object_vars($payload);
    $allowedFields = ['title', 'body'];

    if ($changes === [] || array_diff(array_keys($changes), $allowedFields) !== []) {
        sendError(
            422,
            'invalid_fields',
            'Provide title, body, or both, and do not include other fields.'
        );
    }

    if (
        array_key_exists('title', $changes)
        && !isValidText($changes['title'], MAX_TITLE_BYTES)
    ) {
        sendError(
            422,
            'invalid_title',
            'Title must be a non-empty string of at most 160 bytes.'
        );
    }

    if (
        array_key_exists('body', $changes)
        && !isValidText($changes['body'], MAX_BODY_BYTES)
    ) {
        sendError(
            422,
            'invalid_body',
            'Body must be a non-empty string within the size limit.'
        );
    }
}

try {
    if ($id === null && $method === 'GET') {
        $result = withArticles(
            static function (array &$articles): array {
                return [
                    'status' => 200,
                    'data' => [
                        'data' => array_values($articles),
                        'meta' => ['count' => count($articles)],
                    ],
                    'save' => false,
                ];
            }
        );
    } elseif ($id !== null && $method === 'GET') {
        $result = withArticles(
            static function (array &$articles) use ($id): array {
                $index = findArticleIndex($articles, $id);

                if ($index === null) {
                    return [
                        'status' => 404,
                        'data' => [
                            'error' => [
                                'code' => 'article_not_found',
                                'message' => 'The requested article was not found.',
                            ],
                        ],
                        'save' => false,
                    ];
                }

                return [
                    'status' => 200,
                    'data' => ['data' => $articles[$index]],
                    'save' => false,
                ];
            }
        );
    } elseif ($id === null && $method === 'POST') {
        $result = withArticles(
            static function (array &$articles) use ($payload): array {
                $ids = array_column($articles, 'id');
                $nextId = $ids === [] ? 1 : max($ids) + 1;

                $article = [
                    'id' => $nextId,
                    'title' => trim($payload->title),
                    'body' => trim($payload->body),
                ];

                $articles[] = $article;

                return [
                    'status' => 201,
                    'data' => ['data' => $article],
                    'location' => '/api/v1/articles/' . $nextId,
                    'save' => true,
                ];
            }
        );
    } elseif ($id !== null && $method === 'PATCH') {
        $result = withArticles(
            static function (array &$articles) use ($id, $changes): array {
                $index = findArticleIndex($articles, $id);

                if ($index === null) {
                    return [
                        'status' => 404,
                        'data' => [
                            'error' => [
                                'code' => 'article_not_found',
                                'message' => 'The requested article was not found.',
                            ],
                        ],
                        'save' => false,
                    ];
                }

                foreach ($changes as $field => $value) {
                    $articles[$index][$field] = trim($value);
                }

                return [
                    'status' => 200,
                    'data' => ['data' => $articles[$index]],
                    'save' => true,
                ];
            }
        );
    } else {
        $result = withArticles(
            static function (array &$articles) use ($id): array {
                $index = findArticleIndex($articles, $id);

                if ($index === null) {
                    return [
                        'status' => 404,
                        'data' => [
                            'error' => [
                                'code' => 'article_not_found',
                                'message' => 'The requested article was not found.',
                            ],
                        ],
                        'save' => false,
                    ];
                }

                array_splice($articles, $index, 1);

                return [
                    'status' => 204,
                    'data' => [],
                    'save' => true,
                ];
            }
        );
    }
} catch (Throwable $exception) {
    error_log('Article API failure: ' . $exception->getMessage());

    sendError(
        500,
        'internal_error',
        'The server could not complete the request.'
    );
}

if (isset($result['location'])) {
    header('Location: ' . $result['location']);
}

sendJson($result['status'], $result['data']);
~~~

### 3. Start the local server

From the project directory, start PHP's built-in server with the public directory as its document root and the endpoint as its router:

~~~bash
php -S 127.0.0.1:8000 -t public public/index.php
~~~

Keep this terminal running. Use a second PowerShell terminal for the requests below. The commands use curl.exe explicitly so they work in Windows PowerShell, where curl may otherwise be an alias.

### 4. List all articles

~~~powershell
curl.exe -i http://127.0.0.1:8000/api/v1/articles
~~~

Expected response body:

~~~json
{
  "data": [
    {
      "id": 1,
      "title": "Welcome to the API",
      "body": "This is the first sample article."
    },
    {
      "id": 2,
      "title": "Working with JSON",
      "body": "JSON gives clients a structured response."
    }
  ],
  "meta": {
    "count": 2
  }
}
~~~

The status is 200 OK.

### 5. Read one article

~~~powershell
curl.exe -i http://127.0.0.1:8000/api/v1/articles/1
~~~

The status is 200 OK and the body contains the article with ID 1.

### 6. Create an article

~~~powershell
curl.exe -i -X POST http://127.0.0.1:8000/api/v1/articles -H "Content-Type: application/json" --data-raw '{"title":"A third article","body":"Created through the JSON API."}'
~~~

Expected response:

~~~http
HTTP/1.1 201 Created
Location: /api/v1/articles/3
Content-Type: application/json; charset=utf-8
~~~

~~~json
{
  "data": {
    "id": 3,
    "title": "A third article",
    "body": "Created through the JSON API."
  }
}
~~~

The article is saved in storage/articles.json, outside the public directory.

### 7. Partially update the article

~~~powershell
curl.exe -i -X PATCH http://127.0.0.1:8000/api/v1/articles/3 -H "Content-Type: application/json" --data-raw '{"title":"Updated third article"}'
~~~

The status is 200 OK. The returned article has the new title and its existing body.

### 8. Delete the article

~~~powershell
curl.exe -i -X DELETE http://127.0.0.1:8000/api/v1/articles/3
~~~

The status is 204 No Content and there is no response body. A later GET for article 3 returns 404.

### 9. See validation and method errors

Malformed JSON:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:8000/api/v1/articles -H "Content-Type: application/json" --data-raw '{'
~~~

Expected: 400 Bad Request with the invalid_json error code.

Valid JSON with a missing title:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:8000/api/v1/articles -H "Content-Type: application/json" --data-raw '{"body":"This request has no title."}'
~~~

Expected: 422 Unprocessable Content with the invalid_title error code.

Unsupported method on the collection route:

~~~powershell
curl.exe -i -X DELETE http://127.0.0.1:8000/api/v1/articles
~~~

Expected: 405 Method Not Allowed and an Allow: GET, POST header.

Unsupported request media type:

~~~powershell
curl.exe -i -X POST http://127.0.0.1:8000/api/v1/articles -H "Content-Type: text/plain" --data-raw "not JSON"
~~~

Expected: 415 Unsupported Media Type.

Unknown article:

~~~powershell
curl.exe -i http://127.0.0.1:8000/api/v1/articles/999
~~~

Expected: 404 Not Found with the article_not_found error code.

## Expected behavior

| Operation | Status | Result |
| --- | --- | --- |
| List articles | 200 | JSON collection and count |
| Read an existing article | 200 | JSON representation of that article |
| Create an article | 201 | New representation and Location header |
| Update allowed fields with PATCH | 200 | Updated representation |
| Delete an existing article | 204 | No response body |
| Request an unknown route or article | 404 | Consistent JSON error |
| Send malformed JSON | 400 | Consistent JSON error |
| Send invalid field values | 422 | Validation error |
| Send an unsupported body type | 415 | Media type error |
| Use a method the route does not support | 405 | Error plus Allow header |
| Exceed the request size limit | 413 | Request size error |

## How it works

1. **Resource routing:** The endpoint matches only the collection route and a numeric article ID route. Unknown paths are rejected.
2. **Method semantics:** GET reads, POST creates, PATCH updates selected fields, and DELETE removes a resource.
3. **Bounded JSON parsing:** The endpoint requires application/json, reads at most the configured limit plus one byte, decodes with JSON_THROW_ON_ERROR, and requires an object.
4. **Field validation:** Titles and bodies must be non-empty strings within byte limits. PATCH accepts only title and body.
5. **Consistent responses:** Every response uses JSON, a meaningful status, and a stable error envelope. A 204 response has no body.
6. **Persistent demo storage:** The endpoint locks the JSON data file while reading or changing it, then writes updates before releasing the lock.
7. **Public document root:** The server serves files from public/. The article data is in storage/, outside that root.
8. **Errors:** Invalid client input receives a 4xx response. Unexpected storage or encoding failures are logged server-side and return a generic 500 response.

## Important notes

- This is a local learning example, not a production-ready API.
- It has no authentication or authorization. Do not expose it publicly or use it for real user data.
- The JSON file is a small demonstration store. A database, migrations, framework router, and production-grade transaction handling are more suitable for a real application.
- The API does not implement pagination because the seed collection is deliberately tiny. A real collection endpoint should enforce page size and pagination.
- The example limits the bytes read by PHP. Also set compatible request-body limits in the PHP configuration and web server.
- The validation limits use bytes via strlen(). If a limit should count user-visible Unicode characters, define the rule and use an encoding-aware length check.
- For cookie-authenticated state-changing requests, add CSRF protection. CORS does not replace it.
- Do not expose raw exceptions, request bodies, tokens, or sensitive data in public error messages or logs.
- Reset storage/articles.json to the seed data above if you want to repeat the same create request and see ID 3 again.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [096 - Making HTTP requests with streams and cURL](../lessons/096-making-http-requests-with-streams-and-curl.md) | [Q&A](../qnas/097-building-json-and-rest-apis.md) | Example | [098 - Relational databases and SQL fundamentals](../lessons/098-relational-databases-and-sql-fundamentals.md) |
