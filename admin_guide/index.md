# SlimeCore Admin Guide

This guide should cover everything a world admin needs to know about SlimeCore and how to use it.

In-case you did not read the [Description](../description.md): \
SlimeCore is a datapack that *loads and manages other datapacks* in a way that is more robust than by default. In slightly obtuse terms, it attempts to ensure that all of your world's enabled datapacks stay "happy", and doesn't allow changes that would make them "unhappy". For instance, if datapack A requires datapack B (i.e. B is a dependency of A), SlimeCore will not allow datapack A to load until datapack B is installed and enabled. Likewise, it will not allow datapack B to be disabled/uninstalled until datapack A is also disabled/uninstalled.

## Installing a Frontend

First and foremost, 
SlimeCore is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This means that *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

Because of this paradigm, frontends define how you interact with SlimeCore and SlimeCore-loaded datapacks. it is primarily the responsibilty of your chosen frontend's author to document its usage. With that said, it is likely useful to know the concepts covered in [Key Concepts](#essential-concepts).

If you just want to get started quickly, you can install [Scdev](https://github.com/sixslimemc/scdev), a basic frontend created by the author of SlimeCore, and skip to the [Using Scdev](#using-scdev) section.
