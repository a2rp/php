# 088 - Input validation and normalization: Q&A

## Multiple-choice questions

### 1. What does validation do?

- **A.** Converts every value to lowercase
- **B.** Checks whether a value meets defined rules
- **C.** Escapes a value for HTML
- **D.** Removes all punctuation

**Answer: B.** Validation decides whether a value satisfies a rule, such as an allowed format, range, or option. Normalization, sanitization, and output escaping have different purposes.

### 2. A request normally sends `name` as text. Why should PHP still check `is_string()`?

- **A.** The client can send an array using bracket syntax.
- **B.** PHP converts every text input to a boolean.
- **C.** `is_string()` encrypts the value.
- **D.** Request values are always integers.

**Answer: A.** A client can submit a key such as `name[]`, which PHP may represent as an array. Check the expected shape before using string functions or validators.

### 3. What does `FILTER_DEFAULT` do by itself?

- **A.** Validates an email address.
- **B.** Rejects all special characters.
- **C.** Performs no validation; it aliases `FILTER_UNSAFE_RAW`.
- **D.** HTML-escapes a string.

**Answer: C.** Choose a specific validation filter or a deliberate application rule. The default filter is not a validation step.

### 4. How should a program normally check whether integer validation failed?

- **A.** `if (!$result)`
- **B.** `if ($result === false)`
- **C.** `if ($result === 0)`
- **D.** `if (empty($result)`

**Answer: B.** Strictly compare against `false`. Zero may be a valid integer result, so loose or truthiness checks can reject valid input.

### 5. What does `FILTER_VALIDATE_BOOLEAN` return for invalid text when used with `FILTER_NULL_ON_FAILURE`?

- **A.** `false`
- **B.** `true`
- **C.** `null`
- **D.** An empty string

**Answer: C.** With `FILTER_NULL_ON_FAILURE`, `null` distinguishes invalid input from a valid false value.

### 6. What does a successful `FILTER_VALIDATE_EMAIL` result prove?

- **A.** The mailbox exists and belongs to the current user.
- **B.** The address meets the filter's accepted syntax rules.
- **C.** The domain will accept messages.
- **D.** The address is safe to put into every output context.

**Answer: B.** Syntax validation does not prove that a mailbox exists or that the user controls it. Use a verification message when ownership matters, and still escape output for its destination context.

## Short-answer questions

### 7. Explain the difference between validation and normalization.

**Answer:** Validation decides whether a value meets a rule. Normalization transforms accepted values into a consistent representation, such as trimming surrounding whitespace from a search term.

**Explanation:** A normalized value may still be invalid, so validate after applying the field's intended normalization policy.

### 8. Explain the difference between validation and output escaping.

**Answer:** Validation checks that input is acceptable for the application's rules. Output escaping encodes a value for a particular destination, such as HTML text or a quoted HTML attribute.

**Explanation:** A value can be valid but still contain characters that need escaping when rendered. Escaping does not make an invalid value valid.

### 9. Why should validation happen on the server if the form uses `required`, `maxlength`, or `type="email"`?

**Answer:** Those browser checks help users, but a client can bypass or modify them and send a request directly. The server must enforce the important rules itself.

### 10. What does `trim()` remove by default, and what should developers remember?

**Answer:** It removes a defined set of whitespace and NUL characters from the beginning and end of a string. It does not remove characters from the middle and is not aware of every Unicode whitespace character.

**Explanation:** Use it only when trimming is part of the field's policy. Apply a deliberate Unicode-aware rule if the application requires one.

### 11. Why should a password generally not be trimmed or lowercased during normalization?

**Answer:** Those transformations change the secret the user chose and may cause the user to authenticate with a different value than they entered. Validate the password policy without silently rewriting the password.

### 12. What is an allowlist, and when is it useful?

**Answer:** An allowlist explicitly lists acceptable values. It is useful when the valid choices are limited, such as sort modes `name`, `newest`, or `price`.

**Explanation:** It is usually easier to reason about a known set of acceptable values than an incomplete list of forbidden inputs. Use strict comparisons when appropriate.

### 13. What is the difference between byte length and character length in PHP?

**Answer:** `strlen()` measures bytes. `mb_strlen()` can measure characters in a chosen encoding when the mbstring extension is available.

**Explanation:** In UTF-8, one visible character can use multiple bytes. Align application limits with the database's storage limits and the product's actual meaning of “length.”

## Code and behavior questions

### 14. What is the bug in this integer validation check?

```php
$page = filter_var($rawPage, FILTER_VALIDATE_INT);

if (!$page) {
    echo 'Invalid page';
}
```

**Answer:** It treats a valid zero as failure because zero is falsey in PHP.

**Correct check:**

```php
if ($page === false) {
    echo 'Invalid page';
}
```

**Explanation:** Validation filters commonly use `false` to report failure. Compare the documented failure value strictly.

### 15. What should happen before calling `trim()` on `$_POST['name']`?

**Answer:** Check that the field exists as expected and is a string. If it is missing or an array, handle the input error rather than passing an unexpected type to a string function.

```php
$nameInput = $_POST['name'] ?? null;

if (!is_string($nameInput)) {
    // Record a validation error.
} else {
    $name = trim($nameInput);
}
```

### 16. What can this filter result mean?

```php
$result = filter_var(
    $rawEnabled,
    FILTER_VALIDATE_BOOLEAN,
    FILTER_NULL_ON_FAILURE
);
```

**Answer:** It can be `true` for a recognized true representation, `false` for a recognized false representation, or `null` when the input is not recognized.

**Explanation:** `null` is the invalid case here; `false` is a valid boolean value. Handle these three states explicitly.

### 17. Is this a safe way to validate a user-selected sort order?

```php
$sort = $_GET['sort'] ?? 'name';
```

**Answer:** No. The code only supplies a default when the key is missing. It does not ensure the provided value is a string or one of the supported choices.

**Safer approach:** Check the type, then compare against a fixed allowlist with strict comparison before using the value.

### 18. If an email passes `FILTER_VALIDATE_EMAIL`, can the application treat it as verified?

**Answer:** No. The filter checks a syntax rule, not mailbox existence or ownership. Send a verification link or code if the application needs to confirm control of the address.

### 19. Does validating that an `id` is an integer mean the current user can access that record?

**Answer:** No. Validation only checks the value's shape or range. The server must separately check whether the current user is authorized to access the identified record.

### 20. Does removing `<` and `>` from a value make it safe for SQL and HTML?

**Answer:** No. There is no single character-removal rule that safely handles every context. Use parameterized database statements for SQL and context-appropriate output escaping for HTML. Validate the value separately against its business rules.

## Key takeaways

- Check input type and structure before normalizing or validating.
- Normalize only according to a clear field policy, then validate the resulting value.
- Use explicit ranges and allowlists, and compare validation failures strictly.
- Filter defaults do not validate; email syntax is not mailbox verification.
- Input validation does not provide authorization, SQL safety, CSRF protection, or output escaping.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [087 - HTML forms and GET or POST requests](../lessons/087-html-forms-and-get-or-post-requests.md) | [Lesson](../lessons/088-input-validation-and-normalization.md) | [Example](../examples/088-input-validation-and-normalization.md) | [089 - Response headers, status codes, and redirects](../lessons/089-response-headers-status-codes-and-redirects.md) |