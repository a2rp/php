# 085 - Generating HTML safely with templates

## Learning goals

By the end of this lesson, you will be able to:

- Explain what a server-side HTML template does.
- Render dynamic values without mixing them into large HTML strings.
- Escape untrusted text for HTML output with `htmlspecialchars()`.
- Safely place dynamic values in quoted HTML attributes.
- Explain why HTML escaping does not validate URLs or make JavaScript and CSS contexts safe.
- Include a template from a fixed application path and pass it only the data it needs.

## 1. What is a template?

A template is a file that describes the structure of a response and provides places for dynamic data. In a PHP application, a template commonly contains HTML with small PHP expressions where values should appear.

For example, a profile page may have one reusable layout and receive a different name and biography for each profile. The HTML structure stays in the template; PHP supplies the values.

```php
<h1><?= escapeHtml($profileName) ?></h1>
<p><?= escapeHtml($profileBio) ?></p>
```

The `<?= ... ?>` tag is shorthand for `<?php echo ...; ?>`. A template helps organize output, but it does not automatically make values safe. Every untrusted value still needs the right output handling.

## 2. Prefer templates to concatenating HTML strings

It is possible to assemble markup with concatenation:

```php
$html = '<h1>' . $profileName . '</h1>';
$html .= '<p>' . $profileBio . '</p>';
echo $html;
```

This gets difficult to read as a page grows. It also makes it easy to forget escaping one value or to place a value in the wrong output context.

Keep page structure in an HTML-oriented template instead:

```php
<h1><?= escapeHtml($profileName) ?></h1>
<p><?= escapeHtml($profileBio) ?></p>
```

This does not remove the need for escaping. It makes the boundary between markup and dynamic values easier to see and review.

## 3. Escape values for HTML output

Text from a user, a database, a request, or an external service should be treated as untrusted when it is written into a page. If a value contains markup characters such as `<` or `&`, the browser could interpret it as HTML instead of ordinary text.

Use `htmlspecialchars()` for dynamic text in HTML text nodes and quoted HTML attributes:

```php
function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}
```

- `ENT_QUOTES` converts both kinds of quote, which is useful when a value is placed in a quoted attribute.
- `ENT_SUBSTITUTE` replaces invalid code unit sequences instead of silently dropping the output.
- `ENT_HTML5` applies HTML5 entity rules.
- `'UTF-8'` makes the expected character encoding explicit.

The result represents special characters as entities. For example, `<` becomes `&lt;` and `&` becomes `&amp;`. The browser displays those characters as text instead of treating them as HTML syntax.

Call the escaping function at the point where the value enters the HTML response. Do not assume that a value is safe because it came from your own database: it may originally have come from a visitor or another system.

## 4. Escape at every HTML output location

Escape each dynamic value where it is rendered, including text nodes, attributes, and repeated list items.

```php
<h1><?= escapeHtml($user['name']) ?></h1>

<p title="<?= escapeHtml($user['bio']) ?>">
    <?= escapeHtml($user['bio']) ?>
</p>

<ul>
    <?php foreach ($tags as $tag): ?>
        <li><?= escapeHtml($tag) ?></li>
    <?php endforeach; ?>
</ul>
```

Keep attribute values inside quotes. Escaping and quoting work together: a quoted attribute has a clear boundary, and quote characters in the value are encoded.

Do not escape the whole HTML document as one string. That would encode the application-authored tags and show the markup as visible text. Escape the dynamic values, while keeping the trusted template markup as markup.

## 5. Keep templates focused on presentation

A controller or page handler can gather and validate data, then pass a small view-data structure to a template. Keep database access, request handling, and business rules out of the template when possible.

One simple directory layout is:

```text
project/
├── public/
│   └── profile.php
└── templates/
    └── profile.php
```

Use a fixed path based on `__DIR__`, rather than a path supplied by a request parameter:

```php
require __DIR__ . '/../templates/profile.php';
```

A required template is part of the application; if it is missing, the page cannot be rendered correctly. `require` therefore makes the failure explicit. A template included with `include` or `require` can access variables in the scope where it is included, so deliberately pass the data it needs and avoid relying on a large set of unrelated local variables.

Never build an include path directly from untrusted input, such as `require $_GET['page']`. If a page can choose among templates, map a small allowlist of page identifiers to fixed paths in application code.

## 6. HTML escaping is context-specific

`htmlspecialchars()` is for HTML output contexts. It is not a universal sanitizer or validator.

| Output location | Correct approach |
| --- | --- |
| Text between HTML tags | HTML-escape the value. |
| A quoted ordinary HTML attribute | Quote the attribute and HTML-escape its value. |
| `href` or `src` URL | Validate the URL and allowed scheme, then HTML-escape it for the attribute. |
| JavaScript code | Avoid inserting raw values; use a safe data-transfer approach designed for JavaScript or JSON context. |
| CSS code | Do not concatenate untrusted data into CSS. Validate narrowly if a value must affect styling. |
| HTML that users are allowed to author | Use a maintained allowlist-based HTML sanitizer; escaping would display the tags as text. |

For example, HTML-escaping a URL does not make a dangerous scheme safe. A string such as `javascript:...` contains no HTML markup that `htmlspecialchars()` needs to remove. Validate that a URL uses an allowed scheme (and any other application-specific rules), then escape it for the quoted `href` attribute.

The safest default is to keep untrusted values out of script and style blocks, event-handler attributes such as `onclick`, and other executable contexts. If a page needs data in JavaScript, use a well-defined JSON encoding pattern and understand the HTML context in which that JSON appears.

## 7. Avoid common escaping mistakes

### Escaping too early

Storing HTML entities in the database makes the stored data harder to reuse and can lead to double-encoding. Store the intended data, then encode it for the context where it is output.

### Escaping twice

If `&lt;` is escaped again, it may display as the literal characters `&lt;` instead of `<`. Decide where output encoding happens and apply it once for each output context. Do not disable double encoding globally just to hide inconsistent data handling.

### Treating validation as escaping

Validation decides whether a value is acceptable for a business rule, such as whether a number is in range. Escaping makes a value safe to represent in a particular output context. A page can need both.

### Treating escaping as validation

Encoding an attribute value does not prove that a link points to an allowed destination. Validate a URL's scheme and application rules separately.

### Removing tags as the only defense

Removing some tags does not reliably make arbitrary HTML safe. For plain text, escape it. If users are intentionally allowed to submit a limited set of HTML, use an HTML sanitizer with an explicit allowlist.

## 8. A practical rendering flow

A simple server-rendered page usually follows this order:

1. Read the needed request or application data.
2. Validate the data for the page's rules.
3. Prepare a small view-data structure.
4. Load a known template from the application.
5. Escape each untrusted value for its output context.
6. Send the completed HTML response.

Escaping at output keeps the stored value reusable. For instance, the same name can be escaped for HTML text in one response and encoded differently if later used in another format.

## Key takeaways

- Templates keep page structure readable and separate from the values used to fill it.
- Templates do not automatically escape dynamic data.
- Use `htmlspecialchars()` with explicit flags and UTF-8 for untrusted values in HTML text and quoted attributes.
- Keep attributes quoted; escape values at the output boundary.
- Validate URL schemes separately because HTML escaping does not make a URL safe.
- Use fixed, application-controlled template paths and pass templates only the data they need.
- JavaScript, CSS, URLs, and user-authored rich HTML require handling appropriate to their own contexts.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [084 - HTTP requests, responses, and the PHP request lifecycle](084-http-requests-responses-and-the-php-request-lifecycle.md) | [Q&A](../qnas/085-generating-html-safely-with-templates.md) | [Example](../examples/085-generating-html-safely-with-templates.md) | [086 - Reading query strings and request data](086-reading-query-strings-and-request-data.md) |