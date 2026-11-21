# Project: create a validated contact form — Q&A

Review form validation, CSRF protection, email delivery, usability, privacy, and abuse prevention. Each answer follows its question.

## Multiple-choice questions

### 1. Why must PHP validate form input even when the HTML fields use required and maxlength?

- A. The browser always removes invalid requests.
- B. A request can bypass browser-side checks and send arbitrary values directly.
- C. PHP ignores POST values.
- D. HTML attributes replace server validation.
**Answer: B.** Browser validation improves usability but is not a security boundary. The server must check every submitted value.

### 2. What should the application do before delivering a form submission that uses a session-backed CSRF token?

- A. Check that the submitted token is a string and matches the session token.
- B. Trust the hidden field because the visitor cannot edit it.
- C. Compare only the token lengths.
- D. Skip validation if the session cookie is SameSite.
**Answer: A.** Hidden inputs can be changed. Verify the submitted value against the server-side session token before processing.

### 3. What does FILTER_VALIDATE_EMAIL establish?

- A. The address exists and belongs to the visitor.
- B. The address is syntactically acceptable to the filter.
- C. The message is not spam.
- D. The sender is authorized to contact the site.
**Answer: B.** Syntax validation cannot prove that an inbox exists or that the visitor controls it. Ownership confirmation requires a separate verification process.

### 4. Which address should normally be used as the From header for a contact-form email?

- A. The visitor's raw submitted address.
- B. A fixed, verified sender address configured by the application.
- C. Any text submitted in the message.
- D. The CSRF token.
**Answer: B.** Keep headers under server control. The visitor's validated address can appear in the message body for the owner to reply.

### 5. What does a SameSite session cookie setting do for CSRF protection?

- A. It makes all cross-site requests impossible in every browser.
- B. It is an additional browser defense, but does not replace server-side token verification.
- C. It validates the email address.
- D. It rate-limits submissions.
**Answer: B.** SameSite can reduce some cross-site cookie behavior. A form that needs CSRF protection should still verify its token.

### 6. What is the post/redirect/get pattern used for?

- A. It encrypts the form contents.
- B. It redirects after a successful POST so refreshing the resulting page does not ordinarily repeat the POST.
- C. It proves that the email was delivered.
- D. It blocks bots from loading the form.
**Answer: B.** PRG reduces accidental browser resubmission after success, but it does not guarantee exactly-once delivery.

## Short-answer questions

### 7. Why should the server check that a submitted field is a string?

**Answer:** A request can submit an array or another unexpected shape instead of a normal field value. Confirm the expected type before trimming, validating, or using the value.

### 8. What is the difference between validation and output escaping?

**Answer:** Validation checks whether a value meets application rules. Escaping encodes a value for a particular output context, such as HTML. A value can be valid but still need escaping when displayed.

### 9. Why is HTML required not enough to make a form secure?

**Answer:** A visitor can bypass the browser and send an HTTP request directly. Server-side code must enforce required fields, type checks, and size limits.

### 10. What does a CSRF token do?

**Answer:** A server-generated token is associated with the visitor's session and included in the form. The server checks the submitted token against the stored value before processing, helping distinguish the form flow from a forged cross-site request.

### 11. Why should the application avoid putting untrusted values into email headers?

**Answer:** Header values have protocol structure, and untrusted line breaks or content can alter that structure. Keep sender, recipient, and subject controlled by the server; put the visitor's data in the plain-text body.

### 12. What does it mean if a mail transport accepts a message?

**Answer:** It generally means the transport accepted the request for processing. It does not prove that the message reached the recipient's inbox; delivery and bounce status depend on the provider and should be monitored separately.

### 13. Why should contact-form failures not show raw mail or PHP errors to visitors?

**Answer:** Detailed errors can reveal server paths, configuration, or provider information. Show a safe generic message and keep any necessary operational diagnostics in protected logs without recording tokens or full message contents.

### 14. Why should a contact form use server-side rate limiting?

**Answer:** CSRF tokens do not stop bots that can load the form and submit it. A server-side limit helps control repeated submissions and abuse; a hidden field alone is not a reliable rate limit.

### 15. Why should a form collect only the fields it needs?

**Answer:** Every extra field increases privacy and security responsibility, including access control, storage, retention, and breach impact. Minimize collected data and define how long it is kept.

## Form and email scenarios

### 16. A visitor submits an array as the email field. What should the handler do?

**Answer:** Reject it as an invalid field type before passing it to string functions or the email validator. Display a clear field error and do not attempt delivery.

### 17. A hidden CSRF field is present, but the token does not match the session. Should the message still be sent if the other fields look valid?

**Answer:** No. Stop processing before delivery and show a safe error. A hidden field is not trustworthy merely because the normal page generated it.

### 18. A valid-looking email passes the syntax filter. Can the application claim the address is verified?

**Answer:** No. The filter checks syntax only. To verify control of the inbox, send a verification link or code and require the visitor to complete that separate flow.

### 19. A developer uses the visitor's email directly as the message's From header. What should be changed?

**Answer:** Use a fixed, verified From address from protected server configuration. Put the validated visitor address in the body or use a properly configured reply-to mechanism from the mail library.

### 20. A mail transport call returns success and the form shows “Delivered to your inbox.” Is that accurate?

**Answer:** No. The transport may have accepted the message without confirming final delivery. Use wording such as accepted for processing, and use the provider's delivery and bounce events for confirmation.

### 21. A successful form submission redirects, but a timeout occurred while sending. Does redirect-after-post guarantee the message was sent exactly once?

**Answer:** No. A timeout can occur after the provider accepts a message but before the application receives confirmation. For high-impact messages, use a queue and an idempotency strategy.

### 22. The application logs the CSRF token and full message to debug a delivery problem. What should it do instead?

**Answer:** Remove those logs and avoid recording secrets or full personal message contents. Log limited operational details with access controls and a retention policy; inspect delivery status through the configured provider.

## Practical exercise

### 23. List the server-side checks for a name, email, and message field.

**Answer:** Confirm each submitted value is a string, trim whitespace where appropriate, require non-empty values, enforce maximum sizes, and validate the email syntax. Keep the validated values for delivery and escape any value that is shown again in HTML.

### 24. Describe the safe processing order for a CSRF-protected form.

**Answer:** Start or resume the session before output, create or retrieve a random token, render it in the form, accept POST, validate field types, verify the CSRF token, validate the remaining fields, deliver only if validation succeeds, then redirect after an accepted submission. On errors, render escaped old values and useful messages.

### 25. Name three production concerns beyond field validation.

**Answer:** Examples include HTTPS and secure session-cookie settings, server-side rate limiting, protected mail configuration, private error logging, delivery monitoring, privacy and retention policy, accessibility, and a non-production local mail test strategy.

**Explanation:** A form is a complete request-handling feature, not only a group of inputs. It also needs safe operation, understandable feedback, and a plan for abuse and personal data.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [148 - Project: build a personal portfolio page](../lessons/148-project-build-a-personal-portfolio-page.md) | [Lesson](../lessons/149-project-create-a-validated-contact-form.md) | [Example](../examples/149-project-create-a-validated-contact-form.md) | [150 - Project: build a database-backed CRUD application](../lessons/150-project-build-a-database-backed-crud-application.md) |
