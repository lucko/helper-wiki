helper provides a Messenger abstraction utility, which consists of a few key classes.

* [`Messenger`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/Messenger.java) - an object which manages messaging Channels
* [`Channel`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/Channel.java) - represents an individual messaging channel. Facilitates sending a message to the channel, or creating a ChannelAgent
* [`ChannelAgent`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/ChannelAgent.java) - an agent for interacting with channel messaging streams. Allows you to add/remove ChannelListeners to a channel
* [`ChannelListener`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/ChannelListener.java) - an object listening to messages sent on a given channel

The system is very easy to use, and cuts out a lot of the boilerplate code which usually goes along with using PubSub systems.

As an example, here is a super simple global player messaging system.

```java
public class GlobalMessengerPlugin extends ExtendedJavaPlugin {

    @Override
    public void onEnable() {
        // get the Messenger
        Messenger messenger = getService(Messenger.class);

        // Define the channel data model.
        class PlayerMessage {
            UUID uuid;
            String username;
            String message;

            public PlayerMessage(UUID uuid, String username, String message) {
                this.uuid = uuid;
                this.username = username;
                this.message = message;
            }
        }

        // Get the channel
        Channel<PlayerMessage> channel = messenger.getChannel("pms", PlayerMessage.class);

        // Listen for chat events, and send a message to our channel.
        Events.subscribe(AsyncPlayerChatEvent.class, EventPriority.HIGHEST)
                .filter(Events.DEFAULT_FILTERS.ignoreCancelled())
                .handler(e -> {
                    e.setCancelled(true);
                    channel.sendMessage(new PlayerMessage(e.getPlayer().getUniqueId(), e.getPlayer().getName(), e.getMessage()));
                });

        // Get an agent from the channel.
        ChannelAgent<PlayerMessage> channelAgent = channel.newAgent();
        channelAgent.register(this);

        // Listen for messages sent on the channel.
        channelAgent.addListener((agent, message) -> {
            Scheduler.runSync(() -> {
                Bukkit.broadcastMessage("Player " + message.username + " says " + message.message);
            });
        });
    }
}
```

You can either integrate messenger into your own existing messaging system (using [`AbstractMessenger`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/AbstractMessenger.java), or, use **helper-redis**, which implements Messenger using Jedis and the Redis PubSub system.