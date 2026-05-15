# SlimeCore Admin Guide

This guide should cover everything a world admin needs to know about SlimeCore and how to use it.

- [SlimeCore Briefly](#slimecore-briefly)
- [Installing a Frontend](#installing-a-frontend)
- [TLDR](#tldr)

## SlimeCore Briefly

*See [Description](../description.md) for a more in-depth description.*

SlimeCore is a datapack that *loads and manages other datapacks* in a way that is more robust than by default. It attempts to ensure that all of your world's enabled datapacks are garunteed to load correctly, and doesn't allow changes that would cause otherwise. For instance, if datapack A requires datapack B (i.e. B is a dependency of A), SlimeCore will not allow datapack A to load until datapack B is installed and enabled. Likewise, it will not allow datapack B to be disabled/uninstalled until datapack A is also disabled/uninstalled.


## Installing a Frontend

In order to effectively interact with SlimeCore in-game, a **frontend** datapack must be installed, as SlimeCore intentionally does not implement any user-facing features on it's own.

Requiring an external frontend implementation has the following benefits:

- **Separation of concerns:** SlimeCore itself can focus on doing a single job well, loading datapacks.
- **Customizability:** You, the user/admin, can choose (or create) a SlimeCore interface that works best for you.

Frontends define their own user-facing interface with SlimeCore, and thus their functionality/methods should be documented independently. Importantly, frontends are not "special" in any way, they are installed and managed like any other datapack.

This guide is meant to be frontend-agnostic, but does include [this section](./using_scdev.md) that covers usage of [SCDev](https://github.com/sixslimemc/scdev), a basic and accessible chat-based frontend written by the author of SlimeCore.

> While SlimeCore is still in an early-adoption phase, there unfortunately may not be much of a selection of frontends. However, if you find the current selection to be insufficient, it is not difficult to [create your own](../interface_guide/index.md)!

## TLDR

For those that just want to quickly get started, install [SCDev](https://github.com/sixslimemc/scdev) and skip to [Using SCDev](./using_scdev.md).

---

**Next: [Key Concepts](./key_concepts.md)**