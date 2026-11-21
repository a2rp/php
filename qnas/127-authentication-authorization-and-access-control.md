# 127 - Authentication, authorization, and access control: Questions and answers

Use these questions to review how PHP applications verify identities, establish sessions, and make access decisions for particular actions and resources.

## Multiple-choice questions

### 1. What does authentication establish?

- A. Whether a user may delete any record.
- B. Whether evidence supports the claimed identity.
- C. Whether a database query is valid.
- D. Whether a page has a navigation menu.

**Answer:** B. **Explanation:** Authentication verifies an identity claim, for example by checking a password or another credential.

### 2. A signed-in member changes a document ID in the URL. What must the application do?

- A. Assume the member is allowed because they are authenticated.
- B. Check whether that member may perform the requested action on that document.
- C. Allow access if the ID is hard to guess.
- D. Check authorization only when rendering the page link.

**Answer:** B. **Explanation:** Authentication alone does not grant access to every resource. The server must enforce the resource-specific policy on the request.

### 3. Which PHP API should verify a password against its stored password hash?

- A. hash().
- B. password_verify().
- C. htmlspecialchars().
- D. session_id().

**Answer:** B. **Explanation:** password_verify() checks a submitted password against a hash created by PHP's password hashing APIs.

## Identity and access-control concepts

### 4. What is identification?

**Answer:** Identification is the identity a person or service claims, such as an account email address. **Explanation:** The application still needs to verify evidence for that claim.

### 5. What is authentication?

**Answer:** Authentication verifies that a claimed identity is supported by valid evidence. **Explanation:** A password, passkey, or second factor can provide that evidence.

### 6. What is authorization?

**Answer:** Authorization decides whether an authenticated identity may perform a specific action on a resource. **Explanation:** A successful login does not automatically permit viewing or changing every record.

### 7. What does access control mean?

**Answer:** Access control is the set of policies and checks that enforce authorization decisions. **Explanation:** It includes server-side rules for routes, actions, and individual resources.

### 8. Does a successful login mean the user can access every page and record?

**Answer:** No. **Explanation:** Login establishes the identity for a request; the application must still evaluate the identity's permission for each action and resource.

### 9. What does “deny by default” mean?

**Answer:** Reject access unless a policy explicitly allows it. **Explanation:** Missing, incomplete, or failed permission data should not accidentally grant access.

### 10. What is the principle of least privilege?

**Answer:** Give each account only the actions and data it needs. **Explanation:** A limited account has less potential impact if it is misused or compromised.

## Passwords and login

### 11. How should a PHP application store a new password?

**Answer:** Store a value returned by password_hash(), normally using PASSWORD_DEFAULT or a deliberately selected supported algorithm. **Explanation:** PHP generates the password hash and encodes the algorithm details needed for later verification.

### 12. Should an application create its own salt for password_hash()?

**Answer:** No. **Explanation:** PHP's password APIs manage salts as part of the generated hash; application-created salts add complexity and can cause mistakes.

### 13. Why should passwords not be stored with a fast digest such as MD5 or SHA-256?

**Answer:** Fast general-purpose digests allow attackers to test many guesses quickly after a database leak. **Explanation:** Password hashing needs algorithms and costs designed to slow offline guessing.

### 14. Which PHP function checks a submitted password?

**Answer:** password_verify($submittedPassword, $storedHash). **Explanation:** It safely checks the password against the stored hash without the application comparing raw hash strings itself.

### 15. Why should a login error avoid saying whether the email or password was wrong?

**Answer:** A generic failure message makes account enumeration harder. **Explanation:** Distinct errors can reveal which account names exist to an attacker.

### 16. When should password_needs_rehash() be used?

**Answer:** After a password has been successfully verified, to determine whether to replace its stored hash with one using the current algorithm or cost. **Explanation:** Rehashing on successful login can upgrade old hashes without knowing the plaintext password in advance.

### 17. Why should login attempts be rate-limited?

**Answer:** Rate limits make repeated automated password guesses more difficult. **Explanation:** They should be designed to reduce abuse without creating an easy denial-of-service path against a victim's account.

### 18. Should a password reset email contain the user's current password?

**Answer:** No. **Explanation:** The application should not be able to recover a plaintext password; use a time-limited, single-use reset flow instead.

## Sessions

### 19. What should a session contain after login?

**Answer:** The minimum server-side state needed to identify the authenticated account, such as its user ID. **Explanation:** Do not store or repeatedly reuse the submitted password as session state.

### 20. Why regenerate the session ID after a successful login?

**Answer:** To reduce session-fixation risk by replacing an identifier that may have been known or supplied before authentication. **Explanation:** Call session_regenerate_id() before marking the session authenticated.

### 21. When else should the session ID be regenerated?

**Answer:** After important privilege changes, such as elevating an account's permissions. **Explanation:** Rotating the identifier limits reuse of a session associated with the old privilege state.

### 22. What is the session ID's security role?

**Answer:** It is a bearer credential for the session. **Explanation:** Anyone who obtains it may be able to act as that session, so keep it out of URLs, logs, and analytics.

### 23. What does the Secure cookie attribute do?

**Answer:** It tells browsers to send the cookie only over HTTPS connections. **Explanation:** Production deployments should use HTTPS and configure this setting according to trusted server or proxy configuration.

### 24. What does HttpOnly do for a session cookie?

**Answer:** It prevents ordinary browser scripts from reading the cookie through JavaScript APIs. **Explanation:** It reduces cookie theft through some script attacks, but does not prevent every XSS impact.

### 25. Does SameSite replace CSRF tokens?

**Answer:** No. **Explanation:** SameSite can reduce some cross-site cookie sending, but state-changing cookie-authenticated actions still need the application's CSRF policy.

### 26. Should a session ID appear in a URL?

**Answer:** No. **Explanation:** URLs may be copied, logged, stored in browser history, or sent in referrer data, which can expose a bearer credential.

### 27. What should logout do?

**Answer:** Invalidate the server-side session and expire the session cookie. **Explanation:** Removing only a visible login link or client-side value does not reliably invalidate the credential.

### 28. Should a logout route be protected against CSRF?

**Answer:** Yes, when browser credentials are attached automatically. **Explanation:** Logout changes state and a cross-site request could otherwise force a user out unexpectedly.

## Authorization and object-level access

### 29. Where should an authorization check happen?

**Answer:** On the server, in the handler or service that performs the protected action. **Explanation:** The client can skip hidden buttons and call a route or API directly.

### 30. Is hiding an Admin link enough to protect an administrator route?

**Answer:** No. **Explanation:** Interface visibility is not an access-control boundary; the server must check the current user's permission.

### 31. What is an insecure direct object reference (IDOR)?

**Answer:** It is an access-control flaw where an application accepts a resource identifier without checking that the current user may access that resource. **Explanation:** An attacker may change a URL or request field to refer to someone else's record.

### 32. Do random or unguessable record IDs replace authorization checks?

**Answer:** No. **Explanation:** Identifiers may be leaked, shared, or discovered, and the application still needs to enforce who may use them.

### 33. How can an owner-only database lookup reduce accidental IDORs?

**Answer:** Include both the requested record ID and authenticated owner ID in the query condition. **Explanation:** A record that belongs to another user will not be returned by the scoped query.

### 34. Where should the current user ID come from?

**Answer:** From the authenticated server-side session or trusted authentication context. **Explanation:** Do not accept the acting identity from a user-controlled form field, query parameter, or header.

### 35. Where should roles and permissions come from?

**Answer:** From trusted server-side account and policy data. **Explanation:** A browser-submitted value such as role=admin is just untrusted input and must not grant privileges.

### 36. Are roles always enough for authorization?

**Answer:** No. **Explanation:** A role may need to be combined with ownership, tenant membership, workflow state, and the specific action being requested.

### 37. What is the difference between 401 and 403?

**Answer:** 401 indicates that a valid authenticated identity is missing; 403 indicates the identity is known but the action is not allowed. **Explanation:** Some browser apps redirect unauthenticated users, and some systems intentionally use 404 to conceal resource existence.

### 38. Why might an application return 404 for another user's private record?

**Answer:** To avoid revealing that the record exists. **Explanation:** The response policy can conceal existence, but the application must still deny the operation and avoid leaking the record.

### 39. Should an authorization error be returned after the update already happened?

**Answer:** No. **Explanation:** Check permission before reading protected data or performing side effects; a denial must leave protected state unchanged.

## Review and testing

### 40. What access cases should be tested for an owner-protected document?

**Answer:** Test an anonymous request, the owner's allowed request, another member's denied request, and any explicitly permitted administrator path. **Explanation:** Also try direct URLs, API calls, changed IDs, tenant changes, expired sessions, and client-supplied roles.

### 41. What should a denied request do?

**Answer:** Stop before protected data is disclosed or a state change occurs. **Explanation:** A denial response is not sufficient if the application has already completed the sensitive operation.

### 42. What is the main rule to remember?

**Answer:** Verify identity with secure authentication, then authorize each requested action against trusted identity and resource data on the server. **Explanation:** Deny by default, use least privilege, and test the denied paths as carefully as the allowed ones.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [126 - Preventing cross-site request forgery](../lessons/126-preventing-cross-site-request-forgery.md) | [Lesson](../lessons/127-authentication-authorization-and-access-control.md) | [Example](../examples/127-authentication-authorization-and-access-control.md) | [128 - Session and cookie security](../lessons/128-session-and-cookie-security.md) |