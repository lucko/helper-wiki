Buckets are an extension of the Set interface, which partition elements on insertion to split them up into sub sets.

The primary use case for buckets comes from the way Minecraft servers run. Frequently in plugins, you schedule repeating tasks on a collection of things.

For the sake of explaining this, let's assume we want to iterate through all players on the server every second (20 ticks) and apply an action to each of them. The action we want to apply is fairly intensive, but it cannot be ran async. The only option is to create a scheduler task to perform the action every 20 ticks.

However - these tasks can quickly build up - when for example, lots of plugins are scheduling repeating tasks with delay zero and interval 20. (think about how often you do this!) The server is much more likely to overrun it's allocated 50 milliseconds every 20th tick, as every 20th tick has abnormally high activity, compared with the other 19 in the cycle.

Buckets allow you to divide the collection of elements up - meaning in our example, all players can be acted upon every second, but the actions are spread out over 20 ticks.

```java
Bucket<Player> bucket = BucketFactory.newHashSetBucket(20, PartitioningStrategies.lowestSize());

Events.subscribe(PlayerJoinEvent.class).handler(e -> bucket.add(e.getPlayer()));
Events.subscribe(PlayerQuitEvent.class).handler(e -> bucket.remove(e.getPlayer()));

Scheduler.runTaskRepeatingSync(() -> {
    BucketPartition<Player> part = bucket.asCycle().next();
    for (Player player : part) {
        // perform some action on player
    }
}, 1L, 1L);
```

In this example, players are split into 20 partitions, and a partition is acted upon per tick. Instead of a (potential) lag spike every 20 ticks, the work is divided equally over the period. 😄 