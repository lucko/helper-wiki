helper provides a wrapper class for the BungeeCord Plugin Messaging API, providing callbacks to read response data.

It handles the messaging channels behind the scenes and simply runs the provided callback when the data is returned.

For example...
```java
// sends the player to server "hub"
Player player;
BungeeMessaging.connect(player "hub");
```

And for calls which return responses, the data is captured automatically and returned via the callback.
```java
// requests the global player count and then broadcasts it to all players
BungeeMessaging.playerCount(BungeeMessaging.ALL_SERVERS, count -> Bukkit.broadcastMessage("There are " + count + " players online!"));
```

The class also provides a way to use the "Forward" channel.
```java
// prepare some data to send
ByteArrayDataOutput buf = ByteStreams.newDataOutput();
buf.writeUTF(getServerName());
buf.writeUTF("Hey!");

// send the data
BungeeMessaging.forward(BungeeMessaging.ONLINE_SERVERS, "my-special-channel", buf);

// listen for any messages sent on the special channel
BungeeMessaging.registerForwardCallback("my-special-channel", buf -> {
    String server = buf.readUTF();
    String message = buf.readUTF();

    Log.info("Server " + server + " says " + message);
    return false;
});
```