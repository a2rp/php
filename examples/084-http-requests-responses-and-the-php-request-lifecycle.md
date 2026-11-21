# 084 - HTTP requests, responses, and the PHP request lifecycle: Example

## Question

How can one PHP endpoint read a name from a query string, a form submission, or a JSON request body, validate it, and return an appropriate HTTP status and response body?

## Complete PHP example: `request-demo.php`

```php
<?php

declare(strict_types=1);

function sendText(int $status, string $body): void
{
    http_response_code($status);
    header('Content-Type: text/plain; charset=utf-8');
    echo $body;
    exit;
}

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
$name = null;

if ($method === 'GET') {
    // Query-string values are available in $_GET.
    $name = $_GET['name'] ?? null;
} elseif ($method === 'POST') {
    $contentType = $_SERVER['CONTENT_TYPE'] ?? '';
    $mediaType = strtolower(trim(explode(';', $contentType, 2)[0]));

    if ($mediaType === 'application/json') {
        // JSON bodies are read from php://input, not from $_POST.
        $rawBody = file_get_contents('php://input');

        try {
            $data = json_decode($rawBody, true, 512, JSON_THROW_ON_ERROR);
        } catch (JsonException) {
            sendText(400, 'Invalid JSON body.');
        }

        if (!is_array($data)) {
            sendText(400, 'Expected a JSON object.');
        }

        $name = $data['name'] ?? null;
    } else {
        // PHP populates $_POST for supported form content types.
        $name = $_POST['name'] ?? null;
    }
} else {
    header('Allow: GET, POST');
    sendText(405, 'Method Not Allowed');
}

if (!is_string($name) || trim($name) === '') {
    sendText(400, 'A non-empty name is required.');
}

sendText(200, 'Hello, ' . trim($name) . '!');
```

Start PHP's built-in development server in the directory containing `request-demo.php`:

```sh
php -S 127.0.0.1:8000 request-demo.php
```

Send requests from another terminal.

### GET query-string request

```sh
curl -i 'http://127.0.0.1:8000/?name=Maya'
```

### Form POST request

```sh
curl -i -X POST --data-urlencode 'name=Arjun' 'http://127.0.0.1:8000/'
```

### JSON POST request

```sh
curl -i -X POST \
  -H 'Content-Type: application/json' \
  --data '{"name":"Priya"}' \
  'http://127.0.0.1:8000/'
```

### Unsupported method

```sh
curl -i -X PUT 'http://127.0.0.1:8000/'
```

## Expected responses

Generated headers such as `Date`, `Server`, and `Connection` are omitted because they depend on the local PHP server. The response status, content type, and body will be:

**GET with `name=Maya`:**

```text
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8

Hello, Maya!
```

**Form POST with `name=Arjun`:**

```text
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8

Hello, Arjun!
```

**JSON POST with `{"name":"Priya"}`:**

```text
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8

Hello, Priya!
```

**PUT request:**

```text
HTTP/1.1 405 Method Not Allowed
Allow: GET, POST
Content-Type: text/plain; charset=utf-8

Method Not Allowed
```

**Missing or blank name:**

```text
HTTP/1.1 400 Bad Request
Content-Type: text/plain; charset=utf-8

A non-empty name is required.
```

## Explanation

1. **Read the method:** The endpoint reads `REQUEST_METHOD` from `$_SERVER` and uses `GET` as a fallback for environments where the key is absent.
2. **Read query data:** For a GET request, it reads `name` from `$_GET`.
3. **Read form data:** For a POST request with a supported form content type, it reads `name` from `$_POST`.
4. **Read JSON data:** For a JSON POST, it reads the raw body from `php://input` and decodes it with `JSON_THROW_ON_ERROR`. Invalid JSON produces a `400` response.
5. **Validate the decoded value:** The decoded JSON must be an array before the code tries to read `name`. The name must then be a non-empty string.
6. **Reject other methods:** The endpoint lists its supported methods in `Allow`, returns `405`, and exits without continuing into the success path.
7. **Prepare the response:** `sendText()` sets the status and content type before printing the body. It exits after sending the response so no later output is appended.

## Try these changes

- Request `/?name=` and observe the `400 Bad Request` response.
- Send JSON with an empty body or malformed JSON and observe `Invalid JSON body.`
- Send valid JSON that is not an object, such as `[]`, and observe `Expected a JSON object.`
- Send a form field named `name[]` and observe that the validation rejects the array instead of treating it as a string.
- Add `DELETE` to the allowed methods and implement a deliberate branch for it.

## Key takeaways

- Read each value from the source that matches how the client sent it.
- `$_POST` does not parse JSON; read and decode `php://input` for JSON requests.
- Validate both the shape and content of request values.
- Send the correct status and headers before the body.
- Stop execution after returning a redirect or a handled error response.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [083 - Reflection API](../lessons/083-reflection-api.md) | [Q&A](../qnas/084-http-requests-responses-and-the-php-request-lifecycle.md) | Example | [085 - Generating HTML safely with templates](../lessons/085-generating-html-safely-with-templates.md) |
