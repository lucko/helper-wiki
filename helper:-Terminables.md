Terminables are a way to easily cleanup active objects in plugins when a shutdown or reset is needed.

The system consists of a few key interfaces.

* [`Terminable`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/Terminable.java) - The main interface. An object that can be unregistered, stopped, or gracefully halted.
* [`TerminableConsumer`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/TerminableConsumer.java) - An object which binds with and registers Terminables.
* [`CompositeTerminable`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/composite/CompositeTerminable.java) - An object which itself contains/has a number of Terminables, but does not register them internally.
* [`CompositeTerminableConsumer`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/composite/CompositeTerminableConsumer.java) - A bit like a TerminableConsumer, just for CompositeTerminables
* [`TerminableRegistry`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/registry/TerminableRegistry.java) - An object which is a Terminable itself, but also a TerminableConsumer & CompositeTerminableConsumer, all in one!

Terminables are a really important part of helper, as so much of the utility is accessible from a static context. Terminables are a way to tame these floating, globally built handlers, and register them with the plugin instance.

`ExtendedJavaPlugin` implements TerminableConsumer & CompositeTerminableConsumer, which lets you register Terminables and CompositeTerminables to the plugin. These are all terminated automagically when the plugin disables.

To demonstrate, I'll first define a new CompositeTerminable. Think of this as a conventional Listener class in a regular plugin.
```java
public class DemoListener implements CompositeTerminable {

    @Override
    public void setup(@Nonnull TerminableConsumer consumer) {

        Events.subscribe(PlayerJoinEvent.class)
                .filter(e -> e.getPlayer().hasPermission("silentjoin"))
                .handler(e -> e.setJoinMessage(null))
                .bindWith(consumer);

        Events.subscribe(PlayerQuitEvent.class)
                .filter(e -> e.getPlayer().hasPermission("silentquit"))
                .handler(e -> e.setQuitMessage(null))
                .bindWith(consumer);

    }
}
```

Notice the `.bindWith(...)` calls? All Terminables have this method added via default in the interface. It lets you register that specific terminable with a consumer.

In order to setup our DemoListener, we need a CompositeTerminableConsumer. Luckily, ExtendedJavaPlugin implements this for us!

```java
public class DemoPlugin extends ExtendedJavaPlugin {

    @Override
    protected void enable() {

        // either of these is fine (but don't use both!)
        new DemoListener().bindWith(this);

        bindComposite(new DemoListener());

    }
}
```