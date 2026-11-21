# PHP quick reference and glossary — Q&A

These questions review the PHP syntax, runtime features, and web-development terms summarized in the quick-reference lesson. Each answer follows its question.

## Multiple-choice questions

### 1. Which statement best describes a PHP array?

- A. It can only be a zero-based numeric list.
- B. It is an ordered map that can use integer and string keys.
- C. It can store only one value type.
- D. It is always an object.

**Answer: B.** PHP arrays preserve order and can be used as lists, dictionaries, or mixed structures.

### 2. Which comparison checks both value and type?

- A. ==
- B. =
- C. ===
- D. =>

**Answer: C.** The strict comparison operator checks both the value and its type. A single equals sign assigns a value.

### 3. What does the null coalescing operator provide?

- A. A default when the left value is missing or null
- B. A strict comparison
- C. String concatenation
- D. An exception handler

**Answer: A.** The ?? operator returns the right side if the left side is unset or null.

### 4. Which function is intended to encode special HTML characters before placing dynamic text into HTML?

- A. json_encode()
- B. htmlspecialchars()
- C. serialize()
- D. urlencode()

**Answer: B.** htmlspecialchars() converts special HTML characters to entities. The output context still matters; HTML escaping is not a universal encoder for JavaScript, CSS, or SQL.

### 5. Which HTTP method should normally retrieve a page without changing application state?

- A. GET
- B. POST
- C. PATCH
- D. DELETE

**Answer: A.** GET retrieves a representation. State-changing operations should use an appropriate mutation method and protection.

### 6. What does a PDO placeholder bind?

- A. A SQL table name
- B. A SQL column name
- C. A value in a prepared statement
- D. A PHP namespace

**Answer: C.** Prepared-statement placeholders represent values. They cannot safely substitute arbitrary SQL identifiers or syntax.

### 7. Which version introduced enums in PHP?

- A. PHP 7.4
- B. PHP 8.0
- C. PHP 8.1
- D. PHP 8.3

**Answer: C.** Enums became available in PHP 8.1. Code using them requires a compatible runtime.

### 8. What does php -l check?

- A. Whether a script has valid PHP syntax
- B. Whether every database query is correct
- C. Whether all web routes return the correct response
- D. Whether the app is secure

**Answer: A.** A lint check detects syntax problems. It does not execute the application or verify behavior.

### 9. Which file should generally be committed when Composer manages an application’s dependencies?

- A. Only the vendor directory
- B. composer.json and composer.lock
- C. Only a global php.ini
- D. The developer’s local database

**Answer: B.** composer.json declares dependencies and composer.lock records the resolved versions for repeatable installs. The generated vendor directory is usually recreated by Composer.

### 10. What does a PHP interface primarily define?

- A. A database schema
- B. A contract of methods a class must implement
- C. A function’s local variables
- D. A browser session cookie

**Answer: B.** An interface specifies behavior that implementing classes agree to provide.

## Short-answer questions

### 11. What is the difference between a parameter and an argument?

**Answer:** A parameter is a named input declared in a function definition. An argument is the actual value supplied when the function is called.

### 12. What does declare(strict_types=1) do?

**Answer:** It changes scalar type coercion for calls made from the file where it is declared. It does not validate request data or make arbitrary input safe.

### 13. What is the difference between null and an empty string?

**Answer:** null represents the absence of a value. An empty string is a string containing zero characters. Application logic should distinguish them when that distinction matters.

### 14. When should array_key_exists() be used instead of isset()?

**Answer:** Use array_key_exists() when a key may exist with a null value and you need to distinguish that from a missing key. isset() returns false for both a missing key and a key whose value is null.

### 15. What does a union type such as string|int mean?

**Answer:** It allows a value of either listed type. The function should still handle the accepted alternatives deliberately.

### 16. What is the difference between a closure and an arrow function?

**Answer:** Both are anonymous functions. An arrow function automatically captures outer variables by value and is suited to a short expression. A closure uses a function body and a use clause when it needs outer variables.

### 17. What does match do?

**Answer:** match is an expression that compares values strictly and returns the value from the matching arm. It does not fall through like switch; a default arm is useful when other values are possible.

### 18. What does a namespace help with?

**Answer:** A namespace groups names such as classes, functions, and constants and reduces collisions between parts of an application or packages.

### 19. Why should user input be validated and then escaped separately?

**Answer:** Validation checks application rules, such as required values and length. Escaping makes a value safe for a particular output context. Passing validation does not make text safe to insert into HTML without escaping.

### 20. What does CSRF mean, and what is one common mitigation for form submissions?

**Answer:** Cross-site request forgery is when a browser is tricked into sending an unwanted authenticated request. A random server-generated token included in state-changing forms and verified by the server is a common mitigation. SameSite cookies are an additional defense, not a full replacement.

### 21. Why should an application use a prepared statement for user-supplied SQL values?

**Answer:** It keeps data values separate from SQL structure, reducing SQL injection risk. Prepared statements do not validate business rules or authorize the action.

### 22. What should a web application do with an unexpected exception?

**Answer:** Record useful diagnostic details in protected logs, return a safe response to the visitor, and avoid exposing stack traces, SQL, secrets, or filesystem paths.

## Scenario questions

### 23. A form’s title input has a maxlength attribute, but a direct request submits a longer title. Should the PHP app accept it?

**Answer:** No. Browser constraints are only a convenience. The server must enforce the length rule before using or saving the value.

### 24. Why can empty('0') be surprising?

**Answer:** empty() treats the string '0' as empty, along with other false-like values. Use an explicit comparison or a type-aware check if zero is meaningful input.

### 25. A PHP file uses enum syntax but the server runs PHP 8.0. What happens?

**Answer:** PHP 8.0 cannot parse enum syntax because enums require PHP 8.1 or later. Check the target runtime before using version-specific language features.

### 26. A PDO query needs to order by a client-selected column. Can a placeholder bind the column name?

**Answer:** No. Placeholders bind values, not SQL identifiers. Map a short allowlist of accepted sort names to fixed SQL fragments, and reject everything else.

### 27. A JSON API receives malformed JSON. How can PHP make the decode failure explicit?

**Answer:** Use json_decode() with JSON_THROW_ON_ERROR and catch JsonException. Then return a safe client error, usually 400 Bad Request, without exposing internal details.

### 28. A page displays user text in a script block. Is htmlspecialchars() automatically sufficient?

**Answer:** No. HTML text escaping does not automatically make a value safe for a JavaScript context. Prefer avoiding dynamic script construction; otherwise use a context-appropriate safe encoding strategy.

### 29. A developer runs php -l and sees no errors. Can they conclude the application works?

**Answer:** No. Linting verifies syntax only. The developer still needs behavior checks for routes, data access, validation, error handling, and security requirements.

### 30. Composer has updated composer.json but composer.lock is missing from the commit. What problem may this cause?

**Answer:** Another installation may resolve different package versions. Committing composer.lock for an application helps make dependency installs reproducible.

### 31. The quick reference describes a function, but it is unavailable on the production server. What should be checked?

**Answer:** Check the PHP version and whether the required extension is installed and enabled. Some functions belong to extensions and are not available in every runtime configuration.

## Coding questions

### 32. Write a function that accepts a string and returns a trimmed string.

**Answer:**

~~~php
function cleanLabel(string $value): string
{
    return trim($value);
}
~~~

The parameter and return declarations document the expected types. A value from an HTTP request still needs a type check before being passed to this function.

### 33. Given a nullable nickname, choose the fallback Guest.

**Answer:**

~~~php
$displayName = $nickname ?? 'Guest';
~~~

The fallback is used if nickname is unset or null. An empty string is not null, so it remains an empty string.

### 34. Decode JSON and catch invalid input.

**Answer:**

~~~php
try {
    $value = json_decode($json, false, 512, JSON_THROW_ON_ERROR);
} catch (JsonException $exception) {
    // Return or record a safe parsing error.
}
~~~

With associative mode disabled, JSON objects become PHP objects and JSON arrays become PHP arrays. JSON_THROW_ON_ERROR turns decoding errors into JsonException.

### 35. Show a PDO prepared statement for loading one row by ID.

**Answer:**

~~~php
$statement = $pdo->prepare(
    'SELECT id, title FROM books WHERE id = :id'
);
$statement->execute(['id' => $id]);
$book = $statement->fetch();
~~~

The placeholder represents a value. Validate the ID and check whether the query found a row.

### 36. Escape a dynamic string before writing it into an HTML heading.

**Answer:**

~~~php
echo '<h1>' . htmlspecialchars(
    $title,
    ENT_QUOTES | ENT_SUBSTITUTE,
    'UTF-8'
) . '</h1>';
~~~

This is appropriate for HTML text. Other output contexts may require different handling.

### 37. Which PHP 8.5 feature can express a sequence of callable transformations with the pipe operator?

**Answer:**

~~~php
$slug = $title
    |> trim(...)
    |> strtolower(...);
~~~

The pipe operator was introduced in PHP 8.5. Do not use it in a project that supports older PHP versions.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Course |
| --- | --- | --- | --- |
| [152 - Capstone: plan, build, test, and document an application](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) | [Lesson](../lessons/153-php-quick-reference-and-glossary.md) | [Example](../examples/153-php-quick-reference-and-glossary.md) | End of course |
