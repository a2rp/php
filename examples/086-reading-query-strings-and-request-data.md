# 086 - Reading query strings and request data: Example

This example builds a small search page that reads a search term and page number from the URL. It checks that each query value has the expected shape, validates the page range, builds pagination links with `http_build_query()`, and escapes values when rendering HTML.

## File structure

```text
project/
└── public/
    └── search.php
```

## `public/search.php`

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

function readStringParameter(array $query, string $name, string $default): string
{
    if (!array_key_exists($name, $query)) {
        return $default;
    }

    $value = $query[$name];

    if (!is_string($value)) {
        sendBadRequest('The ' . $name . ' parameter must be a single value.');
    }

    return $value;
}

function sendBadRequest(string $message): void
{
    http_response_code(400);
    header('Content-Type: text/html; charset=UTF-8');

    echo '<!doctype html><html lang="en"><meta charset="utf-8">';
    echo '<title>Bad request</title><h1>Bad request</h1><p>';
    echo escapeHtml($message);
    echo '</p></html>';
    exit;
}

if (($_SERVER['REQUEST_METHOD'] ?? '') !== 'GET') {
    http_response_code(405);
    header('Allow: GET');
    header('Content-Type: text/plain; charset=UTF-8');
    exit('This search page only accepts GET requests.');
}

header('Content-Type: text/html; charset=UTF-8');

$searchTerm = trim(readStringParameter($_GET, 'q', ''));
$rawPage = readStringParameter($_GET, 'page', '1');

if (strlen($searchTerm) > 100) {
    sendBadRequest('The search term must be no longer than 100 bytes.');
}

$page = filter_var(
    $rawPage,
    FILTER_VALIDATE_INT,
    ['options' => ['min_range' => 1, 'max_range' => 1000]]
);

if ($page === false) {
    sendBadRequest('The page parameter must be a positive integer.');
}

$catalog = [
    ['title' => 'PHP Basics', 'category' => 'Language'],
    ['title' => 'PHP Templates', 'category' => 'Web development'],
    ['title' => 'PHP Forms', 'category' => 'Web development'],
    ['title' => 'HTML Basics', 'category' => 'Markup'],
    ['title' => 'Secure URLs', 'category' => 'Security'],
    ['title' => 'CSS Basics', 'category' => 'Stylesheets'],
];

$matches = array_values(array_filter(
    $catalog,
    static function (array $item) use ($searchTerm): bool {
        return $searchTerm === ''
            || stripos($item['title'], $searchTerm) !== false
            || stripos($item['category'], $searchTerm) !== false;
    }
));

$itemsPerPage = 2;
$totalResults = count($matches);
$totalPages = max(1, (int) ceil($totalResults / $itemsPerPage));

if ($page > $totalPages) {
    http_response_code(404);
    header('Content-Type: text/plain; charset=UTF-8');
    exit('That results page does not exist.');
}

$offset = ($page - 1) * $itemsPerPage;
$pageItems = array_slice($matches, $offset, $itemsPerPage);

function searchUrl(string $searchTerm, int $page): string
{
    $query = http_build_query(
        ['q' => $searchTerm, 'page' => $page],
        '',
        '&',
        PHP_QUERY_RFC3986
    );

    return '/search.php?' . $query;
}
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Catalog search</title>
</head>
<body>
    <main>
        <h1>Catalog search</h1>

        <form action="/search.php" method="get">
            <label for="q">Search the catalog</label>
            <input
                id="q"
                name="q"
                type="search"
                value="<?= escapeHtml($searchTerm) ?>"
                maxlength="100"
            >
            <button type="submit">Search</button>
        </form>

        <p><?= $totalResults ?> result(s); page <?= $page ?> of <?= $totalPages ?>.</p>

        <?php if ($pageItems === []): ?>
            <p>No matching items were found.</p>
        <?php else: ?>
            <ul>
                <?php foreach ($pageItems as $item): ?>
                    <li>
                        <strong><?= escapeHtml($item['title']) ?></strong>
                        <span><?= escapeHtml($item['category']) ?></span>
                    </li>
                <?php endforeach; ?>
            </ul>
        <?php endif; ?>

        <nav aria-label="Search result pages">
            <?php if ($page > 1): ?>
                <a href="<?= escapeHtml(searchUrl($searchTerm, $page - 1)) ?>">Previous page</a>
            <?php endif; ?>

            <?php if ($page < $totalPages): ?>
                <a href="<?= escapeHtml(searchUrl($searchTerm, $page + 1)) ?>">Next page</a>
            <?php endif; ?>
        </nav>
    </main>
</body>
</html>
```

## How the example works

1. The endpoint accepts only the `GET` method. The method check is separate from reading `$_GET`, because query parameters can accompany other methods too.
2. `readStringParameter()` supplies a default when a parameter is absent and rejects array-shaped values such as `q[]=php`.
3. The page number is validated as an integer in a bounded range. The code checks the result strictly against `false` before using it in arithmetic.
4. The sample catalog is filtered by the search term, then sliced into pages of two items.
5. `http_build_query()` encodes the query values for pagination URLs. `escapeHtml()` then protects those URLs and other dynamic values when they enter HTML attributes and text.

The in-memory catalog keeps the example focused on query handling. In a database-backed application, validate the query the same way and use parameterized database queries for the search operation.

## Try these URLs

Run the PHP built-in server from the project directory:

```bash
php -S localhost:8000 -t public
```

Then open these paths on the local server:

```text
/search.php?q=php&page=1
/search.php?q=php&page=2
/search.php?q=PHP%20%26%20HTML
/search.php?q%5B%5D=php
/search.php?q=php&page=abc
```

The first URL returns the first two PHP matches. The second returns the remaining PHP match. The third searches for the literal term `PHP & HTML` and returns no matches in this sample catalog. The fourth is rejected because `q` is an array instead of a single string. The fifth is rejected because `abc` is not a valid page number.

## Expected output for `?q=php&page=1`

```text
Catalog search
Search the catalog: [php] [Search]
3 result(s); page 1 of 2.

- PHP Basics — Language
- PHP Templates — Web development

Next page
```

The generated next-page URL is:

```text
/search.php?q=php&page=2
```

The browser displays ordinary text and links. The PHP code generates HTML, but it does not execute JavaScript or render a separate application preview.

## Security and behavior notes

- Query values are client-controlled. The example checks type and range before using them.
- `trim()` only removes surrounding whitespace; it is not a security validator.
- The search term is escaped when put back into the search field. Validation and HTML escaping serve different purposes.
- The page uses `stripos()` to keep the sample short. For multilingual case-insensitive search, use an appropriate Unicode-aware implementation and a suitable database collation.
- The page number has a maximum to limit unreasonable input. Real applications should choose limits based on their actual pagination behavior.
- This demo uses `GET` only for reading results. Do not use a `GET` URL to delete or modify data.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [085 - Generating HTML safely with templates](../lessons/085-generating-html-safely-with-templates.md) | [Q&A](../qnas/086-reading-query-strings-and-request-data.md) | Example | [087 - HTML forms and GET or POST requests](../lessons/087-html-forms-and-get-or-post-requests.md) |