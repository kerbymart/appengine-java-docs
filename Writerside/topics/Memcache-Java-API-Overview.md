# Memcache Java API Overview

  

[Python](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/python/memcache/ "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/php/memcache/ "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/go/memcache/ "View this page in the Go runtime")

[<img src="https://web.archive.org/web/20160425102002im_/https://cloud.google.com/cloud/images/stack_overflow_questions.png" title="Stack Overflow Questions" data-border="0" width="240" height="53" />](https://web.archive.org/web/20160425102002/http://stackoverflow.com/questions/tagged/google-app-engine+memcached)

[](https://web.archive.org/web/20160425102002/http://stackoverflow.com/feeds/tag?sort=votes&tagnames=google-app-engine%2Bmemcached)

<span class="link gc-analytics-event" category="Sidebar" data-action="Stack Overflow question click"></span>

<a href="https://web.archive.org/web/20160425102002/http://stackoverflow.com/questions/tagged/google-app-engine+memcached?sort=votes" class="gc-analytics-event" data-category="Sidebar" data-action="Stack Overflow See more link">See more...</a>

High performance scalable web applications often use a distributed in-memory data cache in front of or in place of robust persistent storage for some tasks. App Engine includes a memory cache service for this purpose.

**Note:** The cache is global and is shared across the application's frontend, backend, and all of its modules/versions.

1.  [When to use a memory cache](#Java_When_to_use_a_memory_cache)
2.  [Limits](#Java_Limits)
3.  [Caching data with the Low-Level API](#Java_Caching_data_with_the_Low-Level_API)
4.  [Caching data with JCache](#Java_Caching_data_with_JCache)
5.  [How cached data expires](#Java_How_cached_data_expires)
6.  [Configuring memcache](#Java_Configuring_memcache)
7.  [Monitoring memcache](#Java_Statistics)
8.  [Safely handling concurrent memcache updates](#Java_Safely_handling_concurrent_memcache_updates)
9.  [Best practices](#Java_best_practices)

## When to use a memory cache

One use of a memory cache is to speed up common datastore queries. If many requests make the same query with the same parameters, and changes to the results do not need to appear on the web site right away, the app can cache the results in the memcache. Subsequent requests can check the memcache, and only perform the datastore query if the results are absent or expired. Session data, user preferences, and any other queries performed on most pages of a site are good candidates for caching.

Memcache may be useful for other temporary values. However, when considering whether to store a value solely in the memcache and not backed by other persistent storage, be sure that your application behaves acceptably when the value is suddenly not available. Values can expire from the memcache at any time, and may be expired prior to the expiration deadline set for the value. For example, if the sudden absence of a user's session data would cause the session to malfunction, that data should probably be stored in the datastore in addition to the memcache.

The memcache service provides best-effort cache space by default. Apps with billing enabled may opt to use [dedicated memcache](#Java_Configuring_memcache), which provides a fixed cache size assigned exclusively to your app.

## Limits

The following limits apply to the use of the memcache service:

-   The maximum size of a cached data value is 1 MiB (2^20 bytes) minus the size of the key minus an implementation-dependent overhead, which is approximately 73 bytes.
-   A key cannot be larger than 250 bytes. In the Java runtime, keys that are objects or strings longer than 250 bytes will be hashed. (Other runtimes behave differently.)
-   The "multi" batch operations can have any number of elements. The total size of the call and the total size of the data fetched must not exceed 32 megabytes.
-   A memcache key cannot contain a null byte.

## Caching data with the Low-Level API

The Low-Level API provides [`MemcacheService`](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/MemcacheService) and [`AsyncMemcacheService`](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/AsyncMemcacheService) for accessing memcache service. This API is richer than the one provided by JCache. For more details see [Low-Level API](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/package-summary).

```
  String key = ..
  byte[] value;

  // Using the synchronous cache.
  MemcacheService syncCache = MemcacheServiceFactory.getMemcacheService();
  syncCache.setErrorHandler(ErrorHandlers.getConsistentLogAndContinue(Level.INFO));
  value = (byte[]) syncCache.get(key); // Read from cache.
  if (value == null) {
    // Get value from another source.
    // ........

    syncCache.put(key, value); // Populate cache.
  }

  // Using the asynchronous cache.
  AsyncMemcacheService asyncCache = MemcacheServiceFactory.getAsyncMemcacheService();
  asyncCache.setErrorHandler(ErrorHandlers.getConsistentLogAndContinue(Level.INFO));
  Future<Object> futureValue = asyncCache.get(key); // Read from cache.
  // ... Do other work in parallel to cache retrieval.
  value = (byte[]) futureValue.get();
  if (value == null) {
    // Get value from another source.
    // ........

    // Asynchronously populate the cache.
    // Returns a Future<Void> which can be used to block until completion.
    asyncCache.put(key, value);
  }
```

## Caching data with JCache

The App Engine Java SDK supports the JCache API. JCache provides a map-like interface to cached data. You store and retrieve values in the cache using keys. Keys and values can be of any Serializable type or class. For more details, see [Using JCache](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/memcache/usingjcache).

## How cached data expires

By default, values stored in memcache are retained as long as possible. Values may be evicted from the cache when a new value is added to the cache if the cache is low on memory. When values are evicted due to memory pressure, the least recently used values are evicted first.

The app can provide an expiration time when a value is stored, as either a number of seconds relative to when the value is added, or as an absolute Unix epoch time in the future (a number of seconds from midnight January 1, 1970). The value will be evicted no later than this time, though it may be evicted for other reasons.

Under rare circumstances, values may also disappear from the cache prior to expiration for reasons other than memory pressure. While memcache is resilient to server failures, memcache values are not saved to disk, so a service failure may cause values to become unavailable.

In general, an application should not expect a cached value to always be available.

You can erase an application's entire cache via the API or in the memcache section of Google Cloud Platform Console, as described in [Managing memcache](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/console/#memcache).

**Note:** The actual removal of expired cache data is handled lazily. An expired item will be removed when someone tries to retrieve it. Alternatively, it will fall out of the cache according to LRU cache behavior, which applies to all items, both live and expired. This means when the cache size is reported in statistics, the number can include live and expired items.

## Configuring memcache

App Engine supports two classes of the memcache service:

-   Shared memcache is the free default for App Engine applications. It provides cache capacity on a best-effort basis and is subject to the overall demand of all the App Engine applications using the shared memcache service.

-   Dedicated memcache provides a fixed cache capacity assigned exclusively to your application. It's billed by the GB-hour of cache size. Having control over cache size means your app can perform more predictably and with fewer accesses to more costly durable storage.

Both memcache service classes use the same API. Select the memcache service class for an app in the Google Cloud Platform Console, as described in [Managing memcache](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/console/#memcache).

Whether shared or dedicated, memcache is not durable storage. Keys may be evicted when the cache fills up, according to the cache's LRU policy. Changes in the cache configuration or datacenter maintenance events may also flush some or all of the cache.

The following table summarizes the differences between the two classes of memcache service:

<table>
<thead>
<tr class="header">
<th width="15%">Feature</th>
<th width="55%">Dedicated Memcache</th>
<th width="30%">Shared Memcache</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Price</td>
<td>$0.06 per GB per hour</td>
<td>Free</td>
</tr>
<tr class="even">
<td>Capacity</td>
<td>1 to 100GB</td>
<td>No guaranteed capacity</td>
</tr>
<tr class="odd">
<td>Performance</td>
<td>Up to 10k reads or 5k writes (exclusive) per second per GB (items &lt; 1KB). For more details, see <a href="https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/console/#monitoring_memcache">Monitoring memcache</a>.</td>
<td>Not guaranteed</td>
</tr>
<tr class="even">
<td>Durable store</td>
<td>No</td>
<td>No</td>
</tr>
<tr class="odd">
<td>SLA</td>
<td>None</td>
<td>None</td>
</tr>
</tbody>
</table>

Dedicated memcache billing is charged in 15 minute increments. When charging in local currency, Google will convert the prices listed into applicable local currency pursuant to the conversion rates published by leading financial institutions.

If your app needs more than 100GB of cache, please contact us at [cloud-accounts-team@google.com](https://web.archive.org/web/20160425102002/mailto:cloud-accounts-team@google.com).

## Monitoring memcache

For information about memcache performance and the relative cost of each operation type, refer to the [memcache section](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/console/#monitoring_memcache) of the page "Using the Google Cloud Platform Console for App Engine".

## Safely handling concurrent memcache updates

If you're updating the value of a memcache key that might receive other concurrent write requests, you must use the memcache methods [`putIfUntouched`](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/MemcacheService#putIfUntouched(java.lang.Object,%20com.google.appengine.api.memcache.MemcacheService.IdentifiableValue,%20java.lang.Object,%20com.google.appengine.api.memcache.Expiration)) and [`getIdentifiable`](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/MemcacheService#getIdentifiable(java.lang.Object)) instead of `put` and `get`. The methods `putIfUntouched` and `getIdentifiable` allows multiple requests that are being handled concurrently to update the value of the same memcache key atomically, avoiding race conditions.

The code snippet below shows one way to safely update the value of a key that might have concurrent update requests from other clients:

```
  String key = ..

  // Using the synchronous cache.
  MemcacheService syncCache = MemcacheServiceFactory.getMemcacheService();

 // Write this value to cache using getIdentifiable and putIfUntouched.
  for (long delayMs = 1; ; delayMs *= 2)  {
    IdentifiableValue oldValue = syncCache.getIdentifiable(key);
    byte[] newValue = foo(oldValue.getValue());  // newValue depends on oldValue
    if (oldValue == null) {
      // Key doesn't exist. We can safely put it in cache.
      syncCache.put(key, newValue);
      break;
    } else if (syncCache.putIfUntouched(key, oldValue, newValue)) {
      // newValue has been successfully put into cache.
      break;
    } else {
      // Some other client changed the value since oldValue was retrieved.
      // Wait a while before trying again, waiting longer on successive loops.
      Thread.sleep(delayMs);
    }
  }
```
**Note:** These methods are also available in the [`AsyncMemcacheService`](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/AsyncMemcacheService).

A refinement you could add to this sample code is to set a limit on the number of retries, to avoid blocking for so long that your App Engine request times out.

## Best practices

Following are some best practices for using memcache:

-   **Handle memcache API failures gracefully.** Memcache operations can fail for various reasons. Applications should be designed to catch failed operations without exposing these errors to end users. This applies especially to Set operations.

-   **Use the batching capability of the API when possible**, especially for small items. This will increase the performance and efficiency of your app.

-   **Distribute load across your memcache keyspace.** Having a single or small set of memcache items represent a disproportionate amount of traffic will hinder your app from scaling. This applies to both operations/sec and bandwidth. The problem can often be alleviated by explicit sharding of your data. For example, a frequently updated counter can be split among several keys, reading them back and summing only when a total is needed. Likewise, a 500K piece of data that must be read on every HTTP request can be split across multiple keys and read back using a single batch API call. (Even better would be to cache the value in instance memory.) For dedicated memcache, the peak access rate on a single key should be 1-2 orders of magnitude less than the per-GB rating.

For more details and more best practices for concurrency, performance, and migration, including sharing memcache between different programming languages, read the article [Best Practices for App Engine Memcache](https://web.archive.org/web/20160425102002/https://cloud.google.com/appengine/articles/best-practices-for-app-engine-memcache).
