# PHP quick reference and glossary

Use this chapter to look up common PHP syntax and terms while working through the lessons. It is a practical reference for the language and web-development patterns used in this course, not an inventory of every built-in function or extension.

The examples use modern PHP 8 syntax. Several features are version-specific, so check the version table before copying code into an older project. The repository’s broader [PHP summary](../php-summary.md) and [PHP keywords reference](../php-keywords.md) provide additional internal reference pages.

## Start a PHP file

~~~php
<?php

declare(strict_types=1);

echo 'Hello, PHP!', PHP_EOL;
~~~

- Use the standard PHP opening tag in PHP-only files.
- Put declare(strict_types=1) at the beginning of a PHP file, before executable statements.
- PHP statements usually end in a semicolon.
- PHP names are case-sensitive in many contexts, including variable names. Follow one consistent naming style.
- In a PHP-only file, omit the closing ?> tag to avoid accidentally sending whitespace to the response.

## Comments

~~~php
// A single-line comment.

/*
 * A multi-line comment.
 */

/** A documentation comment for a class, method, or function. */
~~~

Comments explain decisions and non-obvious behavior. Prefer clear names and small functions over comments that merely repeat the code.

## Values and types

| Type family | Common types |
| --- | --- |
| Scalar | bool, int, float, string |
| Compound | array, object |
| Special | null, resource, callable, iterable, mixed, never, void |

Examples:

~~~php
$active = true;
$count = 3;
$price = 19.95;
$title = 'PHP notes';
$items = ['lesson', 'Q&A', 'example'];
$nothing = null;
~~~

Use strict comparisons and explicit checks where possible. Do not assume that a value received from a form, URL, JSON body, or database has the type you expect.

### Type declarations

~~~php
function formatTitle(string $title): string
{
    return trim($title);
}

function findLesson(int $id): ?array
{
    return $id > 0 ? ['id' => $id] : null;
}

function parseIdentifier(string|int $value): int
{
    return (int) $value;
}
~~~

- A parameter declaration documents and checks accepted input types.
- A return declaration documents and checks the function result.
- A nullable type such as ?array means the value may be an array or null.
- A union type such as string|int allows either listed type.
- Use mixed only when a value is intentionally allowed to have any type.
- strict_types affects scalar type coercion at calls made from that file; it does not validate HTTP input.

### Variables and constants

~~~php
$name = 'Mira';
$name = 'Mira Patel'; // Variables can be reassigned.

const APP_NAME = 'Reading tracker';
define('MAX_ITEMS', 100);
~~~

- Variables begin with $, are case-sensitive, and do not need a declaration.
- Constants use const or define and have no $ prefix.
- Prefer descriptive names such as $bookCount over ambiguous names such as $x.
- Constants are conventionally written in uppercase with underscores.

## Strings

~~~php
$first = 'PHP';
$message = "Welcome to $first";       // Interpolates a variable.
$fullName = $first . ' developer';    // Concatenates with a dot.
$label = "{$first} lessons";          // Braces make the variable boundary clear.
~~~

Single-quoted strings mostly preserve their contents. Double-quoted strings interpolate variables and recognize escape sequences. Use braces around a variable when adjacent text could make its boundary unclear. Avoid deprecated dollar-brace interpolation.

For multiline text, use nowdoc when interpolation is not wanted and heredoc when it is:

~~~php
$plainText = <<<'TEXT'
Variables such as $title remain literal here.
TEXT;

$personalized = <<<TEXT
Welcome, $name.
TEXT;
~~~

## Arrays

PHP arrays are ordered maps. They can act as lists, dictionaries, or mixed structures.

~~~php
$colors = ['blue', 'green', 'red'];
$book = [
    'title' => 'PHP in Practice',
    'status' => 'reading',
];

$colors[] = 'purple';
$title = $book['title'] ?? 'Untitled';
~~~

Common operations:

~~~php
$count = count($colors);
$hasTitle = array_key_exists('title', $book);
$statuses = array_column($allBooks, 'status');
$finished = array_filter(
    $allBooks,
    static fn(array $item): bool => $item['status'] === 'finished'
);
$names = array_map(
    static fn(array $item): string => $item['title'],
    $allBooks
);
~~~

- Use [] for a list or map literal.
- Use array_key_exists when you need to distinguish a missing key from a key whose value is null.
- Use ?? to provide a default when a key is missing or null.
- Use foreach to process items.
- Do not interpolate arbitrary client input as an array key without validating it.

## Operators and comparisons

| Operator | Meaning |
| --- | --- |
| +, -, *, /, % | Arithmetic |
| . | String concatenation |
| = | Assignment |
| ===, !== | Strict comparison of value and type |
| ==, != | Loose comparison after type juggling |
| <, <=, >, >= | Ordering comparison |
| &&, \|\|, ! | Logical operations |
| ?? | Null coalescing |
| ??= | Assign only if unset or null |
| ?: | Shorthand conditional expression |
| ** | Exponentiation |

Use === and !== unless loose comparison is intentional.

~~~php
$displayName = $nickname ?? 'Guest';
$isAdult = $age >= 18;
$result = $isReady ? 'ready' : 'waiting';
~~~

Avoid relying on surprising type conversions in conditions. In particular, empty() treats several different values as empty, including the string '0'. Use an explicit comparison when those values matter.

## Control flow

~~~php
if ($score >= 90) {
    $grade = 'A';
} elseif ($score >= 80) {
    $grade = 'B';
} else {
    $grade = 'Needs review';
}

foreach ($books as $book) {
    echo $book['title'], PHP_EOL;
}

for ($i = 0; $i < 3; $i++) {
    echo $i, PHP_EOL;
}

while ($queue !== []) {
    $item = array_shift($queue);
}
~~~

A switch statement is useful for several branches. A match expression is an alternative when each branch produces a value:

~~~php
$label = match ($status) {
    'planned' => 'Planned',
    'reading' => 'In progress',
    'finished' => 'Finished',
    default => 'Unknown',
};
~~~

match uses strict comparison and returns a value. If no arm matches and there is no default, PHP throws UnhandledMatchError.

## Functions and callables

~~~php
function addTax(float $amount, float $rate = 0.0): float
{
    return $amount + ($amount * $rate);
}

$total = addTax(amount: 100.0, rate: 0.18);

function collectTitles(array ...$books): array
{
    return array_map(
        static fn(array $book): string => $book['title'],
        $books
    );
}
~~~

- Parameters are values passed into a function.
- Arguments are the values supplied when calling it.
- Defaults make an argument optional.
- Variadic parameters collect remaining arguments.
- Named arguments are available in PHP 8.0 and later.
- Return early when it makes a function easier to read.

Closures and arrow functions:

~~~php
$minimumLength = 3;
$longEnough = static fn(string $value): bool =>
    mb_strlen($value, 'UTF-8') >= $minimumLength;

$formatter = static function (string $value): string {
    return trim($value);
};
~~~

Arrow functions automatically capture variables from the outer scope by value. A normal closure uses a use clause when it needs outer variables.

## Classes, objects, interfaces, traits, and enums

~~~php
interface Describable
{
    public function description(): string;
}

final class Book implements Describable
{
    public function __construct(
        public string $title,
        private string $author
    ) {
    }

    public function description(): string
    {
        return $this->title . ' by ' . $this->author;
    }
}
~~~

- A class describes a type of object.
- An object is an instance of a class.
- A property stores object state.
- A method is a function declared in a class.
- An interface defines methods a class agrees to provide.
- A trait reuses method implementations across classes.
- Visibility is public, protected, or private.
- Constructor property promotion is available in PHP 8.0 and later.

Enums represent a closed set of named values and require PHP 8.1 or later:

~~~php
enum ReadingStatus: string
{
    case Planned = 'planned';
    case Reading = 'reading';
    case Finished = 'finished';
}
~~~

Prefer a simple function or value when a class would add no useful behavior.

## Forms, request data, and validation

Common superglobals include:

| Superglobal | Typical use |
| --- | --- |
| $_GET | Query-string values |
| $_POST | Form-encoded POST values |
| $_FILES | Uploaded file metadata |
| $_COOKIE | Cookie values |
| $_SESSION | Values stored for a session |
| $_SERVER | Request and server metadata |
| $_ENV | Environment variables, when configured |
| $GLOBALS | Global symbol table; avoid relying on it for application design |

Treat every request value as untrusted. Validate on the server before using it.

~~~php
$title = $_POST['title'] ?? null;

if (!is_string($title)) {
    $errors['title'] = 'Enter a title.';
} else {
    $title = trim($title);

    if ($title === '') {
        $errors['title'] = 'A title is required.';
    } elseif (mb_strlen($title, 'UTF-8') > 160) {
        $errors['title'] = 'The title is too long.';
    }
}
~~~

Validation checks whether data fits application rules. Sanitizing or transforming data is a separate decision. Do not silently change user input in ways that alter its meaning.

## HTML output safety

Escape dynamic text when it is inserted into HTML:

~~~php
function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

echo '<h1>' . e($title) . '</h1>';
~~~

- Validation does not replace output escaping.
- Use escaping for the output context; HTML, JavaScript, CSS, and URLs have different rules.
- Keep HTML templates separate from database queries.
- Do not print raw exception details to visitors.

## HTTP and forms

- GET retrieves a representation and should not change state.
- POST commonly submits a form or creates a resource.
- PUT commonly replaces a complete resource.
- PATCH changes part of a resource.
- DELETE removes a resource.
- Use an appropriate status code: 200 for success, 201 for creation, 204 for success without a body, 400 for a bad request, 403 for denied access, 404 for missing resources, 422 for invalid fields, and 500 for an unexpected server failure.
- Use Post/Redirect/Get after a successful form submission.
- Use CSRF tokens for state-changing cookie-authenticated form requests.
- CORS is a browser cross-origin policy, not authentication.

## Database access with PDO

~~~php
$pdo = new PDO('sqlite:' . $databasePath);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

$statement = $pdo->prepare(
    'SELECT id, title FROM books WHERE status = :status'
);
$statement->execute(['status' => $status]);
$books = $statement->fetchAll();
~~~

- Use prepared statements for values supplied by users.
- A placeholder represents a value, not a table or column name.
- Allowlist dynamic sort fields and other SQL structure.
- Use a transaction when several writes must succeed together.
- Keep credentials and database files out of the public web directory and source control.

## JSON

~~~php
try {
    $data = json_decode($json, false, 512, JSON_THROW_ON_ERROR);
    $output = json_encode($data, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    // Handle invalid JSON or an encoding failure safely.
}
~~~

json_decode converts JSON into PHP values. With associative mode false, a JSON object becomes a PHP object; a JSON array becomes a PHP array. JSON_THROW_ON_ERROR makes parsing and encoding failures explicit through JsonException.

## Files and paths

~~~php
$contents = file_get_contents($path);

if ($contents === false) {
    throw new RuntimeException('Could not read the file.');
}

if (!is_dir($directory)) {
    mkdir($directory, 0775, true);
}
~~~

Check return values. Do not let a user-controlled path select arbitrary files. Validate a path against an intended directory or map a fixed set of allowed names to fixed paths.

## Errors and exceptions

~~~php
try {
    $result = performOperation();
} catch (InvalidArgumentException $exception) {
    // Handle an expected invalid input.
} catch (Throwable $exception) {
    error_log((string) $exception);
    // Return a safe response.
} finally {
    // Cleanup that must happen either way.
}
~~~

An exception represents an exceptional condition that can be handled. Catch a specific exception when you know how to recover. Log unexpected details privately and show a safe message to the user.

## Namespaces, Composer, and common commands

Namespaces organize code and prevent name collisions:

~~~php
namespace App\Repository;

final class BookRepository
{
}
~~~

Composer manages PHP packages and autoloading. Common commands:

~~~text
composer init
composer require vendor/package
composer require --dev vendor/test-tool
composer install
composer update
composer dump-autoload
~~~

Commit composer.json and composer.lock for an application. Review dependencies before adding them.

Common commands:

~~~text
php -v
php -m
php file.php
php -l path/to/file.php
php -S 127.0.0.1:8000 -t public
composer validate
~~~

The PHP built-in server is for local development, not public production hosting. A syntax check with php -l does not verify application behavior.

## Selected PHP version features

This is a short list of language milestones, not a complete changelog. Confirm that the server and deployment environment support the syntax used by the project.

| Version | Selected language features |
| --- | --- |
| PHP 8.0 | Union types, named arguments, match expressions, nullsafe operator, constructor property promotion, attributes |
| PHP 8.1 | Enums, readonly properties, intersection types, never return type, first-class callable syntax |
| PHP 8.2 | Readonly classes, DNF types, standalone null/false/true types; dynamic properties on ordinary classes are deprecated |
| PHP 8.3 | Typed class constants, json_validate(), readonly amendments |
| PHP 8.4 | Property hooks, asymmetric visibility, new DOM API |
| PHP 8.5 | Pipe operator, URI extension, array_first(), array_last(), clone-with syntax |

Some features depend on extensions or runtime configuration. A function can exist in the language documentation but still be unavailable if its extension is not installed.

## Glossary

| Term | Meaning |
| --- | --- |
| Argument | A value supplied when calling a function or method |
| Array | An ordered map used as a list, dictionary, or combination |
| Autoloading | Loading a class file automatically when a class is first used |
| Class | A definition from which objects can be created |
| Closure | An anonymous function that can retain access to values from its surrounding scope |
| Composer | PHP’s dependency manager and autoloading tool |
| Constant | A named value that cannot be reassigned |
| Cookie | A small value sent by a browser with matching requests |
| CSRF | Cross-site request forgery: a site is tricked into sending an unwanted authenticated request |
| Database transaction | A group of database operations committed or rolled back as a unit |
| Dependency | A package or component required by an application |
| Endpoint | A URL and method combination that exposes an API operation |
| Exception | An object representing an exceptional condition that can be caught |
| Expression | Code that produces a value |
| Extension | An optional or bundled component that adds PHP functions and classes |
| Function | A reusable named block of code |
| HTTP | The request and response protocol commonly used by web applications |
| Interface | A contract describing methods a class must implement |
| JSON | A text format for structured data exchange |
| Method | A function declared in a class |
| Namespace | A named scope for classes, functions, and constants |
| Object | A value created from a class |
| Parameter | A named input declared by a function or method |
| Prepared statement | A database query whose values are bound separately from its SQL structure |
| Property | A value stored on an object |
| Request | A message a client sends to a server |
| Response | A message the server returns, including status, headers, and optional body |
| Scope | The part of code where a variable or name is available |
| Session | Server-side state associated with a browser through a session identifier |
| Superglobal | A predefined variable available in all scopes, such as $_POST |
| Trait | A mechanism for reusing methods across classes |
| Type declaration | A parameter, property, or return type requirement in code |
| Validation | Checking whether input follows application rules |
| Variable | A named value whose name begins with $ |
| View | A template or rendering layer that produces output |
| XSS | Cross-site scripting: untrusted content is interpreted as executable browser code |

## Quick checks before using a snippet

- Which PHP version does the project support?
- Is this value untrusted request data?
- Has the value been validated for its purpose?
- Is it being escaped for the output context?
- Are database values bound through prepared statements?
- Are failures handled without exposing secrets?
- Do required extensions exist on the target runtime?
- Does the example belong in a CLI script, web request, or template?

## Related internal references

- [PHP summary](../php-summary.md)
- [PHP keywords](../php-keywords.md)

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Course |
| --- | --- | --- | --- |
| [152 - Capstone: plan, build, test, and document an application](152-capstone-project-plan-build-test-and-document-an-application.md) | [Q&A](../qnas/153-php-quick-reference-and-glossary.md) | [Example](../examples/153-php-quick-reference-and-glossary.md) | End of course |
