helper provides an alternate system to the Bukkit Metadata API. The main benefits over Bukkit are the use of generic types and automatically expiring, weak or soft values.

The metadata API can be easily integrated with the Event system, thanks to some default filters.
```java
MetadataKey<Boolean> IN_ARENA_KEY = MetadataKey.createBooleanKey("in-arena");

Events.subscribe(PlayerQuitEvent.class)
        .filter(Events.DEFAULT_FILTERS.playerHasMetadata(IN_ARENA_KEY))
        .handler(e -> {
            // clear their inventory if they were in an arena
            e.getPlayer().getInventory().clear();
        });

Events.subscribe(ArenaEnterEvent.class)
        .handler(e -> Metadata.provideForPlayer(e.getPlayer()).put(IN_ARENA_KEY, true));

Events.subscribe(ArenaLeaveEvent.class)
        .handler(e -> Metadata.provideForPlayer(e.getPlayer()).remove(IN_ARENA_KEY));
```

MetadataKeys can also use generic types with guava's TypeToken.
```java
MetadataKey<Set<UUID>> FRIENDS_KEY = MetadataKey.create("friends-list", new TypeToken<Set<UUID>>(){});

Events.subscribe(PlayerQuitEvent.class)
        .handler(e -> {
            Player p = e.getPlayer();

            Set<UUID> friends = Metadata.provideForPlayer(p).getOrDefault(FRIENDS_KEY, Collections.emptySet());
            for (UUID friend : friends) {
                Player pl = Bukkit.getPlayer(friend);
                if (pl != null) {
                    pl.sendMessage("Your friend " + p.getName() + " has left!"); // :(
                }
            }
        });
```

Values can automatically expire, or be backed with Weak or Soft references.
```java
MetadataKey<Player> LAST_ATTACKER = MetadataKey.create("combat-tag", Player.class);

Events.subscribe(EntityDamageByEntityEvent.class)
        .filter(e -> e.getEntity() instanceof Player)
        .filter(e -> e.getDamager() instanceof Player)
        .handler(e -> {
            Player damaged = ((Player) e.getEntity());
            Player damager = ((Player) e.getDamager());

            Metadata.provideForPlayer(damaged).put(LAST_ATTACKER, ExpiringValue.of(damager, 1, TimeUnit.MINUTES));
        });

Events.subscribe(PlayerDeathEvent.class)
        .handler(e -> {
            Player p = e.getEntity();
            MetadataMap metadata = Metadata.provideForPlayer(p);

            Optional<Player> player = metadata.get(LAST_ATTACKER);
            player.ifPresent(pl -> {
                // give the last attacker all of the players experience levels
                pl.setTotalExperience(pl.getTotalExperience() + p.getTotalExperience());
                p.setTotalExperience(0);
                metadata.remove(LAST_ATTACKER);
            });
        });
```

Unlike Bukkit's system, metadata will be removed automatically when a player leaves the server, meaning you need-not worry about creating accidental memory leaks from left over metadata. The API also supports attaching metadata to blocks, worlds and other entities.