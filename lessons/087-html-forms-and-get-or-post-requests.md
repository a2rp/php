# 087 - HTML forms and GET or POST requests

## Learning goals

By the end of this lesson, you will be able to:

- Build a semantic HTML form that sends named controls to a PHP endpoint.
- Choose GET for retrieval and POST for submitting data that may change server state.
- Read URL parameters from `$_GET` and supported form bodies from `$_POST`.
- Check the request method and content type expected by a handler.
- Explain why browser-side constraints and POST do not replace server-side validation or security controls.
- Use Post/Redirect/Get after a successful form submission.

## 1. How does an HTML form send data?

A form groups controls and describes where and how the browser should submit their values:

```html
<form action="/search.php" method="get">
    <label for="q">Search</label>
    <input id="q" name="q" type="search">
    <button type="submit">Search</button>
</form>
```

When the visitor submits the form, the browser sends successful controls: generally controls that have a `name`, are enabled, and are included in the submission. The `id` connects the label to the input; the `name` becomes the parameter name PHP receives. An input without a `name` is not submitted as a form parameter.

For a GET form, entering `PHP notes` creates a request similar to:

```text
GET /search.php?q=PHP+notes HTTP/1.1
```

PHP makes the query parameter available as `$_GET['q']`. The browser URL can be bookmarked or shared.

## 2. Choose GET for retrieval

Use GET when the request asks for a representation or result without changing application state. Search, filtering, sorting, and pagination are common examples.

```html
<form action="/search.php" method="get">
    <label for="search-term">Search tutorials</label>
    <input id="search-term" name="q" type="search">
    <label for="page-size">Results per page</label>
    <select id="page-size" name="limit">
        <option value="10">10</option>
        <option value="25">25</option>
    </select>
    <button type="submit">Search</button>
</form>
```

A submission with `q=PHP` and `limit=25` puts those values in the URL query string. The receiving PHP script reads them through `$_GET` and still validates them before use.

Do not use GET for an action that changes state, such as creating an order, changing a password, or deleting a record. Links can be opened, shared, prefetched, or revisited. A GET request should not cause a user-visible state change.

## 3. Choose POST for submissions

Use POST when the form submits data to be processed, especially when the operation can change state:

```html
<form action="/contact.php" method="post">
    <label for="name">Name</label>
    <input id="name" name="name" autocomplete="name" required>

    <label for="message">Message</label>
    <textarea id="message" name="message" rows="5" required></textarea>

    <button type="submit">Send message</button>
</form>
```

With a normal URL-encoded form, the browser sends the fields in the request body, and PHP exposes them in `$_POST`:

```php
$name = $_POST['name'] ?? null;
$message = $_POST['message'] ?? null;
```

POST keeps the form values out of the URL, but it does not encrypt them or make them secret. Use HTTPS to protect data in transit. Request bodies may still be recorded by application infrastructure, and all values need server-side validation.

POST itself does not prevent cross-site request forgery (CSRF). State-changing endpoints need the application's appropriate authorization and CSRF protections as well as input validation.

## 4. GET and POST compared

| Behavior | GET | POST |
| --- | --- | --- |
| Where ordinary form values go | URL query string | Request body |
| Common use | Retrieve or filter a representation | Submit data for processing or a state change |
| Bookmark/share a particular search | Usually useful | Usually not represented by the submitted body |
| Values exposed in the address bar | Yes | No, for the request body |
| Protects confidentiality by itself | No | No |
| Safe for a state-changing action | No | Often chosen, but still needs authorization and CSRF protection |

The method is a statement about the intended operation; it is not an access-control check. A caller can construct a request without using your form, so the server must enforce the same rules.

## 5. Request method and PHP input source

A PHP handler can check the method explicitly:

```php
$method = $_SERVER['REQUEST_METHOD'] ?? '';

if ($method === 'GET') {
    // Show the form or retrieve search results.
} elseif ($method === 'POST') {
    // Validate and process submitted form data.
} else {
    http_response_code(405);
    header('Allow: GET, POST');
    exit('Method not allowed.');
}
```

Use the source that matches how the client sent the value:

- `$_GET` contains query-string parameters, even if the request method is not GET.
- `$_POST` contains form data for `application/x-www-form-urlencoded` and `multipart/form-data` POST requests.
- JSON and other raw request bodies are not automatically parsed into `$_POST`; read raw data from `php://input` and decode it deliberately.

Avoid `$_REQUEST` when it matters whether a value came from the URL, a form body, or a cookie. Reading `$_GET` or `$_POST` directly makes the input source clear.

## 6. Names, repeated controls, and omitted values

The `name` attribute defines the submitted key. PHP understands bracket notation for grouped values:

```html
<label><input type="checkbox" name="topics[]" value="forms"> Forms</label>
<label><input type="checkbox" name="topics[]" value="security"> Security</label>
```

If both boxes are selected, PHP may receive an array in `$_POST['topics']`. Validate that it is an array and validate every member before using it.

Some controls may be absent from a submission:

- An unchecked checkbox is not submitted.
- A disabled control is not submitted.
- A control with no `name` is not submitted.
- A radio group normally submits only the selected option.
- Only the submit button used to submit the form contributes its name and value.

Do not assume an expected key exists. Use `isset()` or `array_key_exists()` as appropriate, then validate the actual shape and value.

## 7. Browser validation helps users; server validation protects the application

Attributes such as `required`, `type="email"`, `min`, `max`, and `maxlength` can prevent common mistakes in the browser and make the form easier to use:

```html
<input name="email" type="email" required maxlength="254">
<input name="quantity" type="number" min="1" max="20" required>
```

A client can bypass or modify browser checks and send an arbitrary HTTP request. Repeat all important validation on the server. Check types, allowed values, ranges, lengths, authorization, and any business rules before changing data.

Validation and output escaping solve different problems. After accepting a string, still escape it for the output context when rendering HTML. See [Generating HTML safely with templates](085-generating-html-safely-with-templates.md).

## 8. Process POST data and redirect after success

A common browser workflow is Post/Redirect/Get (PRG): process a valid POST, then send a redirect so the browser loads a GET page. This helps prevent a page refresh from resubmitting the same form.

```php
<?php

declare(strict_types=1);

if (($_SERVER['REQUEST_METHOD'] ?? '') !== 'POST') {
    http_response_code(405);
    header('Allow: POST');
    exit('Method not allowed.');
}

$name = $_POST['name'] ?? null;

if (!is_string($name) || trim($name) === '') {
    http_response_code(400);
    exit('A name is required.');
}

// Validate, authorize, and perform the intended application operation here.

header('Location: /contact/thank-you.php', true, 303);
exit;
```

Send the redirect header before any body output, then call `exit` so the old handler does not continue. A `303 See Other` directs the browser to retrieve the target using GET. The redirect does not replace validation or transaction handling.

## 9. Form encoding and file uploads

A regular form using GET or POST submits values as a standard form encoding. PHP parses URL-encoded POST forms and multipart form submissions into the relevant superglobals.

A form that uploads files needs:

```html
<form action="/upload.php" method="post" enctype="multipart/form-data">
    <label for="document">Choose a document</label>
    <input id="document" name="document" type="file" required>
    <button type="submit">Upload</button>
</form>
```

Uploaded file metadata is exposed through `$_FILES`, not `$_POST`. File handling needs additional size, content, storage, and access checks; it is covered separately in the file-upload lesson.

If a client submits JSON instead of an HTML form, `$_POST` will not automatically contain decoded JSON fields. Read `php://input`, decode the JSON, and handle decoding errors as part of that request format.

## Common mistakes

- **Using GET for state changes:** a URL should not delete records or submit purchases.
- **Treating POST as private:** POST changes where data is carried, not whether it is encrypted.
- **Trusting `required` or input types alone:** validate again on the server.
- **Reading form fields from the wrong place:** query strings belong in `$_GET`; standard POST form fields belong in `$_POST`.
- **Assuming every field is present:** unchecked checkboxes and disabled controls are omitted.
- **Expecting JSON in `$_POST`:** raw JSON needs `php://input` and decoding.
- **Redirecting after output:** send headers before the response body and stop execution after redirecting.
- **Assuming POST prevents CSRF:** use a CSRF defense for state-changing requests.

## Key takeaways

- Use GET for retrieval and shareable search/filter state; use POST to submit data for processing.
- Form `name` attributes become submitted parameter names; labels should be associated with controls.
- Read data from `$_GET`, `$_POST`, `$_FILES`, or `php://input` according to its source and encoding.
- Treat all submitted values as client-controlled; validate on the server.
- POST is not encryption, authentication, authorization, or CSRF protection.
- After successful POST processing, a 303 redirect can implement Post/Redirect/Get.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [086 - Reading query strings and request data](086-reading-query-strings-and-request-data.md) | [Q&A](../qnas/087-html-forms-and-get-or-post-requests.md) | [Example](../examples/087-html-forms-and-get-or-post-requests.md) | [088 - Input validation and normalization](088-input-validation-and-normalization.md) |