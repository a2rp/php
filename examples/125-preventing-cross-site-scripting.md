# 125 - Preventing cross-site scripting: Example

## Question

Build a PHP profile page that displays a name and comment supplied by a request. The page must show markup-looking input as plain text, keep a search link safe, and preview the name in the browser without parsing it as HTML.

## Complete example

Save this as a PHP file and serve it through a PHP-enabled web server. The demo uses harmless XSS-shaped text as its default input so the result can be inspected without entering a value.

~~~php
<?php

declare(strict_types=1);

function queryString(string $key, string $default): string
{
    $value = $_GET[$key] ?? $default;

    return is_string($value) ? $value : $default;
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

$displayName = queryString(
    'name',
    '<img src=x onerror=alert("XSS")>'
);

$comment = queryString(
    'comment',
    '<script>alert("XSS")</script> This is a sample comment.'
);

$searchTerm = queryString('q', $displayName);

$searchUrl = '/search?' . http_build_query(
    ['q' => $searchTerm],
    '',
    '&',
    PHP_QUERY_RFC3986
);
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Safe profile output</title>
</head>
<body>
    <main>
        <h1>Safe profile output</h1>

        <section aria-labelledby="profile-heading">
            <h2 id="profile-heading">Profile</h2>

            <p>
                Display name:
                <span><?= escapeHtml($displayName) ?></span>
            </p>

            <label for="display-name">Edit display name</label>
            <input
                id="display-name"
                name="displayName"
                type="text"
                value="<?= escapeHtml($displayName) ?>"
            >

            <p>
                Live preview:
                <span id="preview"></span>
            </p>
        </section>

        <section aria-labelledby="comment-heading">
            <h2 id="comment-heading">Comment</h2>
            <p><?= escapeHtml($comment) ?></p>
        </section>

        <p>
            <a href="<?= escapeHtml($searchUrl) ?>">Search this value</a>
        </p>
    </main>

    <script>
        const nameInput = document.querySelector('#display-name');
        const preview = document.querySelector('#preview');

        function updatePreview() {
            preview.textContent = nameInput.value;
        }

        nameInput.addEventListener('input', updatePreview);
        updatePreview();
    </script>
</body>
</html>
~~~

## Expected output

The page displays the following text. The angle brackets and tags are visible characters; the browser does not create an image or run a script.

~~~text
Safe profile output

Profile
Display name: <img src=x onerror=alert("XSS")>
Edit display name
Live preview: <img src=x onerror=alert("XSS")>

Comment
<script>alert("XSS")</script> This is a sample comment.

Search this value
~~~

No alert appears. The input field contains the name as text, the comment is displayed as text, and the live preview remains text while the name is edited.

## Explanation

### PHP encodes the value where HTML is rendered

The escapeHtml() helper encodes HTML-significant characters, both quote types, and invalid UTF-8 sequences using UTF-8 settings. It is applied separately to the name, comment, and href value at the point each is written into the HTML.

The helper is appropriate here because these destinations are ordinary HTML text or a quoted, ordinary attribute. The element and attribute names are fixed. This helper is not a universal encoder for JavaScript, CSS, or arbitrary HTML.

### The search link uses two protections

The application creates a local search path, then http_build_query() encodes the query parameter value. The completed URL is still passed through escapeHtml() when inserted into the quoted href attribute. URL component encoding and HTML attribute encoding protect different parsing stages.

If a feature accepts a complete URL from a user, it must also validate the scheme and permitted destination. Encoding alone does not make a javascript: URL safe.

### The browser preview uses a text sink

The script reads the value from the input and assigns it to textContent. It never concatenates the value into JavaScript source or parses it through innerHTML. Markup-looking characters therefore remain text in the preview.

The script itself is fixed application code. The request value is supplied to it through the input element, whose value was encoded for its HTML attribute.

### Request shape and validation

queryString() ensures the request value is a string before it reaches the page. A production feature should also enforce its own length and content rules. That validation complements output encoding; it does not replace it.

If users must be allowed to write formatted HTML, do not remove tags with ad hoc string replacements. Use a maintained HTML sanitizer configured with an explicit allowlist, and keep it updated.

## What this example demonstrates

- Encode untrusted strings when writing them into ordinary HTML text or quoted attributes.
- Keep element names, attribute names, and executable code fixed.
- URL-encode query parameter values and then encode the completed URL for its HTML attribute.
- Use textContent when displaying plain text in the DOM.
- Test stored and reflected values at every output location that uses them.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [124 - Preventing SQL injection](../lessons/124-preventing-sql-injection.md) | [Q&A](../qnas/125-preventing-cross-site-scripting.md) | [Lesson](../lessons/125-preventing-cross-site-scripting.md) | [126 - Preventing cross-site request forgery](../lessons/126-preventing-cross-site-request-forgery.md) |