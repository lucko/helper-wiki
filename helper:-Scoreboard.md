helper includes a thread safe scoreboard system, allowing you to easily setup & update custom teams and objectives. It is written directly at the packet level, meaning it can be safely used from asynchronous tasks.

For example....
```java
MetadataKey<ScoreboardObjective> SCOREBOARD_KEY = MetadataKey.create("scoreboard", ScoreboardObjective.class);

BiConsumer<Player, ScoreboardObjective> updater = (p, obj) -> {
    obj.setDisplayName("&e&lMy Server &7(" + Bukkit.getOnlinePlayers().size() + "&7)");
    obj.applyLines(
            "&7Hi and welcome",
            "&f" + p.getName(),
            "",
            "&eRank: &f" + getRankName(p),
            "&eSome data:" + getSomeData(p)
    );
};

Scoreboard sb = Services.load(Scoreboard.class);

Events.subscribe(PlayerJoinEvent.class)
        .handler(e -> {
            // register a new scoreboard for the player when they join
            ScoreboardObjective obj = sb.createPlayerObjective(e.getPlayer(), "null", DisplaySlot.SIDEBAR);
            Metadata.provideForPlayer(e.getPlayer()).put(SCOREBOARD_KEY, obj);

            updater.accept(e.getPlayer(), obj);
        });

Schedulers.async().runRepeating(() -> {
    for (Player player : Bukkit.getOnlinePlayers()) {
        MetadataMap metadata = Metadata.provideForPlayer(player);
        ScoreboardObjective obj = metadata.getOrNull(SCOREBOARD_KEY);
        if (obj != null) {
            updater.accept(player, obj);
        }
    }
}, 3L, 3L);
```