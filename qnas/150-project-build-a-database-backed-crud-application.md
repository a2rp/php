# Project: build a database-backed CRUD application — Q&A

Review persistent task data, PDO, safe SQL, CRUD request handling, and database operations. Each answer follows its question.

## Multiple-choice questions

### 1. In a server-rendered CRUD application, which request method should normally display a task list without changing data?

- A. GET
- B. DELETE
- C. PATCH
- D. CONNECT
**Answer: A.** A GET request reads and displays information. State-changing actions should use a mutation request such as POST.

### 2. What is the main safety benefit of using a PDO prepared statement for a user-supplied task title?

- A. It encrypts the title.
- B. It keeps the parameter value separate from SQL structure.
- C. It automatically authorizes the user.
- D. It escapes the title for HTML.
**Answer: B.** A prepared statement treats the supplied value as data for the query. HTML escaping and authorization are separate concerns.

### 3. Can a prepared-statement placeholder represent an SQL column name?

- A. Yes, for any value supplied by the request.
- B. No. Placeholders represent values; dynamic identifiers must come from a fixed allowlist.
- C. Yes, if the column name is an integer.
- D. Only in a DELETE statement.
**Answer: B.** Parameter markers are for values. Choose dynamic SQL identifiers from trusted, fixed options.

### 4. When should database values be escaped for HTML?

- A. Only before inserting them into the database.
- B. Each time they are rendered in an HTML context.
- C. Only when the database uses SQLite.
- D. Never.
**Answer: B.** A value read from a database is not automatically safe HTML. Escape it for the context where it is output.

### 5. What should protect an update or delete form from a forged cross-site request?

- A. A CSRF token stored with the visitor's session and checked on POST.
- B. A link that uses GET.
- C. A hidden field containing the record ID only.
- D. A CSS class on the submit button.
**Answer: A.** A server-generated token should be included in the form and verified before changing data.

### 6. What does a database constraint add when the application already validates task status?

- A. A second enforcement layer that protects stored data from other write paths.
- B. A replacement for HTML escaping.
- C. Automatic user authentication.
- D. A backup of the database.
**Answer: A.** Application validation gives useful feedback; database constraints preserve important invariants at the data layer.

### 7. What is a transaction useful for?

- A. Making a page responsive.
- B. Grouping related database writes so they can commit or roll back together.
- C. Rendering task descriptions safely.
- D. Replacing prepared statements.
**Answer: B.** Use a transaction when several operations form one business action and must succeed or fail as a unit.

## Short-answer questions

### 8. What do create, read, update, and delete mean in a task manager?

**Answer:** Create inserts a task, read fetches or lists tasks, update changes a task, and delete removes one. The application should define validation and not-found behavior for each operation.

### 9. Why should a CRUD page use post/redirect/get after a successful mutation?

**Answer:** Redirecting after POST makes the browser load a GET page. Refreshing that page does not ordinarily resubmit the previous create, update, or delete request.

### 10. Why does a required attribute on a task title not replace PHP validation?

**Answer:** A request can be sent without using the browser form or its client-side checks. The PHP handler must verify type, required status, and length before storing the value.

### 11. Does a positive task ID prove a visitor may edit that record?

**Answer:** No. An ID identifies a record; it does not prove permission. A multi-user application must authenticate the user and authorize access to each record.

### 12. Why should a database query select named columns instead of relying on SELECT star?

**Answer:** Explicit columns show what the page depends on and reduce accidental coupling to future schema changes or newly added fields.

### 13. What is the difference between a schema initializer and a migration history?

**Answer:** An initializer creates a fresh database in a known shape. A migration history records ordered changes from one deployed schema version to another so existing data can be upgraded safely.

### 14. Why should a writable SQLite database be outside the public document root?

**Answer:** Otherwise a web server may serve the database file directly, exposing stored data. Keep application data and source files outside directories meant for public requests.

### 15. When should a CRUD project use a transaction?

**Answer:** Use one when a logical operation requires multiple related database writes to succeed or fail together. A single insert is usually already one database operation and does not need an extra transaction.

### 16. Why should visitors not see raw PDO exception messages?

**Answer:** Database errors can expose SQL, schema names, paths, and operational details. Show a general message to visitors and keep necessary diagnostic information in protected logs.

### 17. Why can rowCount be unreliable for determining whether an UPDATE found a record?

**Answer:** Drivers differ in whether they report matched rows or only rows whose values changed. Updating a row to the same values may produce a zero count even though the row exists. Define not-found handling explicitly rather than depending on one driver behavior.

## CRUD and database scenarios

### 18. A page runs a DELETE query when a visitor follows a link. What is the risk?

**Answer:** GET requests can be triggered accidentally by crawlers, previews, or navigation. Deletion should be a deliberate POST request protected by CSRF verification.

### 19. A developer builds SQL by appending a title from POST to a query string. What should the reviewer request?

**Answer:** Replace concatenation with a prepared statement and pass the title as a parameter. Also validate its type and length on the server.

### 20. A task's status comes from a select element with open and done choices. Is checking those options in HTML enough?

**Answer:** No. A direct request can submit any value. The server should check the submitted status against the allowed values, and a database constraint can enforce the invariant again.

### 21. A database query fails after a deployment. What should the visitor see?

**Answer:** A safe, general error message that does not reveal SQL, credentials, or stack traces. Record only the protected operational detail needed for diagnosis.

### 22. An application has an update operation that changes a task and inserts an audit record. What database feature may be appropriate?

**Answer:** A transaction can group both writes so they commit together or are rolled back together if either operation fails.

### 23. A record is absent when the visitor opens its edit page. How should the application behave?

**Answer:** Handle the not-found case explicitly, such as showing a clear not-found message or redirecting to the list. Do not assume a database fetch always returns a record.

### 24. The task list sorts by a query parameter that is concatenated into ORDER BY. What should change?

**Answer:** Choose the sort column and direction from fixed allowlisted values. Prepared-statement placeholders cannot safely stand in for SQL identifiers or keywords.

## Practical exercise

### 25. Write a prepared statement to fetch one task by its ID.

**Answer:**

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, description, status FROM tasks WHERE id = :id'
);
$statement->execute(['id' => $taskId]);
$task = $statement->fetch(PDO::FETCH_ASSOC);
~~~

The request value is a bound parameter rather than part of the SQL text. The application should validate the ID and handle the case where fetch returns no record.

### 26. Describe the safe steps for updating a task title and status.

**Answer:** Accept the update through POST, validate the CSRF token, confirm each field is a string, trim and validate the title and allowed status, parse the ID as a positive integer, load the record and authorize access, then execute an UPDATE with prepared parameters. On success, store a brief status message and redirect with a GET request.

### 27. A task title is stored safely in the database. Is it safe to print directly into a page later?

**Answer:** No. Database storage does not make text safe for HTML. Escape the title at the point where it is rendered in HTML.

**Explanation:** SQL parameterization protects query structure. Output escaping protects the browser's HTML parser. They address different boundaries.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [149 - Project: create a validated contact form](../lessons/149-project-create-a-validated-contact-form.md) | [Lesson](../lessons/150-project-build-a-database-backed-crud-application.md) | [Example](../examples/150-project-build-a-database-backed-crud-application.md) | [151 - Project: build a JSON API](../lessons/151-project-build-a-json-api.md) |
