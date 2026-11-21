# 126 - Preventing cross-site request forgery: Questions and answers

Use these questions to review how CSRF abuses browser credentials and how PHP applications can require a valid token before changing state.

## Multiple-choice questions

### 1. What makes a cookie-authenticated application vulnerable to CSRF?

- A. Browsers may attach the site's authentication cookie to a request the user did not intend.
- B. A database has several tables.
- C. A page uses HTML forms.
- D. PHP sessions always expire.

**Answer:** A. **Explanation:** The server may see a valid session cookie and mistake a forged browser request for an intentional user action.

### 2. Which value is appropriate for generating a CSRF token in PHP?

- A. The current timestamp.
- B. rand().
- C. random_bytes().
- D. The user's email address.

**Answer:** C. **Explanation:** random_bytes() provides cryptographically secure random bytes suitable for an unpredictable token.

### 3. What should a state-changing POST handler do when the token is missing or invalid?

- A. Continue if the user is signed in.
- B. Reject the request before performing the action.
- C. Create a new token from the submitted value.
- D. Retry the request using GET.

**Answer:** B. **Explanation:** CSRF validation must happen before side effects and fail closed.

### 4. What is SameSite's role in CSRF protection?

- A. It encrypts the request body.
- B. It can reduce some cross-site cookie sending as an additional defense.
- C. It checks that every form contains a token.
- D. It prevents XSS.

**Answer:** B. **Explanation:** SameSite helps in some browser request contexts but does not replace tokens or safe route design.

## Core concepts

### 5. What is cross-site request forgery?

**Answer:** CSRF tricks a browser into sending an unwanted request to a site where the user is authenticated. **Explanation:** The browser may attach the site's session cookie, making the server believe the action came from the user.

### 6. Why might an attacker not need to read the application's response?

**Answer:** The attack may only need the browser to send a state-changing request. **Explanation:** The same-origin policy can prevent reading a response while still allowing some cross-origin requests to be sent.

### 7. What is the synchronizer-token pattern?

**Answer:** The server stores an unpredictable token in the user's session, includes it in a form or trusted same-origin request, and validates it when the state-changing request arrives. **Explanation:** An unrelated site can cause a request but should not be able to read the application's page and obtain its token.

### 8. Which actions need CSRF protection?

**Answer:** All state-changing operations that rely on browser-attached credentials, such as profile changes, purchases, account changes, and logout. **Explanation:** Protect every route that performs the action, not just the most visible form.

### 9. Should a GET route delete a record or update an account?

**Answer:** No. **Explanation:** Safe methods such as GET should retrieve information without changing server state, and links can be visited or prefetched unexpectedly.

### 10. Does using POST by itself prevent CSRF?

**Answer:** No. **Explanation:** POST is an appropriate method for many state changes, but the server still needs a CSRF defense.

### 11. Does an authenticated session prove the user intended a request?

**Answer:** No. **Explanation:** A forged request may carry the user's session cookie automatically.

## Tokens in PHP

### 12. How can PHP create a printable CSRF token from secure random bytes?

~~~php
$token = bin2hex(random_bytes(32));
~~~

**Answer:** Generate 32 cryptographically secure random bytes and encode them as hexadecimal. **Explanation:** The result is unpredictable and printable for use in a form field.

### 13. Where should a synchronizer token be stored?

**Answer:** In server-side session state associated with the authenticated user. **Explanation:** The server needs a trusted value to compare with the submitted token.

### 14. How is a token commonly included in an HTML form?

**Answer:** As a hidden field in a POST form. **Explanation:** The browser returns the field with the request, and the server validates it before changing state.

### 15. Should a CSRF token be placed in a URL query string?

**Answer:** No. **Explanation:** URLs can appear in browser history, logs, analytics, copied links, and referrer data.

### 16. Why should a simple application often keep one token stable for the session?

**Answer:** It avoids invalidating forms open in other tabs or pages reached with the Back button. **Explanation:** Per-request tokens can reduce a replay window but may create usability problems; a framework can manage more advanced choices.

### 17. What should happen if the token in the request is an array rather than a string?

**Answer:** Reject the request. **Explanation:** Validate request shape before comparison so malformed input cannot trigger unexpected behavior or bypass checks.

### 18. At what point should a handler validate its CSRF token?

**Answer:** Before performing any state-changing operation. **Explanation:** If the application changes data first, rejecting the token afterward is too late.

### 19. How should a PHP handler compare the submitted token with the session token?

**Answer:** Use hash_equals() with the known session token first and the user-supplied token second. **Explanation:** This performs a timing-safe string comparison.

### 20. What should the server do when a token is absent, malformed, or mismatched?

**Answer:** Reject the request, commonly with a forbidden response, and perform no protected action. **Explanation:** Do not silently skip validation or create a replacement from untrusted input.

### 21. Is CSRF token validation a substitute for field validation?

**Answer:** No. **Explanation:** After the token is valid, validate the submitted values according to application rules.

### 22. Is CSRF token validation a substitute for authorization?

**Answer:** No. **Explanation:** A valid token helps show that a request came through the application's session, but the user must still be allowed to perform the requested action.

## Cookies and request origins

### 23. What does the Secure session-cookie attribute do?

**Answer:** It tells browsers to send the cookie only over secure HTTPS connections. **Explanation:** Configure it for an HTTPS application so the session identifier is not sent over plain HTTP.

### 24. What does HttpOnly do for a session cookie?

**Answer:** It prevents ordinary client-side scripts from reading the cookie directly. **Explanation:** It can limit direct cookie theft but does not stop XSS code from making same-origin requests.

### 25. What does SameSite do?

**Answer:** It asks the browser to limit when a cookie is sent with cross-site requests. **Explanation:** It can reduce CSRF exposure but browser behavior and request context matter, so retain a token defense for protected actions.

### 26. Why must GET routes remain safe even when SameSite=Lax is set?

**Answer:** Lax cookies may still accompany some cross-site top-level navigations using safe methods. **Explanation:** If a GET request changes state, that behavior can undermine the intended protection.

### 27. What can an Origin check add?

**Answer:** It can verify that a browser request came from the application's expected origin. **Explanation:** Compare the full scheme, host, and port against a trusted configured value.

### 28. Why should a server avoid building its trusted origin from an unverified Host header?

**Answer:** A request may contain a hostile or manipulated host value. **Explanation:** Behind a proxy, trust only correctly configured proxy headers or use a server-side configured origin.

### 29. Should an application accept the literal Origin value null as trusted?

**Answer:** No. **Explanation:** Opaque origins and redirects can produce null, and an attacker can create some such contexts.

### 30. What should a protected endpoint do if Origin and Referer are missing?

**Answer:** Follow an explicit policy and require another valid defense, such as the session-bound token. **Explanation:** Missing origin information does not prove a request is same-origin.

## CORS, APIs, and related attacks

### 31. Is CORS by itself a complete CSRF defense?

**Answer:** No. **Explanation:** Cross-origin form requests can still be sent even when the attacker cannot read the response.

### 32. Do cookie-authenticated JSON APIs need a CSRF review?

**Answer:** Yes. **Explanation:** Content type alone is not a reliable defense; examine accepted request types, credentials, CORS settings, and server-side token checks.

### 33. How can a same-origin JavaScript client send a CSRF token?

**Answer:** It can send the token in a custom request header to explicitly trusted application endpoints. **Explanation:** The server must validate it, and the client must not attach it to arbitrary destinations.

### 34. Do APIs using bearer tokens have the same CSRF profile as cookie-authenticated APIs?

**Answer:** Not necessarily. **Explanation:** A bearer token explicitly attached by application code is not automatically sent like a cookie, but storage and client behavior must still be reviewed.

### 35. Can XSS defeat a CSRF token?

**Answer:** Often yes. **Explanation:** Script running in the application's origin may be able to read a token from the page or make a request that includes it, so XSS needs its own defenses.

### 36. What is the difference between a synchronizer token and a double-submit cookie pattern?

**Answer:** A synchronizer token is stored in server-side session state and compared with a submitted value. A double-submit design validates a token submitted both in a cookie and request, with additional integrity protections depending on the design. **Explanation:** Do not treat a token cookie by itself as equivalent to a session-bound synchronizer token.

### 37. What should a CSRF test verify for a protected action?

**Answer:** A valid token allows the expected action, while a missing or invalid token is rejected and leaves state unchanged. **Explanation:** Test the side effect as well as the response code.

### 38. What is the main rule for preventing CSRF?

**Answer:** Require a server-validated, unpredictable token for state-changing requests that rely on browser-attached credentials, and keep safe HTTP methods free of side effects. **Explanation:** Add cookie and origin checks as defense in depth, and keep validation, authorization, and XSS defenses separate.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [125 - Preventing cross-site scripting](../lessons/125-preventing-cross-site-scripting.md) | [Lesson](../lessons/126-preventing-cross-site-request-forgery.md) | [Example](../examples/126-preventing-cross-site-request-forgery.md) | [127 - Authentication, authorization, and access control](../lessons/127-authentication-authorization-and-access-control.md) |