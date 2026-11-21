# 085 - Generating HTML safely with templates: Example

This example renders a profile page from a fixed PHP template. It keeps the HTML structure in one file, passes only the view data the template needs, escapes dynamic HTML values, and validates a URL scheme before placing the URL in an attribute.

## File structure

```text
project/
├── public/
│   └── profile.php
└── templates/
    └── profile.php
```

## 1. Page handler: `public/profile.php`

```php
<?php

declare(strict_types=1);

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

function validatedWebUrl(string $url): string
{
    $parts = parse_url($url);

    if ($parts === false) {
        throw new InvalidArgumentException('The profile URL is invalid.');
    }

    $scheme = strtolower($parts['scheme'] ?? '');
    $host = $parts['host'] ?? '';

    if ($host === '' || !in_array($scheme, ['https', 'http'], true)) {
        throw new InvalidArgumentException('The profile URL must use HTTP or HTTPS.');
    }

    return $url;
}

function renderProfile(array $viewData): void
{
    require __DIR__ . '/../templates/profile.php';
}

$profile = [
    'name' => '<img src=x onerror=alert(1)>',
    'bio' => 'PHP notes & examples for beginners.',
    'website' => 'https://example.test/profiles/amina',
];

$viewData = [
    'name' => $profile['name'],
    'bio' => $profile['bio'],
    'website' => validatedWebUrl($profile['website']),
];

renderProfile($viewData);
```

### What the page handler does

- `escapeHtml()` converts HTML-significant characters before a value is rendered.
- `validatedWebUrl()` requires a parseable URL with an `http` or `https` scheme and a host. If the application permits links only to particular sites, add a host allowlist too.
- `renderProfile()` loads one application-controlled template path. No request value can choose a file to include.
- `$viewData` contains only the values needed by the template.
- The sample name intentionally contains an image tag with an event handler to demonstrate that it will be displayed as text rather than parsed as markup.

## 2. Template: `templates/profile.php`

```php
<?php

declare(strict_types=1);

/** @var array{name: string, bio: string, website: string} $viewData */
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title><?= escapeHtml($viewData['name']) ?> - Profile</title>
</head>
<body>
    <main>
        <article class="profile">
            <h1><?= escapeHtml($viewData['name']) ?></h1>
            <p><?= escapeHtml($viewData['bio']) ?></p>
            <a href="<?= escapeHtml($viewData['website']) ?>">Visit website</a>
        </article>
    </main>
</body>
</html>
```

The include runs inside `renderProfile()`, so the template can read that function's `$viewData` variable. The `escapeHtml()` function is available because it was declared in the page handler. Each dynamic value is escaped at its HTML output location, including the document title, text nodes, and quoted `href` attribute.

## Expected HTML output

The relevant response source is:

```html
<title>&lt;img src=x onerror=alert(1)&gt; - Profile</title>
<h1>&lt;img src=x onerror=alert(1)&gt;</h1>
<p>PHP notes &amp; examples for beginners.</p>
<a href="https://example.test/profiles/amina">Visit website</a>
```

The browser displays the name as the literal text `<img src=x onerror=alert(1)>`. It does not create an image or run its event handler. The `&` in the biography is represented as `&amp;` in the HTML source and displayed as `&` in the page.

## 3. Check an unsafe URL

The URL validator rejects a URL using a disallowed scheme:

```php
$unsafeUrl = 'javascript:alert(1)';
$approvedUrl = validatedWebUrl($unsafeUrl);
```

The call throws `InvalidArgumentException` before rendering because the scheme is not `http` or `https`. This validation is separate from `escapeHtml()`: HTML escaping protects the attribute syntax, while URL validation applies the application's destination rules.

## Common mistakes to avoid

```php
<!-- Do not insert untrusted values directly into markup. -->
<h1><?= $viewData['name'] ?></h1>
```

This may let the browser parse a submitted value as markup. Use `escapeHtml()` for HTML text and quoted ordinary attributes.

```php
<!-- Do not let request data select an include path. -->
<?php require __DIR__ . '/../templates/' . $_GET['page'] . '.php'; ?>
```

Use a fixed template path or an application-controlled allowlist instead.

Also avoid inserting untrusted strings into JavaScript, CSS, inline event handlers, or other contexts that have different parsing rules. `escapeHtml()` is not a universal encoder.

## Summary

This example separates page structure from page data, uses a fixed template path, escapes each dynamic HTML value, and validates the URL before rendering it as a link. In a real application, validate all input according to its requirements and add destination allowlists where needed.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [084 - HTTP requests, responses, and the PHP request lifecycle](../lessons/084-http-requests-responses-and-the-php-request-lifecycle.md) | [Q&A](../qnas/085-generating-html-safely-with-templates.md) | Example | [086 - Reading query strings and request data](../lessons/086-reading-query-strings-and-request-data.md) |