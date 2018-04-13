### Entry points

The `Schedulers` class provides easy access to different types of scheduler.

The main entry points are `Schedulers.sync()` and `Schedulers.async()`, which execute tasks on the main server thread and using a asynchronous executor service respectively.

`Scheduler` provides a number of useful methods for executing tasks.

```java
Schedulers.sync().run(() -> {
    // Do something on the main thread
});
```

```java
Schedulers.async().run(() -> {
    // Do something on an async thread
});
```

```java
Schedulers.sync().runLater(() -> {
    // Do something on the main thread after waiting for 10 ticks
}, 10L);
```

### Repeating tasks
```java
// Do something repeatedly on the main thread every 20 ticks, with an initial delay of 10 ticks
AtomicInteger counter = new AtomicInteger(10);
Schedulers.sync().runRepeating(task -> {
    int count = counter.getAndDecrement();
    Bukkit.broadcastMessage(count + "...");
    
    if (count <= 0) {
        Bukkit.broadcastMessage("GO!");
        task.stop();
    }
}, 10L, 20L);
```

All scheduling methods return either `Promise`s or `Tasks`, which allow for easy manipulation of the scheduled computation or task.


### Compatibility with CompletableFuture

The `Scheduler` interface extends `Executor`, meaning these instances can be used in other cases too - most notably with `CompletableFuture`s.

```java
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> { /* something */ }, Schedulers.sync());
```

### Fluent task builders

Schedulers also exposes a fluent builder API to construct tasks.
```java
Schedulers.builder()
        .async() // run async
        .after(25) // wait 25 ticks
        .every(1, TimeUnit.MINUTES) // then execute every minute
        .run(() -> {
            // Do something!
        });
```

### Server thread locks

Ever been in a situation where you're performing some work in an async task, but need to quickly jump back onto the server thread to obtain some information (which would be unsafe to access async)?

`ServerThreadLock` attempts to make this easy (maybe?!)

```java
// in the middle of some async task, we want to jump back onto the main thread for something

EntityType type;

try (ServerThreadLock lock = ServerThreadLock.obtain()) {
    // code inside of this block will be executed on the main thread!

    type = Bukkit.getEntity(entityUuid).getType();
}

// Do something with type (now async again)
boolean success = recordEntityType(entityUuid, type);

// and so on...
```