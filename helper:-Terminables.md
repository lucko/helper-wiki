Terminables are a way to easily encapsulate and handle objects which need to be "terminated" at some point in the future.

The system consists of a few key interfaces.

* [`Terminable`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/Terminable.java) - An object that can be "terminated" (closed) later. Basically an extension of Java's AutoClosable.
* [`TerminableConsumer`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/TerminableConsumer.java) - An object which accepts Terminables, and does something with them. (usually adds them to a registry for terminating later)
* [`TerminableModule`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/module/TerminableModule.java) - A class which "sets up" (produces) a number of terminable instances, to be bound to a consumer. Think of this a bit like a listener class.
* [`CompositeTerminable`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/terminable/composite/CompositeTerminable.java) - A Terminable which is made up of several other Terminables. Usually used as a registry of terminables.


Terminables are a really important part of helper, as so much of the utility is accessible and constructed from a static context. Terminables are a way to tame/bind these floating, globally built handlers, and register them with a concrete instance. (usually the plugin)

### Obtaining a terminable.

Most helper classes that create some sort of listener, activity or state return classes that extend Terminable.

For example...

```java
Terminable handler = Events.subscribe(PlayerJoinEvent.class).handler(e -> e.getPlayer().sendMessage("Hi!"));
```

By returning a Terminable, the API allows us to terminate the listener subscription at a later time.

```java
// Setup a listener
Terminable handler = Events.subscribe(PlayerJoinEvent.class).handler(e -> e.getPlayer().sendMessage("Hi!"));

// then later when we want to close the handler...
handler.close();
```

Terminables can also be bound to a CompositeTerminable. The easiest way to do this is with the fluent `bindWith(...)` method.

```
CompositeTerminable composite = CompositeTerminable.create();

Events.subscribe(PlayerJoinEvent.class)
        .handler(e -> e.getPlayer().sendMessage("Hi!"))
        .bindWith(composite);

// the composite can then be closed later...
composite.close();
```

### Plugins are TerminableConsumers!

If you use helper's `ExtendedJavaPlugin`, you can bind terminables directly to the plugin instance, as it extends `TerminableConsumer`. The registered terminables are terminated when the plugin disables.

```java
public class TestPlugin extends ExtendedJavaPlugin {

    @Override
    protected void enable() {
        Events.subscribe(PlayerJoinEvent.class)
                .handler(e -> e.getPlayer().sendMessage("Hi!"))
                .bindWith(this);
    }
}
```

### Using modules
TerminableModules are an easy way to group Terminables together and initialise objects at the same time.

To demonstrate, I'll first define a new TerminableModule.

```java
public class DemoListener implements TerminableModule {

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

The module can then be initialised in the main plugin instance.

```java
public class TestPlugin extends ExtendedJavaPlugin {

    @Override
    protected void enable() {
        bindModule(new DemoListener());
    }
}
```