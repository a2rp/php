# 134 - HTTPS, security headers, and secure transport

HTTPS protects HTTP traffic with Transport Layer Security (TLS). When configured and used correctly, TLS encrypts traffic in transit, detects tampering, and lets a client authenticate the server certificate. It does not fix insecure application logic, authorize a user, or protect data after it reaches a compromised endpoint.

Use HTTPS for every page and API route, keep certificate validation enabled for outbound requests, and configure browser security headers to reduce specific risks. Security headers are an extra browser-enforced layer; they do not replace safe output encoding, authorization, CSRF defenses, or secure application design.

## Learning goals

By the end of this chapter, you should be able to:

- Explain what HTTPS and TLS protect and what they do not protect.
- Configure a site to use HTTPS consistently and redirect HTTP safely.
- Roll out HSTS without locking out hosts that are not ready for HTTPS.
- Configure a restrictive Content Security Policy and test it before enforcement.
- Set useful browser security headers and secure session cookies.
- Preserve certificate and hostname verification in PHP cURL requests.
- Handle TLS termination at a reverse proxy without trusting spoofed headers.
- Verify headers, redirects, cookies, and certificate failures.

## What HTTPS protects

TLS provides confidentiality against passive network observers, integrity against undetected changes in transit, and authentication of the server when the client validates its certificate. A certificate that is expired, untrusted, or issued for a different hostname must fail validation.

HTTPS does not make every request trustworthy. An authenticated user can still submit malicious input, a vulnerable script can still expose data, and a server can still return unsafe HTML. TLS protects a connection between endpoints; it does not prove that an endpoint or its application is secure.

A site should use HTTPS for its entire origin, not only the login form. Session cookies, password resets, authenticated pages, API routes, and redirects can all expose information if an HTTP route remains available.

## Configure HTTPS at the edge

Configure the web server, load balancer, CDN, or hosting platform to:

- Serve a valid certificate for each public hostname.
- Renew certificates and monitor expiry.
- Support TLS 1.3 and TLS 1.2 where compatibility requires it.
- Disable SSL and obsolete TLS protocol versions.
- Redirect HTTP requests to the canonical HTTPS host.
- Avoid sending credentials or sensitive data over an HTTP redirect.
- Keep internal proxy-to-origin traffic protected when it crosses a network that is not trusted.

Redirect HTTP at the trusted edge when possible. Preserve the request method and path as required, and ensure the destination host is fixed or selected from an allowlist. Do not construct a redirect target from an arbitrary Host header.

If TLS terminates at a reverse proxy, the PHP process may receive plain HTTP from that proxy even though the visitor used HTTPS. Configure the proxy and application integration explicitly. Trust forwarded scheme and client-address headers only from known proxy addresses; a client can send forged headers if the application accepts them directly. Never disable HTTPS checks just to make the proxy setup appear to work.

## Use HSTS carefully

The `Strict-Transport-Security` response header tells a browser that has received it over HTTPS to use HTTPS for that host for a configured period:

~~~http
Strict-Transport-Security: max-age=300
~~~

Start with a short duration while validating the deployment. Increase it only after certificate renewal, all routes, and every relevant hostname work over HTTPS.

The directives `includeSubDomains` and `preload` have a much wider effect. Use them only when every covered subdomain is ready to serve HTTPS and the organization deliberately accepts the long-lived consequences. A forgotten subdomain or failed certificate can become inaccessible to browsers that remember the policy. HSTS does not protect a visitor's very first connection unless the site is already covered by a browser preload list.

Send HSTS only on HTTPS responses. Prefer setting it in the edge or web-server configuration that knows the original client connection scheme. Do not trust a request-supplied `X-Forwarded-Proto` header as proof that the visitor used HTTPS.

## Content Security Policy

Content Security Policy (CSP) restricts which sources a browser may use for scripts, styles, images, frames, connections, and other resources. It can reduce the impact of some injection bugs, but it does not make unsafe output safe.

A restrictive policy for a simple same-origin page might be:

~~~http
Content-Security-Policy: default-src 'self'; base-uri 'self'; object-src 'none'; frame-ancestors 'none'; form-action 'self'
~~~

This policy may block inline scripts, inline styles, external images, analytics, embedded frames, or API connections that the application intentionally uses. Inventory the page's actual resource needs before enforcement. Avoid broadly adding `'unsafe-inline'` or `'unsafe-eval'` just to silence violations. Where inline scripts are required, prefer per-response nonces or hashes and render them correctly.

Begin with `Content-Security-Policy-Report-Only` on representative pages, inspect violations, and refine the policy. Once the policy works for intended features, enforce it with `Content-Security-Policy`. Keep report collection endpoints protected from abuse and avoid putting sensitive values into reports.

## Useful response headers

Choose headers that match the application:

~~~http
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Content-Security-Policy: default-src 'self'; base-uri 'self'; object-src 'none'; frame-ancestors 'none'; form-action 'self'
~~~

- `X-Content-Type-Options: nosniff` tells browsers to respect the declared media type rather than guessing a different one.
- `Referrer-Policy` controls how much URL information a browser sends in the `Referer` header. Do not put secrets in URLs.
- CSP `frame-ancestors` controls which sites may embed a page and can prevent framing entirely.
- `Permissions-Policy` can disable browser features an application does not need. Only add directives after checking the features the application uses.
- `X-Frame-Options` can provide a fallback for older clients, but CSP `frame-ancestors` is the modern framing control.

Set the correct `Content-Type` for every response. For downloads of user-provided or unknown files, use a deliberate `Content-Disposition` and media type, and prevent MIME sniffing. Do not emit a long list of headers without understanding their effect.

## Add application headers in PHP

Set headers before any body output. For an application whose pages use only same-origin resources and do not require inline code, a PHP bootstrap might include:

~~~php
<?php

declare(strict_types=1);

header('X-Content-Type-Options: nosniff');
header('Referrer-Policy: strict-origin-when-cross-origin');
header(
    "Content-Security-Policy: default-src 'self'; base-uri 'self'; "
    . "object-src 'none'; frame-ancestors 'none'; form-action 'self'"
);
~~~

This is an example policy, not a universal drop-in setting. Test it against the application's pages and APIs. A JSON API response that is not rendered as a document may not need the same CSP as an HTML page. Prefer configuring shared headers at the web server or edge when that makes coverage more consistent.

## Secure PHP session cookies

Session identifiers are credentials. Use HTTPS and mark session cookies as secure, HTTP-only, and appropriately same-site. Configure these options before calling `session_start()`:

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

This array form supports the `samesite` option in PHP 7.3 and newer. Choose `Strict` or `Lax` based on the application's cross-site navigation and login flows. SameSite is defense in depth, not a replacement for CSRF tokens and request authorization.

When TLS terminates at a proxy, use a maintained framework or server configuration that correctly identifies HTTPS only behind trusted proxies. Do not turn off the cookie's `secure` flag simply because the PHP-to-proxy connection is HTTP.

## Preserve certificate verification in outbound PHP requests

PHP cURL verifies the peer certificate by default, and hostname verification should remain enabled. Do not use `CURLOPT_SSL_VERIFYPEER => false` or `CURLOPT_SSL_VERIFYHOST => 0` in production. Those settings make an encrypted connection vulnerable to impersonation.

A fixed HTTPS URL and explicit verification settings can be used like this:

~~~php
<?php

declare(strict_types=1);

$curl = curl_init('https://api.example.test/health');

if ($curl === false) {
    throw new RuntimeException('Could not initialize the HTTP client.');
}

curl_setopt_array($curl, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CONNECTTIMEOUT => 5,
    CURLOPT_TIMEOUT => 15,
    CURLOPT_SSL_VERIFYPEER => true,
    CURLOPT_SSL_VERIFYHOST => 2,
    CURLOPT_FOLLOWLOCATION => false,
]);

$response = curl_exec($curl);

if ($response === false) {
    // Do not expose raw cURL details to visitors; redact them before logging.
    curl_close($curl);
    throw new RuntimeException('The upstream HTTPS request failed.');
}

$statusCode = curl_getinfo($curl, CURLINFO_RESPONSE_CODE);
curl_close($curl);

if ($statusCode < 200 || $statusCode >= 300) {
    throw new RuntimeException('The upstream service returned an error.');
}
~~~

The URL above is server-controlled. If a feature accepts a destination from input, it also needs a destination allowlist and SSRF protections. Keep the operating system's CA bundle or cURL CA configuration current; do not bypass certificate checks to work around a missing CA certificate.

## Avoid mixed content and downgrade paths

An HTTPS page should not load scripts, styles, images, frames, or API calls over plain HTTP. Mixed content can expose or alter page resources and may be blocked by browsers. Update resource URLs and verify every page, not only the home page.

Also check:

- Canonical links and redirects always end at HTTPS.
- Login and logout forms submit over HTTPS.
- Session cookies have the `Secure` attribute.
- APIs do not redirect sensitive requests to HTTP.
- Absolute links in email, reset flows, and generated content use a configured HTTPS origin.
- The application does not accept an untrusted host or forwarded-protocol header as the canonical origin.

## Verification checklist

Inspect representative responses and test the deployment:

~~~sh
curl -I https://example.test/
curl -I http://example.test/
~~~

Confirm that the HTTPS response has a valid certificate and the intended headers, and that the HTTP request redirects to the expected HTTPS hostname. Browser developer tools can reveal CSP violations, mixed content, cookie attributes, and blocked resources.

Test that the application:

- Redirects HTTP to one configured HTTPS origin without creating an open redirect.
- Rejects expired, invalid, and hostname-mismatched certificates for outbound requests.
- Keeps TLS peer and host verification enabled.
- Sends HSTS only through the HTTPS deployment path and only for hosts ready for it.
- Applies the intended CSP without breaking required application resources.
- Sets secure session-cookie attributes before creating the session.
- Does not trust forwarded headers from untrusted clients.
- Serves correct content types and avoids mixed-content requests.
- Keeps sensitive values out of URLs, headers, errors, and CSP reports.

## Key takeaways

- HTTPS protects data in transit when the client validates the server certificate.
- Redirect every route to a fixed HTTPS origin and configure TLS at the trusted edge.
- HSTS and CSP require a deliberate rollout; overly broad settings can break legitimate pages or subdomains.
- Security headers add protection but do not replace secure application code.
- Keep PHP cURL certificate and hostname verification enabled.
- Trust proxy headers only from known proxies, and keep session cookies secure.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [133 - Dependency security and software supply chains](133-dependency-security-and-software-supply-chains.md) | [Q&A](../qnas/134-https-security-headers-and-secure-transport.md) | [Example](../examples/134-https-security-headers-and-secure-transport.md) | [135 - Safe error handling and private information](135-safe-error-handling-and-private-information.md) |
