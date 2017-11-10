MenuScheme allows you to easily apply layouts to GUIs without having to think about slot ids.
```java
@Override
public void redraw() {
    new MenuScheme(StandardSchemeMappings.STAINED_GLASS)
            .mask("111111111")
            .mask("110000011")
            .mask("100000001")
            .mask("100000001")
            .mask("110000011")
            .mask("111111111")
            .scheme(14, 14, 1, 0, 10, 0, 1, 14, 14)
            .scheme(14, 0, 0, 14)
            .scheme(10, 10)
            .scheme(10, 10)
            .scheme(14, 0, 0, 14)
            .scheme(14, 14, 1, 0, 10, 0, 1, 14, 14)
            .apply(this);
}
```

The above scheme translates into this menu.

![](https://i.imgur.com/sERK75D.png)

The mask values determine which slots in each row will be transformed. The scheme values relate to the data values of the glass panes.

The scheming system can also be used alongside a `MenuPopulator`, which uses the scheme to add items to the Gui programatically.

```java
@Override
public void redraw() {
    MenuScheme scheme = new MenuScheme().mask("000111000");
    MenuPopulator populator = scheme.newPopulator(this);

    populator.accept(ItemStackBuilder.of(Material.PAPER).name("Item 1").buildItem().build());
    populator.accept(ItemStackBuilder.of(Material.PAPER).name("Item 2").buildItem().build());
    populator.accept(ItemStackBuilder.of(Material.PAPER).name("Item 3").buildItem().build());
}
```