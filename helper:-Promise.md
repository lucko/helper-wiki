A `Promise` is an object that acts as a proxy for a result that is initially unknown, usually because the computation of its value is yet incomplete.

The concept very closely resembles the Java [`CompletionStage`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletionStage.html) and [`CompletableFuture`](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html) APIs.

The main differences between CompletableFutures and Promises are:

* The ability to switch seamlessly between the main 'Server thread' and asynchronous tasks
* The ability to delay an action by a number of game ticks

Promises are really easy to use. To demonstrate how they work, consider this simple reward system. The task flow looks a bit like this.

1. Announce to players that rewards are going to be given out. (sync)
2. Get a list of usernames who are due a reward (async)
3. Convert these usernames to UUIDs (async)
4. Get a set of Player instances for the given UUIDs (sync)
5. Give out the rewards to the online players (sync)
6. Notify the reward storage that these players have been given a reward

Using the Promise API, this might look something like this...

```java
RewardStorage storage = getRewardStorage();

Promise<List<Player>> rewardPromise = Promise.start()
        .thenRunSync(() -> Bukkit.broadcastMessage("Getting ready to reward players in 10 seconds!"))
        // wait 10 seconds, then start
        .thenRunDelayedSync(() -> Bukkit.broadcastMessage("Starting now!"), 200L)
        // get the players which need to be rewarded
        .thenApplyAsync(n -> storage.getPlayersToReward())
        // convert to uuids
        .thenApplyAsync(storage::getUuidsFromUsernames)
        // get players from the uuids
        .thenApplySync(uuids -> {
            List<Player> players = new ArrayList<>();
            for (UUID uuid : uuids) {
                Player player = Bukkit.getPlayer(uuid);
                if (player != null) {
                    players.add(player);
                }
            }
            return players;
        });

// give out the rewards sync
rewardPromise.thenAcceptSync(players -> {
    for (Player player : players) {
        storage.giveReward(player);
    }
});

// notify
rewardPromise.thenAcceptSync(players -> {
    for (Player player : players) {
        storage.announceSuccess(player.getUniqueId());
    }
});
```

However, then consider how this might look if you were just using nested runnables...

```java
RewardStorage storage = getRewardStorage();

Bukkit.getScheduler().runTask(this, () -> {

    Bukkit.broadcastMessage("Getting ready to reward players in 10 seconds!");

    Bukkit.getScheduler().runTaskLater(this, () -> {
        Bukkit.broadcastMessage("Starting now!");

        Bukkit.getScheduler().runTaskAsynchronously(this, () -> {
            List<String> playersToReward = storage.getPlayersToReward();
            Set<UUID> uuids = storage.getUuidsFromUsernames(playersToReward);

            Bukkit.getScheduler().runTask(this, () -> {
                List<Player> players = new ArrayList<>();
                for (UUID uuid : uuids) {
                    Player player = Bukkit.getPlayer(uuid);
                    if (player != null) {
                        players.add(player);
                    }
                }

                for (Player player : players) {
                    storage.giveReward(player);
                }

                Bukkit.getScheduler().runTaskAsynchronously(this, () -> {
                    for (Player player : players) {
                        storage.announceSuccess(player.getUniqueId());
                    }
                });
            });
        });
    }, 200L);
});
```

I'll leave it for you to decide which is better. :smile: