# 130 - SSRF, URL validation, and redirect safety: Questions and answers

Use these questions to review how server-side URL fetching can reach unintended networks and how browser redirects can send users to untrusted destinations.

## Multiple-choice questions

### 1. What is server-side request forgery (SSRF)?

- A. A browser redirects to a different page.
- B. An application is tricked into making a network request to an unintended destination.
- C. A PHP script reads a local file by a fixed path.
- D. A user sends a request from a different tab.

**Answer:** B. **Explanation:** The application server makes the request, so the destination may be able to reach networks that the user cannot.

### 2. Which approach is usually safest when a feature can call one of several known services?

- A. Accept any URL and remove punctuation.
- B. Let the user choose a service ID mapped to a server-controlled URL.
- C. Accept a URL if FILTER_VALIDATE_URL() returns true.
- D. Follow every redirect automatically.

**Answer:** B. **Explanation:** The server retains control over the scheme, host, port, and path instead of trusting a caller-provided destination.

### 3. What should an application do with an HTTP redirect returned by a fetched server?

- A. Follow it automatically to any destination.
- B. Treat the new Location as a new destination and reapply policy, or disable redirects.
- C. Skip hostname validation because the first host was approved.
- D. Forward the user's session cookie to it.

**Answer:** B. **Explanation:** A redirect can change the destination to a private address, another host, or another protocol.

## SSRF basics and URL handling

### 4. What is the difference between SSRF and an open redirect?

**Answer:** SSRF causes the application server to make a request; an open redirect sends the user's browser to an untrusted destination. **Explanation:** The attacker targets different network actors, so each needs its own controls.

### 5. Why can an SSRF request reach resources the user cannot access directly?

**Answer:** It originates from the application server and uses the server's network access. **Explanation:** The server may have routes or credentials that are not available to the public browser.

### 6. Which features commonly create SSRF risk?

**Answer:** URL importers, webhook checkers, remote image fetchers, previews, and server-side API integrations. **Explanation:** Each feature may cause the server to connect to a destination selected or influenced by a user.

### 7. Is an HTTP-only SSRF risk always limited to HTTP?

**Answer:** No. **Explanation:** PHP functions and extensions may support other schemes or wrappers, depending on the API and configuration.

### 8. What does FILTER_VALIDATE_URL() prove?

**Answer:** It provides a syntax-level URL check according to its validation rules. **Explanation:** It does not prove that the scheme, host, port, resolved address, or network destination is safe.

### 9. What does parse_url() do?

**Answer:** It extracts components such as scheme, host, port, user, path, query, and fragment. **Explanation:** Parsing a value does not decide whether the application should connect to it.

### 10. Can FILTER_VALIDATE_URL() accept a loopback URL?

**Answer:** Yes. **Explanation:** A syntactically valid URL can point to localhost; destination policy must reject disallowed hosts and IP addresses.

### 11. Why should a feature prefer a server-side destination ID?

**Answer:** It lets trusted configuration determine the actual URL. **Explanation:** The user cannot change the hostname or scheme merely by editing a submitted URL.

### 12. Which URL scheme should a typical external web integration require?

**Answer:** Usually HTTPS, unless the feature has a reviewed reason to permit another scheme. **Explanation:** Restricting the scheme prevents the client from selecting unrelated protocol handlers.

### 13. Why should URL user information be rejected?

**Answer:** A URL such as trusted.example@attacker.example can make naive checks confuse credentials with the actual host. **Explanation:** Parse the URL and compare the canonical host exactly; do not inspect the raw string with substring matching.

### 14. Why must the port be checked?

**Answer:** A hostname can target an unexpected service by using a nonstandard port. **Explanation:** Permit only the port or small set of ports required by the integration.

### 15. Why is exact host matching safer than a substring check?

**Answer:** An attacker can include the trusted hostname as part of a different hostname or URL component. **Explanation:** Compare the normalized parsed hostname to a configured allowlist entry.

### 16. Is a trailing-dot hostname always equivalent for application policy?

**Answer:** It may resolve to the same DNS name, but the policy must normalize or explicitly reject it. **Explanation:** Inconsistent canonicalization can make equivalent names pass one check and behave differently in another component.

### 17. Should an application strip suspicious characters from an unsafe URL and then use it?

**Answer:** No. **Explanation:** Transforming input can change what the network client interprets; reject invalid destinations instead.

## DNS, IP addresses, and network policy

### 18. Why is hostname allowlisting alone insufficient for SSRF?

**Answer:** DNS can resolve an approved name to an internal or otherwise prohibited address. **Explanation:** The application must control the actual connection destination as well as the hostname.

### 19. What is DNS rebinding?

**Answer:** A hostname returns different IP addresses over time or between checks, potentially changing from an allowed address to a prohibited one. **Explanation:** A validation lookup followed by a separate unchecked connection lookup can be bypassed.

### 20. Which DNS records should a destination policy consider?

**Answer:** Both IPv4 A and IPv6 AAAA answers. **Explanation:** Checking only one address family may leave an alternate route to a prohibited destination.

### 21. What should happen if any resolved address is outside the allowed network policy?

**Answer:** Reject the destination or use a policy that guarantees the client cannot connect to the prohibited address. **Explanation:** Every address and fallback path must be covered, not only the first DNS result.

### 22. Why should an HTTP client connect to an address that was actually validated?

**Answer:** Otherwise it may perform another DNS lookup and connect to a different address. **Explanation:** Connection-time pinning helps close the gap between validation and use.

### 23. When pinning a validated IP, what must remain tied to the approved hostname?

**Answer:** The HTTP Host header, TLS SNI, and certificate hostname verification. **Explanation:** Connecting to an IP must not disable authentication of the intended TLS host.

### 24. Why are handwritten “private IP” checks risky?

**Answer:** IPv4, IPv6, mapped addresses, and special-use ranges make complete classification subtle. **Explanation:** Use maintained libraries and egress controls rather than a short list that misses edge cases.

### 25. What does an outbound firewall or egress proxy add?

**Answer:** It can prevent the application from reaching sensitive networks even if application-level validation fails. **Explanation:** Network restrictions provide another layer and reduce the server's available destinations.

## HTTP clients and redirects

### 26. Why should automatic redirects be disabled for a sensitive fetch?

**Answer:** A trusted first host could redirect the client to a private address or another protocol. **Explanation:** With redirects disabled, the application can decide whether and how to process each Location.

### 27. If redirects are required, what should be checked for every hop?

**Answer:** The resolved destination, scheme, host, port, IP policy, credentials, and redirect count. **Explanation:** Each hop is a new outbound request and must satisfy the same policy.

### 28. Why should a redirect chain have a small hop limit?

**Answer:** It prevents unbounded loops and reduces unexpected network work. **Explanation:** Reject loops and stop once the policy's maximum number of hops is reached.

### 29. Should authentication headers or cookies be forwarded to a different redirect host?

**Answer:** No, unless the application has a deliberate, reviewed credential-sharing policy. **Explanation:** Credentials intended for one host must not leak to a new destination.

### 30. Why set connection and total timeouts?

**Answer:** A destination may stall connection setup or keep a response open. **Explanation:** Limits prevent a fetch from tying up application workers indefinitely.

### 31. Why should response size and content type be limited?

**Answer:** A server could return an unexpectedly large or active payload. **Explanation:** Bound memory and processing, and handle only the response formats the feature needs.

### 32. Does limiting cURL to HTTPS validate the destination hostname?

**Answer:** No. **Explanation:** A protocol restriction prevents other schemes but does not decide which HTTPS host or IP the request may reach.

### 33. What does CURLOPT_FOLLOWLOCATION=false do?

**Answer:** It tells cURL not to automatically follow an HTTP redirect. **Explanation:** The application can inspect and validate the Location before making another request.

### 34. Why should the app avoid forwarding user cookies to a fetched URL?

**Answer:** The remote host could receive credentials that belong to the application or user. **Explanation:** Construct a minimal outbound request with only the data and headers required by the integration.

### 35. Why is file_get_contents() not a safe shortcut for an arbitrary URL?

**Answer:** It may perform a network request or use a stream wrapper based on the supplied value and runtime configuration. **Explanation:** Use a constrained HTTP client with explicit destination policy.

## Browser redirect safety

### 36. What is an open redirect?

**Answer:** A route accepts an untrusted destination and sends the browser there. **Explanation:** Attackers can make a link on a trusted site appear to lead somewhere safe before sending the user to a phishing destination.

### 37. What is safer than accepting a return URL after login?

**Answer:** Accept a short destination ID that maps to a fixed local route. **Explanation:** Server-side mapping avoids interpreting an arbitrary URL as a redirect target.

### 38. What is a scheme-relative URL such as //attacker.example?

**Answer:** A URL that inherits the current page's scheme but names another host. **Explanation:** It can escape an application even though it does not begin with http:// or https://.

### 39. If an external browser redirect is required, how should it be controlled?

**Answer:** Select from an explicit server-side allowlist, or validate parsed URL components and show a clear confirmation when appropriate. **Explanation:** Do not trust a raw URL just because it was submitted by a logged-in user.

## Review and testing

### 40. What SSRF cases should be tested?

**Answer:** Unexpected schemes, loopback and private IPs, IPv6, deceptive hosts, user information, unusual ports, DNS changes, and redirects to prohibited destinations. **Explanation:** Also test credentials, response limits, timeouts, retries, and proxy behavior.

### 41. What redirect cases should be tested?

**Answer:** Unknown destination IDs, external URLs, scheme-relative URLs, encoded paths, and destinations that resemble an approved hostname. **Explanation:** Confirm the application redirects only to its approved local routes or external targets.

### 42. What is the main URL-security rule?

**Answer:** Prefer server-controlled destination IDs; when a URL is necessary, validate its parsed meaning and actual connection destination, then revalidate every redirect. **Explanation:** Syntax checks are only one small part of SSRF and redirect defense.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [129 - Path traversal and filesystem security](../lessons/129-path-traversal-and-filesystem-security.md) | [Lesson](../lessons/130-ssrf-url-validation-and-redirect-safety.md) | [Example](../examples/130-ssrf-url-validation-and-redirect-safety.md) | [131 - Command injection and unsafe deserialization](../lessons/131-command-injection-and-unsafe-deserialization.md) |