# 130 - SSRF, URL validation, and redirect safety

Server-side request forgery (SSRF) happens when an application is tricked into making a network request to a destination an attacker chooses. A feature that fetches an image, checks a webhook, imports a URL, or calls an internal service can become an SSRF path if it accepts arbitrary destinations.

Open redirects are a related but different problem: the application sends a user's browser to an untrusted destination. Prevent both by preferring server-side destination IDs mapped to approved targets. When URLs must be accepted, parse and validate every relevant component, constrain the network destination, and do not follow unvalidated redirects.

## Learning goals

By the end of this chapter, you should be able to:

- Explain SSRF and distinguish it from a browser open redirect.
- Prefer server-side mappings over arbitrary user-supplied URLs.
- Understand the limits of FILTER_VALIDATE_URL() and parse_url().
- Restrict schemes, hosts, ports, paths, and URL credentials.
- Account for DNS resolution, private addresses, and DNS rebinding.
- Configure an HTTP client to limit protocols, redirects, and time.
- Validate every redirect destination or disable automatic redirects.
- Use local route IDs for post-login redirects.
- Test rejected as well as approved destinations.

## What SSRF can do

An SSRF-vulnerable server may be able to reach network locations that the user cannot reach directly. Depending on its network permissions, the request might contact an internal administration service, a cloud metadata endpoint, a database, a local-only API, or a public host.

The browser's same-origin policy does not protect server-to-server requests. The request originates from the application server, so its network access and attached credentials matter. SSRF is not limited to HTTP: PHP and its extensions support multiple stream and protocol handlers, and different request libraries may interpret destinations differently.

Examples of risky features include:

- Downloading an image from a user-submitted URL.
- Testing a webhook or callback URL.
- Importing a remote document.
- Generating a preview from an arbitrary page.
- Passing a user-supplied URL to file_get_contents(), fopen(), cURL, or an SDK.
- Following a redirect returned by an otherwise approved host.

## Prefer server-side destination IDs

The strongest design is to avoid accepting a destination URL. Let the caller select a known destination key, then use a URL stored in trusted server-side configuration:

~~~php
$serviceUrls = [
    'status' => 'https://api.example.test/v1/status',
    'catalog' => 'https://api.example.test/v1/catalog',
];

$serviceId = $_GET['service'] ?? null;

if (!is_string($serviceId) || !isset($serviceUrls[$serviceId])) {
    http_response_code(400);
    exit('Unknown service.');
}

$url = $serviceUrls[$serviceId];
~~~

The request selects a capability the application already exposes. It cannot choose a hostname, port, scheme, or arbitrary path. Authorize the user for that capability as well; an allowlisted target is not automatically available to every account.

If an integration truly needs user-configured destinations, reduce the feature's network authority. Use an explicit hostname allowlist where possible, constrain allowed paths and ports, and enforce network-level egress rules. A valid URL is not necessarily a safe URL.

## Parse URLs, then validate policy

URL parsing and URL validation are separate steps. PHP's FILTER_VALIDATE_URL() checks a URL syntax form, but it is permissive about schemes and can accept loopback hosts. parse_url() extracts components; it does not decide whether those components are safe for the application.

For an allowlisted HTTPS integration, a policy might require:

- The input is a scalar string with a reasonable length.
- The scheme is exactly https.
- The hostname matches an approved canonical hostname exactly.
- No username or password appears in the URL.
- The effective port is the expected port, commonly 443.
- The path is one of the approved endpoint paths if the integration does not need arbitrary paths.
- IP-literal destinations are rejected unless there is a specific, reviewed need for them.

Do not use substring checks such as “contains trusted.example”, suffix checks without a dot boundary, or raw prefix checks. Values like trusted.example.attacker.test and URLs with user information can fool simplistic checks. Normalize hostnames consistently and compare parsed hostnames against an exact allowlist.

Do not try to repair a dangerous URL by stripping suspicious characters and then using the result. Reject invalid input. If the feature accepts a URL, parse the same value that the network client will use and ensure no later transformation changes its meaning.

## DNS and network destination checks

An allowlisted domain name does not guarantee that a connection reaches an approved IP address. DNS can return a loopback, private, link-local, reserved, or otherwise prohibited address. DNS answers may also change between validation and connection, which creates a DNS-rebinding or time-of-check/time-of-use problem.

For a user-configurable host:

1. Resolve all relevant A and AAAA records.
2. Reject addresses outside the destinations the application intends to reach, including internal and special-use ranges.
3. Ensure the HTTP client connects to one of the addresses that was checked instead of performing an unchecked second DNS lookup.
4. Preserve the approved hostname for the HTTP Host header, TLS SNI, and certificate verification.
5. Apply the policy to retries, fallback addresses, proxies, and every redirect hop.
6. Add an outbound firewall or egress proxy so the application server cannot reach sensitive networks unnecessarily.

IP range validation is subtle, especially across IPv4, IPv6, mapped addresses, and different network environments. Prefer a maintained HTTP client or egress service that supports safe destination pinning and use deployment-level network controls. Do not copy a short hand-written list of “private” addresses and assume it covers every prohibited destination.

## Configure outbound HTTP requests

Use an HTTP client with a strict request policy:

- Allow only the required scheme, normally HTTPS.
- Verify TLS certificates and the hostname.
- Set connection and overall timeouts.
- Limit the response size and accepted content type.
- Do not forward the user's cookies, authorization header, or other credentials to the target.
- Disable automatic redirects, or validate every new Location target before following it.
- Do not pass the value to a shell command.

For example, a cURL request can disable automatic redirects and limit the protocol:

~~~php
$handle = curl_init($approvedUrl);

curl_setopt_array($handle, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_PROTOCOLS_STR => 'https',
    CURLOPT_FOLLOWLOCATION => false,
    CURLOPT_MAXREDIRS => 0,
    CURLOPT_CONNECTTIMEOUT => 3,
    CURLOPT_TIMEOUT => 8,
    CURLOPT_SSL_VERIFYPEER => true,
    CURLOPT_SSL_VERIFYHOST => 2,
]);

$responseBody = curl_exec($handle);
$responseCode = curl_getinfo($handle, CURLINFO_RESPONSE_CODE);
$error = curl_error($handle);

curl_close($handle);
~~~

CURLOPT_PROTOCOLS_STR is available with PHP 8.3 and a recent libcurl. On older PHP builds, use the compatible protocol option supported by that build. This configuration limits some client behavior; it does not by itself validate the destination hostname, prevent DNS rebinding, constrain the connected IP, or enforce a network egress policy. Use an approved destination mapping and the connection-time controls described above. If a proxy is configured, ensure its routing and DNS behavior meet the same destination policy.

Do not use file_get_contents() on a caller-supplied URL as a shortcut. Depending on the value, wrapper and URL handling may make it a network request rather than a local file read.

## Treat redirects as new destinations

A redirect response can point somewhere different from the original URL. If an HTTP client follows redirects automatically, an approved public host may redirect the application to a private address or another protocol.

The simplest policy is to disable redirects. If the product needs them, inspect each Location value, resolve relative references against the prior URL, and rerun the complete scheme, host, port, DNS, and network policy before following it. Set a small hop limit and reject loops. A redirect hop must not inherit credentials intended for another host.

The same rule applies to browser redirects, but the risk is different. A browser open redirect can make a trusted application appear to send users to a phishing page. Prefer a short destination ID that maps to a fixed local route:

~~~php
$localDestinations = [
    'dashboard' => '/dashboard',
    'profile' => '/account/profile',
];

$destinationId = $_GET['to'] ?? null;

if (!is_string($destinationId) || !isset($localDestinations[$destinationId])) {
    http_response_code(400);
    exit('Unknown destination.');
}

header('Location: ' . $localDestinations[$destinationId], true, 303);
exit;
~~~

Avoid accepting an arbitrary return URL. If a local return path is required, use a framework's local-URL helper or a carefully designed validator, and reject scheme-relative values such as //attacker.example. For required external redirects, use a server-side allowlist and show a clear confirmation when the destination could surprise the user.

## Common mistakes

- Treating FILTER_VALIDATE_URL() as a complete SSRF defense.
- Treating parse_url() as a safety validator.
- Allowing arbitrary schemes, ports, hosts, or URL credentials.
- Using an exact-looking substring or suffix check for hostnames.
- Trusting a domain allowlist without checking DNS answers and the connected address.
- Validating one DNS answer and allowing the client to resolve the host again later.
- Following redirects without revalidating each destination.
- Forwarding authentication cookies or headers to a new host.
- Accepting an arbitrary return URL after login.
- Using file_get_contents() or another stream function with a user-controlled URL.
- Assuming that blocking localhost alone covers private, link-local, IPv6, and special-use networks.
- Omitting timeouts, response-size limits, TLS verification, or network egress restrictions.

## Testing checklist

Test that the application:

- Accepts only configured destination IDs or explicitly approved parsed URLs.
- Rejects non-string and oversized input.
- Rejects file, php, phar, ftp, and other unneeded schemes.
- Rejects loopback, private, link-local, reserved, and unexpected IP literals.
- Rejects URL user information and unexpected ports.
- Rejects deceptive hostnames, trailing-dot variants not in policy, and scheme-relative URLs.
- Revalidates redirect targets and stops after the configured hop limit.
- Does not send cookies, authorization headers, or secrets to untrusted hosts.
- Enforces connection and total timeouts and an appropriate response-size limit.
- Cannot reach prohibited internal networks due to outbound network policy.
- Redirects only to fixed local routes or approved external destinations.

## Key takeaways

- SSRF is about the server's network access; an open redirect is about where the browser is sent.
- Prefer IDs mapped to server-controlled destinations over arbitrary URLs.
- Parsing and syntax validation do not prove a destination is safe.
- Enforce scheme, exact host, port, DNS/IP, and redirect policy at connection time.
- Disable automatic redirects or validate every hop as a new request.
- Use timeouts, TLS verification, minimal credentials, and network egress restrictions as layers of defense.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [129 - Path traversal and filesystem security](129-path-traversal-and-filesystem-security.md) | [Q&A](../qnas/130-ssrf-url-validation-and-redirect-safety.md) | [Example](../examples/130-ssrf-url-validation-and-redirect-safety.md) | [131 - Command injection and unsafe deserialization](131-command-injection-and-unsafe-deserialization.md) |