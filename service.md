# Overview
The Guava <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html'><code>Service</code></a> interface represents an object with an operational state, with methods to start and stop.  For example, webservers, RPC servers, and timers can implement the `Service` interface.  Managing the state of services like these, which require proper startup and shutdown management, can be nontrivial, especially if multiple threads or scheduling is involved.  Guava provides some skeletons to manage the state logic and synchronization details for you.

# Using a Service

The normal lifecycle of a `Service` is

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>Service.State.NEW</code></a> to
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STARTING'><code>Service.State.STARTING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#RUNNING'><code>Service.State.RUNNING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#STOPPING'><code>Service.State.STOPPING</code></a> to
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#TERMINATED'><code>Service.State.TERMINATED</code></a>

A stopped service may not be restarted.  If the service fails where starting, running, or stopping, it goes into state <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#FAILED'><code>Service.State.FAILED</code></a>.

A service can be started _asynchronously_ using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>startAsync()</code></a>, which returns `this` to enable method chaining.  It is only valid to call `startAsync()` if the service is <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>NEW</code></a>.  So you should structure your application to have a unique place where each service is started.

Stopping the service is analogous, using the _asynchronous_ <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#stopAsync()'><code>stopAsync()</code></a> method.  But unlike <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>startAsync()</code></a>, it is safe to call this method multiple times.  This is to make it possible to handle races that may occur when shutting down services.

Service also provides several methods to wait for service transitions to complete.
* _asynchronously_ using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#addListener()'><code>addListener()</code></a>.  `addListener()` allows you to add a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.Listener.html'><code>Service.Listener</code></a> that will be invoked on every state transition of the service.  N.B. if a service is not <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#NEW'><code>NEW</code></a> when the listener is added, then any state transitions that have already occurred will _not_ be replayed on the listener.
* _synchronously_ using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitRunning()'><code>awaitRunning()</code></a>.  This is uninterruptible, throws no checked exceptions, and returns once the service has finished starting.  If the service fails to start, this throws an `IllegalStateException`.  Similarly, <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#awaitTerminated()'><code>awaitTerminated()</code></a> waits for the service to reach a terminal state (<a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#TERMINATED'><code>TERMINATED</code></a> or <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.State.html#FAILED'><code>FAILED</code></a>).  Both methods also have overloads that allow timeouts to be specified.

The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html'><code>Service</code></a> interface is subtle and complicated.  _We do not recommend implementing it directly_.  Instead please use one of the abstract base classes in guava as the base for your implementation.  Each base class supports a specific threading model.

# Implementations
## AbstractIdleService
The <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html'><code>AbstractIdleService</code></a> skeleton implements a `Service` which does not need to perform any action while in the "running" state -- and therefore does not need a thread while running -- but has startup and shutdown actions to perform.  Implementing such a service is as easy as extending `AbstractIdleService` and implementing the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#startUp()'><code>startUp()</code></a> and <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractIdleService.html#shutDown()'><code>shutDown()</code></a> methods.

```java

protected void startUp() {
  servlets.add(new GcStatsServlet());
}
protected void shutDown() {}
```

Note that any queries to the `GcStatsServlet` already have a thread to run in.  We don't need this service to perform any operations on its own while the service is running.

## AbstractExecutionThreadService
An <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html'><code>AbstractExecutionThreadService</code></a> performs startup, running, and shutdown actions in a single thread.  You must override the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html#run()'><code>run()</code></a> method, and it must respond to stop requests.  For example, you might perform actions in a work loop:

```java
public void run() {
  while (isRunning()) {
    // perform a unit of work
  }
}
```

Alternately, you may override <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractExecutionThreadService.html#triggerShutdown()'><code>triggerShutdown()</code></a> in any way that causes `run()` to return.

Overriding `startUp()` and `shutDown()` is optional, but the service state will be managed for you.

```java
protected void startUp() {
  dispatcher.listenForConnections(port, queue);
}
protected void run() {
  Connection connection;
  while ((connection = queue.take() != POISON)) {
    process(connection);
  }
}
protected void triggerShutdown() {
  dispatcher.stopListeningForConnections(queue);
  queue.put(POISON);
}
```

Note that `start()` calls your `startUp()` method, creates a thread for you, and invokes `run()` in that thread.  `stop()` calls `triggerShutdown()` and waits for the thread to die.

## AbstractScheduledService
An <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html'><code>AbstractScheduledService</code></a> performs some periodic task while running.  Subclasses implement <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#runOneIteration()'><code>runOneIteration()</code></a> to specify one iteration of the task, as well as the familiar `startUp` and `shutDown()` methods.

To describe the execution schedule, you must implement the <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.html#scheduler()'><code>scheduler()</code></a> method.  Typically, you will use one of the provided schedules from <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html'><code>AbstractScheduledService.Scheduler</code></a>, either <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedRateSchedule(long, long, java.util.concurrent.TimeUnit)'><code>newFixedRateSchedule(initialDelay, delay, TimeUnit)</code></a> or <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.Scheduler.html#newFixedDelaySchedule(long, long, java.util.concurrent.TimeUnit)'><code>newFixedDelaySchedule(initialDelay, delay, TimeUnit)</code></a>, corresponding to the familiar methods in `ScheduledExecutorService`.  Custom schedules can be implemented using <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractScheduledService.CustomScheduler.html'><code>CustomScheduler</code></a>; see the Javadoc for details.

## AbstractService
When you need to do your own manual thread management, override <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html'><code>AbstractService</code></a> directly. Typically, you should be well served by one of the above implementations, but implementing `AbstractService` is recommended when, for example, you are modeling something that provides its own threading semantics as a `Service`, you have your own specific threading requirements.

To implement `AbstractService` you must implement 2 methods.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStart()'><code>doStart()</code></a>:  `doStart()` is called directly by the first call to `startAsync()`, your `doStart()` method should perform all initialization and then eventually call <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStarted()'><code>notifyStarted()</code></a> if start up succeeded or <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed(java.lang.Throwable)'><code>notifyFailed()</code></a> if start up failed.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#doStop()'><code>doStop()</code></a>:  `doStop()` is called directly by the first call to `stopAsync()`, your `doStop()` method should shut down your service and then eventually call <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyStopped()'><code>notifyStopped()</code></a> if shutdown succeeded or <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/AbstractService.html#notifyFailed(java.lang.Throwable)'><code>notifyFailed()</code></a> if shutdown failed.

Your `doStart` and `doStop`, methods should be _fast_.  If you need to do expensive initialization, such as reading files, opening network connections, or any operation that might block, you should consider moving that work to another thread.

# Using ServiceManager

In addition to the `Service` skeleton implementations, Guava provides a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html'><code>ServiceManager</code></a> class that makes certain operations involving multiple `Service` implementations easier.  Create a new `ServiceManager` with a collection of `Services`.  Then you can manage them:

* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startAsync()'><code>startAsync()</code></a> will start all the services under management. Much like <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/Service.html#startAsync()'><code>Service#startAsync()</code></a> you can only call this method once, if all services are `NEW`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#stopAsync()'><code>stopAsync()</code></a> will stop all the services under management.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#addListener(com.google.common.util.concurrent.ServiceManager.Listener, java.util.concurrent.Executor)'><code>addListener</code></a> will add a <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.Listener.html'><code>ServiceManager.Listener</code></a> that will be called on major state transitions.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitHealthy()'><code>awaitHealthy()</code></a> will wait for all services to reach the `RUNNING` state.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#awaitStopped()'><code>awaitStopped()</code></a> will wait for all services to reach a terminal state.

Or inspect them:
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#isHealthy()'><code>isHealthy()</code></a> returns `true` if all services are `RUNNING`.
* <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#servicesByState()'><code>servicesByState()</code></a> returns a _consistent_ snapshot of all the services indexed by their state.
    * <a href='http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ServiceManager.html#startupTimes()'><code>startupTimes()</code></a> returns a map from `Service` under management to how long it took for that service to start in milliseconds.  The returned map is guaranteed to be ordered by startup time.

While it is recommended that service lifecycles be managed via `ServiceManager`, state transitions initiated via other mechanisms **do not impact the correctness** of its methods. For example, if the services are started by some mechanism besides `startAsync()`, the listeners will be invoked when appropriate and `awaitHealthy()` will still work as expected.  The only requirement that `ServiceManager` enforces is that all `Services` must be `NEW` when `ServiceManager is constructed.
