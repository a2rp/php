# 088 - Input validation and normalization

## Learning goals

By the end of this lesson, you will be able to:

- Distinguish validation, normalization, sanitization, and output escaping.
- Check that request data has the expected type and shape before processing it.
- Validate scalar values with explicit rules and strict result checks.
- Normalize values deliberately without changing their meaning.
- Apply allowlists, length limits, ranges, and format checks to user input.
- Explain why successful validation does not replace authorization, database parameterization, or output escaping.

## 1. Treat input as untrusted

Request data comes from the client. A browser form is only one way to send a request; a client can omit fields, change values, send arrays, or bypass browser-side constraints.

```php
$name = $_POST['name'] ?? '';
```

This retrieves a value, but it does not prove that the value is a string, non-empty, short enough, or appropriate for the application's rules. First verify the shape; then normalize and validate it before using it.

## 2. Validation, normalization, sanitization, and escaping

These operations have different purposes:

| Operation | Purpose | Example |
| --- | --- | --- |
| Validation | Decide whether a value meets a rule | Confirm a page number is an integer from 1 to 100. |
| Normalization | Convert accepted equivalent forms to one representation | Trim leading and trailing spaces from a search term. |
| Sanitization | Transform or remove characters according to a policy | Remove unsupported punctuation from a deliberately restricted slug. |
| Output escaping | Encode a value for its destination context | Escape text before inserting it into HTML. |

Do not use these names interchangeably. A transformation can change a value without making it valid, and a valid value can still need output escaping. For HTML output, use context-appropriate escaping as described in [Generating HTML safely with templates](085-generating-html-safely-with-templates.md).

## 3. Check type and shape first

PHP form parameters can be strings or arrays. A malicious request may send `name[]=value` even when a form normally sends one text field.

```php
$nameInput = $_POST['name'] ?? null;

if (!is_string($nameInput)) {
    // The field is missing or is not one string.
}
```

Do not pass an unchecked array to `trim()`, a regular expression, or a validator intended for a scalar. For structured data, verify each expected key and nested type before reading it.

Missing and empty values may have different meanings. `??` provides a default for a missing or null value; it does not trim spaces or validate content. Choose deliberately whether an omitted field is optional, invalid, or assigned a default.

## 4. Normalize only what the field's rules allow

Normalization is a deliberate conversion to a canonical form. Examples include trimming surrounding whitespace from a search term or converting line endings to a consistent form before processing text.

```php
$searchTerm = trim($searchTerm);
```

PHP's `trim()` removes a defined set of ASCII whitespace and NUL characters from the beginning and end. It does not remove all possible Unicode whitespace. If an application needs a Unicode-specific policy, implement and document that policy explicitly.

Do not apply a generic "clean everything" transformation to every field. In particular:

- Preserve a person's display name as entered unless the product defines a specific normalization rule.
- Do not lowercase or otherwise rewrite an email address unless the system has a clear account-identity policy. Email verification is still needed to prove control of the mailbox.
- Do not trim, lowercase, or Unicode-normalize passwords; changing the secret can prevent a user from signing in with the value they chose.
- Do not URL-decode values a second time after PHP has already decoded request parameters.

Normalize before applying rules that depend on the normalized representation, then validate the final value and store or use that representation consistently.

## 5. Validate numbers with explicit bounds

A cast is not a validation rule. Casting malformed input can silently produce a value that was not actually submitted.

```php
$rawQuantity = $_POST['quantity'] ?? null;

if (!is_string($rawQuantity)) {
    $errors['quantity'] = 'Enter one quantity.';
} else {
    $quantity = filter_var(
        $rawQuantity,
        FILTER_VALIDATE_INT,
        ['options' => ['min_range' => 1, 'max_range' => 20]]
    );

    if ($quantity === false) {
        $errors['quantity'] = 'Quantity must be a whole number from 1 to 20.';
    }
}
```

Use strict comparison with `false`. Values such as integer `0` can be valid results for some rules but are falsey in a loose conditional.

For decimal quantities or monetary values, define the accepted syntax and precision instead of assuming a floating-point cast is sufficient. Monetary values often need integer minor units or a decimal-safe representation to avoid floating-point rounding problems.

## 6. Validate formats and choices

Use built-in validation filters when they match the field's purpose, and add the application's own constraints:

```php
$email = filter_var($emailInput, FILTER_VALIDATE_EMAIL);

if ($email === false) {
    $errors['email'] = 'Enter an email address in an accepted format.';
}
```

A successful email format check only says the string matches the filter's syntax rules. It does not prove that the mailbox exists or that the user controls it. Use a verification flow when ownership matters.

For values with a small set of valid options, use an allowlist:

```php
$allowedSorts = ['name', 'newest', 'price'];

if (!in_array($sortInput, $allowedSorts, true)) {
    $errors['sort'] = 'Choose a supported sort order.';
}
```

Strict mode (`true`) avoids loose type conversions during comparison. An allowlist is usually easier to reason about than trying to enumerate every forbidden string.

For an identifier with a deliberately narrow ASCII format, define the full accepted pattern:

```php
if (!preg_match('/\A[a-zA-Z0-9_-]{3,30}\z/', $username)) {
    $errors['username'] = 'Use 3 to 30 letters, digits, underscores, or hyphens.';
}
```

The rule is a product decision. Do not use a restrictive ASCII pattern for names or text that should support other writing systems.

## 7. Understand filter return values

`filter_var()` can validate a value and returns `false` when validation fails for common validation filters:

```php
$result = filter_var($rawPage, FILTER_VALIDATE_INT);

if ($result === false) {
    // Invalid integer.
} else {
    // $result is the validated integer, including a possible zero.
}
```

Boolean validation has a different result shape. If invalid input must be distinguished from valid `false`, use `FILTER_NULL_ON_FAILURE`:

```php
$result = filter_var(
    $rawEnabled,
    FILTER_VALIDATE_BOOLEAN,
    FILTER_NULL_ON_FAILURE
);

if ($result === null) {
    // The text is not a recognized boolean representation.
} elseif ($result === true) {
    // Explicitly enabled.
} else {
    // Explicitly disabled.
}
```

The default filter, `FILTER_DEFAULT`, is an alias for `FILTER_UNSAFE_RAW`; it performs no validation by itself. Pick a specific filter and handle its documented failure values.

## 8. Use length and range limits that match the product

Set limits for values before using or storing them. The limit should match the field's purpose and storage constraints, not an arbitrary transformation.

Be clear about the unit:

- `strlen()` measures bytes.
- `mb_strlen()` can measure characters in a specified encoding when the mbstring extension is available.
- Database column limits may use bytes or characters depending on the database and encoding.

For a UTF-8 field, a byte limit and a character limit are not interchangeable. Keep application rules and storage constraints aligned. Also limit collection sizes and nested structures, not just the length of each string.

## 9. Validate form values as a group

A handler commonly gathers validated values and errors before performing a state change:

```php
$errors = [];
$values = [];

$nameInput = $_POST['name'] ?? null;
if (!is_string($nameInput)) {
    $errors['name'] = 'Enter one name.';
} else {
    $name = trim($nameInput);

    if ($name === '') {
        $errors['name'] = 'Name is required.';
    } elseif (strlen($name) > 100) {
        $errors['name'] = 'Name must not exceed 100 bytes.';
    } else {
        $values['name'] = $name;
    }
}

$emailInput = $_POST['email'] ?? null;
if (!is_string($emailInput)) {
    $errors['email'] = 'Enter one email address.';
} else {
    $email = trim($emailInput);
    $validatedEmail = filter_var($email, FILTER_VALIDATE_EMAIL);

    if ($validatedEmail === false) {
        $errors['email'] = 'Enter an email address in an accepted format.';
    } else {
        $values['email'] = $validatedEmail;
    }
}

if ($errors === []) {
    // Continue with authorization and the intended application operation.
}
```

Keep the validated values separate from raw request data. If validation fails, show useful field-specific messages and preserve only values that are safe and appropriate to redisplay. Escape those values and messages when rendering HTML.

Do not write to a database, send email, or perform another state change until required validation succeeds. Also check whether the current user is authorized to perform the operation; valid input does not grant permission.

## 10. Validation is one layer, not a universal security fix

Validation does not replace other controls:

- Use parameterized statements to keep data separate from SQL syntax.
- Use output escaping for HTML, JavaScript, URLs, and other output contexts as appropriate.
- Check authorization independently of whether an identifier or value is well-formed.
- Use CSRF protection for relevant state-changing requests.
- Apply resource limits to input size and collection counts.

Reject values that violate the contract rather than silently turning them into a different value. If normalization is allowed, make it predictable and apply it consistently.

## Key takeaways

- Check request value type and structure before transforming it.
- Normalize only according to an explicit field policy, then validate the normalized value.
- Prefer bounded validation and allowlists; compare filter failures strictly.
- `FILTER_DEFAULT` does not validate, and an email syntax check does not prove mailbox ownership.
- Know whether limits count bytes, characters, or database storage units.
- Validation does not replace authorization, parameterized database access, CSRF controls, or output escaping.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [087 - HTML forms and GET or POST requests](087-html-forms-and-get-or-post-requests.md) | [Q&A](../qnas/088-input-validation-and-normalization.md) | [Example](../examples/088-input-validation-and-normalization.md) | [089 - Response headers, status codes, and redirects](089-response-headers-status-codes-and-redirects.md) |