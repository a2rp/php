# 115 - Caching data and application output

## Goals

After this lesson, you should be able to:

- Explain why an application uses a cache and what correctness risks caching introduces.
- Distinguish application data caches from generated-output, opcode, database, and HTTP response caches.
- Apply the cache-aside pattern with an explicit key, source of truth, and time-to-live (TTL).
- Design keys that include every input that can affect a cached result.
- Choose an invalidation strategy and reason about stale data, concurrent misses, and cache failures.
- Set HTTP caching directives for public and private responses without exposing personalized data.
- Recognize what PSR-6 and PSR-16 standardize and what they leave to an implementation.

## What a cache does

A cache keeps a reusable copy of data or output so that a later operation can avoid repeating expensive work. The source of truth remains the database, service, or other authoritative store. A cache is an optimization: entries can expire, be evicted, or disappear during a restart. Application behavior must not depend on an entry existing forever.

Caching can reduce latency, database load, network traffic, and repeated rendering work. It also introduces a second copy that can become stale. Before adding a cache, identify the expensive operation, how often it repeats, and how old its result is allowed to be. If the result is already cheap or must always be current, caching can add complexity without a useful benefit.

## Different kinds of caching

The word cache describes several distinct layers:

| Cache | What it can reuse | Typical scope |
| --- | --- | --- |
| Application data cache | A query result, computed value, or remote API response | PHP application, often shared across processes |
| Generated-output cache | A rendered template or complete application fragment | Application or template engine |
| Opcode cache | Compiled PHP bytecode | PHP runtime, commonly across requests |
| Database cache | Query plans, pages, or engine-managed data | Database server |
| HTTP response cache | A representation of an HTTP response | Browser, reverse proxy, or CDN |

These layers have separate keys, lifetimes, and invalidation rules. An HTTP `Cache-Control` header does not configure a PSR cache pool. Clearing an application cache does not necessarily clear a browser or CDN copy. Know which layer is serving the value you are debugging.

## Cache-aside for application data

Cache-aside is a common application pattern. The code checks the cache first. On a miss, it loads the value from the source of truth, stores a copy with a bounded lifetime, and returns it.

~~~php
function findProduct(
    string $tenantId,
    int $productId,
    ProductRepository $products,
    Psr\\SimpleCache\\CacheInterface $cache,
): ?ProductData {
    $key = "product:v3:{$tenantId}:{$productId}";
    $cached = $cache->get($key);

    if ($cached instanceof ProductData) {
        return $cached;
    }

    $product = $products->findForTenant($tenantId, $productId);

    if ($product === null) {
        return null;
    }

    $data = ProductData::fromProduct($product);
    $cache->set($key, $data, 300);

    return $data;
}
~~~

This example uses the PSR-16 simple-cache shape. A real project must configure a concrete adapter and ensure the value type can be safely stored and retrieved by that adapter. Do not assume every cache stores PHP objects the same way. Some backends serialize values, impose size limits, or are shared among application instances.

The `300` is a TTL in seconds in this example. Choose it from the data's freshness requirement, not just because it is a familiar number. A short TTL limits how long a missed invalidation can remain visible; it does not make invalidation unnecessary when the application promises faster updates.

## Choose complete and safe cache keys

A cache key identifies one answer to one set of inputs. It must include every value that can change the result. The product key above includes a tenant identifier because two tenants can have different product data. Depending on the operation, a key may also need a locale, currency, permission scope, feature version, or normalized filter and sort options.

Use a stable namespace and a version when the cached value's shape or meaning changes. A version such as `product:v3` lets new code avoid reading an incompatible older value. Normalize inputs before building the key, and hash long or unsuitable input strings if needed. Follow the selected backend's key restrictions.

Do not put passwords, access tokens, session identifiers, or unnecessary personal data in keys. Keys can appear in metrics, logs, diagnostic tools, and cache administration interfaces. For personalized values, include an appropriate identity or permission scope, or do not share the cached value at all. A missing identity or tenant component can become a data leak, not just a stale-data bug.

## Expiration and invalidation

Expiration places an upper bound on normal reuse. Invalidation removes or replaces entries when the underlying data changes. Common approaches include:

- **Time-based expiration:** allow the entry to expire after a TTL.
- **Write-through:** update the source and cache as part of a defined write flow.
- **Delete on write:** after a successful source update, remove the related cache key so the next read reloads it.
- **Versioned keys:** move readers to a new version when the data or representation changes.
- **Tags or indexes:** group related keys so a mutation can invalidate a set of entries, when the cache system supports it.

For delete-on-write, update the source first and invalidate only after the source update succeeds. If invalidation fails, the old entry may remain until its TTL expires; the system should have a deliberate response to that possibility. Avoid an order that deletes the cache and then fails to persist the source while other requests populate the cache with an inconsistent value.

A cache write is not a transaction with the database unless a specific system provides such a guarantee. Concurrent requests can also miss the same key and all perform the expensive load. This is a cache stampede. Depending on the cost and traffic, use a short lock, request coalescing, randomized TTL jitter, prewarming, or a documented stale-while-refresh strategy. Any stale-serving strategy needs a clear maximum-staleness contract.

## Cache interfaces in PHP

PSR-6 defines cache pools and cache items. It supports workflows where code obtains an item, checks whether it is a hit, sets a value and optional expiration, and saves through the pool. PSR-16 defines a simpler cache interface with operations such as `get`, `set`, `delete`, and `clear`.

These standards let libraries agree on an API. They do not make every backend durable, shared across servers, equally fast, or identical in serialization and eviction behavior. A cache adapter can report failures, and an entry can disappear before its requested expiration. Decide whether a cache failure should fail the operation or fall back to the source. For an optional performance cache, fallback is often appropriate if the source remains available and safe to query.

Avoid swallowing all exceptions without observability. A cache outage that silently sends every request to a database can overload the source. Log or measure failures without exposing cache keys that contain sensitive information, and consider rate limits or a circuit breaker when fallback traffic needs protection.

## HTTP response caching

HTTP caches store responses according to response metadata and request properties. They are separate from application caches. The `Cache-Control` response header communicates storage and reuse policy:

~~~php
// A public, identical-for-everyone response can be shared for a short period.
header('Cache-Control: public, max-age=300');

// A personalized response may be stored by the user's browser, not a shared cache.
header('Cache-Control: private, max-age=60');

// Do not store a response, for example when its sensitivity requires that policy.
header('Cache-Control: no-store');
~~~

Use `public` only when shared reuse is safe. Personalized account pages and permission-dependent responses must not accidentally be reused for another user. `private` limits storage to a private cache such as the user's browser; it does not encrypt stored data. `no-store` tells caches not to store the response. Apply it when the application's privacy policy requires that behavior.

`no-cache` has a specific meaning: a cache may store the response, but must validate it with the origin before reusing it. It does not simply mean “do not store.” Use validators such as an entity tag or a last-modified timestamp when conditional validation is useful. A successful validation can let the server return that the stored representation is still current without retransmitting its full body.

`Vary` tells an HTTP cache which request headers influence the selected representation. If output changes based on `Accept-Language`, for example, the cache needs to distinguish those variants. Missing or incorrect variation rules can serve the wrong representation. Do not use `Vary` as a substitute for a complete authorization or personalization policy.

Authenticated responses need particular care. A shared cache must not reuse one user's private response for another user. Configure cache directives and the HTTP cache's authorization rules deliberately; when in doubt, keep user-specific output private or disable storage. Unsafe state-changing requests should also trigger the appropriate invalidation of related cached representations.

## A practical review checklist

Before enabling a cache, answer these questions:

1. What is the source of truth, and what work does caching avoid?
2. What exact inputs determine the result, and are they all part of the key?
3. How stale can the result be, and what invalidates it after a write?
4. Can the value contain private data or depend on the current user's permissions?
5. What happens if the entry expires early, the cache is unavailable, or many requests miss at once?
6. Does the cache belong to the application, a PHP runtime, a database, a browser, or a shared HTTP proxy?
7. Can cache hit rate, errors, latency, and staleness be observed without logging secrets?

## Key takeaways

- A cache is a reusable copy; the authoritative source remains elsewhere.
- Cache-aside checks the cache, loads on a miss, and stores with an intentional TTL.
- Keys must include every tenant, identity, locale, permission, and query input that changes the result.
- TTL limits normal staleness, while invalidation handles changes that need faster visibility.
- Eviction, failures, and concurrent misses are normal conditions that need a fallback policy.
- PSR-6 and PSR-16 standardize interfaces, not backend behavior or persistence guarantees.
- HTTP response caching has independent rules. `private`, `no-store`, `no-cache`, and `Vary` have distinct meanings.
- Never allow a shared cache to serve personalized or authorization-dependent output to the wrong user.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [114 - Application configuration and logging](114-application-configuration-and-logging.md) | [Q&A](../qnas/115-caching-data-and-application-output.md) | [Example](../examples/115-caching-data-and-application-output.md) | [116 - PHP extensions and the extension ecosystem](116-php-extensions-and-the-extension-ecosystem.md) |
