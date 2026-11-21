# 125 - Preventing cross-site scripting: Questions and answers

Use these questions to review how XSS reaches a page and how to handle untrusted data in HTML, URLs, and the browser DOM.

## Multiple-choice questions

### 1. What is the main cause of cross-site scripting?

- A. A page displays data from a database.
- B. The browser interprets untrusted data as markup or executable code.
- C. A server uses HTTPS.
- D. A database query returns an empty result.

**Answer:** B. **Explanation:** XSS occurs when untrusted content reaches a browser context that interprets it as code or markup instead of plain data.

### 2. Which PHP function is appropriate for rendering plain text inside ordinary HTML?

- A. htmlspecialchars()
- B. addslashes()
- C. urldecode()
- D. strip_tags() in every context

**Answer:** A. **Explanation:** htmlspecialchars() encodes HTML-significant characters for ordinary HTML text and quoted text attributes. Other contexts need their own handling.

### 3. Which browser API should be used to display a plain text value?

- A. innerHTML
- B. document.write()
- C. textContent
- D. eval()

**Answer:** C. **Explanation:** textContent creates text rather than parsing the value as HTML, when used on an ordinary element.

### 4. What is the correct role of a Content Security Policy?

- A. It replaces output encoding.
- B. It is an additional layer that can limit some script execution and resource loading.
- C. It validates every form field.
- D. It guarantees a page has no XSS.

**Answer:** B. **Explanation:** CSP can reduce impact, but vulnerable rendering still needs to be fixed.

## Core concepts

### 5. What is cross-site scripting?

**Answer:** XSS is a vulnerability that causes a browser to interpret untrusted data as markup or executable code in a page. **Explanation:** The injected content may run with the privileges of the vulnerable site's page and act through that user's browser.

### 6. What are the three common XSS categories?

**Answer:** Stored XSS, reflected XSS, and DOM-based XSS. **Explanation:** They describe whether the data is saved and shown later, echoed in an immediate response, or handled unsafely by browser-side code.

### 7. What is stored XSS?

**Answer:** Untrusted content is saved and later rendered unsafely to visitors. **Explanation:** A comment or profile field can affect each person who views the page.

### 8. What is reflected XSS?

**Answer:** Request data is included in an immediate response without safe handling. **Explanation:** A query parameter or submitted value may be returned to the browser as executable markup.

### 9. What is DOM-based XSS?

**Answer:** Browser JavaScript takes untrusted data from a source such as a URL fragment and sends it to an unsafe DOM or code sink. **Explanation:** The vulnerable operation can occur entirely on the client without the server reflecting the value into HTML.

### 10. Does data from a database automatically count as trusted?

**Answer:** No. **Explanation:** Users or external systems may have influenced the stored value, so it still needs safe handling when rendered.

### 11. What does output encoding do?

**Answer:** It changes special characters so the browser treats a value as data in a particular output context. **Explanation:** The encoded value is interpreted according to the destination's parsing rules rather than as markup or code.

### 12. When should output encoding usually happen?

**Answer:** At the point where the value is rendered. **Explanation:** The same stored value might later appear in HTML, a URL, or another context, each requiring different protection.

### 13. Should an application permanently HTML-encode user input before storing it?

**Answer:** Usually no; store the original value and encode it for the context where it is rendered. **Explanation:** Pre-encoding can corrupt data and does not provide the right protection for every future output context.

## PHP HTML output

### 14. What is a suitable PHP helper for ordinary HTML text?

~~~php
function escapeHtml(string $value): string
{
    return htmlspecialchars(
        $value,
        ENT_QUOTES | ENT_SUBSTITUTE | ENT_HTML5,
        'UTF-8'
    );
}
~~~

**Answer:** A helper like this explicitly encodes quotes, substitutes invalid UTF-8 sequences, selects HTML5 behavior, and specifies UTF-8. **Explanation:** Matching the output document's character encoding avoids ambiguous handling.

### 15. Is htmlspecialchars() useful for ordinary quoted text attributes?

**Answer:** Yes, when the attribute name and element structure are fixed and the value is quoted. **Explanation:** It encodes HTML syntax characters and both quote types, but it does not make code-interpreting or URL-bearing attributes automatically safe.

### 16. Why should tag names and attribute names remain fixed?

**Answer:** They define the structure the browser parses. **Explanation:** Encoding an untrusted attribute name is not a safe substitute for choosing a known, fixed attribute.

### 17. Does HTML encoding make a javascript: URL safe in an href attribute?

**Answer:** No. **Explanation:** HTML encoding protects the attribute's markup syntax; the URL scheme must also be validated and restricted.

### 18. What is the difference between output encoding and HTML sanitization?

**Answer:** Encoding displays input as text, while sanitization removes or restricts markup so an allowed subset of HTML can remain active. **Explanation:** Use encoding for plain text and a maintained, allowlist-based sanitizer only when the feature intentionally supports user-authored HTML.

### 19. Why are regular expressions not a reliable way to sanitize arbitrary HTML?

**Answer:** HTML parsing has complex rules and browser behaviors that simple text replacements do not model. **Explanation:** Use a maintained sanitizer designed for HTML and keep it updated.

## URL, JavaScript, and CSS contexts

### 20. How should a value added to a URL query parameter be handled?

**Answer:** URL-encode the parameter value with an appropriate URL-building function. **Explanation:** This protects URL syntax for that component; the completed URL still needs validation if it can be controlled by a user.

### 21. What checks are needed before putting a user-supplied URL in a link?

**Answer:** Validate the scheme and allowed destination, reject unsafe schemes such as javascript:, then HTML-encode the accepted URL for the quoted attribute. **Explanation:** URL encoding and HTML attribute encoding protect different parsing stages.

### 22. Should untrusted input be concatenated into JavaScript source?

**Answer:** Avoid doing so. **Explanation:** JavaScript parsing differs from HTML parsing, so HTML encoding alone does not make a value safe inside executable script.

### 23. Why should untrusted content not be inserted into CSS source?

**Answer:** It can alter CSS syntax or introduce behavior the application did not intend. **Explanation:** Prefer fixed styles and strictly constrained values through appropriate DOM or CSS APIs.

### 24. Why are inline event-handler attributes such as onclick risky?

**Answer:** Their values are interpreted as JavaScript. **Explanation:** Use a trusted handler registered with addEventListener() and pass untrusted values as data.

### 25. Which operations are examples of unsafe HTML-parsing sinks?

**Answer:** Assigning untrusted strings to innerHTML, outerHTML, insertAdjacentHTML(), or document.write(). **Explanation:** These APIs parse strings as markup; use text APIs or construct nodes safely when displaying plain data.

### 26. Is eval() an appropriate way to process untrusted text?

**Answer:** No. **Explanation:** It interprets a string as JavaScript code and can execute attacker-controlled input.

## Browser DOM and frameworks

### 27. Why is textContent a safer choice for plain text?

**Answer:** On an ordinary element, it creates or updates a text node without parsing the value as HTML. **Explanation:** The browser displays markup-looking characters as text.

### 28. Is textContent safe for every element and every use?

**Answer:** No. Do not use it to set executable script or style content. **Explanation:** Safety depends on the element and its behavior; URL attributes and other special contexts also need their own checks.

### 29. Do template engines that auto-escape make every template safe?

**Answer:** No. Keep auto-escaping enabled, but review raw or unescaped output helpers and custom HTML fragments. **Explanation:** One bypass can reintroduce the vulnerability.

### 30. How should a framework's raw or safe HTML escape hatch be used?

**Answer:** Only for trusted or appropriately sanitized markup when the feature requires HTML. **Explanation:** Such helpers usually disable automatic output encoding.

## Defense in depth and verification

### 31. Does input validation replace output encoding?

**Answer:** No. **Explanation:** Validation enforces expected formats and product rules; output encoding protects the browser context where the value is rendered.

### 32. Does CSP replace safe output handling?

**Answer:** No. **Explanation:** CSP can limit some exploit paths as an additional layer, but it does not remove the unsafe data flow.

### 33. What do HttpOnly and Secure cookie flags do, and what do they not do?

**Answer:** HttpOnly prevents client-side scripts from reading a cookie directly, and Secure restricts the cookie to HTTPS. Neither prevents XSS itself. **Explanation:** Script running in the page may still make same-origin requests as the user.

### 34. How should SameSite cookie behavior be understood here?

**Answer:** It can reduce some cross-site cookie sending, but it is not an XSS fix. **Explanation:** XSS runs in the vulnerable site's page, so safe rendering remains necessary.

### 35. What should a review do for each untrusted value?

**Answer:** Trace it to every server-rendered and browser-side output, identify the exact context, and verify the matching safe method. **Explanation:** One unprotected rendering path can leave the application vulnerable.

### 36. What should a test use to check ordinary HTML text rendering?

**Answer:** A string containing characters significant to HTML, then verify that it is displayed as text and does not create executable markup. **Explanation:** Check the actual output location and browser behavior rather than only testing input validation.

### 37. Why should stored and reflected values both be tested?

**Answer:** They travel through different application paths, so one may be encoded while another is not. **Explanation:** Testing each path helps find gaps in output handling.

### 38. What is the main rule for preventing XSS?

**Answer:** Keep untrusted values as data and apply the right output handling for every destination context. **Explanation:** Use HTML encoding for ordinary HTML, safe text APIs for plain DOM text, URL validation for links, and a maintained sanitizer only when user-authored HTML is required.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [124 - Preventing SQL injection](../lessons/124-preventing-sql-injection.md) | [Lesson](../lessons/125-preventing-cross-site-scripting.md) | [Example](../examples/125-preventing-cross-site-scripting.md) | [126 - Preventing cross-site request forgery](../lessons/126-preventing-cross-site-request-forgery.md) |