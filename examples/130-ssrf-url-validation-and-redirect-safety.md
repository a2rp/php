# 130 - SSRF, URL validation, and redirect safety: Example

## Question

Create a runnable PHP example that lets a caller select a known outbound service and a local post-login destination by ID. Show that URL syntax validation alone accepts a loopback URL, while the server-side mappings reject it as an unapproved destination.

## Complete example

Save this as `safe-destination-selection.php` and run it with PHP CLI:

~~~sh
php safe-destination-selection.php
~~~

~~~php
<?php

declare(strict_types=1);

$services = [
    'status' => [
        'url' => 'https://api.example.test/v1/status',
    ],
    'catalog' => [
        'url' => 'https://api.example.test/v1/catalog',
    ],
];

$localRedirects = [
    'dashboard' => '/dashboard',
    'account' => '/account/profile',
];

function findService(mixed $serviceId, array $services): ?array
{
    if (!is_string($serviceId) || !isset($services[$serviceId])) {
        return null;
    }

    return $services[$serviceId];
}

function findLocalRedirect(
    mixed $destinationId,
    array $localRedirects
): ?string {
    if (
        !is_string($destinationId)
        || !isset($localRedirects[$destinationId])
    ) {
        return null;
    }

    return $localRedirects[$destinationId];
}

$loopbackUrl = 'http://127.0.0.1/admin';
$urlSyntaxIsValid = filter_var(
    $loopbackUrl,
    FILTER_VALIDATE_URL
) !== false;

echo 'FILTER_VALIDATE_URL accepts loopback URL syntax: '
    . ($urlSyntaxIsValid ? 'yes' : 'no')
    . PHP_EOL;

$service = findService('status', $services);

if ($service !== null) {
    echo 'Service ID status maps to: '
        . $service['url']
        . PHP_EOL;
}

$arbitraryService = findService($loopbackUrl, $services);

echo 'Arbitrary URL used as service ID: '
    . ($arbitraryService === null ? 'rejected' : 'accepted')
    . PHP_EOL;

$localDestination = findLocalRedirect(
    'dashboard',
    $localRedirects
);

echo 'Local redirect ID dashboard maps to: '
    . ($localDestination ?? 'rejected')
    . PHP_EOL;

$externalDestination = findLocalRedirect(
    'https://attacker.example/phishing',
    $localRedirects
);

echo 'External URL used as redirect ID: '
    . ($externalDestination === null ? 'rejected' : 'accepted')
    . PHP_EOL;
~~~

The service and redirect tables are controlled by the application. The script intentionally does not make a network request or send a browser redirect; it demonstrates the destination-selection policy without contacting an external host.

## Expected output

~~~text
FILTER_VALIDATE_URL accepts loopback URL syntax: yes
Service ID status maps to: https://api.example.test/v1/status
Arbitrary URL used as service ID: rejected
Local redirect ID dashboard maps to: /dashboard
External URL used as redirect ID: rejected
~~~

## Explanation

### URL syntax is not a destination policy

filter_var() reports that the loopback URL is syntactically valid. That does not make the URL an approved destination. A server-side request to 127.0.0.1 could reach a service on the application server.

The safe service function accepts only a known ID. The caller cannot substitute a hostname, port, protocol, or path. In a real integration, authorize the caller for that service ID before making a request.

### A production fetch needs network controls too

The service map removes arbitrary user control over the target, but a real server-side fetch still needs a constrained HTTP client. Require HTTPS, verify the TLS peer and hostname, set connection and total timeouts, limit response size, send only necessary headers, and disable automatic redirects unless each hop is revalidated.

The hostname must resolve only to destinations permitted by the application. Prevent unchecked DNS re-resolution between validation and connection, preserve the approved hostname for TLS verification, and restrict outbound traffic with network egress rules. A domain allowlist by itself does not prevent DNS rebinding.

Do not pass a user-provided URL to file_get_contents(), fopen(), a shell command, or an HTTP client. If the application genuinely needs user-configured destinations, use a maintained URL parser, an exact host and port allowlist, connection-time IP enforcement, and an egress policy.

### Browser redirects use a different allowlist

The redirect function accepts a route key such as dashboard and returns a fixed local path. It rejects an external URL because that URL is not a configured key. A real handler can then issue the redirect:

~~~php
$destination = findLocalRedirect(
    $_GET['to'] ?? null,
    $localRedirects
);

if ($destination === null) {
    http_response_code(400);
    exit('Unknown destination.');
}

header('Location: ' . $destination, true, 303);
exit;
~~~

Because the path comes from a server-side map, the request cannot redirect the browser to an arbitrary host. Do not replace the route ID with an unvalidated return_to URL.

## What this example demonstrates

- A loopback URL can pass syntax validation and still be unsafe.
- A caller selects a known service ID instead of a raw outbound URL.
- The service map keeps the outbound destination under server control.
- A post-login redirect ID maps only to fixed local routes.
- An unapproved external URL is rejected as a redirect destination.
- Real network fetching still requires DNS/IP enforcement, constrained HTTP-client settings, and egress controls.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [129 - Path traversal and filesystem security](../lessons/129-path-traversal-and-filesystem-security.md) | [Q&A](../qnas/130-ssrf-url-validation-and-redirect-safety.md) | [Lesson](../lessons/130-ssrf-url-validation-and-redirect-safety.md) | [131 - Command injection and unsafe deserialization](../lessons/131-command-injection-and-unsafe-deserialization.md) |