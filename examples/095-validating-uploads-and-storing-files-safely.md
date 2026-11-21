# 095 - Validating uploads and storing files safely: Example

## Goal

Protect a demo avatar-upload endpoint. It requires a signed-in demo user, checks a CSRF token, restricts uploads to JPEG and PNG, compares extension and server-detected type, bounds image dimensions, decodes the image, and writes a normalized PNG under a generated name outside the public web root.

## Question

How can a PHP application accept an avatar safely without trusting the submitted filename, storing raw active content, or exposing private files directly?

## Complete example: `public/avatar-demo.php`

Copy this file to `public/avatar-demo.php` in a small project. Create a writable `storage/avatars/` directory beside `public/` before running it.

~~~php
<?php

declare(strict_types=1);

if (
    ini_set('session.use_strict_mode', '1') === false
    || ini_set('session.use_only_cookies', '1') === false
) {
    http_response_code(500);
    header('Content-Type: text/plain; charset=UTF-8');
    exit("Could not apply the session settings.\n");
}

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

function avatarUploadError(int $error): string
{
    switch ($error) {
        case UPLOAD_ERR_INI_SIZE:
        case UPLOAD_ERR_FORM_SIZE:
            return 'The image exceeds a server upload limit.';
        case UPLOAD_ERR_PARTIAL:
            return 'The image was only partially uploaded. Try again.';
        case UPLOAD_ERR_NO_FILE:
            return 'Choose an image before submitting the form.';
        case UPLOAD_ERR_NO_TMP_DIR:
        case UPLOAD_ERR_CANT_WRITE:
        case UPLOAD_ERR_EXTENSION:
            return 'The server could not receive the image.';
        default:
            return 'The upload could not be processed.';
    }
}

// Public tutorial fixture. Replace this whole login fixture with the application's
// real authentication and authorization middleware in a production route.
$demoUser = [
    'id' => 7,
    'email' => 'editor@example.test',
    // This public sample hash matches the local-only password: rasmuslerdorf.
    'password_hash' => '$2y$12$4Umg0rCJwMswRw/l.SwHvuQV01coP0eWmGzd61QH2RvAOMANUBGC.',
];

if (!isset($_SESSION['csrf_token']) || !is_string($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
$csrfToken = $_SESSION['csrf_token'];

$sessionUserId = $_SESSION['user_id'] ?? null;
$currentUser = $sessionUserId === $demoUser['id'] ? $demoUser : null;
if ($currentUser === null) {
    unset($_SESSION['user_id']);
}

$maximumBytes = 2 * 1024 * 1024; // 2 MiB.
$maximumDimension = 3000;
$maximumPixels = 8_000_000;
$projectRoot = dirname(__DIR__);
$uploadDirectory = $projectRoot
    . DIRECTORY_SEPARATOR . 'storage'
    . DIRECTORY_SEPARATOR . 'avatars';

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
    $action = $_POST['action'] ?? null;

    if (
        !is_string($submittedToken)
        || !hash_equals($csrfToken, $submittedToken)
    ) {
        $responseStatus = 403;
        $message = 'The form could not be verified. Reload the page and try again.';
        $messageType = 'error';
    } elseif (!is_string($action)) {
        $responseStatus = 400;
        $message = 'The request is invalid.';
        $messageType = 'error';
    } elseif ($action === 'login') {
        $emailInput = $_POST['email'] ?? null;
        $passwordInput = $_POST['password'] ?? null;
        $emailMatches = is_string($emailInput)
            && strtolower(trim($emailInput)) === $demoUser['email'];

        // Verify a password hash even if the submitted email does not match.
        $passwordIsValid = is_string($passwordInput)
            && password_verify($passwordInput, $demoUser['password_hash'])
            && $emailMatches;

        if (!$passwordIsValid) {
            $responseStatus = 401;
            $message = 'Invalid email or password.';
            $messageType = 'error';
        } elseif (!session_regenerate_id()) {
            $responseStatus = 500;
            $message = 'The sign-in could not be completed.';
            $messageType = 'error';
        } else {
            $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
            $_SESSION['user_id'] = $demoUser['id'];
            $_SESSION['authenticated_at'] = time();
            $_SESSION['flash_message'] = 'Signed in. You can now upload an avatar.';
            header('Location: /avatar-demo.php', true, 303);
            exit;
        }
    } elseif ($action === 'logout') {
        $_SESSION = [];
        $cookie = session_get_cookie_params();
        setcookie(session_name(), '', [
            'expires' => time() - 42000,
            'path' => $cookie['path'],
            'domain' => $cookie['domain'],
            'secure' => $cookie['secure'],
            'httponly' => $cookie['httponly'],
            'samesite' => $cookie['samesite'] ?? 'Lax',
        ]);
        session_destroy();

        header('Location: /avatar-demo.php', true, 303);
        exit;
    } elseif ($action === 'upload') {
        if ($currentUser === null) {
            $responseStatus = 401;
            $message = 'Sign in before uploading an avatar.';
            $messageType = 'error';
        } else {
            $file = $_FILES['avatar'] ?? null;

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
                $message = avatarUploadError($file['error']);
                $messageType = 'error';
            } elseif (
                !isset($file['name'], $file['tmp_name'], $file['size'])
                || !is_string($file['name'])
                || !is_string($file['tmp_name'])
                || !is_int($file['size'])
                || !is_uploaded_file($file['tmp_name'])
            ) {
                $responseStatus = 400;
                $message = 'The uploaded image could not be verified.';
                $messageType = 'error';
            } else {
                $actualSize = filesize($file['tmp_name']);

                if ($actualSize === false || $actualSize < 1) {
                    $responseStatus = 400;
                    $message = 'The image is empty or could not be read.';
                    $messageType = 'error';
                } elseif ($actualSize > $maximumBytes) {
                    $responseStatus = 413;
                    $message = 'Choose an image no larger than 2 MiB.';
                    $messageType = 'error';
                } elseif (!is_dir($uploadDirectory) || !is_writable($uploadDirectory)) {
                    error_log('Avatar demo storage directory is missing or not writable.');
                    $responseStatus = 500;
                    $message = 'The server storage is not ready for images.';
                    $messageType = 'error';
                } elseif (!class_exists('finfo') || !function_exists('imagecreatefromstring')) {
                    error_log('Avatar demo requires the Fileinfo and GD extensions.');
                    $responseStatus = 500;
                    $message = 'The server is missing an image-processing component.';
                    $messageType = 'error';
                } else {
                    $allowedExtensions = [
                        'jpg' => 'image/jpeg',
                        'jpeg' => 'image/jpeg',
                        'png' => 'image/png',
                    ];
                    $extension = strtolower(pathinfo($file['name'], PATHINFO_EXTENSION));
                    $expectedMime = $allowedExtensions[$extension] ?? null;

                    $fileInfo = new finfo(FILEINFO_MIME_TYPE);
                    $detectedMime = $fileInfo->file($file['tmp_name']);

                    if ($expectedMime === null || $detectedMime !== $expectedMime) {
                        $responseStatus = 415;
                        $message = 'Choose a JPEG or PNG image with a matching file extension.';
                        $messageType = 'error';
                    } else {
                        // Use getimagesize only for a preliminary dimension limit.
                        // It is not the format validator; the GD decoder below must also succeed.
                        $imageInfo = @getimagesize($file['tmp_name']);

                        $expectedImageType = $detectedMime === 'image/jpeg'
                            ? IMAGETYPE_JPEG
                            : IMAGETYPE_PNG;

                        if (
                            !is_array($imageInfo)
                            || !isset($imageInfo[0], $imageInfo[1], $imageInfo[2])
                            || !is_int($imageInfo[0])
                            || !is_int($imageInfo[1])
                            || $imageInfo[2] !== $expectedImageType
                        ) {
                            $responseStatus = 415;
                            $message = 'The file is not a supported image.';
                            $messageType = 'error';
                        } elseif (
                            $imageInfo[0] < 1
                            || $imageInfo[1] < 1
                            || $imageInfo[0] > $maximumDimension
                            || $imageInfo[1] > $maximumDimension
                            || $imageInfo[0] * $imageInfo[1] > $maximumPixels
                        ) {
                            $responseStatus = 413;
                            $message = 'The image dimensions exceed this demo limit.';
                            $messageType = 'error';
                        } else {
                            $imageBytes = file_get_contents($file['tmp_name']);
                            $image = is_string($imageBytes)
                                ? @imagecreatefromstring($imageBytes)
                                : false;

                            if ($image === false) {
                                $responseStatus = 415;
                                $message = 'The image could not be decoded.';
                                $messageType = 'error';
                            } elseif (
                                imagesx($image) !== $imageInfo[0]
                                || imagesy($image) !== $imageInfo[1]
                            ) {
                                imagedestroy($image);
                                $responseStatus = 415;
                                $message = 'The image dimensions could not be verified.';
                                $messageType = 'error';
                            } else {
                                // Always write normalized PNG output; never keep the original bytes.
                                $storedName = bin2hex(random_bytes(32)) . '.png';
                                $destination = $uploadDirectory . DIRECTORY_SEPARATOR . $storedName;
                                $stream = @fopen($destination, 'xb');

                                if ($stream === false) {
                                    imagedestroy($image);
                                    error_log('Avatar demo could not create a generated image file.');
                                    $responseStatus = 500;
                                    $message = 'The server could not store the image.';
                                    $messageType = 'error';
                                } else {
                                    $writeSucceeded = @imagepng($image, $stream, 6);
                                    $closeSucceeded = fclose($stream);
                                    imagedestroy($image);

                                    if (!$writeSucceeded || !$closeSucceeded) {
                                        @unlink($destination);
                                        error_log('Avatar demo could not finish writing a normalized image.');
                                        $responseStatus = 500;
                                        $message = 'The server could not store the image.';
                                        $messageType = 'error';
                                    } else {
                                        $_SESSION['flash_message'] = 'Image validated and re-encoded. Stored image ID: ' . $storedName;
                                        header('Location: /avatar-demo.php', true, 303);
                                        exit;
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    } else {
        $responseStatus = 400;
        $message = 'The requested action is not available.';
        $messageType = 'error';
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
    <title>Secure avatar upload demo</title>
</head>
<body>
    <main>
        <h1>Secure avatar upload demo</h1>

        <?php if (is_string($message)): ?>
            <p role="<?= $messageType === 'error' ? 'alert' : 'status' ?>">
                <?= escapeHtml($message) ?>
            </p>
        <?php endif; ?>

        <?php if ($currentUser === null): ?>
            <h2>Sign in to continue</h2>
            <form action="/avatar-demo.php" method="post">
                <input type="hidden" name="_csrf" value="<?= escapeHtml($csrfToken) ?>">
                <input type="hidden" name="action" value="login">

                <label for="email">Email</label>
                <input id="email" name="email" type="email" autocomplete="username" required>

                <label for="password">Password</label>
                <input id="password" name="password" type="password" autocomplete="current-password" required>

                <button type="submit">Sign in</button>
            </form>
            <p>Local demo credentials: <code>editor@example.test</code> / <code>rasmuslerdorf</code></p>
            <p>This public sample account is for local learning only.</p>
        <?php else: ?>
            <p>Signed in as <strong><?= escapeHtml($currentUser['email']) ?></strong>.</p>

            <h2>Upload a JPEG or PNG avatar</h2>
            <p>Maximum file size: 2 MiB. Maximum dimensions: 3000 × 3000 pixels and 8 million pixels total.</p>

            <form action="/avatar-demo.php" method="post" enctype="multipart/form-data">
                <input type="hidden" name="_csrf" value="<?= escapeHtml($csrfToken) ?>">
                <input type="hidden" name="action" value="upload">

                <label for="avatar">Avatar image</label>
                <input
                    id="avatar"
                    name="avatar"
                    type="file"
                    accept=".jpg,.jpeg,.png,image/jpeg,image/png"
                    required
                >

                <button type="submit">Validate and store avatar</button>
            </form>

            <form action="/avatar-demo.php" method="post">
                <input type="hidden" name="_csrf" value="<?= escapeHtml($csrfToken) ?>">
                <input type="hidden" name="action" value="logout">
                <button type="submit">Sign out</button>
            </form>
        <?php endif; ?>

        <p>Stored images stay outside the public web root and this demo does not serve them directly.</p>
    </main>
</body>
</html>
~~~

## Project layout

The web server's document root is `public/`. The image directory is a sibling outside that root.

~~~text
project/
├── public/
│   └── avatar-demo.php
└── storage/
    └── avatars/
~~~

In PowerShell, create the storage directory from the project root:

~~~powershell
New-Item -ItemType Directory -Force storage/avatars
~~~

Make sure the account running PHP can write to `storage/avatars/`. Do not expose this directory as a public URL or allow the web server to execute its files.

## Run it

From the project root, start PHP's built-in development server with `public` as the document root:

~~~bash
php -S localhost:8000 -t public
~~~

Open:

~~~text
http://localhost:8000/avatar-demo.php
~~~

Sign in with the local demo credentials shown on the page. Upload a small JPEG or PNG. The page redirects after success and displays a generated image ID. The normalized PNG is written under `storage/avatars/`; the original filename and original bytes are not used as the stored file.

## Expected behavior

| Action | Expected result |
| --- | --- |
| Open the page | A demo sign-in form appears |
| Sign in with the displayed demo account | Session ID rotates and the avatar upload form appears |
| Upload a small valid JPEG or PNG with a matching extension | Status `303` redirect; decoded image is re-encoded to a generated `.png` file outside the web root |
| Rename a text file to `photo.jpg` | Rejected with status `415` because the detected content does not match |
| Submit an unsupported extension | Rejected with status `415` |
| Upload a file above 2 MiB | Rejected with status `413`, unless PHP or the web server rejects the request first |
| Upload an image beyond the dimension or pixel limits | Rejected with status `413` before full decoding |
| Submit without a matching CSRF token | Rejected with status `403` |
| Open the generated storage path in a browser | No public URL is provided by this demo |

## How it works

1. **Demo authentication:** The local fixture uses a password hash and regenerates the session ID after sign-in. Replace it with the application's real login and permission checks.
2. **CSRF and method checks:** Login, upload, and logout are POST actions with a session token checked using `hash_equals()`. Other HTTP methods are rejected.
3. **Upload structure and limits:** The code validates the upload array and PHP error code, checks that the temporary file is a genuine upload, and enforces a 2 MiB server-side byte limit.
4. **Allowlist and MIME comparison:** Only JPEG and PNG extensions are allowed. Fileinfo checks the temporary file's detected MIME type; the browser's claimed type is not used.
5. **Dimension precheck:** `getimagesize()` is used only to reject excessive dimensions before decoding. It is not treated as proof that the file is a valid image.
6. **Actual decode:** GD must decode the full image, and decoded dimensions must match the preliminary values. Width, height, and total pixels are bounded.
7. **Normalization:** The decoded image is always written as PNG to a new, exclusive, randomly named file. The original upload bytes and client filename are never stored.
8. **Private storage:** The output directory sits outside `public/`. The demo does not create a public download link.
9. **Safe errors:** The browser receives generic messages; operational failures are logged without showing internal paths.

## Important notes

- The demo password and hash are public tutorial fixtures. Never deploy them or reuse the password.
- The example requires PHP's Fileinfo and GD extensions. Image decoders process untrusted data; keep them updated and use appropriate memory and execution limits.
- The upload endpoint is protected by a tutorial login and CSRF token, but it does not persist an avatar-to-user database record. A real application must authorize each upload for the correct account and save a trusted storage key and metadata transactionally.
- Re-encoding reduces some risks but does not guarantee harmless output. Apply production quotas, rate limits, cleanup, monitoring, and a suitable processing boundary.
- If users need to download or view images, create a separate handler that looks up a database record, checks access on every request, and sends safe response headers.
- PHP and the web server must allow the request size. If the whole request exceeds `post_max_size`, PHP may leave `$_POST` and `$_FILES` empty.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [094 - Handling file uploads](../lessons/094-handling-file-uploads.md) | [Q&A](../qnas/095-validating-uploads-and-storing-files-safely.md) | Example | [096 - Making HTTP requests with streams and cURL](../lessons/096-making-http-requests-with-streams-and-curl.md) |