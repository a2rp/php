# 128 - Session and cookie security: Questions and answers

Use these questions to review how PHP sessions work, how cookie attributes protect session IDs, and how an application should expire and invalidate sessions.

## Multiple-choice questions

### 1. In a typical PHP session, what does the browser's session cookie contain?

- A. The user's password.
- B. All of the user's profile data.
- C. An opaque session identifier.
- D. The PHP source code.

**Answer:** C. **Explanation:** PHP uses the identifier to find session state stored on the server; the cookie should not carry the user's password or application permissions.

### 2. What does session.use_strict_mode help PHP do?

- A. Reject session IDs that PHP has not initialized.
- B. Encrypt every database record.
- C. Prevent all cross-site scripting.
- D. Make every session permanent.

**Answer:** A. **Explanation:** Strict mode helps prevent PHP from accepting an attacker-supplied, uninitialized session ID.

### 3. What does the Secure cookie attribute do?

- A. Makes the cookie unreadable to the web server.
- B. Tells the browser to send the cookie only over HTTPS.
- C. Adds a user permission to the session.
- D. Ends the session after one request.

**Answer:** B. **Explanation:** Secure helps protect the cookie in transit, but the site still needs HTTPS for the entire authenticated session.

## Session fundamentals and configuration

### 4. What is a PHP session?

**Answer:** A server-side state mechanism that lets PHP associate related requests using a session identifier. **Explanation:** The browser usually sends the identifier in a cookie, and PHP uses it to retrieve the server-side data.

### 5. Why is a session ID considered a bearer credential?

**Answer:** A person who obtains a valid session ID may be able to use the corresponding session. **Explanation:** Treat it as a secret, even though it is only an identifier and not the user's password.

### 6. Should a session cookie store the user's password or permissions?

**Answer:** No. **Explanation:** Keep application state on the server and use an opaque ID to look it up; a cookie containing sensitive state can expose it or invite client-side tampering.

### 7. When should PHP session settings and cookie parameters be configured?

**Answer:** Before calling session_start(). **Explanation:** PHP needs the configuration before it reads the current session or sends the session cookie in response headers.

### 8. What does session.use_only_cookies do?

**Answer:** It tells PHP to accept session IDs through cookies rather than URL or request-body session ID mechanisms. **Explanation:** Keeping the identifier out of URLs reduces leakage through browser history, referrers, and logs.

### 9. Does session.use_strict_mode replace session.use_only_cookies?

**Answer:** No. **Explanation:** Strict mode rejects unknown IDs; cookie-only mode limits how PHP accepts the ID. They address different parts of session handling.

### 10. Why should session_start() happen before output?

**Answer:** PHP may need to send or update a session cookie in the response headers. **Explanation:** Headers cannot reliably be changed after output has been sent.

## Cookie attributes and HTTPS

### 11. What does HttpOnly do?

**Answer:** It prevents ordinary browser scripts from reading the cookie through document.cookie. **Explanation:** It helps protect cookie confidentiality, but injected script may still make requests as the user.

### 12. Does HttpOnly prevent XSS from performing actions?

**Answer:** No. **Explanation:** A script running in the application's origin may still issue requests, and the browser may attach the HttpOnly cookie automatically.

### 13. What is the purpose of SameSite?

**Answer:** It controls whether a browser sends a cookie in some cross-site request contexts. **Explanation:** It can reduce some CSRF risk, but applications still need their CSRF defenses for state-changing requests.

### 14. What must be true when SameSite=None is used?

**Answer:** The cookie must also be Secure. **Explanation:** Cross-site cookies should be enabled only when the application actually requires them and is served over HTTPS.

### 15. Should every application use SameSite=Strict?

**Answer:** Not necessarily. **Explanation:** Strict blocks more cross-site contexts but can interfere with legitimate navigation or external sign-in flows; choose a policy that fits the application.

### 16. Why should the Domain attribute be omitted unless needed?

**Answer:** Omitting it creates a host-only cookie that is not sent to every subdomain. **Explanation:** A broader domain scope exposes the session cookie to more hosts than necessary.

### 17. Does the cookie Path attribute provide a strong security boundary between apps?

**Answer:** No. **Explanation:** Path controls when the browser sends a cookie, but applications should not rely on it as complete isolation between applications on one host.

### 18. What constraints does a __Host- cookie prefix require in supporting browsers?

**Answer:** The cookie must be Secure, use Path=/, and omit Domain. **Explanation:** The browser enforces those attributes to reduce some cookie injection and downgrade risks.

### 19. Does a browser-session cookie guarantee the user is logged out when the browser closes?

**Answer:** No. **Explanation:** Browser session restore may preserve session cookies; enforce expiry and logout on the server.

### 20. Is setting Secure enough if the rest of the site still uses HTTP?

**Answer:** No. **Explanation:** Use HTTPS for the entire authenticated session and configure redirects before issuing session credentials.

## Session lifecycle

### 21. What is session fixation?

**Answer:** An attack where someone tries to make a victim use a session identifier the attacker already knows. **Explanation:** A successful login should replace a pre-authentication identifier before adding authenticated state.

### 22. When should the session ID be rotated?

**Answer:** After login and important privilege changes. **Explanation:** Rotating at those transitions helps prevent an older identifier from continuing to represent the more privileged session.

### 23. Should an application regenerate the session ID on every request?

**Answer:** No. **Explanation:** It can disrupt concurrent requests and tabs; rotate at meaningful transitions and follow the framework's lifecycle guidance.

### 24. Why can deleting an old session immediately during rotation cause problems?

**Answer:** A concurrent request or unstable connection may still be using it. **Explanation:** Session rotation and old-ID invalidation need a lifecycle strategy that fits the application's request patterns.

### 25. What is an idle timeout?

**Answer:** A limit on how long a session may remain inactive. **Explanation:** Store the last activity time on the server and check it on each authenticated request.

### 26. What is an absolute timeout?

**Answer:** A maximum session lifetime measured from its creation or authentication time, even if requests keep arriving. **Explanation:** It limits how long a continuously used or stolen session can remain valid.

### 27. Does PHP's session garbage collector enforce an authentication timeout?

**Answer:** No. **Explanation:** Garbage collection removes old stored session data opportunistically; application code must enforce idle and absolute expiry before serving protected requests.

### 28. Is JavaScript's session countdown sufficient to end a session?

**Answer:** No. **Explanation:** Client-side code can be disabled or modified, so the server must enforce expiration.

### 29. What should happen after a session timeout?

**Answer:** The server should reject the expired session, invalidate it according to the session policy, and require authentication again. **Explanation:** Do this before returning protected data or performing an action.

## Logout, storage, and monitoring

### 30. What are the key parts of logout?

**Answer:** Clear the in-memory session data, invalidate the server-side session, and expire the browser cookie using its original scope. **Explanation:** Each step handles a different place where the session may remain usable.

### 31. Does session_destroy() alone clear $_SESSION and the browser cookie?

**Answer:** No. **Explanation:** Clear the session array and explicitly expire the cookie as well as destroying server-side session data.

### 32. Why must cookie deletion use the same path and domain as cookie creation?

**Answer:** A cookie is identified partly by its name and scope. **Explanation:** A deletion with a different scope may leave the original cookie in the browser.

### 33. Should logout be protected against CSRF?

**Answer:** Yes, for a browser session that uses automatically attached cookies. **Explanation:** Logout is state-changing, so a cross-site request should not be able to trigger it unexpectedly.

### 34. Does deleting the cookie in one browser revoke a copied session ID elsewhere?

**Answer:** No. **Explanation:** To support “log out all devices,” the application needs server-side session tracking and revocation.

### 35. Where should PHP session files be stored?

**Answer:** Outside the public web root, with access restricted to the PHP worker and appropriate system processes. **Explanation:** A web request should not be able to download raw session storage.

### 36. What should an application with several web servers consider?

**Answer:** Use a consistent shared session store or another session mechanism all servers can validate and update safely. **Explanation:** Local session files on one server may not be available when a later request reaches another server.

### 37. Why should session IDs not be logged or placed in URLs?

**Answer:** Logs and URLs can be copied, retained, shared, or sent in referrer information. **Explanation:** Anyone who obtains an unexpired ID may be able to reuse the session.

### 38. Should authentication tokens be stored in localStorage instead of an HttpOnly cookie?

**Answer:** Not as a blanket security improvement. **Explanation:** JavaScript in the same origin can read localStorage, so an XSS flaw may expose the token; choose a credential design based on the application's threat model.

### 39. What does Cache-Control: no-store help with on authenticated pages?

**Answer:** It tells browser and intermediary caches not to store the response. **Explanation:** It can reduce later access to cached sensitive pages, but it cannot erase screenshots or data a user already copied.

### 40. Does a stable IP address prove that the same user still owns a session?

**Answer:** No. **Explanation:** Addresses can change or be shared; use IP and device signals only as risk inputs, not as the authentication credential itself.

### 41. What should an access-control check do when a session is expired?

**Answer:** Deny the request before accessing protected data or causing a side effect. **Explanation:** A timeout is ineffective if the handler performs the action before checking it.

### 42. What is the main session-security rule?

**Answer:** Protect the session ID like a credential, and manage its full server-side lifecycle from creation through expiration and logout. **Explanation:** Cookie attributes, HTTPS, rotation, timeouts, CSRF defenses, and careful storage work together.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [127 - Authentication, authorization, and access control](../lessons/127-authentication-authorization-and-access-control.md) | [Lesson](../lessons/128-session-and-cookie-security.md) | [Example](../examples/128-session-and-cookie-security.md) | [129 - Path traversal and filesystem security](../lessons/129-path-traversal-and-filesystem-security.md) |