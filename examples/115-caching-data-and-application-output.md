# 115 - Caching data and application output: Example

## Goal

Build a small PHP CLI catalog that uses cache-aside for product data, includes a tenant in the cache key, and invalidates the entry after a successful update.

## Question

How can an application avoid repeating a source lookup while ensuring a successful write makes the next read fetch the updated value?

## Assumptions

- PHP 8.1 or newer is available on the command line.
- Commands are run from the project directory.
- This example uses an in-memory cache and repository to make the flow visible without installing packages.
- The in-memory cache lasts only for this process. It is not shared between requests or application servers and is not a production cache backend.
- The PHP code is provided for study; it was not executed here.

## 1. Create the project structure

~~~text
cache-demo/
└── cache-demo.php
~~~

## 2. Add the cache-aside catalog

Create `cache-demo.php`:

~~~php
<?php

declare(strict_types=1);

interface CacheStore
{
    public function get(string $key, mixed $default = null): mixed;

    public function set(string $key, mixed $value, ?int $ttlSeconds = null): void;

    public function delete(string $key): void;
}

final class ArrayCache implements CacheStore
{
    /** @var array<string, array{value: mixed, expires_at: ?int}> */
    private array $items = [];

    public function get(string $key, mixed $default = null): mixed
    {
        $entry = $this->items[$key] ?? null;

        if ($entry === null) {
            return $default;
        }

        if ($entry['expires_at'] !== null && $entry['expires_at'] <= time()) {
            unset($this->items[$key]);

            return $default;
        }

        return $entry['value'];
    }

    public function set(string $key, mixed $value, ?int $ttlSeconds = null): void
    {
        if ($ttlSeconds !== null && $ttlSeconds <= 0) {
            unset($this->items[$key]);

            return;
        }

        $expiresAt = $ttlSeconds === null ? null : time() + $ttlSeconds;

        $this->items[$key] = [
            'value' => $value,
            'expires_at' => $expiresAt,
        ];
    }

    public function delete(string $key): void
    {
        unset($this->items[$key]);
    }
}

interface ProductRepository
{
    /** @return array{id: int, name: string, price: float}|null */
    public function find(int $id): ?array;

    /** @param array{id: int, name: string, price: float} $product */
    public function save(array $product): void;
}

final class InMemoryProductRepository implements ProductRepository
{
    /** @var array<int, array{id: int, name: string, price: float}> */
    private array $products;

    public int $readCount = 0;

    /** @param array<int, array{id: int, name: string, price: float}> $products */
    public function __construct(array $products)
    {
        $this->products = $products;
    }

    public function find(int $id): ?array
    {
        $this->readCount++;

        return $this->products[$id] ?? null;
    }

    public function save(array $product): void
    {
        $this->products[$product['id']] = $product;
    }
}

final class ProductCatalog
{
    public function __construct(
        private CacheStore $cache,
        private ProductRepository $products,
        private string $tenantId,
    ) {
    }

    /** @return array{id: int, name: string, price: float}|null */
    public function find(int $id): ?array
    {
        $key = $this->cacheKey($id);
        $cached = $this->cache->get($key);

        if (is_array($cached)) {
            return $cached;
        }

        $product = $this->products->find($id);

        if ($product !== null) {
            $this->cache->set($key, $product, 300);
        }

        return $product;
    }

    /** @param array{id: int, name: string, price: float} $product */
    public function update(array $product): void
    {
        // Persist first. Invalidate only after the source update succeeds.
        $this->products->save($product);
        $this->cache->delete($this->cacheKey($product['id']));
    }

    private function cacheKey(int $id): string
    {
        return 'product:v1:' . $this->tenantId . ':' . $id;
    }
}

$cache = new ArrayCache();
$repository = new InMemoryProductRepository([
    42 => ['id' => 42, 'name' => 'Notebook', 'price' => 999.00],
]);
$catalog = new ProductCatalog($cache, $repository, 'tenant-a');

$first = $catalog->find(42);
printf("First read: %s, price = %.2f\n", $first['name'], $first['price']);
printf("Source reads after first read: %d\n", $repository->readCount);

$second = $catalog->find(42);
printf("Second read (cache hit): %s, price = %.2f\n", $second['name'], $second['price']);
printf("Source reads after cache hit: %d\n", $repository->readCount);

$catalog->update([
    'id' => 42,
    'name' => 'Notebook',
    'price' => 1200.00,
]);

$updated = $catalog->find(42);
printf("Updated read: %s, price = %.2f\n", $updated['name'], $updated['price']);
printf("Source reads after invalidation: %d\n", $repository->readCount);
~~~

The key includes a tenant and a representation version, so a different tenant or incompatible cache format does not reuse this entry. The five-minute TTL bounds normal reuse. A real application should choose the lifetime based on its freshness requirements and invalidate or version entries when changes need to become visible sooner.

The update writes to the repository first. If that write throws or otherwise fails, execution does not reach cache invalidation. After a successful write, deletion forces the next lookup to load the updated product.

## 3. Run the CLI example

From `cache-demo/`, run:

~~~sh
php cache-demo.php
~~~

Expected output:

~~~text
First read: Notebook, price = 999.00
Source reads after first read: 1
Second read (cache hit): Notebook, price = 999.00
Source reads after cache hit: 1
Updated read: Notebook, price = 1200.00
Source reads after invalidation: 2
~~~

The second lookup does not increment the repository read count because it uses the cached value. After `update()` saves the new price and deletes the key, the next lookup reads the repository again and stores the updated product.

## 4. Set HTTP response policy separately

The CLI data cache above does not control browser, proxy, or CDN response caching. Set HTTP response headers at the route or response boundary according to the representation:

~~~php
// Public page shared by visitors; the language changes the representation.
header('Cache-Control: public, max-age=300');
header('Vary: Accept-Language');
~~~

For a user-specific page that can be kept briefly in that user's private cache:

~~~php
header('Cache-Control: private, max-age=60');
~~~

For a response whose sensitivity requires that caches not store it:

~~~php
header('Cache-Control: no-store');
~~~

Choose one policy for the response; do not emit these examples together. `private` does not encrypt a stored response, `no-store` disallows storage, and `Vary` distinguishes public variants such as language. Never let a shared cache reuse authorization-dependent output across users.

## Important notes

- `ArrayCache` exists only during this PHP process. Use a configured adapter for data shared across requests or servers.
- This local cache contract is deliberately smaller than PSR-16 and is not a replacement for a standard cache implementation.
- The in-memory repository is only a stand-in for a database or other source of truth.
- The example does not cache missing products, preventing an absent result from remaining hidden after a later create operation.
- If negative caching is introduced, give it a deliberate short TTL and invalidate it after creation.
- Cache keys must include all result-changing dimensions, including tenant, authorization scope, locale, and normalized query options where applicable.
- Cache failures, concurrent misses, serialization behavior, and multi-server invalidation need production-specific policies.
- The HTTP examples are response-header snippets, not part of the CLI command.
- The PHP snippets are provided for study and were not executed here.

## Chapter navigation

| Previous chapter | Chapter Q&A | Chapter lesson | Next chapter |
| --- | --- | --- | --- |
| [114 - Application configuration and logging](../lessons/114-application-configuration-and-logging.md) | [Q&A](../qnas/115-caching-data-and-application-output.md) | [Lesson](../lessons/115-caching-data-and-application-output.md) | [116 - PHP extensions and the extension ecosystem](../lessons/116-php-extensions-and-the-extension-ecosystem.md) |
