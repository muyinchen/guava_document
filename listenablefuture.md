Concurrency is a _hard_ problem, but it is significantly simplified by working with powerful and simple abstractions.  To simplify matters, Guava extends the `Future` interface of the JDK with <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html'><code>ListenableFuture</code></a>.

**We strongly advise that you always use `ListenableFuture` instead of `Future` in all of your code**, because:
* Most `Futures` methods require it.
* It's easier than changing to `ListenableFuture` later.
    * Providers of utility methods won't need to provide `Future` and `ListenableFuture` variants of their methods.

# Interface
A traditional `Future` represents the result of an asynchronous computation: a computation that may or may not have finished producing a result yet.  A `Future` can be a handle to an in-progress computation, a promise from a service to supply us with a result.

A `ListenableFuture` allows you to register callbacks to be executed once the computation is complete, or if the computation is already complete, immediately.  This simple addition makes it possible to efficiently support many operations that the basic `Future` interface cannot support.

The basic operation added by `ListenableFuture` is <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html#addListener(java.lang.Runnable, java.util.concurrent.Executor)'><code>addListener(Runnable, Executor)</code></a>, which specifies that when the computation represented by this `Future` is done, the specified `Runnable` will be run on the specified `Executor`.

# Adding Callbacks
Most users will prefer to use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback, java.util.concurrent.Executor)'><code>Futures.addCallback(ListenableFuture&lt;V&gt;, FutureCallback&lt;V&gt;, Executor)</code></a>, or the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#addCallback(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.FutureCallback)'>version</a> which defaults to using `MoreExecutors.sameThreadExecutor()`, for use when the callback is fast and lightweight.  A <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html'><code>FutureCallback&lt;V&gt;</code></a> implements two methods:
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onSuccess(V)'><code>onSuccess(V)</code></a>, the action to perform if the future succeeds, based on its result
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/FutureCallback.html#onFailure(java.lang.Throwable)'><code>onFailure(Throwable)</code></a>, the action to perform if the future fails, based on the failure

# Creation

Corresponding to the JDK <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/ExecutorService.html#submit(java.util.concurrent.Callable)'><code>ExecutorService.submit(Callable)</code></a> approach to initiating an asynchronous computation, Guava provides the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListeningExecutorService.html'><code>ListeningExecutorService</code></a> interface, which returns a `ListenableFuture` wherever `ExecutorService` would return a normal `Future`.  To convert an `ExecutorService` to a `ListeningExecutorService`, just use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/MoreExecutors.html#listeningDecorator(java.util.concurrent.ExecutorService)'><code>MoreExecutors.listeningDecorator(ExecutorService)</code></a>.

```java
ListeningExecutorService service = MoreExecutors.listeningDecorator(Executors.newFixedThreadPool(10));
ListenableFuture<Explosion> explosion = service.submit(new Callable<Explosion>() {
  public Explosion call() {
    return pushBigRedButton();
  }
});
Futures.addCallback(explosion, new FutureCallback<Explosion>() {
  // we want this handler to run immediately after we push the big red button!
  public void onSuccess(Explosion explosion) {
    walkAwayFrom(explosion);
  }
  public void onFailure(Throwable thrown) {
    battleArchNemesis(); // escaped the explosion!
  }
});
```

Alternatively, if you're converting from an API based on <a href='http://docs.oracle.com/javase/1.5.0/docs/api/java/util/concurrent/FutureTask.html'><code>FutureTask</code></a>, Guava offers <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.util.concurrent.Callable)'><code>ListenableFutureTask.create(Callable&lt;V&gt;)</code></a> and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFutureTask.html#create(java.lang.Runnable, V)'><code>ListenableFutureTask.create(Runnable, V)</code></a>.  Unlike the JDK, `ListenableFutureTask` is not meant to be extended directly.

If you prefer an abstraction in which you set the value of the future rather than implementing a method to compute the value, consider extending <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractFuture.html'><code>AbstractFuture&lt;V&gt;</code></a> or using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/SettableFuture.html'><code>SettableFuture</code></a> directly.

If you must convert a `Future` provided by another API to an `ListenableFuture`, you may have no choice but to use the heavyweight <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/JdkFutureAdapters.html'><code>JdkFutureAdapters.listenInPoolThread(Future)</code></a> to convert a `Future` to a `ListenableFuture`.  Whenever possible, it is preferred to modify the original code to return a `ListenableFuture`.

# Application
The most important reason to use `ListenableFuture` is that it becomes possible to have complex chains of asynchronous operations.

```java
ListenableFuture<RowKey> rowKeyFuture = indexService.lookUp(query);
AsyncFunction<RowKey, QueryResult> queryFunction =
  new AsyncFunction<RowKey, QueryResult>() {
    public ListenableFuture<QueryResult> apply(RowKey rowKey) {
      return dataService.read(rowKey);
    }
  };
ListenableFuture<QueryResult> queryFuture =
    Futures.transformAsync(rowKeyFuture, queryFunction, queryExecutor);
```

Many other operations can be supported efficiently with a `ListenableFuture` that cannot be supported with a `Future` alone.  Different operations may be executed by different executors, and a single `ListenableFuture` can have multiple actions waiting upon it.

When several operations should begin as soon as another operation starts -- "fan-out" -- `ListenableFuture` just works: it triggers all of the requested callbacks.  With slightly more work, we can "fan-in," or trigger a `ListenableFuture` to get computed as soon as several other futures have _all_ finished: see <a href='http://google.github.io/guava/releases/snapshot/api/docs/src-html/com/google/common/util/concurrent/Futures.html#line.1276'>the implementation of <code>Futures.allAsList</code></a> for an example.

| Method                                   | Description                              | See also                                 |
| :--------------------------------------- | :--------------------------------------- | :--------------------------------------- |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction, java.util.concurrent.Executor)'><code>transformAsync(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;, Executor)</code></a>`*` | Returns a new `ListenableFuture` whose result is the product of applying the given `AsyncFunction` to the result of the given `ListenableFuture`. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transformAsync(com.google.common.util.concurrent.ListenableFuture, com.google.common.util.concurrent.AsyncFunction)'><code>transformAsync(ListenableFuture&lt;A&gt;, AsyncFunction&lt;A, B&gt;)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function, java.util.concurrent.Executor)'><code>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;, Executor)</code></a> | Returns a new `ListenableFuture` whose result is the product of applying the given `Function` to the result of the given `ListenableFuture`. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#transform(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)'><code>transform(ListenableFuture&lt;A&gt;, Function&lt;A, B&gt;)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList(java.lang.Iterable)'><code>allAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</code></a> | Returns a `ListenableFuture` whose value is a list containing the values of each of the input futures, in order.  If any of the input futures fails or is cancelled, this future fails or is cancelled. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#allAsList(com.google.common.util.concurrent.ListenableFuture...)'><code>allAsList(ListenableFuture&lt;V&gt;...)</code></a> |
| <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList(java.lang.Iterable)'><code>successfulAsList(Iterable&lt;ListenableFuture&lt;V&gt;&gt;)</code></a> | Returns a `ListenableFuture` whose value is a list containing the values of each of the successful input futures, in order.  The values corresponding to failed or cancelled futures are replaced with `null`. | <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#successfulAsList(com.google.common.util.concurrent.ListenableFuture...)'><code>successfulAsList(ListenableFuture&lt;V&gt;...)</code></a> |

`*` An <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AsyncFunction.html'><code>AsyncFunction&lt;A, B&gt;</code></a> provides one method, `ListenableFuture<B> apply(A input)`.  It can be used to asynchronously transform a value.

```java
List<ListenableFuture<QueryResult>> queries;
// The queries go to all different data centers, but we want to wait until they're all done or failed.

ListenableFuture<List<QueryResult>> successfulQueries = Futures.successfulAsList(queries);

Futures.addCallback(successfulQueries, callbackOnSuccessfulQueries);
```

# CheckedFuture
Guava also provides a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/CheckedFuture.html'><code>CheckedFuture&lt;V, X extends Exception&gt;</code></a> interface.  A `CheckedFuture` is a `ListenableFuture` that includes versions of the get methods that can throw a checked exception. This makes it easier to create a future that executes logic which can throw an exception.  To convert a `ListenableFuture` to a `CheckedFuture`, use <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Futures.html#makeChecked(com.google.common.util.concurrent.ListenableFuture, com.google.common.base.Function)'><code>Futures.makeChecked(ListenableFuture&lt;V&gt;, Function&lt;Exception, X&gt;)</code></a>.
