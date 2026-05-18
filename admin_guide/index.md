# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [Installing a Frontend](#installing-a-frontend)
- [`slimecore:-/help`](#slimecore-help)
- [TLDR](#tldr)

## SlimeCore Briefly

*See [Description](../description.md) for a more in-depth description.*

SlimeCore is a datapack that provides a loading system for **other datapacks**. It attempts to garuntee that all SlimeCore-loaded datapacks are loaded correctly and to disallow any changes that would cause them not to. For instance, if datapack A requires datapack B, SlimeCore will not load datapack A until datapack B is installed and enabled, and will make sure that datapack B is loaded *before* datapack A.

Datapacks that are SlimeCore-loaded will not function without SlimeCore installed.

## Installing a Frontend

To interact with SlimeCore in-game, you must install a **frontend** datapack. A frontend provides the user-facing interface and features that SlimeCore itself intentionally does not implement on it's own.

Requiring an external frontend has the following benefits:

- **Separation of Concerns:** SlimeCore itself can focus on doing a single job well, loading datapacks.
- **Customizability:** You, the user/admin, can choose (or create) a SlimeCore interface that works best for you.

Importantly, frontends are not "special" in any way--they are installed and managed like any other datapack, and their usage should be documented independently of SlimeCore.

This guide is frontend-agnostic, but does include [this section](./using_scdev.md) that covers usage of [SCDev](https://github.com/sixslimemc/scdev), a basic and accessible chat-based frontend made by the author of SlimeCore.

> While SlimeCore is still in an early-adoption phase, there unfortunately may not be much of a selection of frontends. However, if you find the current selection to be insufficient, it is not too difficult to [create your own](../interface_guide/index.md).

## `slimecore:-/help`

You can run `/function slimecore:-/help` to recieve a clickable link to these docs as a chat message.

## TLDR

For those that just want to quickly get started, install the [SCDev](https://github.com/sixslimemc/scdev) datapack and skip to [Using SCDev](./using_scdev.md).

---

**Next:** [Key Concepts](./key_concepts.md)