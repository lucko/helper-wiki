helper adds a functional event handling utility. It allows you to dynamically register event listeners on the fly, without having to break out of logic, or define listeners as their own method.

Instead of *implementing Listener*, creating a *new method* annotated with *@EventHandler*, and *registering* your listener with the plugin manager, with helper, you can subscribe to an event with one simple line of code. This allows you to define multiple listeners in the same class, and register then selectively.

```java
Events.subscribe(PlayerJoinEvent.class).handler(e -> e.setJoinMessage(""));
```

It also allows for more advanced handling. You can set listeners to automatically expire after a set duration, or after they've been called a number of times. When constructing the listener, you can use Java 8 Stream-esque `#filter` predicates to refine the handling, as opposed to lines of `if ... return` statements.

```java
Events.subscribe(PlayerJoinEvent.class)
        .expireAfter(2, TimeUnit.MINUTES) // expire after 2 mins
        .expireAfter(1) // or after the event has been called 1 time
        .filter(e -> !e.getPlayer().isOp())
        .handler(e -> e.getPlayer().sendMessage("Wew! You were first to join the server since it restarted!"));
```

The implementation provides a selection of default filters.
```java
Events.subscribe(PlayerMoveEvent.class, EventPriority.MONITOR)
        .filter(Events.DEFAULT_FILTERS.ignoreCancelled())
        .filter(Events.DEFAULT_FILTERS.ignoreSameBlock())
        .handler(e -> {
            // handle
        });
```

You can also merge events together into the same handler, without having to define the handler twice.
```java
Events.merge(PlayerEvent.class, PlayerQuitEvent.class, PlayerKickEvent.class)
        .filter(e -> !e.getPlayer().isOp())
        .handler(e -> {
            Bukkit.broadcastMessage("Player " + e.getPlayer() + " has left the server!");
        });
```

This also works when events don't share a common interface or super class.
```java
Events.merge(Player.class)
        .bindEvent(PlayerDeathEvent.class, PlayerDeathEvent::getEntity)
        .bindEvent(PlayerQuitEvent.class, PlayerEvent::getPlayer)
        .handler(e -> {
            // poof!
            e.getLocation().getWorld().createExplosion(e.getLocation(), 1.0f);
        });
```

Events handling can be done alongside the Cooldown system, allowing you to easily define time restrictions on certain events.
```java
Events.subscribe(PlayerInteractEvent.class)
        .filter(e -> e.getAction() == Action.RIGHT_CLICK_AIR)
        .filter(PlayerInteractEvent::hasItem)
        .filter(e -> e.getItem().getType() == Material.BLAZE_ROD)
        .withCooldown(
                CooldownCollection.create(t -> t.getPlayer().getName(), Cooldown.of(10, TimeUnit.SECONDS)),
                (cooldown, e) -> {
                    e.getPlayer().sendMessage("This gadget is on cooldown! (" + cooldown.remainingTime(TimeUnit.SECONDS) + " seconds left)");
                })
        .handler(e -> {
            // play some spooky gadget effect
            e.getPlayer().playSound(e.getPlayer().getLocation(), Sound.CAT_PURR, 1.0f, 1.0f);
            e.getPlayer().playEffect(EntityEffect.FIREWORK_EXPLODE);
        });
```