# 085 - Generating HTML safely with templates: Q&A

## Multiple-choice questions

### 1. What is the main purpose of a server-side template?

- **A.** To store every database query in an HTML file
- **B.** To describe page structure and place dynamic values in that structure
- **C.** To make all PHP output safe automatically
- **D.** To replace HTTP responses

**Answer: B.** A template keeps the page structure readable and provides locations for dynamic values. It does not automatically escape those values, so output still needs context-appropriate handling.

### 2. Which function is appropriate for encoding untrusted text for an HTML text node?

- **A.** `htmlspecialchars()`
- **B.** `trim()`
- **C.** `intval()`
- **D.** `include()`

**Answer: A.** `htmlspecialchars()` converts HTML-significant characters such as `<`, `>`, `&`, and quotes into entities. The other functions do not perform HTML output encoding.

### 3. Why should an HTML attribute value usually be quoted?

- **A.** Quotes make PHP variables immutable.
- **B.** Quotes establish a clear boundary around the attribute value.
- **C.** Quoted attributes do not need escaping.
- **D.** Quotes validate the URL scheme.

**Answer: B.** Quoting the value and escaping it work together to prevent data from being interpreted as part of the surrounding HTML syntax. Quoting does not replace escaping or URL validation.

### 4. Does `htmlspecialchars()` make every `href` value safe?

- **A.** Yes, because it removes every dangerous URL.
- **B.** Yes, but only when the attribute is single-quoted.
- **C.** No. Validate allowed URL schemes separately, then escape the value for the HTML attribute.
- **D.** No. Never use links in templates.

**Answer: C.** HTML escaping encodes special HTML characters. It does not decide whether a destination such as a `javascript:` URL is allowed by the application.

### 5. What is the safest source for a template include path?

- **A.** A request parameter used directly in `require`
- **B.** A fixed application path or an application-controlled allowlist
- **C.** A value copied from a hidden form field
- **D.** A database value without validation

**Answer: B.** Template paths should be controlled by the application. If several templates are available, map approved identifiers to fixed paths instead of using user input as a file path.

### 6. When should a plain text value generally be escaped for HTML?

- **A.** Before storing it, so every copy contains HTML entities
- **B.** At the point where it is written into the HTML response
- **C.** Only when it contains a `<` character
- **D.** Only if it came from `$_GET`

**Answer: B.** Escape at the output boundary for the context where the value is used. Data from databases or other systems can also originate with users, and pre-escaping storage can cause reuse and double-encoding problems.

## Short-answer questions

### 7. What does `<?= $value ?>` mean in a PHP template?

**Answer:** It is shorthand for `<?php echo $value; ?>`. It outputs the expression's value. It does not escape the value by itself, so untrusted data should be passed through an appropriate escaping function, for example `<?= escapeHtml($value) ?>`.

### 8. What is a useful `escapeHtml()` helper for UTF-8 HTML output?

**Answer:**

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

**Explanation:** `ENT_QUOTES` encodes both quote types, `ENT_SUBSTITUTE` handles invalid sequences with a replacement character, and the explicit encoding makes the expected character set clear. Use this for HTML text and quoted ordinary attributes; it is not a universal encoder for other contexts.

### 9. Why is escaping stored data early often a poor design?

**Answer:** A stored value may later be displayed in HTML, JSON, plain text, or another context, and each context has different output rules. Storing HTML entities also makes the underlying data harder to use and can lead to double-encoding. Keep the intended value and encode it when rendering the relevant output.

### 10. What is the difference between validation and output escaping?

**Answer:** Validation checks whether a value meets an application's rules, such as a maximum length or an allowed URL scheme. Escaping encodes a value so it is represented safely in a particular output context. A value may need both checks.

### 11. Why is `strip_tags()` not a complete defense when rendering user input?

**Answer:** Removing tags does not safely handle every output context or validate every kind of input. For plain text, encode it for HTML output. If the application intentionally permits a restricted set of user-authored HTML, use an allowlist-based HTML sanitizer designed for that purpose.

### 12. What does a template included with `require` know about the calling PHP file?

**Answer:** The included file inherits the variable scope at the line where it is required. Variables available at that point can be visible inside the template. Keep that scope understandable: prepare the view data before rendering and let the template depend only on the values it needs.

## Code and output questions

### 13. What does the browser interpret in this example, and what does the user see?

```php
$name = '<script>alert("hello")</script>';
echo '<p>' . escapeHtml($name) . '</p>';
```

**Answer:** The browser interprets the application-authored `<p>` element. The value is encoded, so the user sees the script-like text inside the paragraph instead of the browser executing it. With the helper from question 8, the output source contains entities such as `&lt;script&gt;` and `&quot;`.

**Explanation:** The dynamic value is encoded before it enters the HTML text node. The template's own `<p>` markup stays intact.

### 14. Identify the security problem in this snippet.

```php
$page = $_GET['page'] ?? 'home';
require __DIR__ . '/templates/' . $page . '.php';
```

**Answer:** Untrusted request data controls a filesystem path. An attacker may try path traversal or select an unintended file, depending on the environment and available files.

**Safer approach:** Map a validated identifier to a fixed path:

```php
$templates = [
    'home' => __DIR__ . '/templates/home.php',
    'about' => __DIR__ . '/templates/about.php',
];

$page = $_GET['page'] ?? 'home';
$templatePath = $templates[$page] ?? $templates['home'];
require $templatePath;
```

**Explanation:** The request can select only from paths already approved by the application. Validation of the identifier does not mean the template should receive arbitrary filesystem paths.

### 15. Why is this link still unsafe even if its attribute is escaped?

```php
<a href="<?= escapeHtml($userProvidedUrl) ?>">Open profile</a>
```

**Answer:** Escaping protects the HTML attribute syntax, but it does not decide whether the URL scheme is allowed. Validate the URL against the application's rules first, such as allowing only `https` and `http` schemes (or a carefully defined set of relative paths), then escape the approved value for the quoted attribute.

**Explanation:** HTML parsing and URL navigation are separate contexts. An encoded attribute can still contain a disallowed destination.

### 16. Should a value be passed through `escapeHtml()` before placing it in a JavaScript string or CSS declaration?

**Answer:** No. `escapeHtml()` is for HTML text and quoted ordinary attributes. JavaScript and CSS have different parsing rules. Prefer to keep untrusted values out of inline script and style code; if data must cross into JavaScript, use a well-defined JSON encoding pattern appropriate to the exact HTML and script context.

## Scenario questions

### 17. A user submits a biography that contains `<b>experienced</b>`. The application intends to show the biography as plain text. What should it do?

**Answer:** Treat the biography as text and HTML-escape it when rendering. The browser will display the angle-bracket characters instead of creating a bold element.

**Explanation:** The product requirement is plain text, so interpreting user markup is unnecessary and increases risk. If rich text later becomes a requirement, use an allowlist-based sanitizer instead of simply omitting escaping.

### 18. A developer sees `&amp;` in a page and sets `double_encode` to `false` everywhere. Is that the right fix?

**Answer:** No. First find where data is being escaped more than once and make output encoding happen consistently at the correct boundary. Disabling double encoding can preserve preexisting entities, but it can also leave attacker-controlled entity sequences untreated if data handling is inconsistent.

### 19. A template contains `onclick="...<?= $value ?>..."`. Is it enough to call `escapeHtml($value)`?

**Answer:** No. An event-handler attribute is a JavaScript context embedded inside HTML. HTML escaping alone is not a general-purpose JavaScript encoder. Avoid placing untrusted values in inline event handlers; attach behavior from a script and transfer data using a context-safe approach.

## Key takeaways

- `<?= ... ?>` outputs a value; it does not escape it.
- Use `htmlspecialchars()` for untrusted values in HTML text and quoted ordinary attributes.
- Validate URLs and template identifiers with rules separate from HTML escaping.
- Render plain text as text; sanitize only when a limited set of rich HTML is a deliberate feature.
- Keep template paths application-controlled and template data narrow.
- Output encoding must match the exact destination context.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [084 - HTTP requests, responses, and the PHP request lifecycle](../qnas/084-http-requests-responses-and-the-php-request-lifecycle.md) | [Lesson](../lessons/085-generating-html-safely-with-templates.md) | [Example](../examples/085-generating-html-safely-with-templates.md) | [086 - Reading query strings and request data](../qnas/086-reading-query-strings-and-request-data.md) |