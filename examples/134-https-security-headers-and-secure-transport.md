# 134 - HTTPS, security headers, and secure transport: Example

## Question

Configure a PHP page for a production site served over HTTPS. The page should send a starter set of browser security headers, set secure session-cookie attributes, and make an outbound HTTPS request without bypassing certificate verification.

The example uses deployment-controlled flags to stage CSP and HSTS. Do not enable HSTS until the public HTTPS deployment and all hosts covered by the policy are ready.

## Complete page example

Save this as `public/index.php`:

~~~php
<?php

declare(strict_types=1);

function enabledSetting(string $name): bool
{
    $value = getenv($name);

    if ($value === false || $value === '0') {
        return false;
    }

    if ($value === '1') {
        return true;
    }

    throw new RuntimeException('Invalid security setting: ' . $name);
}

$cspEnforce = enabledSetting('APP_CSP_ENFORCE');
$hstsEnabled = enabledSetting('APP_HSTS_ENABLED');

header('Content-Type: text/html; charset=UTF-8');
header('X-Content-Type-Options: nosniff');
header('X-Frame-Options: DENY');
header('Referrer-Policy: strict-origin-when-cross-origin');
header('Permissions-Policy: camera=(), microphone=(), geolocation=()');

$csp = "default-src 'self'; base-uri 'self'; object-src 'none'; "
    . "frame-ancestors 'none'; form-action 'self'";

if ($cspEnforce) {
    header('Content-Security-Policy: ' . $csp);
} else {
    header('Content-Security-Policy-Report-Only: ' . $csp);
}

// Enable only after the site has verified its public HTTPS setup.
if ($hstsEnabled) {
    header('Strict-Transport-Security: max-age=300');
}

session_set_cookie_params([
    'lifetime' => 0,
    'path' => '/',
    'secure' => true,
    'httponly' => true,
    'samesite' => 'Lax',
]);

session_start();
$_SESSION['security_example_visited'] = true;

echo '<!doctype html>';
echo '<html lang="en">';
echo '<head><meta charset="utf-8"><title>Secure transport example</title></head>';
echo '<body><h1>Secure response configured</h1>';
echo '<p>Inspect the response headers and session cookie in your browser.</p>';
echo '</body></html>';
~~~

The CSP is intentionally limited to same-origin resources and blocks inline scripts. It may need adjustment for a real site. Keep `APP_CSP_ENFORCE` unset or set it to `0` while reviewing the report-only policy. Set it to `1` only after checking the application's actual pages and required resources.

Leave `APP_HSTS_ENABLED` unset or set it to `0` during local development. Set it to `1` only in a deployment whose public response is HTTPS and whose covered host is ready for HSTS. This example uses a short five-minute policy while validating the deployment; increase the duration only after checking certificate renewal and every relevant route. It deliberately does not add `includeSubDomains` or `preload`.

When TLS terminates at a reverse proxy, configure the edge and the application framework to trust only that proxy. These flags must come from deployment-controlled configuration, never from a request parameter or a client-supplied forwarding header.

## Secure outbound PHP cURL request

Use a fixed, server-controlled HTTPS endpoint and keep peer and hostname verification enabled:

~~~php
<?php

declare(strict_types=1);

function fetchHealthStatus(): string
{
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

    $body = curl_exec($curl);

    if ($body === false) {
        curl_close($curl);
        throw new RuntimeException('The upstream HTTPS request failed.');
    }

    $status = curl_getinfo($curl, CURLINFO_RESPONSE_CODE);
    curl_close($curl);

    if ($status < 200 || $status >= 300) {
        throw new RuntimeException('The upstream service returned an error.');
    }

    return $body;
}
~~~

This client does not follow redirects, so it cannot silently send the request to a different host. If a feature accepts a destination from a user, fixed HTTPS syntax alone is not enough; apply an explicit destination allowlist and the SSRF controls from the previous lesson.

## Verify the HTTPS deployment

After deploying the page behind a valid HTTPS endpoint, inspect its response:

~~~sh
curl -I https://app.example.test/
curl -I http://app.example.test/
~~~

Confirm that:

- The HTTPS response presents a valid certificate for the hostname.
- The HTTP URL redirects to the one expected HTTPS origin.
- The HTTPS response includes `X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`, and the selected CSP header.
- HSTS appears only after `APP_HSTS_ENABLED` is enabled for the verified HTTPS deployment.
- The session cookie includes `Secure`, `HttpOnly`, and `SameSite=Lax`.
- Browser developer tools show no unexpected CSP violations or mixed-content requests.

Do not test a long-lived HSTS policy on a domain or subdomain that is not ready for HTTPS. A browser that has stored HSTS may continue enforcing it after the server configuration changes.

## Explanation

- The page applies headers before sending its HTML body.
- CSP begins in report-only mode and has a separate deployment-controlled switch for enforcement.
- HSTS is separately gated and starts at a short duration. It is not sent on every local development response by default.
- Session cookies are configured before `session_start()` and use secure attributes.
- The cURL client uses a fixed HTTPS URL, checks the certificate and hostname, has time limits, and does not follow redirects.
- The example returns generic upstream failures rather than displaying raw TLS diagnostics.
- A security header does not replace output encoding, authorization, CSRF protection, or correct TLS configuration at the server and proxy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter Q&A | Next chapter |
| --- | --- | --- | --- |
| [133 - Dependency security and software supply chains](../lessons/133-dependency-security-and-software-supply-chains.md) | [Lesson](../lessons/134-https-security-headers-and-secure-transport.md) | [Q&A](../qnas/134-https-security-headers-and-secure-transport.md) | [135 - Safe error handling and private information](../lessons/135-safe-error-handling-and-private-information.md) |