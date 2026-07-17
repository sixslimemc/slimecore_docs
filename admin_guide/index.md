# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [Installing a Frontend](#installing-a-frontend)
- [`slimecore:-/help`](#slimecore-help)
- [TLDR](#tldr)

## SlimeCore Briefly


SlimeCore is a datapack that is a loading system for **other datapacks**. It essentially facilitates datapacks "playing nice" with eachother, much more than Minecraft's default datapack loading system. Most prominently, it allows datapacks to require other datapacks (dependencies) be installed/enabled before loading. Don't fret though, SlimeCore should make installing/managing dependencies easy.

*See [Description](../description.md) for a more in-depth description.*

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