# Project: build a personal portfolio page — Q&A

Check your understanding of the portfolio project, PHP rendering, HTML safety, accessibility, and deployment. Each answer follows its question.

## Multiple-choice questions

### 1. What does PHP do when the portfolio page is requested?

- A. It sends PHP source code to the visitor's browser.
- B. It runs on the server and returns an HTML response.
- C. It automatically creates a database.
- D. It replaces the need for HTML.
**Answer: B.** PHP generates the response on the server. The browser receives and renders the resulting HTML, not the PHP source.

### 2. Why should the web server's document root point to the public directory?

- A. To make application source and private files directly downloadable.
- B. To expose only intended public files while keeping source code outside the web root.
- C. To prevent PHP from running.
- D. To make CSS private.
**Answer: B.** A separate public directory reduces the chance that source files or private configuration are served directly.

### 3. What is the purpose of htmlspecialchars when rendering a project description as HTML text?

- A. It validates that the description is true.
- B. It converts special HTML characters so text is displayed as text instead of interpreted as markup.
- C. It checks whether a URL uses HTTPS.
- D. It removes all punctuation.
**Answer: B.** HTML escaping protects the output context by encoding characters such as angle brackets. It is not general-purpose validation or URL checking.

### 4. Is HTML escaping enough to make an arbitrary project URL safe?

- A. Yes, for every URL and every HTML context.
- B. No. The application should also allow only expected URL schemes or construct destinations from trusted data.
- C. Yes, if the URL has a title.
- D. No, URLs should never be used in a portfolio.
**Answer: B.** Escaping prevents HTML syntax from being interpreted as markup, but it does not decide whether a destination such as a javascript URL is acceptable.

### 5. What is PHP's built-in web server intended for?

- A. Development and controlled demonstrations.
- B. Permanent public production hosting.
- C. Replacing a deployment review.
- D. Storing project data.
**Answer: A.** Use it to run the project locally. A production application should use a properly configured PHP-capable web server.

### 6. Which element should contain the primary content of the page?

- A. The main element.
- B. The title element.
- C. The style element.
- D. The script element.
**Answer: A.** The main element identifies the page's primary content and provides a useful landmark for assistive technology.

## Short-answer questions

### 7. Why store project details in a PHP array instead of repeating the same HTML for every project?

**Answer:** A structured list keeps project content separate from presentation and lets one template render consistent cards. Updating a project does not require copying and editing a separate block of markup.

### 8. What should a portfolio do when a project has no public destination URL?

**Answer:** Show the project name and description without creating a fake link. Do not use a placeholder destination such as a bare hash that appears interactive but goes nowhere.

### 9. What is the difference between escaping text and validating data?

**Answer:** Escaping prepares a value for a specific output context, such as HTML text. Validation checks whether a value meets the application's rules, such as whether a URL scheme is allowed. One does not replace the other.

### 10. Why should a PHP include path not come directly from a request parameter?

**Answer:** An untrusted path may cause the application to include or expose an unintended file. Use fixed include paths built from the current source file's location.

### 11. What should a personal portfolio consider before publishing contact details?

**Answer:** Publish only the details needed for the portfolio's purpose and consider spam, unwanted contact, and privacy exposure. A public email address may be copied or collected by automated systems.

### 12. Why are semantic HTML elements useful?

**Answer:** Elements such as header, nav, main, section, and footer describe the page structure. This helps browsers, assistive technology, and maintainers understand the content and navigation.

### 13. What is a useful heading structure for a one-page portfolio?

**Answer:** Use one clear page-level heading, then section headings that describe the content, with nested headings only when there is a real subsection. Choose heading levels for document structure rather than visual size.

### 14. What is the purpose of the escape helper's UTF-8 encoding argument?

**Answer:** It tells htmlspecialchars which character encoding to use while converting special characters. Explicitly using UTF-8 makes the intended document encoding clear and helps handle text consistently.

## Project scenarios

### 15. A project title contains the text &lt;script&gt;alert(1)&lt;/script&gt;. What should happen when it is rendered as a title?

**Answer:** The page should display the characters as text, not execute them as a script. Escape the value for HTML output before inserting it into the document.

### 16. A portfolio link is supplied as javascript:alert(1). Is escaping it enough?

**Answer:** No. Validate or construct link destinations so only expected schemes, such as HTTPS, are permitted. HTML escaping alone does not make an unsafe URL scheme acceptable.

### 17. A visitor says the portfolio is difficult to use without a mouse. What should you review?

**Answer:** Check that links and controls are keyboard reachable, focus is visible, navigation order is logical, and interactive content uses real links or buttons. Also verify headings and landmarks make sense when read in sequence.

### 18. A page looks good on a desktop but clips project cards on a phone. What should change?

**Answer:** Review flexible widths, columns, spacing, and media breakpoints. Start with a readable narrow-screen layout and let the design expand at wider viewports.

### 19. The entire project directory is configured as the web document root. Why should this be corrected?

**Answer:** Files that are meant to remain server-side, such as source code or configuration, may become requestable. Configure the document root to the public directory and keep application code outside it.

### 20. The portfolio project scope includes adding a message form and sending email. Should that be part of the first static version?

**Answer:** Not unless it is a real requirement. A form adds input validation, spam, privacy, mail delivery, abuse prevention, and error-handling work. Keep the first version static and plan a secure form as a separate feature if needed.

### 21. A page has several section headings, all marked as h1 only because they should look large. What is the issue?

**Answer:** Heading tags express document hierarchy, not just appearance. Use a logical heading structure and use CSS to control size.

## Practical exercise

### 22. Write an escape helper for HTML text and explain when to use it.

**Answer:**

~~~php
function escape(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE,
        'UTF-8'
    );
}
~~~

Call the helper when inserting dynamic text into HTML text or a quoted HTML attribute. URL schemes still need separate validation, and JavaScript or CSS contexts need their own safe handling.

### 23. Your portfolio contains a project with a name, description, and URL. What checks should you make before rendering it as a linked card?

**Answer:** Escape the name and description for HTML output. Confirm the URL uses an allowed scheme or was built from trusted local data. If there is no valid destination, render the content without a link. Use descriptive link text and ensure the card remains usable with a keyboard.

### 24. Give a local development command for this project's file layout and name one limitation.

**Answer:** From the project root, run:

~~~sh
php -S localhost:8000 -t public
~~~

This starts PHP's built-in development server with the public directory as its document root. It is for local development or controlled demonstrations, not a public production server.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [147 - Code review, maintenance, and upgrades](../lessons/147-code-review-maintenance-and-upgrades.md) | [Lesson](../lessons/148-project-build-a-personal-portfolio-page.md) | [Example](../examples/148-project-build-a-personal-portfolio-page.md) | [149 - Project: create a validated contact form](../lessons/149-project-create-a-validated-contact-form.md) |
