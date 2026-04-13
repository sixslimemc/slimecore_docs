# SlimeCore Admin Guide

## Acquiring a Frontend
[SlimeCore](https://github.com/sixslimemc/slimecore) is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This is primarily so *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

Because of this paradigm, frontends practically define how you interact with SlimeCore and SlimeCore-loaded datapacks; it is primarily their responsibilty to document their own methods. With that being said, most frontends should share a base level of functionality and conform to the concepts covered in [Essential Concepts](#essential-concepts).

If you just want to get started quickly, [Scdev](https://github.com/sixslimemc/scdev) is a very minimal frontend/utility created by the author of SlimeCore and should be sufficient for most use cases. Usage of Scdev for common administration is covered in the next section.


## With Scdev


## Essential Concepts