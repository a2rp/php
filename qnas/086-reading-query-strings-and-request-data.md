# 086 - Reading query strings and request data: Q&A

## Multiple-choice questions

### 1. Which part of this URL is the query string?

```text
/search.php?q=php&page=2
```


- **A.** `/search.php`
- **B.** `?q=php&page=2`
- **C.** `&page=2`
- **D.** `https://`

**Answer: B.** The query string begins at `?` and contains the parameters `q=php` and `page=2`. The ampersand separates the two parameter pairs.

### 2. Which PHP superglobal contains query-string parameters?


- **A.** `$_POST`
- **B.** `$_COOKIE`
- **C.** `$_GET`
- **D.** `$_FILES`

**Answer: C.** PHP exposes URL query parameters in `$_GET`. It represents the query string even when the HTTP method is not `GET`.

### 3. What does PHP usually do to encoded query parameter values before storing them in `$_GET`?


- **A.** It HTML-escapes them.
- **B.** It URL-decodes them.
- **C.** It validates them as integers.
- **D.** It removes all punctuation.

**Answer: B.** PHP decodes query-string values while populating `$_GET`. It does not validate or HTML-escape them. Do not URL-decode a value a second time without a specific reason.

### 4. What is the main reason to check a query value's type before treating it as a string?


- **A.** A query parameter can use bracket syntax and arrive as an array.
- **B.** PHP stores every query value as a boolean.
- **C.** Arrays can only come from `$_POST`.
- **D.** Type checks encode the value for HTML.

**Answer: A.** A request such as `?page[]=2` can produce an array at `$_GET['page']`. Check the shape before calling string functions or passing it to validation logic intended for a single string.

### 5. Which function is intended to encode parameter values when building a query string?


- **A.** `htmlspecialchars()`
- **B.** `http_build_query()`
- **C.** `filter_input()`
- **D.** `parse_url()`

**Answer: B.** `http_build_query()` constructs an encoded query string from an array of parameters. If the resulting URL is inserted into HTML, it still needs HTML-attribute escaping.

## Short-answer questions

### 6. What is the difference between a missing parameter and an empty parameter?

**Answer:** A missing parameter is absent from the query, such as `/search.php`. An empty parameter is present with an empty value, such as `/search.php?q=`. The application may handle these differently, for example using a default when omitted but reporting an error when explicitly empty.

**Explanation:** `$_GET['q'] ?? ''` turns a missing key into an empty string, so check `array_key_exists()` first if the distinction matters.

### 7. Does `$_GET` prove that a request used the GET method?

**Answer:** No. `$_GET` represents the URL query string, which can be present with any HTTP request method. Inspect `$_SERVER['REQUEST_METHOD']` separately when behavior depends on the method.

### 8. What is the difference between validation and output escaping?

**Answer:** Validation checks whether a value follows the application's expected rules, such as being an integer within a range. Escaping encodes the value for a specific output context, such as HTML text. A valid value may still need escaping before it is rendered.

### 9. What does `FILTER_DEFAULT` do in PHP's filter functions?

**Answer:** It is an alias for `FILTER_UNSAFE_RAW`, so by itself it performs no validation or sanitization. Choose a specific validation filter or apply explicit application rules.

### 10. When should an application avoid `$_REQUEST`?

**Answer:** Avoid it when the source of a value matters. Depending on PHP configuration, `$_REQUEST` can combine query parameters, form data, and cookies with configurable precedence. Reading the specific source such as `$_GET` or `$_POST` makes the origin clear.

### 11. Why should secrets not be placed in query strings?

**Answer:** URLs may be saved in browser history, copied, recorded in server logs, or included in referrer information. Use an appropriate protected mechanism for secrets instead of putting them in the URL.

## Code and output questions

### 12. For this request, what are the values of `$term` and `$page`?

```text
/search.php?q=PHP%20%26%20HTML&page=2
```

```php
$term = $_GET['q'] ?? '';
$page = $_GET['page'] ?? '1';
```

**Answer:** `$term` is the string `PHP & HTML`, and `$page` is the string `2`.

**Explanation:** PHP URL-decodes the query values when creating `$_GET`. It does not automatically convert numeric-looking values into integers, so validate and convert `page` deliberately before using it as a number.

### 13. What is the issue with this code?

```php
$page = (int) ($_GET['page'] ?? 1);
```

**Answer:** Casting is not validation. Unexpected or malformed input can be silently converted to a different integer, and array-shaped input is not handled as an expected single value.

**Safer approach:** Check that the value is a string, then validate it with `FILTER_VALIDATE_INT` and the required range. Handle `false` as invalid input.

### 14. What does this validation code return for a valid page number from 1 to 100?

```php
$page = filter_var(
    $rawPage,
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 100]]
);
```

**Answer:** It returns the integer value when validation succeeds. It returns `false` when the input does not meet the integer and range rules.

**Explanation:** Compare the result strictly with `false`; a loose comparison can confuse a valid numeric value with a failure value.

### 15. What query string does this code create?

```php
$query = http_build_query(
    ['q' => 'PHP & HTML', 'page' => 2],
    '',
    '&',
    PHP_QUERY_RFC3986
);
```

**Answer:**

```text
q=PHP%20%26%20HTML&page=2
```

**Explanation:** `http_build_query()` encodes the space and ampersand inside the value so they are not mistaken for query syntax. The ampersand between `q` and `page` remains the parameter separator.

### 16. What shape can this request produce in PHP?

```text
/search.php?filter[color]=blue&filter[size]=large
```

**Answer:** `$_GET['filter']` can be an associative array with `color` set to `blue` and `size` set to `large`.

**Explanation:** PHP supports bracket notation for external variables. Validate that the top-level value is an array and validate each expected key and value before relying on the structure.

## Scenario questions

### 17. A search page expects `q` to be one string, but receives `?q[]=php`. What should it do?

**Answer:** Reject the value or apply a documented default. Check `is_string($_GET['q'])` before using string operations; do not assume every query parameter is scalar.

**Explanation:** The client controls the URL and can choose bracket syntax. Type checking is part of validating the input contract.

### 18. A page needs to generate a link for the search `C++ & PHP`. Should it concatenate the raw value into a URL?

**Answer:** No. Pass the parameter array to `http_build_query()` so reserved characters are encoded. If the generated URL is then written into an HTML `href` attribute, HTML-escape the completed URL too.

**Explanation:** URL query encoding protects URL structure. HTML escaping protects the surrounding HTML attribute. They are separate output contexts.

### 19. A developer changes a value in `$_GET` and then calls `filter_input(INPUT_GET, 'page', ...)`. Will `filter_input()` read that changed array entry?

**Answer:** Not necessarily. `filter_input()` reads the original input value provided by the SAPI, before user code modifies the superglobal. To validate a value already transformed or modified by the application, use `filter_var()` on that variable.

### 20. Should a link with `?action=delete&id=42` delete a database record when it is opened?

**Answer:** No. A `GET` request should retrieve a representation and should not perform a state-changing action. Use an appropriate state-changing request and enforce authentication, authorization, and other protections on the server.

**Explanation:** Query parameters are user-controlled and can be copied, prefetched, or opened as links. A URL alone is not proof that a user intended or is allowed to change state.

## Key takeaways

- `$_GET` contains decoded query parameters, not validated values.
- Query parameters can be absent, empty, strings, or arrays; check the expected shape.
- Validate according to application rules and check filter failure values strictly.
- `http_build_query()` encodes values when constructing query strings.
- Query strings are public URL data; keep secrets out and do not use `GET` for state changes.
- Keep query strings, form bodies, raw request bodies, and cookies distinct.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [085 - Generating HTML safely with templates](../lessons/085-generating-html-safely-with-templates.md) | [Lesson](../lessons/086-reading-query-strings-and-request-data.md) | [Example](../examples/086-reading-query-strings-and-request-data.md) | [087 - HTML forms and GET or POST requests](../lessons/087-html-forms-and-get-or-post-requests.md) |