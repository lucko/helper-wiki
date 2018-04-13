The `Schedulers` class provides easy access to different types of scheduler.

The main entry points are `Schedulers.sync()` and `Schedulers.async()`, which execute tasks on the main server thread and using a asynchronous executor service respectively.

`Scheduler` provides a number of useful methods for executing tasks.

```java
Schedulers.sync().run(() -> {
    // Do something on the main thread
});

Schedulers.async().run(() -> {
    // Do something on an async thread
});

Schedulers.sync().runLater(() -> {
    // Do something on the main thread after waiting for 10 ticks
}, 10L);

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

The `Scheduler` interface extends `Executor`, meaning these instances can be used in other cases too - most notably with `CompletableFuture`s.

```java
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {/*something*/}, Schedulers.sync());
```

It also exposes a fluent builder API to construct tasks.
```java
Schedulers.builder()
        .async() // run async
        .after(25) // wait 25 ticks
        .every(1, TimeUnit.MINUTES) // then execute every minute
        .run(() -> {
            // Do something!
        });
```
