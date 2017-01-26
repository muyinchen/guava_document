# Example

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .expireAfterWrite(10, TimeUnit.MINUTES)
       .removalListener(MY_LISTENER)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
```

# Applicability

缓存在各种各样的用例中非常有用。 例如，当值计算或检索付出代价很高时，并且您将需要在某个需求上多次使用这个值，应考虑使用高速缓存。

`Cache`类似于`ConcurrentMap`，但也不完全相同。 最根本的区别是，一个`ConcurrentMap`持久化添加到它的所有元素，直到它们被显式删除。 另一方面，`Cache`通常配置为自动清除内容，以限制其内存占用。 在某些情况下，`LoadingCache`可能是有用的，即使它不清除内容，因为它会自动加载缓存。

一般来说，Guava缓存实用程序适用于：

*你愿意花一些内存空间来提高速度。

*你期望某些键有时会被查询不止一次。

*缓存中存放的数据总量不会超出内存容量。 （Guava Cache 是单个应用运行时的本地缓存。它不把数据存放到文件或外部服务器。如果这不符合你的需要，考虑一个工具，如[Memcached](http://memcached.org/)。

如果这些都适用于你的用例，那么Guava缓存实用程序可能适合你！

如上面的示例代码所示，使用`CacheBuilder`构建器模式来获取`Cache`，但是自定义缓存才是有趣的部分。

_Note：_如果你不需要一个`Cache`的特性，`ConcurrentHashMap`更具内存效率 - 但 Cache 的大多数特性都很难基于旧有的 ConcurrentMap 来复制完成，甚至根本不可能做到。


# Population

在使用缓存前，首先问自己一个问题：有没有合理的默认方法来加载或计算与键关联的值？ 如果有的话，你应该使用一个`CacheLoader`。 如果没有，或者如果你需要重写默认值，但你仍然想要原子的“get-if-absent-compute”语义，你应该在调用 `get `时传入一个 `Callable `实例。使用`Cache.put` 元素可以直接插入，但自动缓存加载是首选，因为它使得更容易推理所有缓存内容的一致性。
### From a [CacheLoader](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html)

`LoadingCache `是附带 `CacheLoader `构建而成的缓存实现。创建自己的 `CacheLoader `通常只需要简单地实现` V load(K key) throws Exception` 方法。例如，你可以用下面的代码构建 `LoadingCache`：

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });

...
try {
  return graphs.get(key);
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

The canonical way to query a `LoadingCache` is with the method [get(K)](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#get(K)). This will either return an already cached value, or else use the cache's `CacheLoader` to atomically load a new value into the cache. Because `CacheLoader` might throw an `Exception`, `LoadingCache.get(K)` throws `ExecutionException`. If you have defined a `CacheLoader` that does _not_ declare any checked exceptions then you can perform cache lookups using `getUnchecked(K)`; however care must be taken not to call `getUnchecked` on caches whose `CacheLoader`s declare checked exceptions.

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .expireAfterAccess(10, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });

...
return graphs.getUnchecked(key);
```

Bulk lookups can be performed with the method `getAll(Iterable<? extends K>)`.  By default, `getAll` will issue a a separate call to `CacheLoader.load` for each key which is absent from the cache. When bulk retrieval is more efficient than many individual lookups, you can override <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#loadAll(java.lang.Iterable)'><code>CacheLoader.loadAll</code></a> to exploit this.  The performance of `getAll(Iterable)` will improve accordingly.

Note that you can write a `CacheLoader.loadAll` implementation that loads values for keys that were not specifically requested.  For example, if computing the value of any key from some group gives you the value for all keys in the group, `loadAll` might load the rest of the group at the same time.

### From a <a href='http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/Callable.html'><code>Callable</code></a>

All Guava caches, loading or not, support the method <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#get(java.lang.Object,java.util.concurrent.Callable)'><code>get(K, Callable&lt;V&gt;)</code></a>.  This method returns the value associated with the key in the cache, or computes it from the specified `Callable` and adds it to the cache.  No observable state associated with this cache is modified until loading completes. This method provides a simple substitute for the conventional "if cached, return; otherwise create, cache and return" pattern.

```java

Cache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .build(); // look Ma, no CacheLoader
...
try {
  // If the key wasn't in the "easy to compute" group, we need to
  // do things the hard way.
  cache.get(key, new Callable<Value>() {
    @Override
    public Value call() throws AnyException {
      return doThingsTheHardWay(key);
    }
  });
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

### Inserted Directly

Values may be inserted into the cache directly with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#put(K, V)'><code>cache.put(key, value)</code></a>.  This overwrites any previous entry in the cache for the specified key.  Changes can also be made to a cache using any of the `ConcurrentMap` methods exposed by the `Cache.asMap()` view. Note that no method on the `asMap` view will ever cause entries to be automatically loaded into the cache. Further, the atomic operations on that view operate outside the scope of automatic cache loading, so `Cache.get(K, Callable<V>)` should always be preferred over `Cache.asMap().putIfAbsent` in caches which load values using either `CacheLoader` or `Callable`.

# Eviction

The cold hard reality is that we almost _certainly_ don't have enough memory to cache everything we could cache.  You must decide: when is it not worth keeping a cache entry? Guava provides three basic types of eviction: size-based eviction, time-based eviction, and reference-based eviction.

## Size-based Eviction

If your cache should not grow beyond a certain size, just use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumSize(long)'><code>CacheBuilder.maximumSize(long)</code></a>.  The cache will try to evict entries that haven't been used recently or very often. _Warning_: the cache may evict entries before this limit is exceeded -- typically when the cache size is approaching the limit.

Alternately, if different cache entries have different "weights" -- for example, if your cache values have radically different memory footprints -- you may specify a weight function with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weigher(com.google.common.cache.Weigher)'><code>CacheBuilder.weigher(Weigher)</code></a> and a maximum cache weight with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumWeight(long)'><code>CacheBuilder.maximumWeight(long)</code></a>.  In addition to the same caveats as `maximumSize` requires, be aware that weights are computed at entry creation time, and are static thereafter.

```java

LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumWeight(100000)
       .weigher(new Weigher<Key, Graph>() {
          public int weigh(Key k, Graph g) {
            return g.vertices().size();
          }
        })
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });
```

## Timed Eviction

`CacheBuilder` provides two approaches to timed eviction:

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterAccess(long, java.util.concurrent.TimeUnit)'><code>expireAfterAccess(long, TimeUnit)</code></a> Only expire entries after the specified duration has passed since the entry was last accessed by a read or a write. Note that the order in which entries are evicted will be similar to that of [size-based eviction](#Size-based-Eviction).
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterWrite(long, java.util.concurrent.TimeUnit)'><code>expireAfterWrite(long, TimeUnit)</code></a> Expire entries after the specified duration has passed since the entry was created, or the most recent replacement of the value. This could be desirable if cached data grows stale after a certain amount of time.

Timed expiration is performed with periodic maintenance during writes and occasionally during reads, as discussed below.

### Testing Timed Eviction

Testing timed eviction doesn't have to be painful...and doesn't actually have to take you two seconds to test a two-second expiration.  Use the [Ticker](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Ticker.html) interface and the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#ticker(com.google.common.base.Ticker)'><code>CacheBuilder.ticker(Ticker)</code></a> method to specify a time source in your cache builder, rather than having to wait for the system clock.

## Reference-based Eviction

Guava allows you to set up your cache to allow the garbage collection of entries, by using [weak references](http://docs.oracle.com/javase/6/docs/api/java/lang/ref/WeakReference.html) for keys or values, and by using [soft references](http://docs.oracle.com/javase/6/docs/api/java/lang/ref/SoftReference.html) for values.

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakKeys()'><code>CacheBuilder.weakKeys()</code></a> stores keys using weak references.  This allows entries to be garbage-collected if there are no other (strong or soft) references to the keys.  Since garbage collection depends only on identity equality, this causes the whole cache to use identity (`==`) equality to compare keys, instead of `equals()`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakValues()'><code>CacheBuilder.weakValues()</code></a> stores values using weak references.  This allows entries to be garbage-collected if there are no other (strong or soft) references to the values.  Since garbage collection depends only on identity equality, this causes the whole cache to use identity (`==`) equality to compare values, instead of `equals()`.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#softValues()'><code>CacheBuilder.softValues()</code></a> wraps values in soft references.  Softly referenced objects are garbage-collected in a globally least-recently-used manner, _in response to memory demand_.  Because of the performance implications of using soft references, we generally recommend using the more predictable [maximum cache size](#Size-based-Eviction) instead.  Use of `softValues()` will cause values to be compared using identity (`==`) equality instead of `equals()`.

## Explicit Removals

At any time, you may explicitly invalidate cache entries rather than waiting for entries to be evicted.  This can be done:

* individually, using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidate(java.lang.Object)'><code>Cache.invalidate(key)</code></a>
* in bulk, using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll(java.lang.Iterable)'><code>Cache.invalidateAll(keys)</code></a>
    * to all entries, using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll()'><code>Cache.invalidateAll()</code></a>

## Removal Listeners

You may specify a removal listener for your cache to perform some operation when an entry is removed, via <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#removalListener(com.google.common.cache.RemovalListener)'><code>CacheBuilder.removalListener(RemovalListener)</code></a>.  The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListener.html'><code>RemovalListener</code></a> gets passed a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalNotification.html'><code>RemovalNotification</code></a>, which specifies the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalCause.html'><code>RemovalCause</code></a>, key, and value.

Note that any exceptions thrown by the `RemovalListener` are logged (using `Logger`) and swallowed.

```java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
  public DatabaseConnection load(Key key) throws Exception {
    return openConnection(key);
  }
};
RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
  public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
    DatabaseConnection conn = removal.getValue();
    conn.close(); // tear down properly
  }
};

return CacheBuilder.newBuilder()
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

**Warning**: removal listener operations are executed synchronously by default, and since cache maintenance is normally performed during normal cache operations, expensive removal listeners can slow down normal cache function! If you have an expensive removal listener, use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListeners.html#asynchronous(com.google.common.cache.RemovalListener, java.util.concurrent.Executor)'><code>RemovalListeners.asynchronous(RemovalListener, Executor)</code></a> to decorate a `RemovalListener` to operate asynchronously.

## When Does Cleanup Happen?

Caches built with `CacheBuilder` do _not_ perform cleanup and evict values "automatically," or instantly after a value expires, or anything of the sort.  Instead, it performs small amounts of maintenance during write operations, or during occasional read operations if writes are rare.

The reason for this is as follows: if we wanted to perform `Cache` maintenance continuously, we would need to create a thread, and its operations would be competing with user operations for shared locks.  Additionally, some environments restrict the creation of threads, which would make `CacheBuilder` unusable in that environment.

Instead, we put the choice in your hands.  If your cache is high-throughput, then you don't have to worry about performing cache maintenance to clean up expired entries and the like.  If your cache does writes only rarely and you don't want cleanup to block cache reads, you may wish to create your own maintenance thread that calls <a href='http://google.github.io/guava/releases/11.0.1/api/docs/com/google/common/cache/Cache.html#cleanUp()'><code>Cache.cleanUp()</code></a> at regular intervals.

If you want to schedule regular cache maintenance for a cache which only rarely has writes, just schedule the maintenance using <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/ScheduledExecutorService.html'><code>ScheduledExecutorService</code></a>.

## Refresh

Refreshing is not quite the same as eviction.  As specified in <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#refresh(K)'><code>LoadingCache.refresh(K)</code></a>, refreshing a key loads a new value for the key, possibly asynchronously.  The old value (if any) is still returned while the key is being refreshed, in contrast to eviction, which forces retrievals to wait until the value is loaded anew.

If an exception is thrown while refreshing, the old value is kept, and the exception is logged and swallowed.

A `CacheLoader` may specify smart behavior to use on a refresh by overriding <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#reload(K, V)'><code>CacheLoader.reload(K, V)</code></a>, which allows you to use the old value in computing the new value.

```java
// Some keys don't need refreshing, and we want refreshes to be done asynchronously.
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .refreshAfterWrite(1, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return getGraphFromDatabase(key);
             }

             public ListenableFuture<Graph> reload(final Key key, Graph prevGraph) {
               if (neverNeedsRefresh(key)) {
                 return Futures.immediateFuture(prevGraph);
               } else {
                 // asynchronous!
                 ListenableFutureTask<Graph> task = ListenableFutureTask.create(new Callable<Graph>() {
                   public Graph call() {
                     return getGraphFromDatabase(key);
                   }
                 });
                 executor.execute(task);
                 return task;
               }
             }
           });
```

Automatically timed refreshing can be added to a cache using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#refreshAfterWrite(long, java.util.concurrent.TimeUnit)'><code>CacheBuilder.refreshAfterWrite(long, TimeUnit)</code></a>.  In contrast to `expireAfterWrite`, `refreshAfterWrite` will make a key _eligible_ for refresh after the specified duration, but a refresh will only be actually initiated when the entry is queried.  (If `CacheLoader.reload` is implemented to be asynchronous, then the query will not be slowed down by the refresh.)  So, for example, you can specify both `refreshAfterWrite` and `expireAfterWrite` on the same cache, so that the expiration timer on an entry isn't blindly reset whenever an entry becomes eligible for a refresh, so if an entry isn't queried after it comes eligible for refreshing, it is allowed to expire.

# Features

## Statistics

By using <a href='http://google.github.io/guava/releases/12.0/api/docs/com/google/common/cache/CacheBuilder.html#recordStats()'><code>CacheBuilder.recordStats()</code></a>, you can turn on statistics collection for Guava caches. The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#stats()'><code>Cache.stats()</code></a> method returns a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html'><code>CacheStats</code></a> object, which provides statistics such as

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#hitRate()'><code>hitRate()</code></a>, which returns the ratio of hits to requests
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#averageLoadPenalty()'><code>averageLoadPenalty()</code></a>, the average time spent loading new values, in nanoseconds
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#evictionCount()'><code>evictionCount()</code></a>, the number of cache evictions

and many more statistics besides.  These statistics are critical in cache tuning, and we advise keeping an eye on these statistics in performance-critical applications.

## `asMap`
You can view any `Cache` as a `ConcurrentMap` using its `asMap` view, but how the `asMap` view interacts with the `Cache` requires some explanation.

* `cache.asMap()` contains all entries that are _currently loaded_ in the cache.  So, for example, `cache.asMap().keySet()` contains all the currently loaded keys.
* `asMap().get(key)` is essentially equivalent to `cache.getIfPresent(key)`, and never causes values to be loaded.  This is consistent with the `Map` contract.
    * Access time is reset by all cache read and write operations (including `Cache.asMap().get(Object)` and `Cache.asMap().put(K, V)`), but not by `containsKey(Object)`, nor by operations on the collection-views of `Cache.asMap()`.  So, for example, iterating through `cache.entrySet()` does not reset access time for the entries you retrieve.

# Interruption

Loading methods (like `get`) never throw `InterruptedException`. We could have designed these methods to support `InterruptedException`, but our support would have been incomplete, forcing its costs on all users but its benefits on only some. For details, read on.

`get` calls that request uncached values fall into two broad categories: those that load the value and those that await another thread's in-progress load. The two differ in our ability to support interruption. The easy case is waiting for another thread's in-progress load: Here we could enter an interruptible wait. The hard case is loading the value ourselves. Here we're at the mercy of the user-supplied `CacheLoader`. If it happens to support interruption, we can support interruption; if not, we can't.

So why not support interruption when the supplied `CacheLoader` does? In a sense, we do (but see below): If the `CacheLoader` throws `InterruptedException`, all `get` calls for the key will return promptly (just as with any other exception). Plus, `get` will restore the interrupt bit in the loading thread. The surprising part is that the `InterruptedException` is wrapped in an `ExecutionException`.

In principle, we could unwrap this exception for you. However, this forces all `LoadingCache` users to handle `InterruptedException`, even though the majority of `CacheLoader` implementations never throw it. Maybe that's still worthwhile when you consider that all _non-loading_ threads' waits could still be interrupted. But many caches are used only in a single thread. Their users must still catch the impossible `InterruptedException`. And even those users who share their caches across threads will be able to interrupt their `get` calls only _sometimes_, based on which thread happens to make a request first.

Our guiding principle in this decision is for the cache to behave as though all values are loaded in the calling thread. This principle makes it easy to introduce caching into code that previously recomputed its values on each call. And if the old code wasn't interruptible, then it's probably OK for the new code not to be, either.

I said that we support interruption "in a sense." There's another sense in which we don't, making `LoadingCache` a leaky abstraction. If the loading thread is interrupted, we treat this much like any other exception. That's fine in many cases, but it's not the right thing when multiple `get` calls are waiting for the value. Although the operation that happened to be computing the value was interrupted, the other operations that need the value might not have been. Yet all of these callers receive the `InterruptedException` (wrapped in an `ExecutionException`), even though the load didn't so much "fail" as "abort." The right behavior would be for one of the remaining threads to retry the load. We have [a bug filed for this](https://github.com/google/guava/issues/1122). However, a fix could be risky. Instead of fixing the problem, we may put additional effort into a proposed `AsyncLoadingCache`, which would return `Future` objects with correct interruption behavior.
