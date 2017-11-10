"Bindings" refers to a feature exposed in the Java Scripting API. Expressed simply, bindings represent the objects, mapped to keys (variable names) which are available during the scripts execution.

The bindings provided by helper are encapsulated by [`SystemScriptBindings`](https://github.com/lucko/helper/blob/master/helper-js/src/main/java/me/lucko/helper/js/bindings/SystemScriptBindings.java), and implemented by [`HelperScriptBindings`](https://github.com/lucko/helper/blob/master/helper-js/src/main/java/me/lucko/helper/js/HelperScriptBindings.java).

helper itself provides a number of bindings for convenience:

* `exports` - these are explained later
* `server` - the Bukkit server instance
* `plugin` - the helper-js plugin instance
* `services` - the Bukkit services manager
* `colorize` - a function which accepts a string, and passes it through `Color#colorize`
* `newMetadataKey` - a function which accepts a string, and returns a new `MetadataKey` for the object.

Plus a number of more general bindings for working with Java objects:

* newArrayList
* newLinkedList
* newHashSet
* newHashMap
* newCopyOnWriteArrayList
* newConcurrentHashSet
* newConcurrentHashMap
* listOf
* setOf
* immutableListOf
* immutableSetOf

Each script then appends its own bindings.

* `loader` - the script loader which loaded the script. Useful if you want to load dependant scripts independently
* `registry` - a TerminableRegistry instance used by the script
* `logger` - the scripts logger instance
* `cwd` - the "current working directory" - effectively the scripts location relative to the loader directory
* `rsd` - the "root scripts directory" - the path (relative to the server root) to the scripts directory