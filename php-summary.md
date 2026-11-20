# PHP: Complete Summary

This is a course-wide overview of PHP, from its core language and runtime to web applications, databases, security, testing, and deployment. Use it for review and navigation. The [README lesson list](README.md#lessons) links the detailed lessons, Q&A, and examples.

PHP is a general-purpose programming language commonly used to build server-side applications. The PHP runtime executes a script, often in response to an HTTP request, and the server sends the script's output to the client. PHP also runs from the command line and can be used for automation and other programs. PHP itself is separate from the web server, database, framework, and browser.

## Contents

- [How PHP runs](#how-php-runs)
- [Source files and syntax](#source-files-and-syntax)
- [Values, variables, and types](#values-variables-and-types)
- [Expressions and control flow](#expressions-and-control-flow)
- [Functions and reusable code](#functions-and-reusable-code)
- [Arrays, strings, and data](#arrays-strings-and-data)
- [Objects and modern PHP](#objects-and-modern-php)
- [Errors and debugging](#errors-and-debugging)
- [Web requests and responses](#web-requests-and-responses)
- [Databases and APIs](#databases-and-apis)
- [Composer and project structure](#composer-and-project-structure)
- [Security essentials](#security-essentials)
- [Testing, performance, and deployment](#testing-performance-and-deployment)
- [Problem solving and application design](#problem-solving-and-application-design)
- [Everyday coding checklist](#everyday-coding-checklist)
- [Course map](#course-map)

## How PHP runs

PHP code is interpreted by the PHP runtime. A web server can pass a request to PHP through an integration such as PHP-FPM. PHP can also execute a script directly from a terminal.

~~~sh
php --version
php -m
php script.php
php -S 127.0.0.1:8000 -t public
~~~

The built-in server is for local development, not a production web server. A PHP installation includes a runtime and selected extensions. Available behavior depends on the PHP version, enabled extensions, configuration, and the environment in which the script runs.

Keep the target runtime in mind. New syntax and APIs may not exist on older versions. Check the minimum PHP version required by a project and its dependencies, and use supported releases for deployed applications.

## Source files and syntax

PHP source is commonly saved with a `.php` extension. The opening tag starts PHP code. In a PHP-only file, omit the closing tag to avoid accidental whitespace being sent as output.

~~~php
<?php
declare(strict_types=1);

$name = 'Asha';
echo "Hello, {$name}!", PHP_EOL;
~~~

PHP statements usually end with a semicolon. Braces group blocks. Comments use `//`, `#`, or `/* ... */`. Names are generally case-sensitive, though PHP keywords and many built-in names are not. Variable names begin with `$`.

PHP can be embedded in an HTML template, but keep presentation and application logic organized. Do not mix untrusted data into HTML without escaping it for the correct output context.

## Values, variables, and types

A variable is a name that refers to a value. Assignment copies a value or shares an object handle according to PHP's value model.

~~~php
$count = 3;
$title = 'Learning PHP';
$isReady = true;
$nothing = null;
~~~

PHP has values such as null, booleans, integers, floating-point numbers, strings, arrays, objects, resources, and callables. A variable does not need a type declaration, but types can be declared for parameters, return values, and class properties. PHP also supports union and intersection types, nullable types, enums, and special types such as `mixed`, `void`, and `never`.

PHP can convert values automatically in some contexts. This is called type juggling. Prefer explicit conversion and validation where input or behavior could be ambiguous. Use strict comparisons when the type matters:

~~~php
if ($submittedId === $expectedId) {
    // Both value and type match.
}
~~~

The equality operator `==` may convert types before comparing. The identity operator `===` does not. The same distinction applies to `!=` and `!==`.

Constants are values that do not change after definition. Class constants belong to a class, and magic constants such as `__FILE__` and `__DIR__` are resolved by the runtime.

~~~php
const MAX_ATTEMPTS = 5;

if (isset($user['name'])) {
    $name = $user['name'];
}
~~~

`isset()` returns false for an unset value and for null. Use `array_key_exists()` when the difference between a missing key and a key containing null matters.

## Expressions and control flow

Expressions produce values. Operators combine values or perform actions. PHP includes arithmetic, assignment, comparison, logical, string, array, bitwise, null-coalescing, and nullsafe operators. Operator precedence affects how expressions are grouped, so use parentheses when intent could be unclear.

~~~php
$total = $price * $quantity;
$displayName = $name ?? 'Guest';
$canContinue = $isSignedIn && $hasPermission;
~~~

Use `if`, `elseif`, and `else` for conditional branches. Use `switch` for multi-branch control with its fall-through rules in mind. The `match` expression returns a value, uses strict comparison, and requires its arms to cover the input or provide a default.

~~~php
$label = match ($status) {
    'draft' => 'Draft',
    'published' => 'Published',
    default => 'Unknown',
};
~~~

Use `while`, `do ... while`, and `for` for repetition. `foreach` is the usual way to iterate over arrays and iterables. `break` leaves a loop, while `continue` starts its next iteration. Keep loop conditions and exit behavior easy to follow.

`include` and `require` load another PHP file. The `_once` forms avoid loading the same file again. Use `require` when the program cannot continue without the file. For project classes and libraries, Composer autoloading is usually preferable to a long chain of manual includes.

## Functions and reusable code

Functions package behavior behind a name, parameters, and an optional return value. Give parameters and return values types when useful. Validate assumptions at boundaries, and keep functions focused.

~~~php
function total(array $values): int
{
    $sum = 0;

    foreach ($values as $value) {
        if (!is_int($value)) {
            continue;
        }

        $sum += $value;
    }

    return $sum;
}
~~~

PHP functions support default values, named arguments, variadic parameters, argument unpacking, anonymous functions, closures, and arrow functions. A closure can capture variables with `use`. Arrow functions capture referenced outer variables automatically.

A callable can refer to a function or method. Modern PHP also supports first-class callable syntax. PHP 8.6 adds partial function application. Check the project's minimum runtime before using recent language features.

PHP supports passing and returning by reference with `&`. References change how values are shared, so use them only when their semantics are needed. Generators use `yield` to produce values lazily and can avoid building a large result array all at once.

## Arrays, strings, and data

PHP arrays are ordered maps. They can act as indexed lists, associative maps, stacks, queues, or mixed collections. Array keys are integers or strings, and certain key values may be converted. Choose a consistent shape and document data structures passed between parts of an application.

~~~php
$person = [
    'name' => 'Asha',
    'roles' => ['author', 'editor'],
];

foreach ($person['roles'] as $role) {
    echo $role, PHP_EOL;
}
~~~

Strings are byte sequences. Use the appropriate encoding-aware functions when handling Unicode text. Use formatted output or string interpolation for readable messages, and avoid assuming that a byte offset is always a character boundary.

Use the standard array functions for searching, mapping, filtering, sorting, and combining arrays. Be mindful of whether a function preserves keys, changes the original array, or returns a new value. Regular expressions use PCRE functions for pattern matching.

Date and time work should use the date/time classes where practical, with an explicit timezone for application logic. JSON is a common interchange format. Use `json_encode()` and `json_decode()` with error handling, and validate decoded data before trusting it. Avoid unserializing untrusted data.

Filesystem functions, streams, and wrappers provide file and network access. Check return values, handle failures, and construct paths safely. Never let untrusted input choose arbitrary filesystem paths or stream wrappers.

## Objects and modern PHP

A class defines behavior and data for objects. Objects can have properties and methods. Visibility controls access: `public`, `protected`, and `private`. Constructors initialize objects, while interfaces define contracts that classes can implement.

~~~php
interface Notifier
{
    public function send(string $message): void;
}

final class EmailNotifier implements Notifier
{
    public function __construct(
        private readonly string $address,
    ) {}

    public function send(string $message): void
    {
        // Send the message through an email service.
    }
}
~~~

Use composition to build behavior from collaborating objects. Use inheritance when a real subtype relationship exists. Abstract classes can share implementation while requiring subclasses to complete defined behavior. Traits provide code reuse, but they do not replace a clear object design.

Modern PHP includes typed and readonly properties, constructor property promotion, enums, attributes, anonymous classes, and property hooks. Namespaces organize names and reduce collisions. Composer's PSR-4 autoloader maps namespaces to files. Reflection can inspect classes, methods, attributes, and types at runtime.

Static properties and methods belong to a class rather than an individual object. Magic methods such as `__construct()`, `__toString()`, and `__invoke()` are called by language behavior. Use them intentionally because they can hide control flow.

Fibers support cooperative concurrency primitives. Generators provide lazy iteration. These features do not make ordinary PHP code automatically asynchronous. Application behavior still depends on the runtime, server, libraries, and how work is scheduled.

## Errors and debugging

PHP has errors and exceptions. Both implement `Throwable`, but they represent different categories of failure. Exceptions can be thrown and caught. Do not silently swallow errors that should be reported or handled.

~~~php
try {
    $result = performOperation();
} catch (InvalidArgumentException $exception) {
    error_log($exception->getMessage());
    throw $exception;
}
~~~

Use exceptions for exceptional failures, and return clear values for ordinary expected outcomes. Catch only failures the current layer can handle. In production, show visitors a generic error message and log details privately. Avoid exposing stack traces, secrets, SQL, or filesystem paths.

Use a debugger and logs to find the source of a problem. Validate assumptions with narrow checks. Avoid logging passwords, tokens, session identifiers, or unnecessary personal data. Assertions are for development assumptions, not input validation or authorization.

## Web requests and responses

A PHP web application receives an HTTP request and produces a response. A request has a method, path, headers, cookies, and possibly query or body data. The server exposes request information through superglobals such as `$_GET`, `$_POST`, `$_COOKIE`, `$_FILES`, and `$_SERVER`.

Treat every request value as untrusted, even when it came from a form or a logged-in user. Validate its shape, type, length, and allowed values before using it. Server-side validation is required even when the browser also validates a form.

Output must be encoded for its destination. For HTML text and quoted attributes, a helper can use `htmlspecialchars()` with an explicit UTF-8 encoding:

~~~php
function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8',
    );
}
~~~

HTML escaping does not replace JavaScript, CSS, URL, or SQL protections. Use context-appropriate encoding, prepared SQL statements, and safe APIs.

Headers and status codes must be sent before response body output. After a redirect, stop the current request with `exit`. Set response content types deliberately. Avoid returning secrets or internal exception details in responses.

Cookies are browser-held values. Configure security attributes such as Secure, HttpOnly, and SameSite when appropriate. Sessions store server-side state associated with a session identifier. Start sessions before output, regenerate the session identifier after authentication or privilege changes, and protect state-changing requests against CSRF.

For authentication, store password hashes created with `password_hash()` and verify them with `password_verify()`. Do not store or compare plaintext passwords. Authorization checks should be made on the server for every protected action.

## Databases and APIs

Use a database abstraction to keep connection and query behavior explicit. PDO provides a consistent API for multiple database drivers. Handle connection and query failures, and use transactions when several changes must succeed or fail together.

~~~php
$statement = $pdo->prepare(
    'SELECT id, display_name FROM users WHERE email = :email'
);
$statement->execute(['email' => $email]);
$user = $statement->fetch();
~~~

Prepared statements separate SQL structure from data values and help prevent SQL injection. They do not automatically make dynamic table names, column names, or sort directions safe. Whitelist any SQL identifiers that must vary.

Use a schema that represents the application's data and relationships. Add database constraints, indexes, and migrations deliberately. Paginate large result sets. Avoid one query per row when a single appropriate query can fetch the needed data.

JSON APIs should define request and response shapes, status codes, validation rules, and error formats. Encode JSON with error handling. For outgoing HTTP requests, set timeouts, inspect response status, and handle network failures.

## Composer and project structure

Composer manages PHP packages and autoloading. The `composer.json` file declares project metadata and dependency constraints. The lockfile records resolved package versions and should generally be committed for application projects.

~~~sh
composer install
composer require vendor/package
composer check-platform-reqs
~~~

Use `composer install` to install the versions recorded in the lockfile. Use `composer update` when intentionally resolving newer versions and reviewing the resulting changes. Set the project's minimum PHP version accurately. Do not assume a package supports every PHP release.

A clear project can separate public entry points, application code, configuration, templates, tests, and generated or private files. Keep secrets out of source control. Keep business rules out of request parsing and HTML templates where possible. Use interfaces and dependency injection when they make code easier to change or test, not as ceremony.

Frameworks can provide routing, middleware, templates, validation, database access, and security features. Learn the PHP behavior underneath them, then follow the framework's conventions instead of rebuilding its core features in application code.

## Security essentials

- Validate and normalize user input on the server.
- Escape output for its exact context to prevent cross-site scripting.
- Use prepared statements for SQL values and allowlists for dynamic SQL identifiers.
- Protect state-changing actions against CSRF.
- Use `password_hash()` and `password_verify()` for passwords.
- Check authorization on every protected action. Authentication alone is not permission.
- Secure sessions and cookies. Regenerate session IDs after login or privilege changes.
- Validate uploaded file size, content, and storage location. Use generated filenames and keep private uploads outside the public document root.
- Prevent path traversal, command injection, unsafe redirects, and server-side request forgery.
- Avoid deserializing untrusted data and avoid executing user-controlled code.
- Keep credentials in environment or secret-management configuration, not in committed files.
- Disable detailed error display in production. Keep logs private and avoid logging secrets.
- Keep PHP, extensions, and Composer dependencies updated. Review dependency changes and run security checks.
- Use HTTPS in production and configure the web server and PHP runtime safely.

Security is a property of the whole application. A safe PHP function cannot compensate for incorrect authorization, unsafe architecture, or exposed secrets elsewhere.

## Testing, performance, and deployment

Test behavior at more than one level. Unit tests cover small pieces of logic. Integration tests exercise boundaries such as a database or HTTP endpoint. End-to-end tests verify a user workflow. Tests should cover successful cases, invalid input, permission failures, and important edge cases.

Use a formatter, linter, static analyzer, and automated tests to catch mistakes early. Keep tests repeatable. Avoid relying on production services or real user data in test runs.

Measure performance before optimizing. Profile slow requests, inspect database query counts, reduce repeated work, and consider caching where stale data is acceptable. OPcache improves PHP bytecode reuse. PHP-FPM manages PHP worker processes for common web-server deployments. Memory and concurrency behavior depend on the selected runtime and architecture.

Deploy with explicit configuration for the target environment. Keep development diagnostics out of production. Verify the PHP version, required extensions, file permissions, environment variables, database connectivity, and web-server routing. Plan database changes so application versions can deploy safely.

PHP 8.6 introduces partial function application and new runtime APIs. New features are version-dependent, so check compatibility before using them in a project that targets older PHP releases.

## Problem solving and application design

A practical workflow is:

1. Clarify the expected input, output, and failure cases.
2. Separate trusted internal data from untrusted external input.
3. Choose data structures and functions that express the domain clearly.
4. Validate at boundaries and keep core logic independent of HTTP details.
5. Use the right persistence and encoding mechanisms for each destination.
6. Handle expected failures and log unexpected failures without exposing internals.
7. Test success paths, edge cases, invalid requests, and permission checks.
8. Review security, maintainability, and compatibility before deployment.

Prefer simple code with clear ownership. Keep functions and classes focused. Name values after their meaning. Avoid hidden global state and unnecessary cleverness. Document decisions that are difficult to infer from the code.

## Everyday coding checklist

- Does the code run on the PHP version declared by the project?
- Are parameters, return values, and important properties typed?
- Is external input validated on the server?
- Is output encoded for its exact context?
- Are database values passed through prepared statements?
- Are authorization and CSRF protections applied where needed?
- Are errors handled and logged without leaking private details?
- Are secrets and uploaded files stored safely?
- Are dependencies, tests, formatting, and static analysis up to date?
- Is the code clear enough for another developer to maintain?

## Course map

- Foundations: [introduction](lessons/001-introduction-to-php.md), [runtime setup](lessons/004-install-php-and-inspect-the-runtime.md), [syntax](lessons/008-statements-instruction-separation-and-comments.md), [variables](lessons/010-variables-and-assignment.md), and [types](lessons/013-the-php-type-system-and-value-model.md).
- Language fundamentals: [operators](lessons/024-operator-precedence-and-associativity.md), [control flow](lessons/030-conditional-branching-with-if-and-switch.md), [functions](lessons/037-defining-and-calling-functions.md), [arrays](lessons/049-working-with-arrays.md), and [errors](lessons/057-errors-exceptions-and-throwable.md).
- Object-oriented PHP: [classes](lessons/062-classes-objects-and-object-oriented-php.md), [interfaces](lessons/070-interfaces-and-contracts.md), [enums](lessons/079-enumerations-and-backed-enums.md), [namespaces](lessons/080-namespaces-imports-and-name-resolution.md), and [attributes](lessons/082-attributes-and-metadata.md).
- Web development: [request lifecycle](lessons/084-http-requests-responses-and-the-php-request-lifecycle.md), [forms](lessons/087-html-forms-and-get-or-post-requests.md), [sessions](lessons/091-sessions-and-server-side-state.md), and [JSON APIs](lessons/097-building-json-and-rest-apis.md).
- Data and packages: [PDO](lessons/099-connecting-to-databases-with-pdo.md), [prepared statements](lessons/100-prepared-statements-and-safe-database-queries.md), [Composer](lessons/107-composer-and-dependency-management.md), and [autoloading](lessons/108-composer-autoloading-and-psr-4.md).
- Quality and security: [testing](lessons/117-testing-php-applications-with-phpunit.md), [static analysis](lessons/121-static-analysis-with-phpstan-and-psalm.md), [secure coding](lessons/123-secure-coding-and-threat-modeling.md), and [deployment](lessons/141-deployment-environments-and-release-workflows.md).
- Practice: [validated form](lessons/149-project-create-a-validated-contact-form.md), [CRUD application](lessons/150-project-build-a-database-backed-crud-application.md), [JSON API](lessons/151-project-build-a-json-api.md), and the [capstone](lessons/152-capstone-project-plan-build-test-and-document-an-application.md).

## Continue learning

- [PHP keywords](php-keywords.md)
- [PHP quick reference and glossary](lessons/153-php-quick-reference-and-glossary.md)
- [All lessons, Q&A, and examples](README.md#lessons)
