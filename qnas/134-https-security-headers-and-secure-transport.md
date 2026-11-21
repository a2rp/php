# 134 - HTTPS, security headers, and secure transport: Questions and answers

Use these questions to review TLS, HTTPS redirects, HSTS, Content Security Policy, secure cookies, and certificate verification in PHP.

## Multiple-choice questions

### 1. What does HTTPS with correctly validated TLS provide?

- A. Encryption in transit, integrity protection, and authentication of the server.
- B. Guaranteed security of the PHP application.
- C. Authentication of every visitor.
- D. Protection from all malicious browser scripts.

**Answer:** A. **Explanation:** TLS protects traffic between endpoints and lets the client verify the server identity. It does not automatically authenticate the user or fix vulnerable application code.

### 2. What should a PHP cURL client do when an HTTPS certificate is expired or does not match the hostname?

- A. Disable certificate checks and continue.
- B. Fail the request and report a controlled error.
- C. Retry the request over HTTP.
- D. Accept the certificate if the response is JSON.

**Answer:** B. **Explanation:** Certificate and hostname validation help the client detect an impersonated server. Do not disable them to work around a certificate or CA configuration problem.

### 3. When should a browser receive an HSTS header?

- A. On an HTTP response before the redirect.
- B. On an HTTPS response from the host covered by the policy.
- C. Only when a user logs in.
- D. On every third-party request.

**Answer:** B. **Explanation:** Browsers accept HSTS policy over a secure connection. It tells a browser that has received the policy to use HTTPS for that host for the configured period.

### 4. What is a safe rollout approach for HSTS?

- A. Start immediately with a long max-age, includeSubDomains, and preload for every domain.
- B. Begin with a short max-age, confirm HTTPS and certificate operations, then expand deliberately.
- C. Send HSTS only on HTTP pages.
- D. Enable HSTS as a substitute for a certificate.

**Answer:** B. **Explanation:** Long-lived policies and subdomain coverage can lock browsers to HTTPS. Confirm all covered hosts and renewal processes before increasing the scope.

### 5. What is the purpose of Content Security Policy?

- A. To encrypt HTTP responses.
- B. To restrict which resource sources a browser may load and execute.
- C. To authorize users for application routes.
- D. To replace output encoding.

**Answer:** B. **Explanation:** CSP is a browser-enforced extra layer that can reduce the impact of some injection bugs. It does not replace safe application code or authorization.

### 6. What is a good first step before enforcing a new CSP?

- A. Add `'unsafe-inline'` and `'unsafe-eval'` to avoid all violations.
- B. Test with a report-only policy on representative pages and review violations.
- C. Disable CSP reporting and assume the policy works.
- D. Put the CSP in a database and let each user choose it.

**Answer:** B. **Explanation:** Report-only mode reveals which legitimate resources the policy would block without enforcing it. Refine the policy and then enforce the reviewed version.

### 7. What does `X-Content-Type-Options: nosniff` do?

- A. It prevents PHP from emitting an error.
- B. It tells browsers not to guess a different media type from the declared Content-Type.
- C. It forces every response to be JSON.
- D. It prevents a page from being framed.

**Answer:** B. **Explanation:** Correct media types plus `nosniff` help reduce MIME-confusion risks. The header does not validate or sanitize the response body.

### 8. What is the best way to handle `X-Forwarded-Proto`?

- A. Trust it from every incoming request.
- B. Trust it only when it is set by a known, configured reverse proxy.
- C. Use it as a redirect destination.
- D. Ignore the TLS configuration and disable secure cookies.

**Answer:** B. **Explanation:** A direct client can supply a forged forwarded header. Configure the application or framework to trust forwarding information only from specific proxies.

## Short-answer questions

### 9. Does HTTPS prove that a PHP route is safe?

**Answer:** No. HTTPS protects transport to the endpoint, but the endpoint may still have injection flaws, broken authorization, unsafe output, or compromised code. **Explanation:** Transport security and application security address different risks.

### 10. Why should every page and API route use HTTPS instead of protecting only the login form?

**Answer:** Session cookies, reset tokens, private responses, API data, and redirects can appear on any route. **Explanation:** Leaving an HTTP route available can expose or downgrade a session even if the login page itself used HTTPS.

### 11. Why is a valid server certificate important for an outbound PHP request?

**Answer:** It lets the client verify that it reached the intended server rather than an impersonator. **Explanation:** Encryption without certificate and hostname verification can encrypt traffic to an attacker-controlled endpoint.

### 12. What is mixed content?

**Answer:** It occurs when an HTTPS page loads one or more resources using plain HTTP. **Explanation:** An HTTP resource can be observed or modified in transit, and browsers may block it. Convert resource references to HTTPS and check all page features.

### 13. What does the CSP `frame-ancestors` directive control?

**Answer:** It controls which sites may embed the response in a frame, or can prohibit framing. **Explanation:** This is the modern framing policy and can help mitigate clickjacking.

### 14. Why is SameSite not a complete CSRF defense?

**Answer:** Browser behavior and application flows vary, and SameSite does not authorize the operation or replace an application-level CSRF defense. **Explanation:** Use appropriate CSRF tokens and validate authorization for state-changing requests.

### 15. Why can `includeSubDomains` make a previously harmless HSTS change risky?

**Answer:** It applies the policy to subdomains too. Any subdomain that lacks HTTPS or a valid certificate may become inaccessible to browsers that received the policy. **Explanation:** Inventory and prepare every covered host before enabling broad scope.

### 16. Why should a redirect use a fixed or allowlisted HTTPS hostname?

**Answer:** A request-controlled hostname can create an open redirect or send a user to an attacker-controlled destination. **Explanation:** Redirect targets and canonical origins are security policy, not arbitrary input.

## Code review questions

### 17. What is dangerous about this cURL setting?

~~~php
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false);
~~~

**Answer:** It disables verification of the server certificate. **Explanation:** The request may accept an attacker-controlled endpoint. Keep peer verification enabled and repair the CA trust configuration if verification fails.

### 18. What is wrong with this proxy detection?

~~~php
$isHttps = ($_SERVER['HTTP_X_FORWARDED_PROTO'] ?? '') === 'https';
~~~

**Answer:** It trusts a header that a direct client may be able to supply. **Explanation:** Only use forwarded scheme values through a trusted-proxy configuration; otherwise the application can misidentify the connection.

### 19. Why can this HSTS policy cause an outage?

~~~http
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
~~~

**Answer:** It applies a long-lived HTTPS requirement to the host and all subdomains, and requests preload inclusion. If any covered host or certificate is not ready, browsers may refuse access for an extended period. **Explanation:** Roll out a short policy first and use broad directives only after deliberate verification of all affected hosts and preload requirements.

### 20. Is this policy a safe universal CSP?

~~~http
Content-Security-Policy: default-src * 'unsafe-inline' 'unsafe-eval'
~~~

**Answer:** No. It broadly permits resource sources and inline or evaluated code, removing much of CSP's value. **Explanation:** Start from the application's required resources, prefer a restrictive policy, and use report-only mode before enforcement.

### 21. What problem may occur if this PHP session setting is used on an HTTP development site?

~~~php
session_set_cookie_params(['secure' => true]);
~~~

**Answer:** The browser will not send the Secure cookie over plain HTTP, so the local session may appear not to persist. **Explanation:** Keep the Secure flag enabled in HTTPS environments; configure local development deliberately rather than weakening production policy.

## Applied scenarios

### 22. TLS terminates at a load balancer, and PHP receives HTTP from it. Should the app turn off Secure session cookies?

**Answer:** No. Configure trusted-proxy handling so the application understands the original client connection, and keep Secure cookies enabled for the public HTTPS site. **Explanation:** The internal proxy hop is separate from the browser-to-edge connection; disabling Secure cookies weakens the browser session.

### 23. A staging hostname is not ready for HTTPS, but the main site is. Can the main site send HSTS with `includeSubDomains`?

**Answer:** Not until the staging hostname and every other covered subdomain are ready for HTTPS and valid certificate renewal. Use a narrower policy or prepare those hosts first. **Explanation:** The directive's scope includes subdomains regardless of whether they are production or staging.

### 24. A browser reports a CSP violation for an image hosted by a required trusted CDN. What should the team do?

**Answer:** Confirm the CDN is required and trusted, then add only the necessary source to the appropriate directive and retest the policy. **Explanation:** Do not respond by allowing every source or enabling unsafe inline code.

### 25. cURL reports a certificate verification error for an upstream API. What is the safe response?

**Answer:** Check the API hostname, certificate chain and expiry, server CA bundle, and PHP/cURL trust configuration. Keep verification enabled and fail the request safely until the issue is corrected. **Explanation:** Turning off peer or hostname verification hides the trust failure instead of fixing it.

## Coding exercise

### 26. Configure a PHP session cookie with Secure, HttpOnly, and SameSite attributes. What must happen before the session starts?

**Answer:**

~~~php
<?php

declare(strict_types=1);

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();
~~~

**Explanation:** Configure cookie parameters before `session_start()` so PHP sends the intended attributes when it creates the session cookie. Use this on the HTTPS deployment and retain CSRF protections for state-changing actions.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [133 - Dependency security and software supply chains](../lessons/133-dependency-security-and-software-supply-chains.md) | [Lesson](../lessons/134-https-security-headers-and-secure-transport.md) | [Example](../examples/134-https-security-headers-and-secure-transport.md) | [135 - Safe error handling and private information](../lessons/135-safe-error-handling-and-private-information.md) |