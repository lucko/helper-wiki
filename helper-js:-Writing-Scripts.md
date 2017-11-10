Now you have a (rough) understanding of how helper-js is formed, let's have a look at a script.

Everything starts with the initialisation script. By default, this is a script called "init.js", in the root of the scripts directory. This script is called when the plugin starts. Everything branches from this main script.

To test the script is loading, we can create a simple "Hello world" example.

```javascript
logger.info("Hello world!")
```

All things going well, this should produce the following output in the console.

```
[INFO]: [helper-js] Enabling helper-js v1.1.0
[INFO]: [helper-js] Using script directory: plugins\helper-js\scripts
[INFO]: [helper-js] [LOADER] Loaded script: init.js
[INFO]: [helper-js] [init] Hello world!
```

We can load other scripts from init.js by making a call to the script loader, like so:

```javascript
loader.watch("event-test.js")
loader.watch("gui-test.js")
```

Which will neatly lead onto two other examples :)

#### Listening to events
We can use the helper events API to construct event listeners easily from javascript.

```javascript
Events.subscribe(BlockBreakEvent.class)
    .filter(e => e.player.hasPermission("notify.break"))
    .handler(e => {
        e.player.sendMessage(colorize("&7You broke: &e" + e.block.type))

        for (var p of server.getOnlinePlayers()) {
            p.sendMessage(colorize("&e" + e.player.name + " &7broke &e" + e.block.type + "&7."))
        }

    })
    .bindWith(registry);

Events.subscribe(BlockPlaceEvent.class)
    .filter(e => e.player.hasPermission("notify.place"))
    .handler(e => {
        e.player.sendMessage(colorize("&7You placed: &e" + e.block.type))

        for (var p of server.getOnlinePlayers()) {
            p.sendMessage(colorize("&e" + e.player.name + " &7placed &e" + e.block.type + "&7."))
        }

    })
    .bindWith(registry);
```

Since scripts are frequently reloaded, it's even more important to bind Terminables to the scripts registry, otherwise, duplicate listeners are left floating around when a script is terminated.

#### Creating a GUI

And now for a slightly more complicated GUI example. This implements exactly the same interface as the one found in the GUI example earlier.

```javascript
// the display book
var display = new MenuScheme().mask("000010000")

// the keyboard buttons
var buttons = new MenuScheme()
    .mask("000000000")
    .mask("000000001")
    .mask("111111111")
    .mask("111111111")
    .mask("011111110")
    .mask("000010000")

// we're limited to 9 keys per line, so add 'P' one line above.
var keys = "PQWERTYUIOASDFGHJKLZXCVBNM"

function makeTypewriterGui(player) {
    var gui = new Gui(player, 6, "&7Typewriter", {
        message: new StringBuilder(""),
        redraw: () => {
            var message = this.message;

             // perform initial setup.
            if (gui.isFirstDraw()) {
                // when the GUI closes, send the resultant message to the player
                gui.bindRunnable(() => {
                    gui.player.sendMessage(colorize("&7Your typed message was: &f" + message.toString()))
                })

                // place the buttons
                var populator = buttons.newPopulator(gui);

                for (let k of keys) {
                    populator.accept(ItemStackBuilder.of(Material.CLAY_BALL)
                        .name("&f&l" + k)
                        .lore("")
                        .lore("&7Click to type this character")
                        .build(() => {
                            message.append(k);
                            gui.redraw();
                        })
                    );
                }

                // space key
                populator.accept(ItemStackBuilder.of(Material.CLAY_BALL)
                    .name("&f&lSPACE")
                    .lore("")
                    .lore("&7Click to type this character")
                    .build(() => {
                        message.append(" ");
                        gui.redraw();
                    })
                );
            }

            // update the display every time the GUI is redrawn.
            display.newPopulator(gui).accept(ItemStackBuilder.of(Material.BOOK)
                .name("&f" + message + "&7_")
                .lore("")
                .lore("&f> &7Use the buttons below to type your message.")
                .lore("&f> &7Hit ESC when you're done!")
                .buildItem().build()
            );
        }
    });
    return gui;
}

Commands.create()
    .assertPlayer()
    .handler(c => {
        makeTypewriterGui(c.sender()).open();
    })
    .register(plugin, "typewriter");
```

Although these two examples only showcase the manipulation of a few systems, hopefully it demonstrates the usefulness of the system. :smile: