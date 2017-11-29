The scheduler class provides easy static access to the Bukkit Scheduler.
```java
Scheduler.runAsync(() -> {
    // Do something     
});
```

All scheduling methods return either `Promise`s or `Tasks`, which allow for easy manipulation of the scheduled computation or task.
```java
Scheduler.supplyLaterAsync(() -> getValue(), 10L)
        .thenAcceptSync(value -> {
            // Do something with 'value' on the main thread.
        });
```

It also exposes asynchronous and synchronous `Executor` instances.
```java
CompletableFuture<Void> completableFuture = CompletableFuture.runAsync(() -> {/*something*/}, Scheduler.sync());
```

The Scheduler provides a `Task` class, allowing for fine control over the status of repeating events.
```java
Scheduler.runTaskRepeatingSync(task -> {
    if (task.getTimesRan() >= 10) {
        task.stop();
        return;
    }

    // some repeating task
}, 20L, 20L);
```

You can also use a fluent builder API to construct tasks.
```java
Scheduler.builder()
        .async() // run async
        .after(25) // wait 25 ticks
        .every(1, TimeUnit.MINUTES) // then execute every minute
        .run(() -> {
            // Do something!
        });
```
