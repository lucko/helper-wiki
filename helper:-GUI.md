helper provides a highly adaptable and flexible GUI abstraction class.

All you have to do is extend `Gui` and override the `#redraw` method.

To demonstrate how the class works, I wrote a simple typewriter menu.
```java
public class TypewriterGui extends Gui {

    // the display book
    private static final MenuScheme DISPLAY = new MenuScheme().mask("000010000");

    // the keyboard buttons
    private static final MenuScheme BUTTONS = new MenuScheme()
            .mask("000000000")
            .mask("000000001")
            .mask("111111111")
            .mask("111111111")
            .mask("011111110")
            .mask("000010000");

    // we're limited to 9 keys per line, so add 'P' one line above.
    private static final String KEYS = "PQWERTYUIOASDFGHJKLZXCVBNM";

    private StringBuilder message = new StringBuilder();

    public TypewriterGui(Player player) {
        super(player, 6, "&7Typewriter");
    }

    @Override
    public void redraw() {

        // perform initial setup.
        if (isFirstDraw()) {

            // when the GUI closes, send the resultant message to the player
            bindRunnable(() -> getPlayer().sendMessage("Your typed message was: " + message.toString()));

            // place the buttons
            MenuPopulator populator = BUTTONS.newPopulator(this);
            for (char keyChar : KEYS.toCharArray()) {
                populator.accept(ItemStackBuilder.of(Material.CLAY_BALL)
                        .name("&f&l" + keyChar)
                        .lore("")
                        .lore("&7Click to type this character")
                        .build(() -> {
                            message.append(keyChar);
                            redraw();
                        }));
            }

            // space key
            populator.accept(ItemStackBuilder.of(Material.CLAY_BALL)
                    .name("&f&lSPACE")
                    .lore("")
                    .lore("&7Click to type this character")
                    .build(() -> {
                        message.append(" ");
                        redraw();
                    }));
        }

        // update the display every time the GUI is redrawn.
        DISPLAY.newPopulator(this).accept(ItemStackBuilder.of(Material.BOOK)
                .name("&f" + message.toString() + "&7_")
                .lore("")
                .lore("&f> &7Use the buttons below to type your message.")
                .lore("&f> &7Hit ESC when you're done!")
                .buildItem().build());
    }
}
```

You can call the `#redraw` method from within click callbacks to easily change the menu structure as players interact with the menu.

ItemStackBuilder provides a number of methods for creating item stacks easily, and can be used anywhere. (not just in GUIs!)

The GUI class also provides a number of methods which allow you to
* Define "fallback" menus to be opened when the current menu is closed
* Setup ticker tasks to run whilst the menu remains open
* Add invalidation tasks to be called when the menu is closed
* Manipulate ClickTypes to only fire events when a certain type is used
* Create automatically paginated views in a "dictionary" style