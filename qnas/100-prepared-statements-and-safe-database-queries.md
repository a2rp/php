# 100 - Prepared statements and safe database queries: Q&A

## Multiple-choice questions

### 1. What is the main security benefit of a prepared statement?

- **A.** It separates SQL structure from values supplied at execution time
- **B.** It encrypts the database connection automatically
- **C.** It validates every value against business rules
- **D.** It grants database permissions to the current user

**Answer: A.** Placeholders keep supplied values as data instead of allowing them to change the SQL syntax. Prepared statements do not replace validation, authorization, or secure connection configuration.

### 2. Which PDO pattern is appropriate for a search value from a request?

- **A.** Concatenate the value into a SQL string
- **B.** Put the value in a placeholder and execute the prepared statement with that value
- **C.** Escape quotes manually and concatenate the result
- **D.** Put the value inside the SQL comment

**Answer: B.** Bind the search value through a placeholder. Manual escaping is easy to get wrong and depends on the database and character encoding.

### 3. What does a placeholder represent in a prepared statement?

- **A.** A table name
- **B.** A column name
- **C.** A data value
- **D.** A complete SQL clause

**Answer: C.** Parameters stand for values such as an ID, title, or status. They do not stand for identifiers or arbitrary SQL fragments.

### 4. Which is a valid way to use a named placeholder in PDO?

- **A.** Write WHERE id = :id and then pass ['id' => $id] to execute()
- **B.** Write WHERE id = ':id' and then pass the value to execute()
- **C.** Write WHERE :column = :value to choose any column dynamically
- **D.** Write WHERE id = :id and concatenate the value after prepare()

**Answer: A.** Leave the placeholder unquoted in SQL and pass a matching parameter value to execute(). Placeholder naming conventions can vary with driver requirements, so follow PDO's documented form.

### 5. Can one prepared statement mix named and positional placeholders?

- **A.** Yes, in every PDO driver
- **B.** No, use one placeholder style consistently in a statement
- **C.** Only for SELECT statements
- **D.** Only if every value is an integer

**Answer: B.** Do not mix named markers such as :id with positional markers such as ?. Consistent marker style also makes parameter mapping easier to review.

### 6. What is the difference between bindValue() and bindParam()?

- **A.** bindValue() binds a value now; bindParam() binds a variable by reference for use when the statement executes
- **B.** bindValue() is only for SELECT and bindParam() only for INSERT
- **C.** bindValue() escapes SQL and bindParam() validates input
- **D.** There is no difference

**Answer: A.** bindParam() takes a variable by reference, so its value is read when execute() runs. bindValue() binds the value provided at the time the method is called.

### 7. Which method is useful when an explicit PDO parameter type is needed for a supplied value?

- **A.** bindValue()
- **B.** query()
- **C.** fetchAll()
- **D.** lastInsertId()

**Answer: A.** bindValue() accepts a PDO parameter type such as PDO::PARAM_INT. Choose the type that matches the value and the database operation.

### 8. What happens if a request value is concatenated into SQL before prepare() is called?

- **A.** The value is automatically isolated as data
- **B.** The SQL may already have been changed by that value
- **C.** PDO removes all special characters
- **D.** The database rejects every query

**Answer: B.** Preparing a string after unsafe concatenation does not undo the injection risk. Keep values out of the SQL text and bind them separately.

### 9. Can a placeholder be used for ORDER BY :column to select a column dynamically?

- **A.** Yes, it safely selects any identifier
- **B.** No, values cannot act as SQL identifiers; select an identifier from a fixed allowlist
- **C.** Yes, if the column name comes from a form
- **D.** Only for SQLite

**Answer: B.** A bound parameter is a value, not a table or column identifier. Map allowed user choices to fixed SQL fragments and reject unknown choices.

### 10. What is the safe pattern for a user-selected sort direction?

- **A.** Bind the raw direction as a SQL value after ORDER BY
- **B.** Concatenate the raw direction from the request
- **C.** Map an accepted choice such as asc or desc to a fixed SQL token
- **D.** Remove spaces from the raw direction and concatenate it

**Answer: C.** SQL keywords and identifiers cannot be supplied as ordinary bound values. Use a strict allowlist and provide a safe default or reject invalid choices.

### 11. Why does WHERE id IN (?) not bind an array of IDs as several list items?

- **A.** A placeholder represents one value, not a variable-length SQL list
- **B.** PDO supports arrays only for UPDATE
- **C.** IN accepts no parameters
- **D.** The database converts every array to separate SQL values

**Answer: A.** Generate one placeholder for each ID, then pass the individual values in matching order. Handle an empty list before constructing the SQL.

### 12. What should code do when a dynamically generated IN list has no values?

- **A.** Emit IN () and hope every database accepts it
- **B.** Decide explicitly that the result is empty or use another intentional branch
- **C.** Remove the WHERE clause
- **D.** Add a raw value such as NULL without considering its meaning

**Answer: B.** An empty list needs an explicit application decision. Returning no results early is often appropriate; removing the filter could accidentally broaden a query.

### 13. What do percent and underscore mean in a SQL LIKE pattern?

- **A.** They are always literal characters
- **B.** Percent matches a sequence of characters and underscore matches one character
- **C.** Percent matches one character and underscore matches a sequence
- **D.** They are PDO placeholder markers

**Answer: B.** Binding a LIKE pattern prevents it from changing SQL syntax, but wildcard characters still have their normal pattern-matching meaning.

### 14. Does binding a LIKE value make percent and underscore literal search characters?

- **A.** Yes, binding removes LIKE wildcard behavior
- **B.** No, escape wildcard characters when literal matching is the intended behavior
- **C.** Only when using bindParam()
- **D.** Only in INSERT statements

**Answer: B.** Binding protects the SQL structure; it does not change LIKE semantics. If searching for literal percent or underscore characters, use the database's appropriate LIKE escape behavior.

### 15. Which statement about input validation and prepared statements is correct?

- **A.** Prepared statements make all input validation unnecessary
- **B.** Validation checks application rules; parameters protect the SQL structure
- **C.** Validation alone always prevents SQL injection
- **D.** Validation should be performed by concatenating the value into SQL

**Answer: B.** Validate type, range, length, and business rules for the application. Independently bind values to keep them from becoming SQL syntax.

### 16. Do prepared statements verify that the current user is allowed to read a requested record?

- **A.** Yes, the database infers authorization from the placeholder
- **B.** No, application authorization must be checked separately
- **C.** Yes, but only for integer IDs
- **D.** No, so prepared statements should not be used

**Answer: B.** A safely parameterized query can still expose another user's data if authorization checks are missing. Check access rights as part of the application flow.

### 17. What is a safer way to report a PDOException to a public visitor?

- **A.** Display the full message and SQL text
- **B.** Return a generic error and log useful details securely
- **C.** Ignore the exception and report success
- **D.** Print the database credentials for debugging

**Answer: B.** Exception details can expose schema names, paths, SQL, or infrastructure information. Keep diagnostic logs protected and avoid leaking them in public responses.

### 18. What should be considered before reusing a prepared statement in a loop?

- **A.** Prepare once when the SQL is unchanged, then execute with new values for each iteration
- **B.** Rebuild the SQL by concatenating each loop value
- **C.** Reuse it only if every value is identical
- **D.** Call fetchAll() before every execute()

**Answer: A.** Preparing an unchanged statement once can avoid repeated preparation work. Each execution still needs its own correctly bound values and error handling.

## Short-answer questions

### 19. Explain the difference between SQL structure and a bound value.

**Answer:** SQL structure is the statement syntax, including keywords, tables, columns, and operators. A bound value is data supplied separately for a placeholder. **Explanation:** Keeping request data in placeholders prevents it from being interpreted as SQL syntax.

### 20. Why should a placeholder not be surrounded by SQL quotes?

**Answer:** The placeholder itself is the parameter marker; quoting it makes it part of a SQL string literal instead of a parameter. **Explanation:** Write WHERE email = :email, then bind the value without adding quotes around :email in the SQL.

### 21. When should bindParam() be useful?

**Answer:** It is useful when binding a variable by reference, especially if the variable's value will be assigned or changed before execute(). **Explanation:** The statement reads the variable's value when executed. Use bindValue() when you want the value captured at the binding call.

### 22. How can an application safely build an IN query for a variable number of IDs?

**Answer:** Create one placeholder per ID, interpolate only that generated placeholder list, and pass the IDs as separate bound values. **Explanation:** The list length shapes the SQL, but its untrusted contents never do. Handle the empty-list case explicitly.

### 23. Why is a fixed allowlist appropriate for a dynamic sort column?

**Answer:** Column names are SQL identifiers and cannot be safely supplied through value placeholders. **Explanation:** Convert a recognized choice to one of a small number of developer-written column names; reject any other choice.

### 24. What does parameterization do to an input such as apostrophe OR 1 equals 1 comment?

**Answer:** It treats the whole string as a value rather than SQL operators and a comment. **Explanation:** The database compares or stores those characters as data according to the query, so they do not rewrite the statement's conditions.

### 25. Why can input validation not replace parameterized SQL?

**Answer:** Validation enforces expected application rules, while parameterization separates data from SQL syntax. **Explanation:** Validation can be incomplete or change over time; bind values even when they have already been validated.

### 26. What does binding a LIKE pattern protect, and what does it not protect against?

**Answer:** Binding protects SQL structure from the parameter value. It does not make percent and underscore literal or remove their wildcard meaning. **Explanation:** Apply an appropriate LIKE escape rule if the feature must search for those characters literally.

### 27. What is the difference between logging an exception and returning it to the client?

**Answer:** A protected log can preserve details for authorized debugging; a client response should normally reveal only a safe, generic failure. **Explanation:** Exception messages may disclose internal SQL or database details.

### 28. Does a safely prepared query guarantee that an UPDATE changed a row?

**Answer:** No. It guarantees safer separation of SQL and values, not that a row matched or changed. **Explanation:** Check the application's expected result and interpret rowCount() with the database driver's behavior in mind.

## Practical scenarios

### 29. A login query is built by concatenating an email value into the SQL string. What should be changed?

**Answer:** Prepare the SQL with a placeholder for email and execute it with the supplied email as a parameter. **Explanation:** Do not concatenate request data into SQL, even if the form appears to limit the input.

### 30. A statement uses ORDER BY ? and receives the string created_at. Why might the result not be sorted by that column?

**Answer:** The placeholder represents a value, not an identifier, so the database does not treat it as the column name. **Explanation:** Use an allowlist mapping from accepted choices to fixed column names, and bind other values normally.

### 31. A search should find the exact title 100 percent ready, but the bound LIKE pattern matches too many titles. What is likely happening?

**Answer:** The percent sign is still interpreted as a LIKE wildcard. **Explanation:** Binding prevents SQL injection but preserves pattern semantics; escape percent and underscore according to the database's LIKE escape rules for literal search.

### 32. A filter receives no IDs, and the code removes its WHERE clause. What is the risk?

**Answer:** The query may return every row instead of none. **Explanation:** Define the empty-list behavior explicitly, commonly by returning an empty result before querying.

### 33. A request supplies an unsupported sort field. What should the application do?

**Answer:** Reject it or select a documented safe default from an allowlist. **Explanation:** Never place the raw request string into the SQL identifier position.

### 34. A developer manually escapes input and then concatenates it into SQL. Is that an equivalent replacement for parameters?

**Answer:** No. Use prepared statements for data values. **Explanation:** Correct escaping is context- and database-sensitive and is easy to apply incorrectly; parameterized queries are the intended separation mechanism.

### 35. A query is parameterized but fetches a record by an arbitrary account ID from the URL. What security check may still be missing?

**Answer:** Authorization to confirm the signed-in user may access that account's record. **Explanation:** Parameterization protects query syntax but does not decide whether the user is permitted to see a row.

### 36. A prepared statement executes with a parameter array whose keys do not match the named placeholders. What should be checked?

**Answer:** Ensure every required placeholder is supplied with the matching parameter name and that no extra prefix convention is being used incorrectly. **Explanation:** Parameter names must correspond to the SQL markers; alternatively bind values explicitly and verify the execution result or exception.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [099 - Connecting to databases with PDO](../lessons/099-connecting-to-databases-with-pdo.md) | [Lesson](../lessons/100-prepared-statements-and-safe-database-queries.md) | [Example](../examples/100-prepared-statements-and-safe-database-queries.md) | [101 - Database transactions and error handling](../lessons/101-database-transactions-and-error-handling.md) |
