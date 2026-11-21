# Project: create a validated contact form

A contact form accepts information from a visitor and passes it to an application. It is a useful project because it combines form markup, HTTP requests, validation, sessions, security checks, error handling, and an external delivery service.

A form is not safe just because the browser marks fields as required. Visitors can send requests without using the page, and submitted values must be treated as untrusted on the server.

## Project goals

Create a contact form that:

- Accepts a visitor's name, email address, and message.
- Shows field-specific validation errors.
- Preserves safe input after validation fails.
- Uses a server-generated CSRF token and checks it on submission.
- Limits input sizes and rejects malformed values.
- Escapes submitted values when redisplaying them.
- Sends messages only through a configured server-side mail transport.
- Avoids exposing mail configuration or internal errors.
- Gives a clear success or failure message.
- Includes basic spam and privacy considerations.

The example project uses a small PHP mail integration for demonstration. A real application should use the mail provider or maintained mail library chosen by the project, store credentials in server configuration, and monitor delivery failures.

## Plan the data flow

A normal form submission follows this sequence:

1. The browser requests the form page.
2. PHP starts or resumes a session and creates a random CSRF token if one does not exist.
3. PHP renders the form with the token in a hidden field.
4. The visitor submits the form using POST.
5. PHP confirms that submitted fields are strings and checks the CSRF token.
6. PHP trims values, validates required fields, and enforces size limits.
7. If validation fails, PHP renders the form again with escaped values and useful errors.
8. If validation succeeds, the application passes the message to its configured delivery service.
9. The application redirects after a successful submission so refreshing the page does not resend the POST.

The CSRF token helps confirm that the submission came from a form associated with the visitor's session. A SameSite cookie setting is an additional browser defense; it is not a replacement for checking a token.

## Separate validation, escaping, and delivery

These steps solve different problems:

- **Validation** decides whether a submitted value meets the application's rules.
- **Escaping** makes a value safe for the output context, such as HTML.
- **Delivery** transfers the validated message through an email or messaging service.

Do not treat an email validation filter as proof that an address exists. It checks syntax; confirming that a person controls an address requires a verification flow.

Do not place visitor input into email headers. Keep the destination, sender, subject, and headers controlled by server configuration. Include the visitor's validated address in the plain-text message body when the owner needs to reply.

## Validate on the server

For each submitted field:

- Confirm the value has the expected scalar type. An attacker can submit an array where the form expects a string.
- Trim whitespace where appropriate.
- Enforce required fields and maximum sizes on the server.
- Validate the email syntax.
- Choose explicit behavior for empty and invalid values.
- Keep the raw value only when needed to redisplay the form, and escape it when rendering.

Client-side attributes such as required, maxlength, and type=email improve the visitor's experience. They are not security controls because a request can bypass the browser's checks.

Keep validation errors understandable. Avoid displaying stack traces, SMTP responses, filesystem paths, or configuration details to visitors.

## Protect the POST request

### CSRF token

Generate a token on the server using a cryptographically secure random source. Store it in the visitor's session, include it as a hidden form value, and compare the submitted value with the session value using a timing-safe comparison such as hash_equals.

A missing token, non-string token, or mismatch should stop processing before delivery.

### Session cookie settings

Use HTTPS in production and configure the session cookie with Secure, HttpOnly, and an appropriate SameSite policy. Start the session before sending HTML or other output. When HTTPS is terminated by a reverse proxy, configure the application to recognize HTTPS only from a trusted proxy.

A CSRF token is not a spam filter. Bots can load the form and submit it with a valid token. Consider server-side rate limits and monitoring, and choose additional anti-spam controls with accessibility and privacy in mind.

## Avoid duplicate submissions

After a valid submission is accepted by the delivery transport, store a short success message in the session and return a redirect with status 303. The browser then makes a GET request for the page.

This post/redirect/get flow prevents an ordinary page refresh from resubmitting the same POST. It does not guarantee exactly-once email delivery: a mail service can accept a message and then time out before the application receives confirmation. If duplicate delivery has significant impact, use a queue and an idempotency strategy.

## Email delivery and configuration

Do not hard-code production mail credentials or recipient addresses into a public repository. Read configuration from the deployment environment or a protected secret manager.

Use a fixed, verified sender address configured for the application's domain. Do not use the visitor's address as the From header. Put the validated visitor address in the message body or in a carefully controlled reply-to mechanism provided by the mail library.

A successful call to a mail transport may mean only that the transport accepted the message, not that it reached an inbox. Production systems should track delivery and bounce events through their provider, avoid logging message contents, and show a neutral status to the visitor.

## Privacy and abuse prevention

A contact message may contain personal information. Decide:

- Who can access it.
- Whether it is stored, emailed, or discarded after delivery.
- How long it is retained.
- How delivery failures and abuse are handled.
- What the visitor is told about the submission.

Collect only the fields needed for the purpose. Do not log full message bodies or session tokens. Rate-limit repeated submissions with a server-side strategy; do not rely on a hidden form field alone.

## Accessibility and usability

A usable form should have:

- A visible label for each field.
- Appropriate input types, such as email for the email field.
- A clear indication of required fields.
- Error messages that identify the affected field and explain how to correct it.
- An error summary or alert when multiple fields fail.
- Focus and keyboard behavior that remains understandable after an error.
- A success message after the request is accepted.

Do not communicate errors by color alone. Associate field errors with their input using descriptive text and, where appropriate, aria-describedby and aria-invalid.

## Local development and deployment

Run the form on a local PHP development server to check rendering and request handling. The development server is not intended to host a public production application.

For a production release, confirm:

- HTTPS is enabled and the session Secure cookie is set.
- Session storage and cookie settings are suitable for the deployment.
- Mail transport configuration exists and is protected.
- Rate limits and abuse monitoring are in place.
- PHP errors are logged privately, not displayed to visitors.
- The success message accurately reflects what the delivery provider confirms.
- The form's privacy notice matches actual storage and retention behavior.

## Common mistakes

- Trusting required or maxlength attributes without server-side checks.
- Assuming every value in POST is a string.
- Validating a value but rendering it later without escaping.
- Treating email syntax validation as proof that an inbox exists.
- Using the visitor's address in a From header.
- Building mail headers from untrusted values.
- Treating SameSite cookies as complete CSRF protection.
- Sending the message before checking the CSRF token.
- Revealing mail server errors or secrets in the page.
- Logging session tokens or full message contents.
- Assuming a successful transport call proves inbox delivery.
- Adding a CAPTCHA without considering accessibility, privacy, and alternatives.

## Completion checklist

| Area | Complete when |
| --- | --- |
| Form | Every field has a label, sensible limits, and a server-side rule |
| Request handling | Only the intended request method triggers submission |
| Input types | Unexpected arrays and malformed scalar values are rejected |
| CSRF | A session token is generated, included, and checked before delivery |
| Validation | Required fields, sizes, and email syntax are checked server-side |
| Output | Redisplayed values and error text are escaped for HTML |
| Mail | Recipient and sender come from protected configuration; visitor input is not used in headers |
| Duplicate handling | Successful POST uses redirect-after-post behavior |
| Abuse | Server-side rate limiting and operational monitoring are considered |
| Privacy | Collection, access, logging, and retention are intentional |
| Production | HTTPS, session settings, mail transport, and private error logging are configured |

## Key takeaways

- A browser form is only an interface; PHP must validate every submitted value.
- Validate, escape, and deliver data as separate steps.
- Use a session-backed CSRF token and check it before processing.
- Keep email headers and delivery configuration under server control.
- Redirect after successful POST and avoid promising more delivery certainty than the provider gives.
- Plan for spam, privacy, accessibility, and operational monitoring.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [148 - Project: build a personal portfolio page](148-project-build-a-personal-portfolio-page.md) | [Q&A](../qnas/149-project-create-a-validated-contact-form.md) | [Example](../examples/149-project-create-a-validated-contact-form.md) | [150 - Project: build a database-backed CRUD application](150-project-build-a-database-backed-crud-application.md) |
