# Capstone: plan, build, test, and document an application — Q&A

These questions review how to scope a PHP project, build it in understandable steps, verify its behavior, and prepare it for another developer. Each answer follows its question.

## Multiple-choice questions

### 1. What is the main purpose of acceptance criteria?

- A. To choose a CSS framework before understanding the project
- B. To describe observable behavior that can be checked
- C. To list every PHP function in the application
- D. To guarantee that the app has no defects

**Answer: B.** Acceptance criteria turn a broad request into specific outcomes. They help guide implementation and provide evidence for deciding whether a feature is complete.

### 2. Which is the best first step for a capstone project?

- A. Add every feature that might be useful
- B. Define the problem, users, scope, and success criteria
- C. Create a generic framework
- D. Choose database indexes before identifying the data

**Answer: B.** Clear scope and requirements prevent unnecessary work and give the project a direction before implementation details are selected.

### 3. What is a vertical slice?

- A. A complete path through a small feature, from request to response and persistence
- B. A CSS layout technique
- C. A database backup
- D. A list of future features

**Answer: A.** A vertical slice delivers one small workflow end to end. For example, it can include the form, validation, insert query, redirect, and list display for creating a book.

### 4. Which responsibility belongs in a repository function?

- A. Choosing the page title
- B. Escaping text for an HTML attribute
- C. Running a prepared SQL query to load or save a record
- D. Deciding the button color

**Answer: C.** Repository functions centralize data access. Templates render HTML, while validation and routing have their own responsibilities.

### 5. Where should server-side validation happen?

- A. Only in browser JavaScript
- B. At the server boundary before values are used
- C. Only after the database insert
- D. In the CSS file

**Answer: B.** Requests can bypass the browser form. The server must validate types, required values, lengths, and allowed choices before performing the requested operation.

### 6. What does a CSRF token help verify?

- A. That a submitted state-changing form includes a secret value associated with the user’s session
- B. That the SQL query is fast
- C. That the user’s password is strong
- D. That a title contains no HTML

**Answer: A.** A server-generated token submitted with the form helps reject forged cross-site requests. The server should compare it safely with the session value.

### 7. What should happen to a user-supplied title before it is placed in an HTML text node?

- A. It should be concatenated directly.
- B. It should be HTML-escaped for the output context.
- C. It should be inserted into SQL first.
- D. It should be converted to a filename.

**Answer: B.** Validation checks application rules; escaping protects the output context. Escape dynamic text when rendering it, even if it passed validation.

### 8. Which test most directly checks that an HTTP form submission redirects after creating a book?

- A. Unit test of the title-length validator
- B. Request-level test of the POST response and Location header
- C. CSS visual check
- D. Database schema inspection only

**Answer: B.** A request-level test exercises the HTTP behavior, including the response status and redirect.

## Short-answer questions

### 9. How do user stories differ from implementation tasks?

**Answer:** A user story describes the result a person needs, such as adding a book to a reading list. Implementation tasks describe work that helps deliver it, such as creating a table, writing a repository function, and building a form.

### 10. Why should version-one scope include explicit exclusions?

**Answer:** Exclusions keep the first release small enough to finish. They make it clear that accounts, uploads, or recommendations are future work rather than hidden requirements.

### 11. What makes a good acceptance criterion?

**Answer:** It states a specific behavior and an observable result. For example, “Submitting a blank title shows a title error and does not add a database row” can be checked directly.

### 12. What is the Post/Redirect/Get pattern?

**Answer:** After a successful POST, the server sends a redirect, commonly 303 See Other, to a GET page. Refreshing that page repeats the GET rather than resubmitting the form.

### 13. What is the difference between a unit test and an integration test?

**Answer:** A unit test checks a small unit, often a pure function, in isolation. An integration test checks that components work together, such as a repository function reading and writing an isolated SQLite database.

### 14. What is a request-level test?

**Answer:** It sends an HTTP request through the application and checks the response, such as its status, headers, rendered content, redirect, or saved result.

### 15. Why should tests use an isolated database?

**Answer:** An isolated database prevents test runs from changing a developer’s personal or production data. It also makes the starting state repeatable.

### 16. Why is SameSite not a replacement for a CSRF token?

**Answer:** SameSite is a browser cookie policy that can reduce cross-site cookie sending, but it is only an additional mitigation. State-changing forms should still verify an application-generated CSRF token.

### 17. Why validate and escape separately?

**Answer:** Validation answers whether a value is allowed by the application. Escaping makes a value safe for a particular output context. A valid title can still contain characters that must be escaped before HTML rendering.

### 18. What should the README explain for a new developer?

**Answer:** It should explain the project purpose, prerequisites, setup, database initialization, run commands, main routes, tests or verification steps, data location, security limitations, and known future work.

## Scenario questions

### 19. A project has finished its create workflow, but update, delete, and validation are not yet implemented. What should happen next?

**Answer:** Finish and check one complete workflow at a time. Confirm the create acceptance criteria, then add another vertical slice such as editing. Avoid adding unrelated features before the core requirements work.

### 20. A form has a maxlength attribute, but a direct POST sends a longer value. Should the server accept it?

**Answer:** No. The server must enforce the limit because browser attributes can be bypassed. Reject the value, preserve it safely for correction where appropriate, and do not write it to the database.

### 21. A reading tracker stores a note containing an HTML script element. How should the list page display it?

**Answer:** Escape the note as HTML text. The browser should display the markup characters instead of interpreting them as executable content. Validation alone is not a substitute for output escaping.

### 22. A database exception occurs during a request. What should the visitor see, and where should diagnostic information go?

**Answer:** The visitor should see a safe error page or message without a stack trace or database details. Record diagnostic information in protected server logs for investigation.

### 23. The application works locally, but its database file is inside the public directory. What is the risk and how should it be corrected?

**Answer:** The database could be exposed through the web server. Move it outside the document root, serve only the public directory, and configure permissions and backups deliberately.

### 24. The project uses SQLite and a built-in PHP server. Is it automatically production-ready?

**Answer:** No. A working local demo is not the same as a production deployment. Production needs appropriate hosting, HTTPS, protected logs, backups, access control, session configuration, error handling, and operational review.

### 25. A feature is not included in version one but may be useful later. What should the team do?

**Answer:** Record it as future work with a reason or priority. Keep it outside the current acceptance criteria until the project has capacity and a clear requirement for it.

## Coding and review questions

### 26. Show a PHP helper that escapes a string for HTML output.

**Answer:**

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

Use it in text and quoted attribute contexts. Other output contexts, such as JavaScript or CSS, need context-appropriate handling.

### 27. What checks belong in a test for the book title validator?

**Answer:** Check that an empty title is rejected, a title within the character limit is accepted, a title beyond the limit is rejected, and a non-string value such as an array is rejected. Also check trimming behavior if surrounding whitespace is removed.

### 28. What should a regression check do after fixing a delete route that ignored CSRF tokens?

**Answer:** Repeat the original failing case with a missing or invalid token and confirm the record remains. Then submit a valid token and confirm that the intended delete still works.

### 29. How can a project prove that its data persists across requests?

**Answer:** Create a record, make another request to list or read it, and confirm it remains after restarting the PHP server. The test should use the intended local database, not only in-memory state.

### 30. What evidence supports calling a capstone complete?

**Answer:** Each in-scope acceptance criterion has been checked; main and failure paths work; data access and output handling follow the stated security rules; setup and verification are documented; and known limitations are recorded.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [151 - Project: build a JSON API](../lessons/151-project-build-a-json-api.md) | [Lesson](../lessons/152-capstone-project-plan-build-test-and-document-an-application.md) | [Example](../examples/152-capstone-project-plan-build-test-and-document-an-application.md) | [153 - PHP quick reference and glossary](../lessons/153-php-quick-reference-and-glossary.md) |
