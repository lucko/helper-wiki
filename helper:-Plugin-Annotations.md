With helper, you can automagically create the standard `plugin.yml` files at compile time using annotation processing.

Simply annotate your main class with `@Plugin` and fill in the name and version. The processor will take care of the rest!

```java
@Plugin(name = "MyPlugin", version = "1.0.0")
public class MyPlugin extends JavaPlugin {

}
```

The annotation also supports defining load order, setting a description and website, and defining (soft) dependencies. Registering commands and permissions is not necessary with helper, as `ExtendedJavaPlugin` provides a method for registering these at runtime.

```java
@Plugin(
        name = "MyPlugin",
        version = "1.0",
        description = "A cool plugin",
        load = PluginLoadOrder.STARTUP,
        authors = {"Luck", "Some other guy"},
        website = "www.example.com",
        depends = {@PluginDependency("Vault"), @PluginDependency(value = "ASpecialPlugin", soft = true)},
        loadBefore = {"SomePlugin", "SomeOtherPlugin"}
)
public class MyPlugin extends JavaPlugin {

}
```