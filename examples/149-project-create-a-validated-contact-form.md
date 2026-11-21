# Project: create a validated contact form — Example

## Goal

Build a contact page that validates submissions on the server, checks a CSRF token stored in the visitor's session, escapes values shown back in HTML, and sends a plain-text message through configured PHP mail transport.

The example requires PHP 8.0 or later and the mbstring extension. Actual delivery also requires a working mail transport configured for the environment.

## Project structure

~~~text
contact-form/
├── public/
│   ├── contact.php
│   └── assets/
│       └── contact.css
└── src/
    └── contact-form.php
~~~

Keep **src/** outside the public document root. Configure the web server to serve **public/**.

## 1. Request handling and validation

### File: src/contact-form.php

~~~php
<?php
declare(strict_types=1);

function startContactSession(): void
{
    if (session_status() === PHP_SESSION_ACTIVE) {
        return;
    }

    $isHttps = isset($_SERVER['HTTPS'])
        && $_SERVER['HTTPS'] !== ''
        && $_SERVER['HTTPS'] !== 'off';

    ini_set('session.use_strict_mode', '1');

    session_set_cookie_params([
        'lifetime' => 0,
        'path' => '/',
        'secure' => $isHttps,
        'httponly' => true,
        'samesite' => 'Lax',
    ]);

    session_start();
}

function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}

function readPostString(string $key): string
{
    $value = $_POST[$key] ?? null;

    if (!is_string($value)) {
        return '';
    }

    return trim($value);
}

/**
 * @param array{name: string, email: string, message: string} $values
 * @return array<string, string>
 */
function validateContactValues(array $values): array
{
    $errors = [];

    if ($values['name'] === '') {
        $errors['name'] = 'Enter your name.';
    } elseif (
        !mb_check_encoding($values['name'], 'UTF-8')
        || mb_strlen($values['name'], 'UTF-8') > 100
    ) {
        $errors['name'] = 'Name must be valid UTF-8 and no more than 100 characters.';
    }

    if ($values['email'] === '') {
        $errors['email'] = 'Enter your email address.';
    } elseif (
        strlen($values['email']) > 254
        || filter_var($values['email'], FILTER_VALIDATE_EMAIL) === false
    ) {
        $errors['email'] = 'Enter a valid email address.';
    }

    if ($values['message'] === '') {
        $errors['message'] = 'Enter a message.';
    } elseif (
        !mb_check_encoding($values['message'], 'UTF-8')
        || mb_strlen($values['message'], 'UTF-8') > 5000
    ) {
        $errors['message'] = 'Message must be valid UTF-8 and no more than 5000 characters.';
    }

    return $errors;
}

/**
 * Send a validated message through the configured PHP mail transport.
 *
 * @param array{name: string, email: string, message: string} $values
 */
function deliverContactMessage(array $values): bool
{
    $recipient = getenv('CONTACT_TO');
    $sender = getenv('CONTACT_FROM');

    if (
        !is_string($recipient)
        || !is_string($sender)
        || filter_var($recipient, FILTER_VALIDATE_EMAIL) === false
        || filter_var($sender, FILTER_VALIDATE_EMAIL) === false
    ) {
        return false;
    }

    $subject = 'New contact form message';
    $body = "Name: {$values['name']}\n"
        . "Email: {$values['email']}\n\n"
        . $values['message'];

    // These headers use server configuration, never submitted form values.
    $headers = "From: {$sender}\r\n"
        . "Content-Type: text/plain; charset=UTF-8";

    return mail($recipient, $subject, $body, $headers);
}

/**
 * Handle the form and return the values, errors, status, and CSRF token to render.
 *
 * @return array{
 *     values: array{name: string, email: string, message: string},
 *     errors: array<string, string>,
 *     success: ?string,
 *     csrfToken: string
 * }
 */
function handleContactForm(): array
{
    startContactSession();

    if (
        !isset($_SESSION['contact_csrf_token'])
        || !is_string($_SESSION['contact_csrf_token'])
    ) {
        $_SESSION['contact_csrf_token'] = bin2hex(random_bytes(32));
    }

    $success = $_SESSION['contact_success'] ?? null;
    unset($_SESSION['contact_success']);

    $values = [
        'name' => '',
        'email' => '',
        'message' => '',
    ];
    $errors = [];

    if (($_SERVER['REQUEST_METHOD'] ?? 'GET') === 'POST') {
        $values = [
            'name' => readPostString('name'),
            'email' => readPostString('email'),
            'message' => readPostString('message'),
        ];

        $submittedToken = $_POST['csrf_token'] ?? null;
        $sessionToken = $_SESSION['contact_csrf_token'];

        if (
            !is_string($submittedToken)
            || !hash_equals($sessionToken, $submittedToken)
        ) {
            $errors['form'] = 'The form could not be verified. Reload it and try again.';
        } else {
            $errors = validateContactValues($values);
        }

        if ($errors === []) {
            if (!deliverContactMessage($values)) {
                $errors['form'] = 'The message could not be sent. Please try again later.';
            } else {
                $_SESSION['contact_success'] =
                    'Thank you. Your message was accepted by the mail transport.';
                $_SESSION['contact_csrf_token'] = bin2hex(random_bytes(32));

                header('Location: contact.php', true, 303);
                exit;
            }
        }
    }

    return [
        'values' => $values,
        'errors' => $errors,
        'success' => is_string($success) ? $success : null,
        'csrfToken' => $_SESSION['contact_csrf_token'],
    ];
}
~~~

## 2. Contact page and form

### File: public/contact.php

~~~php
<?php
declare(strict_types=1);

require_once __DIR__ . '/../src/contact-form.php';

$state = handleContactForm();
$values = $state['values'];
$errors = $state['errors'];
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta
        name="description"
        content="Send a message using the contact form."
    >
    <title>Contact</title>
    <link rel="stylesheet" href="assets/contact.css">
</head>
<body>
    <main class="page" id="main-content">
        <a class="back-link" href="index.php">Back to portfolio</a>

        <header class="page-header">
            <p class="eyebrow">Get in touch</p>
            <h1>Contact</h1>
            <p>Complete the form and I will get back to you.</p>
        </header>

        <?php if ($state['success'] !== null): ?>
            <p class="success-message" role="status">
                <?= escapeHtml($state['success']) ?>
            </p>
        <?php endif; ?>

        <?php if ($errors !== []): ?>
            <div class="error-summary" role="alert" tabindex="-1">
                <p>Please review the following:</p>
                <ul>
                    <?php foreach ($errors as $error): ?>
                        <li><?= escapeHtml($error) ?></li>
                    <?php endforeach; ?>
                </ul>
            </div>
        <?php endif; ?>

        <form action="contact.php" method="post">
            <input
                type="hidden"
                name="csrf_token"
                value="<?= escapeHtml($state['csrfToken']) ?>"
            >

            <div class="field">
                <label for="name">Your name</label>
                <input
                    id="name"
                    name="name"
                    type="text"
                    value="<?= escapeHtml($values['name']) ?>"
                    autocomplete="name"
                    maxlength="100"
                    required
                    aria-invalid="<?= isset($errors['name']) ? 'true' : 'false' ?>"
                    aria-describedby="name-help<?= isset($errors['name']) ? ' name-error' : '' ?>"
                >
                <p class="field-help" id="name-help">Maximum 100 characters.</p>
                <?php if (isset($errors['name'])): ?>
                    <p class="field-error" id="name-error">
                        <?= escapeHtml($errors['name']) ?>
                    </p>
                <?php endif; ?>
            </div>

            <div class="field">
                <label for="email">Email address</label>
                <input
                    id="email"
                    name="email"
                    type="email"
                    value="<?= escapeHtml($values['email']) ?>"
                    autocomplete="email"
                    maxlength="254"
                    required
                    aria-invalid="<?= isset($errors['email']) ? 'true' : 'false' ?>"
                    aria-describedby="email-help<?= isset($errors['email']) ? ' email-error' : '' ?>"
                >
                <p class="field-help" id="email-help">
                    This address is included in the message so the recipient can reply.
                </p>
                <?php if (isset($errors['email'])): ?>
                    <p class="field-error" id="email-error">
                        <?= escapeHtml($errors['email']) ?>
                    </p>
                <?php endif; ?>
            </div>

            <div class="field">
                <label for="message">Message</label>
                <textarea
                    id="message"
                    name="message"
                    rows="8"
                    maxlength="5000"
                    required
                    aria-invalid="<?= isset($errors['message']) ? 'true' : 'false' ?>"
                    aria-describedby="message-help<?= isset($errors['message']) ? ' message-error' : '' ?>"
                ><?= escapeHtml($values['message']) ?></textarea>
                <p class="field-help" id="message-help">
                    Maximum 5000 characters. Do not include passwords or other secrets.
                </p>
                <?php if (isset($errors['message'])): ?>
                    <p class="field-error" id="message-error">
                        <?= escapeHtml($errors['message']) ?>
                    </p>
                <?php endif; ?>
            </div>

            <button type="submit">Send message</button>
        </form>
    </main>
</body>
</html>
~~~

## 3. Form stylesheet

### File: public/assets/contact.css

~~~css
:root {
    font-family: system-ui, sans-serif;
    color: #1c2733;
    background: #f4f7fa;
    line-height: 1.6;
}

*,
*::before,
*::after {
    box-sizing: border-box;
}

body {
    margin: 0;
}

.page {
    width: min(100% - 2rem, 42rem);
    margin: 0 auto;
    padding-block: 2rem 5rem;
}

.back-link {
    display: inline-block;
    margin-bottom: 2rem;
    color: #075e9b;
}

a:focus-visible,
input:focus-visible,
textarea:focus-visible,
button:focus-visible {
    outline: 3px solid #b84416;
    outline-offset: 3px;
}

.page-header {
    margin-bottom: 2rem;
}

.eyebrow {
    margin-bottom: 0.25rem;
    color: #526476;
    font-size: 0.85rem;
    font-weight: 700;
    letter-spacing: 0.08em;
    text-transform: uppercase;
}

h1 {
    margin: 0 0 0.5rem;
    font-size: clamp(2.25rem, 8vw, 3.5rem);
    line-height: 1.1;
}

.page-header > p:last-child {
    color: #394b5c;
}

.field {
    margin-bottom: 1.5rem;
}

label {
    display: block;
    margin-bottom: 0.4rem;
    font-weight: 700;
}

input,
textarea {
    display: block;
    width: 100%;
    padding: 0.75rem;
    border: 1px solid #8293a3;
    border-radius: 0.35rem;
    color: inherit;
    background: #fff;
    font: inherit;
}

textarea {
    resize: vertical;
}

[aria-invalid="true"] {
    border-color: #a12424;
}

.field-help,
.field-error {
    margin: 0.35rem 0 0;
    font-size: 0.95rem;
}

.field-help {
    color: #526476;
}

.field-error {
    color: #8b1e1e;
    font-weight: 650;
}

button {
    padding: 0.75rem 1.1rem;
    border: 0;
    border-radius: 0.35rem;
    color: #fff;
    background: #075e9b;
    font: inherit;
    font-weight: 700;
    cursor: pointer;
}

button:hover {
    background: #043c64;
}

.error-summary,
.success-message {
    margin-bottom: 1.5rem;
    padding: 1rem;
    border-radius: 0.35rem;
}

.error-summary {
    border: 1px solid #a12424;
    color: #681818;
    background: #fff0f0;
}

.success-message {
    border: 1px solid #28633d;
    color: #174326;
    background: #eef9f0;
}

.error-summary p {
    margin-top: 0;
    font-weight: 700;
}

.error-summary ul {
    margin-bottom: 0;
}
~~~

## 4. Configure and run locally

Set recipient and sender addresses in the server environment. These values must be addresses controlled by the site owner and permitted by the mail service.

Example PowerShell setup for a local environment:

~~~powershell
$env:CONTACT_TO = 'owner@example.com'
$env:CONTACT_FROM = 'website@example.com'
php -S localhost:8000 -t public
~~~

Open **http://localhost:8000/contact.php**. PHP's built-in server does not configure a mail provider by itself. If no mail transport is configured, valid submissions will show the safe send-failure message. Do not put real credentials in this Markdown file, source code, or version control.

For production, configure HTTPS and a real mail service. A maintained SMTP mail library or provider API is usually a better fit than relying on the basic PHP mail transport. The sender must be authorized by the mail service and domain configuration.

## Expected behavior

| Action | Result |
| --- | --- |
| Open the page | The form displays labeled name, email, and message fields with a session-backed CSRF token |
| Submit blank fields | The server displays required-field errors and keeps the page usable |
| Submit an invalid email | The server reports an email error without attempting delivery |
| Submit an array instead of a string field | The server treats that field as invalid rather than passing it into string validation |
| Submit a missing or changed CSRF token | The server stops before mail delivery and asks the visitor to reload the form |
| Submit valid values with configured mail | The server passes a plain-text message to the mail transport, then redirects with status 303 |
| Submit valid values without configured mail | The page shows a generic delivery failure without exposing configuration details |

A successful call to PHP's mail transport means that the transport accepted the message for processing. It does not confirm that the message reached the recipient's inbox.

## Security and accessibility notes

- CSRF tokens are generated with random_bytes, kept in the session, and compared using hash_equals.
- Session cookies are HttpOnly, use SameSite=Lax, and are marked Secure when the request is served over HTTPS. Configure HTTPS detection correctly when using a trusted reverse proxy.
- Every POST field is checked as a string before trimming.
- Validation runs on the server even though the HTML also uses required and maxlength.
- Old values, errors, and the CSRF token are escaped before HTML output.
- The visitor's address appears only in the plain-text body; it is never used to build a mail header.
- The From address and recipient come from environment configuration and are validated before use.
- Visible labels, associated error descriptions, aria-invalid, and keyboard focus styles support form navigation.
- This small example does not implement rate limiting, a provider queue, delivery webhooks, or a privacy-retention workflow. Add and operate those deliberately before a public launch.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [148 - Project: build a personal portfolio page](../lessons/148-project-build-a-personal-portfolio-page.md) | [Lesson](../lessons/149-project-create-a-validated-contact-form.md) | [Q&A](../qnas/149-project-create-a-validated-contact-form.md) | [150 - Project: build a database-backed CRUD application](../lessons/150-project-build-a-database-backed-crud-application.md) |
