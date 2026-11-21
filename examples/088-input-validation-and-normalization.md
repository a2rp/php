# 088 - Input validation and normalization: Example

This single-file example accepts a profile form, checks the shape of each submitted value, normalizes selected text fields, validates the values against explicit rules, and redisplays errors or accepted values. It does not create or save an account; a real application should perform its authorized persistence step only after validation succeeds.

## `public/profile.php`

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

function validateProfile(array $input): array
{
    $errors = [];
    $values = [];

    $nameInput = $input['name'] ?? null;
    if (!is_string($nameInput)) {
        $errors['name'] = 'Enter one display name.';
    } else {
        $name = trim($nameInput);

        if ($name === '') {
            $errors['name'] = 'Display name is required.';
        } elseif (strlen($name) > 100) {
            $errors['name'] = 'Display name must not exceed 100 bytes.';
        } else {
            $values['name'] = $name;
        }
    }

    $emailInput = $input['email'] ?? null;
    if (!is_string($emailInput)) {
        $errors['email'] = 'Enter one email address.';
    } else {
        $email = trim($emailInput);

        if (strlen($email) > 254 || filter_var($email, FILTER_VALIDATE_EMAIL) === false) {
            $errors['email'] = 'Enter an email address in an accepted format.';
        } else {
            // Keep the supplied casing; mailbox verification is a separate step.
            $values['email'] = $email;
        }
    }

    $ageInput = $input['age'] ?? null;
    if (!is_string($ageInput)) {
        $errors['age'] = 'Enter one age.';
    } else {
        $age = filter_var(
            $ageInput,
            FILTER_VALIDATE_INT,
            ['options' => ['min_range' => 13, 'max_range' => 120]]
        );

        if ($age === false) {
            $errors['age'] = 'Age must be a whole number from 13 to 120.';
        } else {
            $values['age'] = $age;
        }
    }

    $usernameInput = $input['username'] ?? null;
    if (!is_string($usernameInput)) {
        $errors['username'] = 'Enter one username.';
    } else {
        $username = trim($usernameInput);

        if (!preg_match('/\A[a-zA-Z0-9_-]{3,30}\z/', $username)) {
            $errors['username'] = 'Use 3 to 30 letters, digits, underscores, or hyphens.';
        } else {
            $values['username'] = $username;
        }
    }

    $planInput = $input['plan'] ?? null;
    $allowedPlans = ['basic', 'pro'];
    if (!is_string($planInput) || !in_array($planInput, $allowedPlans, true)) {
        $errors['plan'] = 'Choose a supported plan.';
    } else {
        $values['plan'] = $planInput;
    }

    if (($input['terms'] ?? null) !== 'accepted') {
        $errors['terms'] = 'Accept the terms to continue.';
    }

    return ['values' => $values, 'errors' => $errors];
}

function oldString(array $input, string $key): string
{
    return isset($input[$key]) && is_string($input[$key])
        ? $input[$key]
        : '';
}

$method = $_SERVER['REQUEST_METHOD'] ?? '';
if (!in_array($method, ['GET', 'POST'], true)) {
    http_response_code(405);
    header('Allow: GET, POST');
    header('Content-Type: text/plain; charset=UTF-8');
    exit('Method not allowed.');
}

$input = $method === 'POST' ? $_POST : [];
$values = [];
$errors = [];

if ($method === 'POST') {
    $result = validateProfile($input);
    $values = $result['values'];
    $errors = $result['errors'];

    if ($errors !== []) {
        http_response_code(422);
    }
}

$nameForForm = oldString($input, 'name');
$emailForForm = oldString($input, 'email');
$ageForForm = oldString($input, 'age');
$usernameForForm = oldString($input, 'username');
$planForForm = oldString($input, 'plan');
$termsChecked = ($input['terms'] ?? null) === 'accepted';

header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Profile input validation</title>
</head>
<body>
    <main>
        <h1>Profile input validation</h1>

        <?php if ($errors !== []): ?>
            <div role="alert">
                <h2>Please correct these fields</h2>
                <ul>
                    <?php foreach ($errors as $fieldError): ?>
                        <li><?= escapeHtml($fieldError) ?></li>
                    <?php endforeach; ?>
                </ul>
            </div>
        <?php elseif ($method === 'POST'): ?>
            <section aria-labelledby="accepted-heading">
                <h2 id="accepted-heading">Input accepted</h2>
                <p>Name: <?= escapeHtml($values['name']) ?></p>
                <p>Email: <?= escapeHtml($values['email']) ?></p>
                <p>Age: <?= $values['age'] ?></p>
                <p>Username: <?= escapeHtml($values['username']) ?></p>
                <p>Plan: <?= escapeHtml($values['plan']) ?></p>
                <p>This demonstration validates the values but does not save an account.</p>
            </section>
        <?php endif; ?>

        <form action="/profile.php" method="post">
            <label for="name">Display name</label>
            <input
                id="name"
                name="name"
                maxlength="100"
                required
                value="<?= escapeHtml($nameForForm) ?>"
            >

            <label for="email">Email address</label>
            <input
                id="email"
                name="email"
                type="email"
                maxlength="254"
                required
                value="<?= escapeHtml($emailForForm) ?>"
            >

            <label for="age">Age</label>
            <input
                id="age"
                name="age"
                type="number"
                min="13"
                max="120"
                required
                value="<?= escapeHtml($ageForForm) ?>"
            >

            <label for="username">Username</label>
            <input
                id="username"
                name="username"
                minlength="3"
                maxlength="30"
                required
                value="<?= escapeHtml($usernameForForm) ?>"
            >

            <label for="plan">Plan</label>
            <select id="plan" name="plan" required>
                <option value="">Choose a plan</option>
                <option value="basic" <?= $planForForm === 'basic' ? 'selected' : '' ?>>Basic</option>
                <option value="pro" <?= $planForForm === 'pro' ? 'selected' : '' ?>>Pro</option>
            </select>

            <label>
                <input
                    type="checkbox"
                    name="terms"
                    value="accepted"
                    <?= $termsChecked ? 'checked' : '' ?>
                >
                I accept the terms
            </label>

            <button type="submit">Validate profile</button>
        </form>
    </main>
</body>
</html>
```

## How the example works

1. A GET request displays the form. A POST request passes `$_POST` to `validateProfile()`.
2. Each value is checked to be one string before string functions or filters use it. The plan uses a strict allowlist.
3. The name, email, and username are trimmed according to this demo's policy. The password field is intentionally omitted; passwords should not be silently trimmed or lowercased.
4. The age is validated as an integer from 13 through 120. Validation compares the result with `false` strictly.
5. Validation errors are collected and displayed together. Invalid values are not treated as accepted profile data.
6. Values retained in the form are escaped. The accepted integer age is an actual validated integer; other accepted strings are HTML-escaped when rendered.

The email check verifies only that the string matches PHP's accepted syntax. It does not verify that the mailbox exists or belongs to the user. A real account flow should send an email verification challenge.

## Try these submissions

Start a local PHP server from the project directory:

```bash
php -S localhost:8000 -t public
```

Open `/profile.php` and try:

- Valid values such as `Amina Shah`, `amina@example.test`, age `25`, username `amina_25`, and plan `basic`.
- Age `12` or `twenty-five` to see range and integer validation fail.
- Username `a!` to see the allowlist rule fail.
- Plan `enterprise` to see the strict plan allowlist reject an unsupported option.
- A display name containing `<script>` text to see it redisplayed as text, not executed as HTML.
- A request with a field such as `name[]=Amina` to see the handler reject an array where one string is expected.

## Expected behavior

With valid values, the page displays **Input accepted** and the normalized display name and email. It does not create or save an account.

With invalid values, the server responds with status 422, displays field-specific messages, and preserves string form fields safely. The browser's `required`, `type`, `min`, `max`, and length attributes improve the form experience, while PHP independently enforces the rules.

## Important notes

- This demo uses byte limits with `strlen()`. Adjust the policy and storage constraints if the product requires Unicode character counts.
- Trimming is an explicit policy here; it is not applied to passwords or every type of data.
- Validation does not replace output escaping, authorization, CSRF protection, or parameterized database queries.
- A real state-changing profile or account handler should add appropriate CSRF and authorization checks before persistence.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [087 - HTML forms and GET or POST requests](../lessons/087-html-forms-and-get-or-post-requests.md) | [Q&A](../qnas/088-input-validation-and-normalization.md) | Example | [089 - Response headers, status codes, and redirects](../lessons/089-response-headers-status-codes-and-redirects.md) |