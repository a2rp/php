# 129 - Path traversal and filesystem security

Path traversal happens when an application combines untrusted input with a filesystem path and an attacker can make the result point somewhere the application did not intend. A vulnerable file viewer might be changed from a permitted document name to a path that reads configuration, credentials, or application source files.

The safest design is to avoid accepting filesystem paths from users. Let a request select a known file by an application identifier, then map that identifier to a server-controlled path. When a path must be built, resolve it and verify containment before using it. Also restrict what the PHP process can read and write.

## Learning goals

By the end of this chapter, you should be able to:

- Recognize path traversal and unsafe dynamic file inclusion.
- Explain why string replacement, basename(), and extension checks are not complete defenses.
- Map user-facing IDs to server-controlled files.
- Canonicalize existing paths with realpath() and enforce directory containment.
- Handle symlinks, nonexistent files, platform differences, and race conditions.
- Store uploads and application data outside the public web root.
- Use filesystem permissions and open_basedir as defense in depth.
- Test normal, invalid, and adversarial paths.

## How path traversal happens

Consider a download route that accepts a filename:

~~~php
$filename = $_GET['file'] ?? '';
$contents = file_get_contents(__DIR__ . '/documents/' . $filename);
~~~

The application expects a name such as report.txt, but the request can contain path separators and parent-directory components. If the resulting path escapes the documents directory, the application may disclose a file the caller should not read.

Paths have more forms than a single ../ spelling. Systems may interpret forward slashes, backslashes, absolute paths, drive prefixes, encoded characters, repeated separators, or symbolic links. PHP may also understand stream-wrapper names for some filesystem APIs. A filter that searches for one exact substring can miss another representation.

Do not let an external request choose an arbitrary path for file_get_contents(), fopen(), include, require, unlink(), rename(), or similar functions. Dynamic include paths are especially dangerous because an unintended file can be executed as PHP rather than merely read.

## Prefer identifiers and an allowlist

The strongest ordinary design is to accept a logical identifier and map it to a fixed server-side path:

~~~php
$documents = [
    'welcome' => 'welcome.txt',
    'rules' => 'rules.txt',
];

$documentId = $_GET['id'] ?? null;

if (!is_string($documentId) || !array_key_exists($documentId, $documents)) {
    http_response_code(404);
    exit('Document not found.');
}

$filename = $documents[$documentId];
~~~

The request selects a known key. It does not provide a path or a filename. For user-owned records, store a server-generated storage name in the database and authorize the requesting user before resolving or reading the file.

Use identifiers that are distinct from storage paths. A public document number can map to an opaque server-generated filename without revealing directory layout. Never assume that an unguessable ID replaces the authorization check.

If users need to select from a fixed set of templates, use a strict allowlist that maps each accepted key to its intended resource. Do not transform a supplied path into a “safe-looking” path and then assume it is contained.

## Resolve existing paths and enforce containment

When a path must include a filename, build it from a fixed base directory, resolve the existing target, and verify that its canonical path remains within the canonical base:

~~~php
function resolveExistingFile(string $baseDirectory, string $filename): ?string
{
    $base = realpath($baseDirectory);

    if ($base === false || !is_dir($base)) {
        return null;
    }

    $candidate = realpath($base . DIRECTORY_SEPARATOR . $filename);

    if ($candidate === false || !is_file($candidate)) {
        return null;
    }

    $basePrefix = rtrim($base, DIRECTORY_SEPARATOR)
        . DIRECTORY_SEPARATOR;

    $baseForComparison = $basePrefix;
    $candidateForComparison = $candidate;

    if (DIRECTORY_SEPARATOR === '\') {
        $baseForComparison = strtolower($baseForComparison);
        $candidateForComparison = strtolower($candidateForComparison);
    }

    if (!str_starts_with($candidateForComparison, $baseForComparison)) {
        return null;
    }

    return $candidate;
}
~~~

This containment check includes a directory separator after the base. A plain prefix check would incorrectly treat a sibling such as /srv/private-files-old as if it were inside /srv/private-files.

realpath() resolves dot components and symbolic links for an existing path. It returns false when the target does not exist or cannot be resolved, so handle that result before using the value. The check above is for an existing regular file; do not use it as a recipe for validating a path that has not been created yet.

On Windows, filesystem comparisons are commonly case-insensitive and paths use drive and separator rules. The example normalizes case for its prefix comparison on Windows. For production code, test path behavior on the actual operating system and filesystem; network shares, junctions, and mount behavior can have additional details.

A canonicalization check is not sufficient if an attacker can change the directory tree between the check and the later file operation. This is a time-of-check/time-of-use race. Keep directories and their parent directories writable only by trusted processes, avoid attacker-controlled symlinks, and use platform or framework APIs that enforce containment during the open operation when available.

## Why common filters are not enough

- **Removing ../** can miss encoded, repeated, mixed-separator, or platform-specific forms. Replacement can also change the meaning of the remaining input.
- **Calling basename()** strips directory components but does not prove the result is an allowed file or that a symbolic link stays inside the intended directory.
- **Checking an extension** does not constrain the directory. A name can have the expected suffix while still referring to an unintended location.
- **Checking file_exists() before opening** does not authorize the file. It can also create a race if the path changes after the check.
- **Checking only a string prefix** can confuse sibling names unless the comparison ends at a path-component boundary.
- **URL-decoding twice** can turn data that passed one check into separators after a second decode. Decode once at the request boundary, then validate and use the same representation.
- **Using open_basedir alone** does not express which files a particular user may access and is not a complete security boundary.

Reject invalid identifiers rather than trying to repair them. Keep the path construction and the access-control decision separate: a file may be safely contained but still belong to another user.

## Do not use request data in include paths

Never select executable code directly from a request:

~~~php
// Unsafe: the request chooses what PHP file to include.
include $_GET['page'] . '.php';
~~~

Route requests through explicit application logic. If a fixed set of templates is needed, map approved keys to fixed template paths in server-side code. Do not allow user uploads or writable data directories to contain files that the web server executes as PHP.

## Keep private files outside the public web root

A file that is not meant to be public should not be reachable as a static URL. Store uploads, exports, backups, and generated private documents outside the web root. Serve an authorized download through an application handler that checks access, resolves a server-controlled path, and sends the response safely.

Use restrictive operating-system permissions. The PHP worker should be able to access only the files and directories it needs. Avoid making private directories world-readable or world-writable. A writable directory should not also be a place where executable PHP files are served.

Temporary files and backups need the same care. Do not leave database dumps, source archives, logs, or secrets in a publicly served directory.

## Handle uploads as untrusted files

For an uploaded file, PHP provides a temporary path in $_FILES. The original client filename and the submitted MIME type are untrusted metadata; do not use them as the server-side storage path or proof of file type.

A safer upload flow should:

1. Check the upload error code and impose a size limit.
2. Validate the file according to its intended use; do not trust the browser's Content-Type header.
3. Generate a server-side storage name, such as a random identifier.
4. Store the file outside the public web root when it is private or active content.
5. Use move_uploaded_file() to move a valid PHP upload from its temporary location.
6. Keep a database mapping from the authorized application record to the generated storage name.
7. Enforce authorization and safe download headers when serving the file.

For images or documents that will be rendered inline, consider whether the format can contain active content. Use a separate origin or force downloads where appropriate. Upload validation does not replace output encoding or malware handling required by the product.

## Filesystem limits are defense in depth

The PHP process should run as a low-privilege account and have only the filesystem permissions it requires. Keep secrets and private content outside the document root. Use deployment configuration, container isolation, operating-system permissions, and a carefully configured open_basedir restriction to limit accidental exposure.

open_basedir can reduce the files PHP is allowed to access, but PHP documents it as an additional safety net, not a comprehensive security boundary. It does not decide whether the current application user is authorized to read a particular document. Application-level path containment and authorization are still required.

PHP filesystem functions may support stream wrappers such as php://, phar://, or remote wrappers depending on the function, configuration, and installed extensions. Do not pass a user-controlled value that could select a wrapper or URL to a filesystem function. Settings such as allow_url_include do not prevent local path traversal and should not be treated as a substitute for validation.

## Return safe errors

Do not show raw filesystem paths, server directories, or PHP warnings to an untrusted caller. Return a generic response such as “Document not found” for missing or disallowed files if the application wants to avoid revealing their existence. Log operational failures on the server with suitable access controls, but do not log secrets or sensitive file contents.

Treat permissions as a separate decision. First establish that the user may request the document, then resolve its server-side storage location and access it. If the resource is not authorized, do not reveal whether the underlying file exists.

## Testing checklist

Test normal and denied behavior, including:

- A valid allowlisted document identifier.
- An unknown identifier and an identifier supplied as an array.
- Parent-directory sequences using slash and backslash forms.
- Encoded and repeated traversal sequences, including the effects of URL decoding.
- Absolute paths and platform-specific path forms.
- A symlink inside the storage directory that points outside it.
- A missing target, a directory instead of a regular file, and unreadable files.
- A request for another user's document.
- Attempts to use wrapper-like values such as php:// or phar://.
- A writable directory and whether it is also web-accessible or executable.
- Uploads with a misleading extension, MIME type, size, or original filename.

Confirm that invalid and unauthorized requests neither disclose file contents or paths nor modify or delete files.

## Common mistakes

- Concatenating a request parameter directly into a filesystem path.
- Dynamically including a PHP file selected by the caller.
- Assuming basename(), an extension check, or a ../ replacement is enough.
- Comparing a canonical target with a base using an unsafe string prefix.
- Forgetting symbolic links or a check-to-open race.
- Expecting realpath() to validate a file that does not exist yet.
- Using a client filename or MIME type as trusted upload metadata.
- Storing private files or backups under the public web root.
- Giving the PHP process broad filesystem permissions.
- Treating open_basedir or allow_url_include as the sole defense.
- Returning raw paths and filesystem errors to the user.

## Key takeaways

- Prefer an application identifier mapped to a server-controlled storage name.
- If a path must be constructed, resolve it and verify a path-component boundary inside a fixed base directory.
- Account for symlinks, missing targets, operating-system path rules, and race conditions.
- Keep private and uploaded files outside the public web root with least-privilege permissions.
- Authorize the user separately from validating the path.
- Treat open_basedir and other runtime restrictions as defense in depth, not as the primary fix.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [128 - Session and cookie security](128-session-and-cookie-security.md) | [Q&A](../qnas/129-path-traversal-and-filesystem-security.md) | [Example](../examples/129-path-traversal-and-filesystem-security.md) | [130 - SSRF, URL validation, and redirect safety](130-ssrf-url-validation-and-redirect-safety.md) |