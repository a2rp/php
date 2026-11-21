# 125 - Preventing cross-site scripting

Cross-site scripting (XSS) happens when a page causes the browser to interpret untrusted data as executable code or markup. The browser then runs or renders content that the application did not intend to trust.

Prevent XSS by using safe output methods and applying the right protection for the exact place where data is inserted. For ordinary HTML text in PHP, encode the value when rendering it. In browser code, prefer APIs that treat values as text. Validate URLs and sanitize user-authored HTML when those features are required.

## Learning goals

By the end of this chapter, you should be able to:

- Recognize stored, reflected, and DOM-based XSS.
- Encode untrusted values for ordinary HTML text and quoted attributes.
- Use safe DOM methods such as textContent for plain text.
- Identify contexts that need different handling, including URLs, JavaScript, CSS, and user-authored HTML.
- Explain why input validation, cookies, and Content Security Policy do not replace safe output handling.
- Review and test every place untrusted data reaches a page.

## How XSS happens

A browser parses HTML, JavaScript, CSS, and URLs according to different rules. A value that is harmless in one place can become executable in another.

For example, suppose an application stores a display name and then prints it directly:

~~~php
echo '<p>' . $displayName . '</p>';
~~~

If the value contains HTML syntax, the browser may parse it as markup instead of displaying it as text. Depending on where the value is inserted, an attacker may be able to run script in the application's origin. This can expose data available to the page, perform actions as the current user, or change what the user sees.

Treat user-controlled and externally supplied data as untrusted, including:

- Form fields, route parameters, query strings, and URL fragments.
- Comments, names, and other values stored in the database.
- Cookies and request headers.
- Uploaded file names and imported data.
- Responses from third-party services.

Data that came from a database is not automatically safe. It may have been written by a user earlier and later displayed to someone else.

## The common XSS categories

### Stored XSS

The application saves untrusted content, then displays it later. A comment is a common example. Each viewer may be affected when the comment is rendered unsafely.

### Reflected XSS

The application includes request data in an immediate response without safely encoding it. A value from a query string or form may be reflected into the page.

### DOM-based XSS

Browser-side JavaScript reads an untrusted source, such as a URL fragment, and sends it to a DOM API that parses it as HTML or code. The unsafe operation can happen entirely in the browser.

These categories describe how the data reaches a page. The core review question is the same: where does untrusted data enter a context that interprets markup or code?

## Encode when rendering ordinary HTML

For HTML text and ordinary quoted text attributes, use PHP's htmlspecialchars() with an explicit character set and quote handling:

~~~php
function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}
~~~

Display a value as text:

~~~php
<p><?= escapeHtml($displayName) ?></p>
~~~

Use the helper for a quoted, ordinary text attribute while keeping the element and attribute names fixed:

~~~php
<input
    type="text"
    name="displayName"
    value="<?= escapeHtml($displayName) ?>"
>
~~~

This converts characters that have special meaning in HTML into entities. The browser displays them as text rather than treating them as tag or attribute syntax.

Escape at the point where the value is rendered. Store the original value when the application needs it later; do not permanently HTML-encode database values on input. The same value may be used in different output contexts and needs a different treatment in each.

Set the document character set to UTF-8 consistently, for example with an appropriate response header and HTML metadata. The encoding passed to the helper should match the document encoding.

## Encoding is context-specific

HTML encoding is not a universal escaping function. Browsers use different parsers for HTML, JavaScript, CSS, and URLs. Apply protection for the actual destination, and avoid putting untrusted values into contexts that are difficult to secure.

### Ordinary HTML text and attributes

For text between normal HTML tags and for quoted ordinary text attributes such as title or value, HTML-encode the value. Keep tag names and attribute names fixed.

Do not use an ordinary attribute as though it were safe when it actually interprets code or markup. Event-handler attributes such as onclick, srcdoc, and style have additional parsing rules. Avoid building them from untrusted values.

### URL values

When a value is part of a URL query parameter, URL-encode the parameter value with a URL-building function. Do not URL-encode an entire URL indiscriminately.

When a complete URL comes from a user, validate the scheme and the application's allowed destinations. In particular, do not allow a javascript: URL to be placed in a link or other URL-bearing attribute. Then HTML-encode the validated URL when placing it in a quoted HTML attribute. URL encoding by itself does not make an unsafe URL scheme safe.

Prefer application-generated relative paths for internal navigation. Keep the surrounding href or src attribute fixed.

### JavaScript and CSS

Do not concatenate untrusted text into JavaScript source, event-handler attributes, CSS source, selectors, property names, or declaration blocks. Use a well-defined framework or library mechanism for values that truly must enter one of these contexts, and follow its context-specific guidance.

A safer design is often to keep data in a quoted, HTML-encoded data attribute or retrieve it as JSON, then use it as data in a separate script. Avoid inline event handlers and move behavior into trusted JavaScript functions.

### User-authored rich HTML

If a feature is intended to display user-authored formatting, encoding all markup would remove that formatting. Use a maintained HTML sanitizer configured with an explicit allowlist of supported elements and attributes. Keep the sanitizer updated, and do not modify sanitized markup in ways that can reintroduce unsafe content.

Do not attempt to make arbitrary HTML safe with a few string replacements or regular expressions. Sanitizing HTML is different from encoding plain text.

## Use safe browser DOM APIs

When JavaScript needs to display plain text, use a text sink on an ordinary element:

~~~javascript
const message = new URLSearchParams(window.location.search).get('message') ?? '';
document.querySelector('#message').textContent = message;
~~~

textContent creates a text node instead of parsing the value as HTML. The text remains data.

Avoid assigning untrusted strings to HTML-parsing or code-executing sinks such as innerHTML, outerHTML, insertAdjacentHTML(), document.write(), or eval(). When a page needs to create elements, use createElement(), set fixed safe attributes, assign plain text through textContent, and append the resulting nodes.

textContent is not a universal sanitizer. Do not use it to set executable script or style content. URL-bearing attributes still require URL validation, and attribute names must remain fixed and safe.

## Input validation still matters

Validate input to enforce product rules: a display name may have a length limit, an email address may need an expected format, and a sort choice may be restricted to known values.

Validation does not replace output encoding. A value can be valid according to the product and still contain characters that need encoding when placed in HTML. The safest approach is to validate at input boundaries and encode at each output boundary.

Do not try to remove every character that looks like markup. Such filters often break legitimate text and may miss browser parsing cases.

## Templates and frameworks

Many template engines escape values by default. Keep that protection enabled and use the framework's normal text interpolation features.

Be cautious with features named raw, safe, unescaped, or HTML. These commonly bypass automatic encoding. Use them only for trusted, appropriately sanitized markup.

Review custom template helpers, server-rendered HTML fragments, and client-side DOM updates. A secure default can be bypassed by one unsafe escape hatch or sink.

## Defense in depth

### Content Security Policy

A Content Security Policy (CSP) can restrict which scripts and other resources a browser may load or execute. A carefully designed policy can reduce the impact of some XSS mistakes, especially when it avoids unrestricted inline scripts and limits resource origins.

CSP is an additional protection, not a replacement for output encoding, safe DOM methods, or HTML sanitization. A policy can also break existing features if applied without review. Introduce it deliberately, monitor violations, and tighten it based on the application's real requirements.

### Cookie protections

Mark session cookies HttpOnly so client-side scripts cannot read them directly, and use Secure over HTTPS. SameSite can reduce some cross-site cookie sending. These settings are valuable but do not prevent XSS: malicious script running on the page can still act through the user's browser even if it cannot read the cookie value.

### Limit what the page can access

Use least privilege for application features and protect sensitive actions with authorization and CSRF controls. XSS running in a user's page can often make same-origin requests as that user, so keep sensitive data and capabilities limited.

## A practical review checklist

For each value that can be influenced outside trusted application code:

1. Find every place it is rendered on the server and in the browser.
2. Identify the exact context: HTML text, ordinary attribute, URL, JavaScript, CSS, or HTML markup.
3. Use the context-appropriate encoder, safe text API, URL validation, or maintained HTML sanitizer.
4. Keep element names, attribute names, and code structure fixed.
5. Search for unsafe APIs such as innerHTML, document.write(), eval(), inline event handlers, and raw template output.
6. Test values containing HTML-like characters in each relevant output location.
7. Confirm CSP and cookie settings provide additional protection without being treated as the main fix.
8. Recheck every new or changed rendering path during review.

## Key takeaways

- XSS occurs when a browser interprets untrusted data as markup or code.
- Encode at output for the exact context; HTML encoding is not universal.
- Use htmlspecialchars() for ordinary HTML text and quoted text attributes in PHP.
- Use textContent for plain text in the DOM; avoid HTML and code parsing sinks.
- Validate URL schemes and sanitize user-authored HTML when those features are required.
- Validation, CSP, and cookie flags add protection but do not replace safe rendering.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [124 - Preventing SQL injection](124-preventing-sql-injection.md) | [Q&A](../qnas/125-preventing-cross-site-scripting.md) | [Example](../examples/125-preventing-cross-site-scripting.md) | [126 - Preventing cross-site request forgery](126-preventing-cross-site-request-forgery.md) |