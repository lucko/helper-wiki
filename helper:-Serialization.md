helper provides a few classes with are useful when trying to serialize plugin data. It makes use of Google's GSON to convert from Java Objects to JSON.

* [`Position`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/Position.java) - similar to Bukkit's location, but without pitch/yaw
* [`BlockPosition`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/BlockPosition.java) - the location of a block within a world
* [`ChunkPosition`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/ChunkPosition.java) - the location of a chunk within a world
* [`Region`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/Region.java) - the area bounded by two Positions
* [`BlockRegion`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/BlockRegion.java) - the area bounded by two BlockPositions
* [`ChunkRegion`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/ChunkRegion.java) - the area bounded by two ChunkPositions
* [`Direction`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/Direction.java) - the yaw and pitch
* [`Point`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/Point.java) - a position + a direction

And finally, [`Serializers`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/Serializers.java), containing serializers for ItemStacks and Inventories.

There is also an abstraction for conducting file I/O. [`FileStorageHandler`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/serialize/FileStorageHandler.java) is capable of handling the initial creation of storage files, as well as automatically creating backups and saving when the server stops.

It's as simple as creating a class to handle serialization/deserialization, and then calling a method when you want to load/save data.

```java
public class DemoStorageHandler extends FileStorageHandler<Map<String, String>> {
    private static final Splitter SPLITTER = Splitter.on('=');

    public DemoStorageHandler(JavaPlugin plugin) {
        super("demo", ",json", plugin.getDataFolder());
    }

    @Override
    protected Map<String, String> readFromFile(Path path) {
        Map<String, String> data = new HashMap<>();

        try (BufferedReader reader = Files.newBufferedReader(path, StandardCharsets.UTF_8)) {
            // read all the data from the file.
            reader.lines().forEach(line -> {
                Iterator<String> it = SPLITTER.split(line).iterator();
                data.put(it.next(), it.next());
            });
        } catch (IOException e) {
            e.printStackTrace();
        }

        return data;
    }

    @Override
    protected void saveToFile(Path path, Map<String, String> data) {
        try (BufferedWriter writer = Files.newBufferedWriter(path, StandardCharsets.UTF_8)) {
            for (Map.Entry<String, String> e : data.entrySet()) {
                writer.write(e.getKey() + "=" + e.getValue());
                writer.newLine();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

Then, to save/load, just create an instance of the handler, and use the provided methods.
```java
DemoStorageHandler handler = new DemoStorageHandler(this);
handler.save(ImmutableMap.of("some key", "some value"));

// or, to save a backup of the previous file too
handler.saveAndBackup(ImmutableMap.of("some key", "some value"));
```

helper also provides a handler which uses Gson to serialize the data.
```java
GsonStorageHandler<List<String>> gsonHandler = new GsonStorageHandler<>("data", ".json", getDataFolder(), new TypeToken<List<String>>(){});
gsonHandler.save(ImmutableList.of("some key", "some value"));
```