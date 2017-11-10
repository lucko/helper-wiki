helper provides a very simple command abstraction, designed to reduce some of the boilerplate needed when writing simple commands.

Specifically:
* Checking if the sender is a player/console sender, and then automatically casting.
* Checking for permission status
* Checking for argument usage
* Checking if the sender is able to use the command.
* Easily parsing arguments (not just a String[] like the Bukkit interface)

For example, a simple /msg command condenses down into only a few lines.

```java
Commands.create()
        .assertPermission("message.send")
        .assertPlayer()
        .assertUsage("<player> <message>")
        .handler(c -> {
            Player other = c.arg(0).parseOrFail(Player.class);
            Player sender = c.sender();
            String message = c.args().subList(1, c.args().size()).stream().collect(Collectors.joining(" "));

            other.sendMessage("[" + sender.getName() + " --> you] " + message);
            sender.sendMessage("[you --> " + sender.getName() + "] " + message);
        })
        .register(this, "msg");
```

All invalid usage/permission/argument messages can be altered when the command is built.
```java
Commands.create()
        .assertConsole("&cUse the console to shutdown the server!")
        .assertUsage("[countdown]")
        .handler(c -> {
            ConsoleCommandSender sender = c.sender();
            int delay = c.arg(0).parse(Integer.class).orElse(5);

            sender.sendMessage("Performing graceful shutdown!");

            Scheduler.runTaskRepeatingSync(task -> {
                int countdown = delay - task.getTimesRan();

                if (countdown <= 0) {
                    Bukkit.shutdown();
                    return;
                }

                Players.forEach(p -> p.sendMessage("Server restarting in " + countdown + " seconds!"));
            }, 20L, 20L);
        })
        .register(this, "shutdown");
```