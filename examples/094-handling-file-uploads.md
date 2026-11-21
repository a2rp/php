# 094 - Handling file uploads: Example

## Goal

Build a small PHP form that accepts one plain-text file. The handler checks the request, CSRF token, PHP upload result, size, and server-detected content type, then moves the file to a generated name outside the public web root.

## Question

How can a PHP page receive a file through a browser form, handle upload errors, and keep the stored file separate from public executable content?

## Complete example: `public/upload-demo.php`

Copy this file to `public/upload-demo.php` in a small project. Create a writable `storage/uploads/` directory beside `public/` before running it.

~~~php
<?php

declare(strict_types=1);

// Set session cookie properties before starting the session.
$isHttps = isset($_SERVER['HTTPS'])
    && $_SERVER['HTTPS'] !== ''
    && $_SERVER['HTTPS'] !== 'off';

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => $isHttps,
    'httponly' => true,
    'samesite' => 'Lax',
]);

if (!session_start()) {
    http_response_code(500);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Could not start the session.\n");
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}

function uploadErrorMessage(int $error): string
{
    switch ($error) {
        case UPLOAD_ERR_INI_SIZE:
        case UPLOAD_ERR_FORM_SIZE:
            return 'The selected file is too large.';
        case UPLOAD_ERR_PARTIAL:
            return 'The file was only partially uploaded. Please try again.';
        case UPLOAD_ERR_NO_FILE:
            return 'Choose a text file before submitting the form.';
        case UPLOAD_ERR_NO_TMP_DIR:
        case UPLOAD_ERR_CANT_WRITE:
        case UPLOAD_ERR_EXTENSION:
            return 'The server could not complete the upload.';
        default:
            return 'The upload could not be processed.';
    }
}

if (!isset($_SESSION['csrf_token']) || !is_string($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$csrfToken = $_SESSION['csrf_token'];

$maximumBytes = 1024 * 1024; // 1 MiB application limit.
$projectRoot = dirname(__DIR__);
$uploadDirectory = $projectRoot
    . DIRECTORY_SEPARATOR . 'storage'
    . DIRECTORY_SEPARATOR . 'uploads';

$message = $_SESSION['flash_message'] ?? null;
unset($_SESSION['flash_message']);
$messageType = 'status';
$responseStatus = 200;

$method = $_SERVER['REQUEST_METHOD'] ?? 'GET';
if (!in_array($method, ['GET', 'POST'], true)) {
    http_response_code(405);
    header('Allow: GET, POST');
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Method not allowed.\n");
}

if ($method === 'POST') {
    $submittedToken = $_POST['_csrf'] ?? null;

    if (
        !is_string($submittedToken)
        || !hash_equals($csrfToken, $submittedToken)
    ) {
        $responseStatus = 403;
        $message = 'The form could not be verified. Reload the page and try again.';
        $messageType = 'error';
    } else {
        $file = $_FILES['document'] ?? null;

        if (!is_array($file) || !isset($file['error']) || !is_int($file['error'])) {
            $responseStatus = 400;
            $message = 'The upload request is invalid.';
            $messageType = 'error';
        } elseif ($file['error'] !== UPLOAD_ERR_OK) {
            $responseStatus = match ($file['error']) {
                UPLOAD_ERR_INI_SIZE, UPLOAD_ERR_FORM_SIZE => 413,
                UPLOAD_ERR_NO_FILE, UPLOAD_ERR_PARTIAL => 400,
                default => 500,
            };
            $message = uploadErrorMessage($file['error']);
            $messageType = 'error';
        } elseif (
            !isset($file['tmp_name'], $file['size'])
            || !is_string($file['tmp_name'])
            || !is_int($file['size'])
            || !is_uploaded_file($file['tmp_name'])
        ) {
            $responseStatus = 400;
            $message = 'The uploaded file could not be verified.';
            $messageType = 'error';
        } else {
            $actualSize = filesize($file['tmp_name']);

            if ($actualSize === false || $actualSize < 1) {
                $responseStatus = 400;
                $message = 'The text file is empty or could not be read.';
                $messageType = 'error';
            } elseif ($actualSize > $maximumBytes) {
                $responseStatus = 413;
                $message = 'Choose a file no larger than 1 MiB.';
                $messageType = 'error';
            } elseif (!class_exists('finfo')) {
                $responseStatus = 500;
                $message = 'The server is missing the Fileinfo extension.';
                $messageType = 'error';
            } else {
                $fileInfo = new finfo(FILEINFO_MIME_TYPE);
                $detectedType = $fileInfo->file($file['tmp_name']);

                if ($detectedType !== 'text/plain') {
                    $responseStatus = 415;
                    $message = 'This demo accepts plain-text files only.';
                    $messageType = 'error';
                } elseif (!is_dir($uploadDirectory) || !is_writable($uploadDirectory)) {
                    error_log('Upload demo storage directory is missing or not writable.');
                    $responseStatus = 500;
                    $message = 'The server storage is not ready for uploads.';
                    $messageType = 'error';
                } else {
                    // Never use the client-supplied name as a destination path.
                    $storedName = bin2hex(random_bytes(32)) . '.txt';
                    $destination = $uploadDirectory . DIRECTORY_SEPARATOR . $storedName;

                    if (file_exists($destination) || !move_uploaded_file($file['tmp_name'], $destination)) {
                        error_log('Upload demo could not move the temporary upload.');
                        $responseStatus = 500;
                        $message = 'The server could not store the uploaded file.';
                        $messageType = 'error';
                    } else {
                        $_SESSION['flash_message'] = 'Upload received. Stored file ID: ' . $storedName;
                        header('Location: /upload-demo.php', true, 303);
                        exit;
                    }
                }
            }
        }
    }
}

http_response_code($responseStatus);
header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>PHP text upload demo</title>
</head>
<body>
    <main>
        <h1>Upload a plain-text file</h1>
        <p>This demo accepts one non-empty text file up to 1 MiB.</p>

        <?php if (is_string($message)): ?>
            <p role="<?= $messageType === 'error' ? 'alert' : 'status' ?>">
                <?= escapeHtml($message) ?>
            </p>
        <?php endif; ?>

        <form action="/upload-demo.php" method="post" enctype="multipart/form-data">
            <input type="hidden" name="_csrf" value="<?= escapeHtml($csrfToken) ?>">
            <input type="hidden" name="MAX_FILE_SIZE" value="<?= $maximumBytes ?>">

            <label for="document">Text file</label>
            <input
                id="document"
                name="document"
                type="file"
                accept=".txt,text/plain"
                required
            >

            <button type="submit">Upload</button>
        </form>

        <p>The file is stored outside the public web root and is not served by this demo.</p>
    </main>
</body>
</html>
~~~

## Project layout

The public directory is the document root. The storage directory is its sibling, so a direct request cannot address an uploaded file by URL when the web server is configured with `public/` as its document root.

~~~text
project/
├── public/
│   └── upload-demo.php
└── storage/
    └── uploads/
~~~

The `storage/uploads/` directory must already exist and be writable by the PHP process. Keep its permissions limited to the application account.

## Run it

From the project root, start PHP's built-in development server with `public` as the document root:

~~~bash
php -S localhost:8000 -t public
~~~

Open this page in a browser:

~~~text
http://localhost:8000/upload-demo.php
~~~

Choose a small, non-empty `.txt` file and submit it. The page redirects after a successful upload and displays a generated file ID. The file is stored in `storage/uploads/`; the original client filename is not used as a path and the demo does not provide a public download URL.

## Expected behavior

| Action | Expected result |
| --- | --- |
| Open the page | A CSRF-protected multipart upload form appears |
| Upload a small plain-text file | The page redirects with status `303` and displays the generated stored file ID |
| Upload a different detected type, such as a PNG image | The request is rejected with status `415` |
| Upload a file larger than 1 MiB | The application rejects it with status `413`, unless PHP or the web server rejects the request first |
| Submit without the matching CSRF token | The request is rejected with status `403` |
| Leave the file input empty | The browser's `required` field blocks normal submission; a direct request with no file receives a safe error |
| Inspect the destination path | The stored file is under `storage/uploads/`, outside the `public/` document root |

## How it works

1. **Multipart form:** The form uses `POST` and `multipart/form-data`. The `accept` attribute and hidden size field help the browser but are not trusted security checks.
2. **CSRF check:** The page creates a random session token and compares it with the submitted token using `hash_equals()`.
3. **Shape and error checks:** The handler verifies that the expected upload entry exists and that PHP reports `UPLOAD_ERR_OK` before using the temporary path.
4. **Size check:** It checks the actual temporary-file size against the server-side 1 MiB limit. The client cannot raise this application limit by editing the form.
5. **Content check:** Fileinfo detects the temporary file's MIME type. The demo accepts only `text/plain` and never trusts the browser's `type` field.
6. **Safe destination:** The handler creates a random name using `random_bytes()` and writes only to the pre-created directory outside the public document root.
7. **Success response:** After a successful move, it stores a short flash message and sends a `303` redirect, preventing a browser refresh from resubmitting the upload.
8. **Safe errors:** User-facing errors do not include temporary or destination paths. Server-side logs record only a brief operational message.

## Important notes

- This is a learning demo, not a complete production upload service. A public endpoint also needs authentication and authorization when files belong to users, request and storage quotas, monitoring, and deployment-specific limits.
- Fileinfo's MIME result is useful but not a guarantee that complex content is harmless. This demo stores text outside the web root and does not render or serve the uploaded file.
- Configure `upload_max_filesize`, `post_max_size`, proxy limits, and application limits together. If the whole request exceeds `post_max_size`, PHP may provide empty `$_POST` and `$_FILES` arrays.
- Keep error display disabled for visitors in production and log diagnostics on the server.
- For a real document or image feature, use a narrow format-specific validation policy and a controlled download handler. Chapter 095 covers deeper validation and storage safeguards.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [093 - Authentication and authorization](../lessons/093-authentication-and-authorization.md) | [Q&A](../qnas/094-handling-file-uploads.md) | Example | [095 - Validating uploads and storing files safely](../lessons/095-validating-uploads-and-storing-files-safely.md) |