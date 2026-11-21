# Project: build a database-backed CRUD application

CRUD stands for create, read, update, and delete. This project builds a small task manager whose records persist in a database instead of disappearing when a request ends.

The application combines a form, PHP request handling, validation, PDO, SQL, database constraints, prepared statements, output escaping, and clear separation between public files and private application data.

## Project goals

Build a single-user task manager that lets a visitor:

- Create a task with a title, description, and status.
- View saved tasks.
- Edit an existing task.
- Delete a task.
- Keep task data between requests.
- See useful validation feedback.
- Submit state-changing actions through POST.
- Use CSRF protection for mutations.
- Avoid SQL injection and unsafe HTML output.

This project is a learning application, not a public multi-user product. Anyone who can access the demo can manage its tasks. A public deployment must add authentication and per-record authorization.

## CRUD and HTTP methods

A common mapping for a server-rendered application is:

| Operation | Typical request | Effect |
| --- | --- | --- |
| Create | POST | Add a record |
| Read | GET | Display records or a single record |
| Update | POST | Change an existing record |
| Delete | POST | Remove an existing record |

GET requests should not change application state. Links can request a page or select a record to edit; forms submit changes with POST. After a successful POST, redirect to a GET page so refreshing does not ordinarily repeat the mutation.

## Choose the data model

A task record needs:

- A primary key so the application can identify it.
- A required title.
- A description that may be empty.
- A status restricted to known values.
- Creation and update timestamps.

A database schema might express some rules directly:

~~~sql
CREATE TABLE tasks (
    id INTEGER PRIMARY KEY,
    title TEXT NOT NULL,
    description TEXT NOT NULL DEFAULT '',
    status TEXT NOT NULL CHECK (status IN ('open', 'done')),
    created_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TEXT NOT NULL DEFAULT CURRENT_TIMESTAMP
);
~~~

Application validation improves the visitor experience, while database constraints protect stored data from invalid writes through other code paths. Use both for important invariants.

## Connect with PDO

Use PDO to keep database access behind a consistent PHP interface. Configure the connection once, use exception mode, and keep credentials or writable database paths outside public files.

For a SQLite learning project:

~~~php
$pdo = new PDO('sqlite:' . $databasePath);
$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
~~~

A production application may use another database and a protected DSN. Confirm the required PDO driver, access permissions, backup approach, and deployment configuration.

## Use prepared statements for values

Do not concatenate user input into SQL. Prepare a query and pass values separately:

~~~php
$statement = $pdo->prepare(
    'SELECT id, title, description, status FROM tasks WHERE id = :id'
);
$statement->execute(['id' => $taskId]);
$task = $statement->fetch(PDO::FETCH_ASSOC);
~~~

Prepared statements keep parameter values separate from the SQL statement. Use them for values in create, read, update, and delete operations.

Placeholders represent values, not SQL identifiers or keywords. If a query needs a dynamic sort column or direction, choose it from a fixed allowlist rather than inserting arbitrary request text into the SQL.

## Validate before persistence

Treat request data as untrusted. For each submitted field:

1. Confirm the request method and expected action.
2. Confirm each value has the expected type.
3. Trim values where the application's rules require it.
4. Check required fields and length limits.
5. Restrict enumerated values such as task status to allowed choices.
6. Validate the record ID and confirm the target record exists.
7. Check that the current user is allowed to make the change in a multi-user application.

A form's required and maxlength attributes help a visitor but do not replace server-side validation.

## Separate database operations

Keep SQL in small repository functions or a repository class. Typical operations include:

- List tasks in a predictable order.
- Find one task by ID.
- Insert a task.
- Update a task.
- Delete a task.

The page handler coordinates requests and user feedback. Templates render HTML. The database layer owns the SQL. This separation makes each part easier to review and change.

Do not rely on a broad SELECT * for a long-lived data contract. Select the columns the page needs so schema changes are easier to reason about.

## Handle update and delete carefully

Before updating or deleting a task:

- Parse the ID as a positive integer.
- Look up the record and handle the not-found case.
- Confirm authorization for that record when the application has users.
- Require a valid CSRF token.
- Use a prepared statement.

Avoid using rowCount() as a universal way to detect whether an update succeeded. Database drivers differ in whether an update that writes identical values reports a changed row. For deletes, a positive affected-row count can be useful, but still define the application's not-found behavior explicitly.

A delete button should submit a POST form with a CSRF token. A link that deletes on GET can be activated by crawlers, previews, or accidental navigation.

## Escape output

Database data is not automatically safe HTML. Escape values each time they are inserted into HTML text or quoted attributes:

~~~php
function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}
~~~

Escaping protects the HTML output context. It does not validate a value, authorize a user, or make an unsafe URL acceptable.

## Use transactions for related changes

A single insert or update is normally one database operation. Use a transaction when a business action requires several writes to succeed or fail together, such as creating an order and its line items.

~~~php
$pdo->beginTransaction();

try {
    // Perform all related writes.
    $pdo->commit();
} catch (Throwable $exception) {
    if ($pdo->inTransaction()) {
        $pdo->rollBack();
    }

    throw $exception;
}
~~~

Do not wrap unrelated work in a transaction without a reason. Database engines differ in their transaction and DDL behavior, so plan schema migrations separately from ordinary application writes.

## Handle errors without exposing internals

Use PDO exceptions to detect database failures. In a development environment, inspect diagnostic details privately. In a public response, show a general error and log only the operational detail needed to investigate.

Do not show SQL, stack traces, DSNs, credentials, filesystem paths, or database contents to visitors. Avoid logging task contents unless there is a clear need and a privacy policy.

## Add schema changes safely

A schema file can initialize a fresh learning database, but a maintained application needs a migration history. A migration records a change from one schema version to another and should be applied in a controlled, repeatable way.

For a change that must coexist with old application code, use backward-compatible steps:

1. Add the new structure.
2. Deploy code that can work with old and new data.
3. Backfill or transition records.
4. Switch reads and writes.
5. Remove the old structure after it is no longer needed.

Back up important data and verify restoration, not only backup creation.

## Project workflow

1. Create the project directories and keep the database outside the public document root.
2. Define the schema and a repeatable database initialization or migration step.
3. Configure one PDO connection with exception mode.
4. Implement and review each repository operation with prepared statements.
5. Add server-side validation and allowlisted status values.
6. Add POST handlers, CSRF checks, and post/redirect/get behavior.
7. Render forms and records with semantic HTML and escaped values.
8. Check empty, invalid, missing-record, and database-error paths.
9. Add authentication and authorization before exposing records publicly.
10. Plan database backups, migrations, and deployment permissions.

## Common mistakes

- Building SQL by concatenating form values.
- Assuming required or maxlength attributes validate server requests.
- Escaping HTML only when data is first stored instead of when output is rendered.
- Allowing arbitrary status text or sort-column names.
- Performing a delete through a GET link.
- Forgetting CSRF protection for update and delete requests.
- Treating an integer record ID as proof of authorization.
- Assuming a record exists without checking.
- Revealing database exceptions or credentials in the response.
- Storing a writable SQLite database under the public document root.
- Using a schema creation script as the only long-term migration strategy.
- Treating rowCount identically across all database drivers.
- Deploying a multi-user database app without authentication and per-record access checks.

## Completion checklist

| Area | Complete when |
| --- | --- |
| Schema | IDs, required fields, allowed status values, and timestamps are defined |
| Create | Valid records are inserted using a prepared statement |
| Read | Lists and individual records are read with explicit columns and stable ordering |
| Update | Existing records can be changed after validation and authorization |
| Delete | A POST form and CSRF check protect deletion |
| Validation | Types, lengths, allowed values, and IDs are checked server-side |
| Output | Database values are escaped when rendered in HTML |
| Errors | Visitors see safe messages; diagnostics stay in protected logs |
| Data | Database files, credentials, backups, and migrations are handled deliberately |
| Access control | Public deployments authenticate users and authorize each record operation |

## Key takeaways

- CRUD connects HTTP request handling to persistent data operations.
- Use prepared statements for every user-supplied SQL value.
- Validate before writing and escape when rendering.
- Protect state-changing requests with POST and CSRF checks.
- Keep database code, request handling, and templates understandable.
- Add transactions for related multi-step writes.
- Plan authorization, migrations, backups, and recovery before public deployment.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [149 - Project: create a validated contact form](149-project-create-a-validated-contact-form.md) | [Q&A](../qnas/150-project-build-a-database-backed-crud-application.md) | [Example](../examples/150-project-build-a-database-backed-crud-application.md) | [151 - Project: build a JSON API](151-project-build-a-json-api.md) |
