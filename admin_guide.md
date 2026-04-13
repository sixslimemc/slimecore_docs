# SlimeCore Admin Guide

## Acquiring a Frontend
[SlimeCore](https://github.com/sixslimemc/slimecore) is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This is primarily so *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

If you are new to using SlimeCore, [Scdev](https://github.com/sixslimemc/scdev) is a very minimal frontend/utility created by the author of SlimeCore and should be sufficient for most use cases. This guide will reference Scdev for demonstration, but any frontend should provide similar functionality with obvious steps--as long as you have one, you should be good to go.

## Loading Datapacks

To simply install and play with SlimeCore-loaded datapack(s), it is likely that all you need to do is ensure that the SlimeCore datapack is installed and enabled, and the datapack(s) should just work. If you have SlimeCore installed and your datapack(s) still do not load, it is most likely because they require the installation of other datapacks, **dependencies**.

#### Status Messages (Scdev)

With Scdev, players with the tag `scdev.listen` recieve messages containing useful information and errors upon every `/reload`. World admins should have this tag.

```mcfunction
tag @s add scdev.listen
```

### Dependencies



