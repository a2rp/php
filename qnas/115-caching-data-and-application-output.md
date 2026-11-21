# 115 - Caching data and application output: Q&A

## Multiple-choice questions

### 1. What is the main purpose of a cache?

- **A.** Keep a reusable copy so repeated work can sometimes be avoided
- **B.** Replace the authoritative database permanently
- **C.** Guarantee that data never changes
- **D.** Store every application secret

**Answer: A.** A cache can reduce repeated work and latency. **Explanation:** The authoritative source remains the database, service, or other source of truth; cache entries can expire, be evicted, or disappear.

### 2. In cache-aside, what normally happens after a cache miss?

- **A.** Load from the source, store the result with a chosen lifetime, and return it
- **B.** Return an empty response without checking the source
- **C.** Permanently disable the cache
- **D.** Treat the missing entry as proof that the record does not exist

**Answer: A.** Cache-aside loads from the source on a miss and may then cache the result. **Explanation:** A cache miss is not the same as a missing record; the source must decide whether the data exists.

### 3. What does a cache TTL usually express?

- **A.** The requested lifetime before the entry expires
- **B.** The number of users allowed to read a value
- **C.** A guarantee that a backend will never evict the entry early
- **D.** The time a database transaction takes

**Answer: A.** TTL is the requested time-to-live. **Explanation:** A backend may evict an entry earlier, so application correctness must not depend on the entry surviving for the full TTL.

### 4. What belongs in a cache key?

- **A.** Every input that can change the cached result
- **B.** Only the PHP class name
- **C.** A random value generated on every request
- **D.** A password so the key is unique

**Answer: A.** Keys must distinguish results that differ. **Explanation:** Depending on the operation, inputs may include tenant, identity or permission scope, locale, currency, normalized filters, and a representation version.

### 5. What is PSR-16 intended to provide?

- **A.** A simple cache interface with operations such as get, set, delete, and clear
- **B.** A universal distributed cache server
- **C.** A required file format for all cache values
- **D.** A guarantee that all cache adapters have identical persistence

**Answer: A.** PSR-16 standardizes a simple cache API. **Explanation:** It does not make different backends identical in storage, durability, eviction, or serialization behavior.

### 6. Which description best matches PSR-6?

- **A.** A cache pool and cache item model
- **B.** A browser response-header standard
- **C.** A PHP extension installer
- **D.** A database schema migration format

**Answer: A.** PSR-6 models cache items obtained from a pool and saved through that pool. **Explanation:** It provides an interoperable API, not a guarantee about the underlying cache server.

### 7. When using delete-on-write, when should the application normally invalidate the cache?

- **A.** After the source update succeeds
- **B.** Before attempting the source update, regardless of its outcome
- **C.** Only when a user reports stale data
- **D.** Before reading the source value

**Answer: A.** Invalidate after the authoritative update succeeds. **Explanation:** If the source update fails after deletion, another request could repopulate the cache with the old value.

### 8. What is a cache stampede?

- **A.** Many requests miss the same key and repeat the same expensive load concurrently
- **B.** A cache key containing a namespace
- **C.** A successful cache hit
- **D.** An HTTP cache validating a representation

**Answer: A.** Simultaneous misses can overwhelm the source with duplicate work. **Explanation:** Depending on traffic and cost, mitigation may include locks, request coalescing, TTL jitter, or prewarming.

### 9. What does an HTTP `Vary` response header do?

- **A.** Identifies request headers that distinguish cached response variants
- **B.** Clears all PHP cache pools
- **C.** Prevents a browser from storing any response
- **D.** Encrypts a response in a shared cache

**Answer: A.** `Vary` tells an HTTP cache which request headers affect representation selection. **Explanation:** Incorrect variation can cause one request variant to receive another variant's response.

### 10. What does HTTP `Cache-Control: no-cache` mean?

- **A.** A response may be stored, but must be validated before reuse
- **B.** The response can never be stored anywhere
- **C.** The response is encrypted before storage
- **D.** A PSR-16 entry is cleared

**Answer: A.** `no-cache` requires successful validation before a stored response is reused. **Explanation:** `no-store` is the directive that tells caches not to store a response.

### 11. What does HTTP `Cache-Control: private` indicate?

- **A.** The response is intended for a private cache, not a shared cache
- **B.** The response body is encrypted at rest
- **C.** The browser must never store the response
- **D.** The response is safe to serve to every user

**Answer: A.** `private` restricts reuse to private caches such as a user's browser. **Explanation:** It does not encrypt data or make unsafe personalized output safe for a shared cache.

### 12. When is `Cache-Control: no-store` useful?

- **A.** When policy requires caches not to store the response
- **B.** Whenever a public image is cached for a day
- **C.** To make a PHP cache entry expire after five minutes
- **D.** To force a response to be shared among users

**Answer: A.** `no-store` communicates that the response should not be stored by caches. **Explanation:** Choose response directives according to the sensitivity and reuse requirements of the output.

## Short-answer questions

### 13. Why should an application treat its cache as an optimization rather than its source of truth?

**Answer:** Cache entries can expire, be evicted, or vanish during failures and restarts. The authoritative store must still be able to determine the correct value. **Explanation:** Treating cached state as permanent can make normal eviction appear to be data loss.

### 14. How are an application data cache and an HTTP response cache different?

**Answer:** An application cache stores values used by application code, while an HTTP cache stores HTTP representations according to response metadata and request properties. **Explanation:** They have separate keys, policies, and invalidation paths; changing one does not automatically clear the other.

### 15. Why must a tenant identifier be included when the same lookup can return tenant-specific data?

**Answer:** Without it, two tenants could map to the same key and receive each other's cached data. **Explanation:** Key completeness is a security boundary as well as a freshness concern.

### 16. Why might a cache key include a version such as `product:v3`?

**Answer:** The version can separate values produced under different schemas or meanings. **Explanation:** New code can stop reading an incompatible old entry without needing to discover every prior key.

### 17. Is a five-minute TTL by itself a guarantee that data is never more than five minutes stale?

**Answer:** No. TTL bounds normal reuse if the backend honors it, but entries can be evicted early, and application behavior around refresh and invalidation still matters. **Explanation:** The allowed staleness contract must consider writes, concurrent reads, outages, and backend behavior.

### 18. What is the purpose of invalidating a related cache key after a successful data update?

**Answer:** It prevents future reads from continuing to reuse a known old value and causes them to reload from the source. **Explanation:** Invalidation should be tied to the mutation that changed the authoritative data.

### 19. What should a cache-aside reader do if the cache entry is absent?

**Answer:** Query the authoritative source and handle its result normally. **Explanation:** A cache miss says only that there is no reusable cached value; it does not say whether the source record exists.

### 20. Why can cache failures need monitoring even if the application falls back to the source?

**Answer:** Silent fallback can increase database or upstream traffic enough to cause a secondary outage. **Explanation:** Track cache errors and fallback load, while avoiding logs that expose sensitive keys or values.

### 21. What problem can TTL jitter help reduce?

**Answer:** It can spread expirations that would otherwise happen at the same time and trigger a burst of simultaneous reloads. **Explanation:** Jitter reduces synchronized expiry; it does not replace a sound refresh or locking strategy for expensive values.

### 22. Why should code avoid assuming every cache adapter stores PHP objects identically?

**Answer:** Backends can use different serialization, size, and supported-value rules. **Explanation:** Define and test the value representation expected by the configured adapter, or cache a portable representation.

### 23. What do HTTP entity tags or last-modified validators enable?

**Answer:** They let a client or intermediary validate whether a stored representation is still current, often without downloading the full body again. **Explanation:** Validation is useful when a response can be reused conditionally and the origin can reliably identify changes.

### 24. Why is `private` not a complete security measure for a personalized response?

**Answer:** It limits shared-cache storage but does not encrypt the response, authorize the user, or ensure a private cache is on a trusted device. **Explanation:** The application must still protect the endpoint and select storage directives appropriate to the data.

## Scenario questions

### 25. A product lookup is cached under `product:42`, but product 42 differs by tenant. What should change?

**Answer:** Include a normalized tenant identifier in the key, for example `product:v1:<tenant>:42`, and invalidate the tenant-specific key after a successful update. **Explanation:** Tenant data must not collide in a shared cache namespace.

### 26. The application caches a user's dashboard under `dashboard:home`. What is the risk and a better approach?

**Answer:** Different users may receive the same cached dashboard. Include the appropriate user or permission scope in an application-cache key, or avoid caching it. For HTTP, prevent shared reuse and choose private or no-store behavior according to the data. **Explanation:** Authorization-dependent output must never be served to an unintended user.

### 27. A database update fails after the application has already deleted the cache entry. Another request repopulates it from the unchanged database. What was the ordering problem?

**Answer:** The cache was invalidated before the authoritative write had succeeded. Update the source first, then invalidate after success. **Explanation:** Otherwise requests can refill the cache from the old source state while a write is failing.

### 28. A popular cache key expires and hundreds of requests query the database at once. What happened, and name two mitigations.

**Answer:** This is a cache stampede. Possible mitigations include per-key locking, coalescing concurrent requests, TTL jitter, prewarming, or a carefully bounded stale-while-refresh policy. **Explanation:** Select a strategy that fits the source's capacity and the data's freshness contract.

### 29. A public page changes according to the request's `Accept-Language` header, but the response has no corresponding variation policy. What can go wrong?

**Answer:** An HTTP cache may reuse one language's response for a request asking for another language. Declare the relevant variation, such as `Vary: Accept-Language`, and ensure the cache key and allowed variants are appropriate. **Explanation:** Every request property that selects a different representation must be accounted for.

### 30. A response says `Cache-Control: no-cache`. A developer assumes that no copy can be stored. Is that correct?

**Answer:** No. The response may be stored, but it must be validated before reuse. Use `no-store` when storage itself is disallowed. **Explanation:** The directive names are easy to confuse but specify different cache behavior.

### 31. An account endpoint returns different private information for each signed-in user, and a CDN is configured as a shared cache. What should the team do?

**Answer:** Do not allow the CDN to reuse one user's response for another. Configure private/non-shared response policy or disable storage, and verify the CDN's authorization behavior. **Explanation:** A generic cache key based only on the URL is not sufficient for personalized output.

### 32. The cache is unavailable, but the database is healthy. Should every request necessarily fail?

**Answer:** Not necessarily. If caching is optional, the application can fall back to the source while reporting the cache error and protecting the source from overload. **Explanation:** A fallback policy should be explicit; unbounded fallback can turn a cache outage into a database outage.

### 33. A value contains permission-filtered results, but the cache key includes only the record ID. What correction is needed?

**Answer:** Include the permission scope or another stable authorization dimension in the key, or do not cache the filtered result. Re-check authorization on every request. **Explanation:** A cache hit must not bypass access control or reuse one role's view for another role.

### 34. A template fragment is regenerated by the application and then the complete page is also cached by a browser. Does clearing the template fragment necessarily clear the browser's page copy?

**Answer:** No. These are separate cache layers. The application must handle fragment invalidation and separately control or invalidate HTTP response caching. **Explanation:** The stale copy may be at a different layer than the one being cleared.

### 35. A deployment changes a cached array's shape, and old entries cause errors. Give a low-risk key strategy.

**Answer:** Add a schema or representation version to the key namespace, such as changing `settings:v2:<scope>` to `settings:v3:<scope>`. **Explanation:** New code then reads the new representation while old entries expire or are removed independently.

### 36. A response contains a one-time recovery token. What HTTP caching policy should be considered?

**Answer:** Use `Cache-Control: no-store` when the security policy requires that the response not be stored, and avoid exposing the token through logs or shared infrastructure. **Explanation:** `private` alone still allows a private cache to store a response and does not encrypt the stored copy.

## Chapter navigation

| Previous chapter | Chapter lesson | Chapter example | Next chapter |
| --- | --- | --- | --- |
| [114 - Application configuration and logging](../lessons/114-application-configuration-and-logging.md) | [Lesson](../lessons/115-caching-data-and-application-output.md) | [Example](../examples/115-caching-data-and-application-output.md) | [116 - PHP extensions and the extension ecosystem](../lessons/116-php-extensions-and-the-extension-ecosystem.md) |
