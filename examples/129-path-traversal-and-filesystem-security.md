# 129 - Path traversal and filesystem security: Example

## Question

Build a PHP download route that accepts a document ID, not a filesystem path. It must require an authenticated user, allow only explicitly permitted documents, keep files outside the public web root, and reject a symbolic link or resolved path that escapes the private storage directory.

## Complete example

Arrange the application like this:

~~~text
project/
├── private-files/
│   ├── welcome.txt
│   └── rules.txt
└── public/
    └── download.php
~~~

Put the following text in the two private files:

~~~text
private-files/welcome.txt
Welcome, authorized reader.
~~~

~~~text
private-files/rules.txt
Private project rules.
~~~

Save this as `public/download.php`. Serve the site through HTTPS. The login handler from the previous chapters must establish the session after authenticating the user and store the integer user ID in `$_SESSION['user_id']`.

~~~php
<?php

declare(strict_types=1);

ini_set('session.use_strict_mode', '1');
ini_set('session.use_only_cookies', '1');

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();

$userId = $_SESSION['user_id'] ?? null;

if (!is_int($userId)) {
    http_response_code(401);
    exit('Sign in is required.');
}

$documents = [
    'welcome' => [
        'filename' => 'welcome.txt',
        'allowed_user_ids' => [42],
    ],
    'rules' => [
        'filename' => 'rules.txt',
        'allowed_user_ids' => [42, 84],
    ],
];

$documentId = $_GET['id'] ?? null;

if (!is_string($documentId) || !isset($documents[$documentId])) {
    http_response_code(404);
    exit('Document not found.');
}

$document = $documents[$documentId];

if (!in_array($userId, $document['allowed_user_ids'], true)) {
    http_response_code(404);
    exit('Document not found.');
}

function resolvePrivateFile(
    string $baseDirectory,
    string $filename
): ?string {
    $base = realpath($baseDirectory);

    if ($base === false || !is_dir($base)) {
        return null;
    }

    $candidate = realpath(
        $base . DIRECTORY_SEPARATOR . $filename
    );

    if ($candidate === false || !is_file($candidate)) {
        return null;
    }

    $basePrefix = rtrim($base, DIRECTORY_SEPARATOR)
        . DIRECTORY_SEPARATOR;

    $baseForComparison = $basePrefix;
    $candidateForComparison = $candidate;

    if (DIRECTORY_SEPARATOR === chr(92)) {
        $baseForComparison = strtolower($baseForComparison);
        $candidateForComparison = strtolower($candidateForComparison);
    }

    if (!str_starts_with($candidateForComparison, $baseForComparison)) {
        return null;
    }

    return $candidate;
}

$privateDirectory = dirname(__DIR__)
    . DIRECTORY_SEPARATOR
    . 'private-files';

$filePath = resolvePrivateFile(
    $privateDirectory,
    $document['filename']
);

if ($filePath === null) {
    http_response_code(404);
    exit('Document not found.');
}

$content = @file_get_contents($filePath);

if ($content === false) {
    error_log('Failed to read approved document ID: ' . $documentId);
    http_response_code(500);
    exit('The document could not be read.');
}

header('Cache-Control: private, no-store');
header('Content-Type: text/plain; charset=UTF-8');
header('X-Content-Type-Options: nosniff');

echo $content;
~~~

The allowlist and permission IDs are server-side demonstration data. In a real application, load document metadata and permissions from trusted storage. Do not accept filenames or user IDs from the request as authorization data.

## Expected output

Assume the signed-in user has ID 42 and requests `/download.php?id=welcome`. The browser displays:

~~~text
Welcome, authorized reader.
~~~

User 42 may also request `/download.php?id=rules` and sees:

~~~text
Private project rules.
~~~

User 84 may request the rules document, but is not allowed to request the welcome document. The denied response is HTTP 404:

~~~text
Document not found.
~~~

A traversal string such as `/download.php?id=../../private-files/rules.txt` is not a known document ID, so it receives the same HTTP 404 response. A missing or non-integer session user ID receives HTTP 401 before the file lookup.

## Explanation

### The request selects an ID, not a path

The route accepts only a scalar ID present in the server-side documents map. The map supplies the filename and the permitted user IDs. Array-shaped query parameters and unknown IDs fail before filesystem access.

In production, replace the fixture map with a database lookup. Check the current user's permission before resolving or reading the storage file. An ID that is difficult to guess is not a substitute for that authorization check.

### The private directory is outside the web root

The web server serves files from public/, while private-files/ is its sibling. A browser cannot fetch those text files through a static URL. The PHP route can read the file only after its authentication and authorization checks pass.

Keep the private directory writable only by trusted deployment or application processes. Avoid placing user-writable files in a directory where PHP scripts can be executed.

### The resolved target must remain inside the base directory

resolvePrivateFile() canonicalizes the base and existing candidate with realpath(). That resolves traversal components and follows a symbolic link. It then checks that the canonical candidate starts with the base directory plus a separator, preventing a similarly named sibling directory from passing a simple prefix check.

If an attacker can replace files or links between the realpath() check and file_get_contents(), a time-of-check/time-of-use race may still exist. Keep the directory tree and its parents protected from untrusted writes; use a platform or framework API that enforces containment during opening when available.

### The response does not reveal filesystem paths

Unknown IDs, unauthorized documents, missing files, and targets that escape the directory use a generic not-found response. The route does not print the resolved path or PHP warning to the user.

The selected file is returned as plain text with nosniff and a private no-store cache policy. If the application serves another file type, choose safe response headers and consider whether the format can contain active content.

## What this example demonstrates

- A user-facing document ID maps to a server-controlled filename.
- The route gets identity from the server-side session and applies a separate permission check.
- Private files live outside the public web root.
- realpath() resolves the existing target and the code enforces base-directory containment.
- A traversal attempt or unauthorized document does not expose contents or paths.
- Filesystem permissions and safe response headers complement application checks.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [128 - Session and cookie security](../lessons/128-session-and-cookie-security.md) | [Q&A](../qnas/129-path-traversal-and-filesystem-security.md) | [Lesson](../lessons/129-path-traversal-and-filesystem-security.md) | [130 - SSRF, URL validation, and redirect safety](../lessons/130-ssrf-url-validation-and-redirect-safety.md) |