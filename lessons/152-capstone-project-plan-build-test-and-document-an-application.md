# Capstone: plan, build, test, and document an application

A capstone brings skills from earlier lessons into one small application. The goal is not to write the most code. The goal is to turn a clear requirement into a working, secure, maintainable result and show how you know it works.

This chapter uses a personal reading tracker as its running project. A user can record books, review the list, change a reading status, edit details, and remove a record. The application uses PHP, SQLite, server-rendered HTML, validation, sessions, CSRF protection, and PDO.

The example is intentionally single-user and local. It is a learning application, not a public multi-user service.

## Define the project before coding

Start with a short problem statement:

> I want a simple place to record books I plan to read, am reading, or have finished.

Turn that statement into user stories:

- As a reader, I can add a book with its title and author.
- As a reader, I can see all saved books and their status.
- As a reader, I can edit a book.
- As a reader, I can delete a book after choosing a clear action.
- As a reader, I receive useful feedback when submitted data is invalid.

User stories describe outcomes. They do not decide the implementation too early.

## Set scope and boundaries

Write down what the first version includes and excludes.

| In scope | Out of scope for version one |
| --- | --- |
| Add, list, edit, and delete books | User accounts and multiple libraries |
| Status values: planned, reading, finished | Reviews, ratings, and recommendations |
| Title, author, optional notes | Cover image uploads |
| SQLite persistence | Cloud synchronization |
| One local user | Public deployment |

A deliberate boundary helps finish the project. Additional features can be considered after the core workflows work.

## Make acceptance criteria observable

Acceptance criteria describe behavior that can be checked. For example:

1. Submitting a valid title and author creates one book.
2. The book remains after restarting the PHP server.
3. The list shows its title, author, status, and notes.
4. A blank title or author does not create a record and shows a field error.
5. Only the three documented status values are accepted.
6. An edit changes the chosen book and preserves fields not changed by the form.
7. Delete requires a POST form with a valid CSRF token.
8. User-supplied text is rendered as text, not interpreted as HTML.
9. A missing book shows a 404 page.
10. Database details and stack traces are not shown to visitors.

Acceptance criteria make “done” concrete and become the basis for the test plan.

## Sketch the data and routes

Choose the smallest data model that meets the requirements:

| Field | Rule |
| --- | --- |
| id | Integer primary key |
| title | Required text, at most 160 characters |
| author | Required text, at most 120 characters |
| status | planned, reading, or finished |
| notes | Optional text, at most 2,000 characters |
| created_at | Database-generated timestamp |

Choose routes based on the resource and action:

| Method | Path | Action |
| --- | --- | --- |
| GET | /books | List books |
| GET | /books/new | Show the create form |
| POST | /books | Validate and create a book |
| GET | /books/{id}/edit | Show the edit form |
| POST | /books/{id} | Validate and update a book |
| POST | /books/{id}/delete | Validate and delete a book |

GET requests display pages. State changes use POST. After a successful POST, redirect with 303 See Other so a browser refresh does not resubmit the form.

## Keep the architecture small and understandable

Use boundaries that clarify responsibility:

- The front controller reads the path and method and selects an action.
- Validation checks submitted values and returns values plus field errors.
- Repository functions contain SQL and persistence operations.
- Templates render HTML and escape dynamic text.
- Bootstrap code creates the PDO connection and shared security helpers.

A small project does not need a framework to have clear boundaries. Avoid building an elaborate service container or generic routing system for a handful of routes.

A typical request flow is:

1. The browser sends a request.
2. The route is matched and the method is checked.
3. A POST verifies its CSRF token.
4. Submitted values are validated.
5. A repository function reads or writes the database.
6. A template renders an escaped response, or the app redirects after success.

## Build one complete workflow at a time

Prefer a vertical slice: implement one path from request through persistence to response before adding every route.

A practical order is:

1. Create the schema and initialize the database.
2. Implement listing so you can see that the connection works.
3. Add the create form, server validation, and insert query.
4. Redirect after successful creation and show a confirmation.
5. Add edit and update, preserving values when validation fails.
6. Add delete as a POST action with CSRF protection.
7. Add missing-record and unexpected-error pages.
8. Add styling and improve keyboard and screen-reader usability.
9. Review the acceptance criteria and documentation.

This order creates a usable result early and makes failures easier to locate.

## Validate at the boundary

Treat browser submissions as untrusted, even if the form has required fields or length limits. A client can send any request it wants.

For each field:

- Check that it is a scalar string before calling string functions.
- Trim values where leading and trailing spaces are not meaningful.
- Enforce required fields, length limits, and allowed values on the server.
- Preserve submitted values when redisplaying a form.
- Keep database constraints as a second line of defense.

Do not use unvalidated request values as SQL fragments, file paths, class names, or template names.

## Protect state-changing requests

A session can store a random CSRF token. Include the token in every state-changing form and compare the submitted token with the session token using hash_equals(). Reject a request with a missing or invalid token.

Use secure session-cookie settings appropriate to the deployment:

- HttpOnly prevents client-side scripts from reading the cookie.
- Secure sends the cookie only over HTTPS.
- SameSite=Lax or Strict can reduce cross-site cookie sending.
- Strict session mode rejects uninitialized session IDs.

SameSite is a useful additional measure, not a replacement for CSRF tokens. Cookie settings should account for HTTPS and any trusted reverse proxy in production.

## Escape when rendering HTML

Validate input for application rules, but still escape it when placing it into HTML. Validation does not make a string safe for every output context.

~~~php
function e(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}
~~~

Use the escaping helper for text and quoted attribute values. HTML escaping is context-specific; JavaScript, CSS, URL, and SQL contexts need their own appropriate handling. Use prepared statements for SQL values.

## Keep database work predictable

Give each repository function a clear job. Bind values in prepared statements, select only the columns needed, and use deterministic ordering.

If a workflow updates several related records, use a transaction so all related writes succeed or fail together. Avoid transactions for simple single-statement changes unless they are part of a larger unit of work.

Keep database credentials and generated database files out of source control. Store production secrets in deployment configuration, not in committed PHP files.

## Create a test plan before the final pass

Test the behavior promised by the acceptance criteria, including failure paths.

### Unit-level checks

Test pure functions such as validation without starting the web server or using the real database:

- Empty title is rejected.
- A title at the limit is accepted and one beyond it is rejected.
- An unknown status is rejected.
- Notes longer than the limit are rejected.
- A submitted array is rejected where a string is expected.

### Integration checks

Exercise repository functions against an isolated test database:

- Insert a book and read it back.
- Update one record without changing another.
- Delete a record and confirm it is absent.
- Verify invalid status values cannot bypass database constraints.

Use a separate test database so running the suite cannot damage personal data.

### Request-level checks

Exercise the application through HTTP:

- GET each page and check the status and expected content.
- POST valid form data and confirm the redirect and saved row.
- POST invalid data and confirm errors are shown without saving.
- Try a missing ID and an unsupported method.
- Submit a missing and an invalid CSRF token.
- Submit text containing HTML and confirm it is displayed as text.

### Regression checks

After a fix, repeat the failing case and the related successful flow. A change to validation can affect both create and edit forms, so check both.

A useful test matrix maps a requirement to evidence:

| Requirement | Test input | Expected result |
| --- | --- | --- |
| Create book | Valid title and author | Redirect; one new row |
| Required fields | Blank title | Form shown with title error; no row |
| Status allowlist | status=archived | Rejected |
| CSRF protection | Missing token on delete | Request rejected; row remains |
| Output escaping | Title contains HTML markup | Markup appears as text |
| Missing record | Edit an unknown ID | 404 response |
| Persistence | Restart server then list | Existing row remains |

## Document how to use and maintain the app

A useful README should answer:

- What does the application do?
- Which PHP version and extensions are needed?
- How do you initialize the database and start the local server?
- Which routes and workflows are available?
- Where is application data stored?
- How do you run checks or repeat the manual test plan?
- What security limitations remain?
- What should a future maintainer improve next?

Record important decisions and trade-offs. For example, SQLite is convenient for a local single-user app, while a larger deployed system may need a managed database, backups, migrations, and operational monitoring.

Keep setup steps reproducible. A new developer should be able to clone the project, follow the README, and reach the same working state without guessing.

## Review accessibility and usability

A technically correct form can still be difficult to use. Check that:

- Every input has a visible label.
- Errors are associated with the relevant field and are understandable.
- Links and buttons describe their action.
- Forms work with a keyboard.
- Color is not the only way to communicate status.
- Success and failure messages are easy to find.
- Destructive actions are clearly identified.

Keep CSS separate from PHP templates where possible. The page should remain usable if decorative styles fail to load.

## Prepare for deployment deliberately

A development server is not a production server. Before deployment:

- Serve only the public directory.
- Enable HTTPS and secure session cookies.
- Configure error reporting so details go to protected logs.
- Add authentication and per-record authorization for multiple users.
- Protect secrets and database files outside the web root.
- Plan backups, restoration, and schema migrations.
- Set request limits and monitor failures.
- Review permissions and trusted proxy configuration.

Do not call a local demo production-ready simply because it works on your computer.

## Definition of done

A capstone is ready for review when:

- Each in-scope acceptance criterion has evidence.
- The main workflow and its failure paths behave predictably.
- Validation is server-side and database values use prepared statements.
- State-changing requests have appropriate protections.
- Dynamic output is escaped for its context.
- Errors shown to users are safe and useful.
- Setup, usage, testing, and limitations are documented.
- Another person can reproduce the local setup.
- Known limitations and the next improvements are recorded.

## Key takeaways

- Define users, scope, and observable acceptance criteria before coding.
- Build a complete vertical slice, then extend it.
- Keep routing, validation, persistence, and rendering understandable.
- Test normal behavior, invalid input, security boundaries, and regressions.
- Document setup, decisions, limitations, and repeatable verification.
- Treat deployment and multi-user access as separate work that needs explicit security design.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [151 - Project: build a JSON API](151-project-build-a-json-api.md) | [Q&A](../qnas/152-capstone-project-plan-build-test-and-document-an-application.md) | [Example](../examples/152-capstone-project-plan-build-test-and-document-an-application.md) | [153 - PHP quick reference and glossary](153-php-quick-reference-and-glossary.md) |
