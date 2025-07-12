# Using JCache

  

The App Engine Java SDK supports the JCache interface ([JSR 107](https://web.archive.org/web/20160424230647/http://jcp.org/en/jsr/detail?id=107)) for accessing memcache. The interface is included in the `javax.cache` package.

With JCache, you can set and get values, control how values expire from the cache, inspect the contents of the cache, and get statistics about the cache. You can also use "listeners" to add custom behavior when setting and deleting values.

The JCache API standard is still in development. The App Engine implementation tries to implement a loyal subset. (For more information about JCache, see [JSR 107](https://web.archive.org/web/20160424230647/http://jcp.org/en/jsr/detail?id=107).) However, instead of using JCache, you may want to consider using the [low-level Memcache API](https://web.archive.org/web/20160424230647/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/package-summary) to access more features of the underlying service.

1.  [Putting and Getting Values](#Putting_and_Getting_Values)
2.  [Configuring Expiration](#Configuring_Expiration)
3.  [Configuring the Set Policy](#Configuring_the_Set_Policy)
4.  [Cache Statistics](#Cache_Statistics)
5.  [JCache Features Not Supported](#Features_Not_Supported)
6.  [Features of the Low-level API](#Features_of_the_Low_level_API)

## Getting a Cache Instance

You use an implementation of the `javax.cache.Cache` interface to interact with the cache. You obtain a Cache instance using a CacheFactory, which you obtain from a static method on the CacheManager. The following code gets a Cache instance with the default configuration:

```
import java.util.Collections;
import javax.cache.Cache;
import javax.cache.CacheException;
import javax.cache.CacheFactory;
import javax.cache.CacheManager;

// ...
        Cache cache;
        try {
            CacheFactory cacheFactory = CacheManager.getInstance().getCacheFactory();
            cache = cacheFactory.createCache(Collections.emptyMap());
        } catch (CacheException e) {
            // ...
        }
```

The CacheFactory's `createCache()` method takes a Map of configuration properties. These properties are discussed below. To accept the defaults, give the method an empty Map.

## Putting and Getting Values

The Cache behaves like a Map: You store keys and values using the `put()` method, and retrieve values using the `get()` method. You can use any Serializable object for either the key or the value.

```
        String key;      // ...
        byte[] value;    // ...

        // Put the value into the cache.
        cache.put(key, value);

        // Get the value from the cache.
        value = (byte[]) cache.get(key);
```

To put multiple values, you can call the `putAll()` method with a Map as its argument.

To remove a value from the cache (to evict it immediately), call the `remove()` method with the key as its argument. To remove every value from the cache for the application, call the `clear()` method.

The `containsKey()` method takes a key, and returns a `boolean` (`true` or `false`) to indicate whether a value with that key exists in the cache. The `isEmpty()` method tests whether the cache is empty. The `size()` method returns the number of values currently in the cache.

## Configuring Expiration

By default, all values remain in the cache as long as possible, until evicted due to memory pressure, removed explicitly by the app, or made unavailable for another reason (such as an outage). The app can specify a expiration time for values, a maximum amount of time the value will be available. The expiration time can be set as an amount of time relative to when the value is set, or as an absolute date and time.

You specify the expiration policy using configuration properties when you create the Cache instance. All values put with that instance use the same expiration policy. For example, to configure a Cache instance to expire values one hour (3,600 seconds) after they are set:

```
import java.util.HashMap;
import java.util.Map;
import javax.cache.Cache;
import javax.cache.CacheException;
import javax.cache.CacheFactory;
import javax.cache.CacheManager;
import javax.concurrent.TimeUnit;
import com.google.appengine.api.memcache.jsr107cache.GCacheFactory;

// ...
        Cache cache;
        try {
            CacheFactory cacheFactory = CacheManager.getInstance().getCacheFactory();
            Map properties = new HashMap<>();
            properties.put(GCacheFactory.EXPIRATION_DELTA, TimeUnit.HOURS.toSeconds(1));
            cache = cacheFactory.createCache(properties);
        } catch (CacheException e) {
            // ...
        }
```

The following properties control value expiration:

-   `GCacheFactory.EXPIRATION_DELTA`: expire values the given amount of time relative to when they are put, as an Integer number of seconds
-   `GCacheFactory.EXPIRATION_DELTA_MILLIS`: expire values the given amount of time relative to when they are put, as an Integer number of milliseconds
-   `GCacheFactory.EXPIRATION`: expire values at the given date and time, as a java.util.Date

## Configuring the Set Policy

By default, setting a value in the cache adds the value if there is no value with the given key, and replaces a value if there is a value with the given key. You can configure the cache to only add (protect existing values) or only replace values (do not add).

```
import java.util.HashMap;
import java.util.Map;
import com.google.appengine.api.memcache.MemcacheService;

// ...
        Map props = new HashMap<>();
        properties.put(MemcacheService.SetPolicy.ADD_ONLY_IF_NOT_PRESENT, true);
```

The following properties control the set policy:

-   `MemcacheService.SetPolicy.SET_ALWAYS`: add the value if no value with the key exists, replace an existing value if a value with the key exists; this is the default
-   `MemcacheService.SetPolicy.ADD_ONLY_IF_NOT_PRESENT`: add the value if no value with the key exists, do nothing if the key exists
-   `MemcacheService.SetPolicy.REPLACE_ONLY_IF_PRESENT`: do nothing if no value with the key exists, replace an existing value if a value with the key exists

## Cache Statistics

The app can retrieve statistics about its own use of the cache. These statistics are useful for monitoring and tuning cache behavior. You access statistics using a CacheStatistics object, which you get by calling the Cache's `getCacheStatistics()` method.

Available statistics include the number of cache hits (gets for keys that existed), the number of cache misses (gets for keys that did not exist), and the number of values in the cache.

```
import javax.cache.CacheStatistics;

// ...
        CacheStatistics stats = cache.getCacheStatistics();
        int hits = stats.getCacheHits();
        int misses = stats.getCacheMisses();
```

The App Engine implementation does not support resetting the hit and miss counts. Hit and miss counts are maintained indefinitely, but may be reset due to transient conditions of the memcache servers.

## JCache Features Not Supported

The JCache listener API is partially supported for listeners that can execute during the processing of a app's API call, such as for onPut and onRemove listeners. Listeners that require background processing, like onEvict, are not supported.

An app can test whether the cache contains a given key, but it cannot test whether the cache contains a given value (`containsValue()` is not supported).

An app cannot dump the contents of the cache's keys or values.

An app cannot manually reset cache statistics.

The `put()` method does not return the previous known value for a key. It always returns `null`.

## Features of the Low-level API

The low-level Memcache API includes a number of additional features such as:

-   Atomically increment and decrement integer counter values
-   Expose more cache statistics, such as the amount of time since the least-recently-used entry was accessed, and the total size of all items in the cache.
-   Check and set operations to conditionally store data
-   Perform memcache operations asynchronously, using the [AsyncMemcacheService](https://web.archive.org/web/20160424230647/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/AsyncMemcacheService).

For more information on the low-level API, see the [Memcache Javadoc](https://web.archive.org/web/20160424230647/https://cloud.google.com/appengine/docs/java/javadoc/com/google/appengine/api/memcache/package-summary).
