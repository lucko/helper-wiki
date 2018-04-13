helper provides a Messenger abstraction utility, which consists of a few key classes.

* [`Messenger`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/Messenger.java) - an object which manages messaging Channels
* [`Channel`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/Channel.java) - represents an individual messaging channel. Facilitates sending a message to the channel, or creating a ChannelAgent
* [`ChannelAgent`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/ChannelAgent.java) - an agent for interacting with channel messaging streams. Allows you to add/remove ChannelListeners to a channel
* [`ChannelListener`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/ChannelListener.java) - an object listening to messages sent on a given channel

The system is very easy to use, and cuts out a lot of the boilerplate code which usually goes along with using PubSub systems.

As an example, here is a super simple global player messaging system.

```java
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
    Schedulers.sync().run(() -> {
        Bukkit.broadcastMessage("Player " + message.username + " says " + message.message);
    });
});
```

You can either integrate messenger into your own existing messaging system (using [`AbstractMessenger`](https://github.com/lucko/helper/blob/master/helper/src/main/java/me/lucko/helper/messaging/AbstractMessenger.java), or, use **helper-redis**, which implements Messenger using Jedis and the Redis PubSub system.


### Conversations

helper also provides an additional abstraction for working with "conversation channels" - where sent messages require some sort of reply.

```java
public class PrivateMessageSystem {
    private final ConversationChannel<PrivateMessage, PrivateMessageReply> channel;

    public PrivateMessageSystem(Messenger messenger) {
        this.channel = messenger.getConversationChannel("private-messages", PrivateMessage.class, PrivateMessageReply.class);

        ConversationChannelAgent<PrivateMessage, PrivateMessageReply> channelAgent = this.channel.newAgent();
        channelAgent.addListener((agent, message) -> {
            Promise<PrivateMessageReply> reply = Schedulers.sync().supply(() -> {
                Player player = Bukkit.getPlayerExact(message.to);
                if (player == null) {
                    return null;
                }

                player.sendMessage("You got a message from " + message.from + " saying " + message.message);
                return new PrivateMessageReply(message.getConversationId(), player.getName());
            });
            
            return ConversationReply.ofPromise(reply);
        });
    }

    public void sendMessage(Player from, String to, String message) {

        // create a new message object
        PrivateMessage pm = new PrivateMessage(from.getUniqueId(), message, to);

        this.channel.sendMessage(pm, new ConversationReplyListener<PrivateMessageReply>() {
            @Nonnull
            @Override
            public RegistrationAction onReply(@Nonnull PrivateMessageReply reply) {
                from.sendMessage("Your message was delivered successfully to " + reply.deliveredTo);
                return RegistrationAction.STOP_LISTENING;
            }

            @Override
            public void onTimeout(@Nonnull List<PrivateMessageReply> replies) {
                from.sendMessage("Unable to deliver your message to " + to);
            }
        }, 2, TimeUnit.SECONDS);
    }

    private static final class PrivateMessage implements ConversationMessage {
        private final UUID conversationId;
        private final UUID from;
        private final String message;
        private final String to;

        private PrivateMessage(UUID from, String message, String to) {
            this.conversationId = UUID.randomUUID();
            this.from = from;
            this.message = message;
            this.to = to;
        }

        @Nonnull
        @Override
        public UUID getConversationId() {
            return this.conversationId;
        }
    }

    private static final class PrivateMessageReply implements ConversationMessage {
        private final UUID conversationId;
        private final String deliveredTo;

        private PrivateMessageReply(UUID conversationId, String deliveredTo) {
            this.conversationId = conversationId;
            this.deliveredTo = deliveredTo;
        }

        @Nonnull
        @Override
        public UUID getConversationId() {
            return this.conversationId;
        }
    }

}
```