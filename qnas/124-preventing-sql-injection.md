# 124 - Preventing SQL injection: Questions and answers

Use these questions to review how SQL injection happens, how prepared statements protect values, and what still needs separate validation or authorization.

## Multiple-choice questions

### 1. What is the main cause of SQL injection?

- A. A database has too many rows.
- B. Untrusted input is combined with SQL so it can change the query's instructions.
- C. A PHP page contains a form.
- D. A query uses a prepared statement.

**Answer:** B. **Explanation:** Injection occurs when input is parsed as part of the SQL command instead of remaining a separate data value.

### 2. Which is the normal way to pass a user-supplied email into a PDO query?

~~~php
$statement = $pdo->prepare(
    'SELECT id FROM users WHERE email = :email'
);
$statement->execute(['email' => $email]);
~~~

**Answer:** Prepare the fixed query and supply the email separately, as shown. **Explanation:** The database receives the email as a value, so SQL-shaped text in the email cannot add query instructions.

### 3. A user can choose a sort column. Which approach is safe?

- A. Put the requested column in a placeholder.
- B. Escape the requested column with `htmlspecialchars()`.
- C. Map the user's choice to a fixed, application-owned allowlist of column names.
- D. Remove spaces from the requested column and concatenate it.

**Answer:** C. **Explanation:** Placeholders represent values, not identifiers. A fixed allowlist limits the SQL structure to known column names.

### 4. What does a placeholder represent in a prepared statement?

- A. Any fragment of SQL.
- B. One complete data value.
- C. A table name only.
- D. A PHP variable name.

**Answer:** B. **Explanation:** A placeholder keeps a value separate from the query syntax. It cannot stand for a table name, operator, keyword, or arbitrary SQL fragment.

## Core concepts

### 5. What is SQL injection?

**Answer:** SQL injection is a vulnerability in which untrusted input changes the meaning of a database query. **Explanation:** It happens when an application lets data become part of SQL instructions instead of keeping it as a value.

### 6. What is the difference between SQL instructions and query data?

**Answer:** Instructions define what the query does; data supplies values the query operates on. **Explanation:** A secure query keeps these roles separate so a value cannot alter the command's structure.

### 7. Why is concatenating a request value into SQL unsafe?

**Answer:** The database parses the combined text as SQL, so special input may become query syntax. **Explanation:** The application has already mixed instructions and data before the database can distinguish their intended roles.

### 8. Which inputs should an application treat as untrusted?

**Answer:** Request fields, route and query parameters, cookies, headers, uploaded or imported content, and stored or remote values that a user could have influenced. **Explanation:** Trust depends on control and validation, not on where the value is currently stored.

### 9. What is a prepared statement?

**Answer:** It is a query template with placeholders whose values are supplied separately. **Explanation:** Keeping data separate from the SQL text prevents a value from changing the query's syntax.

### 10. Can validation replace prepared statements?

**Answer:** No. Validate values for business rules and still bind them in SQL. **Explanation:** A value can be valid for the product while still being dangerous to concatenate into a query.

### 11. Can output encoding such as `htmlspecialchars()` prevent SQL injection?

**Answer:** No. HTML output encoding protects a value when it is rendered into an HTML context. **Explanation:** SQL parameterization and HTML output encoding protect different boundaries and are both needed where applicable.

## PDO and placeholders

### 12. What are the two common PDO placeholder styles?

**Answer:** Named placeholders such as `:email` and positional question-mark placeholders such as `?`. **Explanation:** Both let the statement receive values separately from its SQL structure.

### 13. May named and positional placeholders be mixed in one statement?

**Answer:** No. Use one placeholder style consistently for a statement. **Explanation:** Mixing marker styles is unsupported and makes the mapping between markers and values error-prone.

### 14. Should a placeholder be written inside SQL quotes?

**Answer:** No. Write the marker as the complete value position, without surrounding quotes. **Explanation:** A marker inside a quoted SQL string is treated as ordinary string text rather than a parameter.

### 15. Why does preparing a query after concatenating user input fail to fix it?

**Answer:** The unsafe input is already part of the query template before preparation. **Explanation:** Preparation does not inspect and undo earlier string construction; keep the SQL template fixed from the start.

### 16. Can a PDO placeholder represent an ORDER BY column?

**Answer:** No. A placeholder represents a value, not a column identifier. **Explanation:** Use a fixed allowlist to choose among approved columns.

### 17. Can a placeholder represent ASC or DESC?

**Answer:** No. Sort direction is part of SQL structure. **Explanation:** Map a user's choice to one of the fixed direction strings the application permits.

### 18. How should an application choose a dynamic column safely?

**Answer:** Translate a small user-facing key, such as `name` or `newest`, into a fixed column name stored in an application-owned allowlist. **Explanation:** Never concatenate the raw request value into the query.

### 19. What should happen when a requested sort key is unknown?

**Answer:** Reject it or use a documented safe default from the allowlist. **Explanation:** An unknown value must not be allowed to become query syntax.

### 20. How should a page-size value be handled?

**Answer:** Validate that it is an integer within an allowed range and bind it with an integer type if the driver requires it. **Explanation:** Validation enforces the application's limits, while binding keeps the value separate from SQL syntax.

### 21. How should a LIKE search term be passed safely?

**Answer:** Build the complete search pattern in PHP and bind it as one value. **Explanation:** Do not put a marker inside quoted SQL text, such as `LIKE '%:term%'`.

### 22. Does binding a LIKE pattern make percent and underscore characters literal?

**Answer:** No. Those characters may still act as LIKE wildcards. **Explanation:** Binding prevents SQL injection; escaping wildcard characters is a separate search-behavior requirement.

### 23. Can a single placeholder bind an entire array for an IN clause?

**Answer:** No. One placeholder represents one value. **Explanation:** Create one placeholder per item, bind each item separately, and handle an empty list explicitly.

### 24. Why should an IN list have a maximum size?

**Answer:** A limit prevents oversized requests from creating an unnecessarily large query or consuming excessive database resources. **Explanation:** Parameterization protects query syntax, while resource limits address availability and performance.

## Other database APIs and defenses

### 25. What does the MySQLi type string passed to bind_param() describe?

**Answer:** The data types of the bound values, using markers such as s for string and i for integer. **Explanation:** Values are still supplied separately; the type string does not make SQL identifiers bindable.

### 26. Is addslashes() a reliable substitute for prepared statements?

**Answer:** No. **Explanation:** Correct escaping depends on query context, driver, character set, and database behavior. Prefer parameter binding for values.

### 27. Do ORMs automatically prevent every SQL injection vulnerability?

**Answer:** No. ORM parameterized methods are useful, but raw query methods and manually assembled fragments can still be unsafe. **Explanation:** Review how the query is constructed and bind values through the library's supported API.

### 28. Are stored procedures automatically safe from SQL injection?

**Answer:** No. A procedure can construct unsafe dynamic SQL internally. **Explanation:** Parameterize values inside procedures and review any dynamic query construction.

### 29. Do prepared statements enforce record-level authorization?

**Answer:** No. **Explanation:** They prevent values from changing SQL syntax, but the application must still check whether the current user may access the requested record.

### 30. What does least privilege mean for a database account?

**Answer:** Give the account only the permissions needed for its application tasks. **Explanation:** If a flaw is exploited, restricted permissions can reduce the damage.

### 31. How should an application handle a database exception shown to a visitor?

**Answer:** Return an appropriate generic response and keep detailed diagnostics in protected logs. **Explanation:** Raw errors can expose query details or schema information. Logs should also avoid secrets.

## Applying and checking the defense

### 32. What should a test verify when a search value contains quotes or SQL-shaped text?

**Answer:** The value is treated as a literal search value and cannot change the query's filters or return unrelated records. **Explanation:** Test the security property at the database boundary rather than only checking that a form rejects unusual characters.

### 33. What should be checked when reviewing a dynamic ORDER BY query?

**Answer:** Every interpolated identifier and direction must come from fixed application-owned choices, and ordinary data values must still use placeholders. **Explanation:** Query structure and query data have different safe handling rules.

### 34. What is a useful test for an IN query?

**Answer:** Verify that each item is bound separately, an empty list follows a deliberate path, and an oversized list is limited. **Explanation:** This checks both SQL safety and predictable behavior for edge cases.

### 35. Why should an application inspect raw SQL in an ORM or query builder?

**Answer:** Raw query APIs may bypass automatic parameterization or assemble unsafe fragments. **Explanation:** The security property comes from how the final query and values are built, not simply from choosing a framework.

### 36. What should an application do if query preparation or execution fails?

**Answer:** Handle the failure through the application's error policy, avoid disclosing internals, and do not continue as if the query succeeded. **Explanation:** A failed database operation must not accidentally grant access or produce an unsafe fallback.

### 37. What is a good database-account setup for a read-only feature?

**Answer:** Use an account or role with only the required read permissions. **Explanation:** The feature should not inherit write or administrative privileges it does not need.

### 38. What is the core rule for preventing SQL injection?

**Answer:** Keep SQL structure fixed and pass untrusted values through prepared-statement parameters. **Explanation:** Use allowlists for dynamic identifiers, and enforce validation, authorization, least privilege, safe errors, and resource limits as additional safeguards.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [123 - Secure coding and threat modeling](../lessons/123-secure-coding-and-threat-modeling.md) | [Lesson](../lessons/124-preventing-sql-injection.md) | [Example](../examples/124-preventing-sql-injection.md) | [125 - Preventing cross-site scripting](../lessons/125-preventing-cross-site-scripting.md) |