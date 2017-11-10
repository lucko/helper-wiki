The scheduler class provides easy static access to the Bukkit Scheduler. All future methods return `Promise`s, allowing for easy use of callbacks.

It also exposes asynchronous and synchronous `Executor` instances.

```java
Scheduler.runLaterSync(() -> {
    for (Player player : Bukkit.getOnlinePlayers()) {
        if (!player.isOp()) {
            player.sendMessage("Hi!");
        }
    }
}, 10L);
```

It also provides a `Task` class, allowing for fine control over the status of repeating events.
```java
Scheduler.runTaskRepeatingSync(task -> {
    if (task.getTimesRan() >= 10) {
        task.stop();
        return;
    }

    // some repeating task
}, 20L, 20L);
```
