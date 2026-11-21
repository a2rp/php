# 086 - Reading query strings and request data

## Learning goals

By the end of this lesson, you will be able to:

- Identify a query string in a URL and read its values from `$_GET`.
- Explain how PHP decodes query-string values and maps bracket notation into arrays.
- Handle missing, empty, malformed, and unexpectedly shaped values.
- Validate query data according to an application's rules before using it.
- Build a query string safely with `http_build_query()`.
- Distinguish query-string data from form bodies and raw request bodies.

## 1. What is a query string?

A query string is the part of a URL after `?`. It carries name-value pairs, commonly separated by `&`:

```text
/search.php?q=php&page=2
```

In this example, `q` has the value `php`, and `page` has the value `2`. A query string is part of the URL and is sent with the request. It is often used for search terms, filters, sorting choices, pagination, and other values that identify or customize a requested representation.

The question mark separates the path from the query string. An ampersand separates parameters. A value can be percent-encoded when it contains characters that have a special role in a URL.

## 2. Read query values with `$_GET`

PHP exposes query-string parameters in the `$_GET` superglobal, an associative array available throughout the script:

```php
// For /search.php?q=php&page=2
$searchTerm = $_GET['q'] ?? '';
$pageNumber = $_GET['page'] ?? '1';
```

PHP decodes query-string values when it populates `$_GET`. For example, a space encoded as `+` or `%20` is available as a space in the value. Do not call `urldecode()` on `$_GET` values again; double decoding can change the data unexpectedly.

`$_GET` describes the query string, not necessarily the HTTP method. A query string can be present on a request made with methods other than `GET`; use `$_SERVER['REQUEST_METHOD']` when the method itself matters.

## 3. Handle missing and empty parameters

A request may omit a parameter, or include it with an empty value:

```text
/search.php
/search.php?q=
/search.php?q=php
```

These requests have different intent. The null coalescing operator supplies a default when a key is absent or its value is `null`; it does not replace an empty string:

```php
$query = $_GET['q'] ?? '';

if ($query === '') {
    // The parameter is missing or explicitly empty after this default.
}
```

If the application needs to distinguish “not supplied” from “supplied but empty,” check whether the key exists first:

```php
if (!array_key_exists('q', $_GET)) {
    $query = null; // not supplied
} elseif ($_GET['q'] === '') {
    $query = '';   // supplied with an empty value
} else {
    $query = $_GET['q'];
}
```

In real request handling, also verify the value's type before applying string operations. A query may use PHP's bracket syntax to create an array instead of a string.

## 4. Validate the expected shape and value

All request values are controlled by the client. A value that normally looks like a string can be replaced with a structured value:

```text
/search.php?page[]=2
```

With bracket syntax, PHP can represent `page` as an array. Check the type before treating it as a string or number.

For a page number between 1 and 100:

```php
$rawPage = $_GET['page'] ?? '1';

if (!is_string($rawPage)) {
    http_response_code(400);
    exit('The page parameter must be a single value.');
}

$page = filter_var(
    $rawPage,
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 100]]
);

if ($page === false) {
    http_response_code(400);
    exit('The page parameter must be an integer from 1 to 100.');
}
```

After validation, `$page` is an integer. Do not replace validation with a cast like `(int) $rawPage`; casting can silently turn malformed input into a different value, such as `0`.

Validation rules should match the application's meaning. For a sort choice, an allowlist is usually clearer than accepting arbitrary strings:

```php
$allowedSorts = ['name', 'newest', 'price'];
$rawSort = $_GET['sort'] ?? 'newest';

if (!is_string($rawSort) || !in_array($rawSort, $allowedSorts, true)) {
    http_response_code(400);
    exit('Unsupported sort order.');
}

$sort = $rawSort;
```

Validation is different from output escaping. Validate whether a value is acceptable for the operation; escape it separately when rendering it in HTML. See [Generating HTML safely with templates](085-generating-html-safely-with-templates.md).

## 5. Query parameter arrays

PHP supports bracket notation for groups of related values:

```text
/search.php?filter[color]=blue&filter[size]=large
```

The resulting value is structured approximately like this:

```php
$_GET['filter'] = [
    'color' => 'blue',
    'size' => 'large',
];
```

A repeated parameter name without brackets is not a reliable way to collect a list: later occurrences may replace earlier values. When a list is intended, use a documented bracket format such as `tag[]=php&tag[]=web`, then verify that the result is an array and validate each element.

```php
$tags = $_GET['tag'] ?? [];

if (!is_array($tags)) {
    http_response_code(400);
    exit('Tags must be supplied as a list.');
}

foreach ($tags as $tag) {
    if (!is_string($tag) || strlen($tag) > 40) {
        http_response_code(400);
        exit('Each tag must be a short string.');
    }
}
```

Also set reasonable limits on the number of accepted values and validate each value against the application's rules. Never assume nested query data has the shape you expect.

PHP normalizes dots and spaces in incoming variable names to underscores in some external-variable handling. Prefer simple, predictable parameter names and check the keys PHP actually exposes rather than relying on unusual names.

## 6. Use `filter_input()` or `filter_var()` deliberately

`filter_input()` can retrieve and filter a value from a specific PHP input source:

```php
$page = filter_input(
    INPUT_GET,
    'page',
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 100]]
);

if ($page === null) {
    // The parameter was not present.
} elseif ($page === false) {
    // The parameter was present but failed validation.
}
```

This form is useful when reading the original value from PHP's input source. It reads the SAPI-provided input rather than a later, manually modified copy of `$_GET`. Use `filter_var()` when validating a value already stored in a variable or when application code has transformed the input.

Be explicit about the filter. `FILTER_DEFAULT` is an alias for `FILTER_UNSAFE_RAW`; it does not validate or sanitize a value. A filter result can also be `false` or `null`, so compare strictly and handle failure intentionally.

For many applications, directly reading `$_GET`, checking the type, and validating with a focused rule is clear and sufficient. A filter function is a tool, not a substitute for defining the input contract.

## 7. Generate query strings with `http_build_query()`

When constructing a URL from values, use `http_build_query()` instead of concatenating raw values:

```php
$params = [
    'q' => 'PHP & HTML',
    'page' => 2,
];

$queryString = http_build_query(
    $params,
    '',
    '&',
    PHP_QUERY_RFC3986
);

$url = '/search.php?' . $queryString;
```

The generated query string encodes parameter values so reserved characters do not accidentally become separators. With the example values, the result is:

```text
/search.php?q=PHP%20%26%20HTML&page=2
```

If the resulting URL is inserted into an HTML attribute, it still needs HTML attribute escaping. URL encoding and HTML escaping solve different problems.

## 8. Query strings are public request data

Do not put passwords, session secrets, private personal data, or other sensitive values in a query string. URLs can be saved in browser history, copied, recorded in server logs, and sent in referrer information to other sites.

A query string is suitable for data that can be part of a shareable or bookmarkable URL, such as a search term or selected page. It is not a security boundary: users can edit every parameter, so authorization and business rules must be checked on the server.

Use request methods intentionally. A `GET` request should retrieve a representation and should not perform an action that changes application state, such as deleting a record. Form submission details and the differences between GET and POST are covered in [HTML forms and GET or POST requests](087-html-forms-and-get-or-post-requests.md).

## 9. Keep request sources distinct

Use the source that matches where the value was sent:

| Source | Typical PHP access | What it represents |
| --- | --- | --- |
| URL query string | `$_GET` | Parameters after `?` in the URL, regardless of request method. |
| Form body encoded as URL-encoded or multipart data | `$_POST` | Common POST form fields. |
| Raw request body, such as JSON | `php://input` | Body content that PHP has not parsed into `$_POST`. |
| Cookies | `$_COOKIE` | Values sent in the `Cookie` request header. |

Avoid using `$_REQUEST` when the input source matters. Its contents and precedence can depend on PHP configuration, which can make it unclear whether a value came from the query string, form body, or cookies.

## Key takeaways

- `$_GET` contains decoded query-string parameters; it can be populated even when the request method is not `GET`.
- Distinguish absent parameters from empty values when the application needs to.
- Verify whether a value is a string or array before using it, then validate it for the intended operation.
- `FILTER_DEFAULT` does not validate data, and filters can return `false` or `null`.
- Use `http_build_query()` to encode values while constructing query strings.
- Keep secrets out of URLs, and do not confuse query data with request-body data.
- HTML output still needs context-appropriate escaping after input validation.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [085 - Generating HTML safely with templates](085-generating-html-safely-with-templates.md) | [Q&A](../qnas/086-reading-query-strings-and-request-data.md) | [Example](../examples/086-reading-query-strings-and-request-data.md) | [087 - HTML forms and GET or POST requests](087-html-forms-and-get-or-post-requests.md) |