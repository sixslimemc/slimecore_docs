# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [Installing a Frontend](#installing-a-frontend)
- [`slimecore:-/help`](#slimecore-help)
- [Quick Start](#tldr)

## SlimeCore Briefly


SlimeCore is a datapack that is a loading system for **other datapacks**. It essentially facilitates datapacks "playing nice" with eachother, much more than Minecraft's default datapack loading system. Importantly, it allows datapacks to require other datapacks (dependencies) be installed/enabled before loading. Don't fret though, SlimeCore should make installing/managing dependencies easy.

*See [Description](../description.md) for a more in-depth description.*

## Install a Frontend

In all practical cases, you should install a **frontend** datapack. SlimeCore does not include any UI on its own; a frontend provides such UI. This guide is intended to be frontend-agnostic.

Frontends are developed and documented independently of SlimeCore like any other datapack; your chosen frontend should provide its own documentation on its specific usage.

> While SlimeCore is still in an early-adoption phase, there unfortunately may not be much of a selection of frontends. However, if you find the current selection to be insufficient, it is not too difficult to [create your own](../interface_guide/index.md).

### Frontend Not Loading

If your frontend doesn't seem to work/load, other datapacks may be silently causing rebuild errors and not allowing your frontend to load. Try temporarily removing all datapacks except your frontend (and it's dependencies, if any) from your world's `datapacks/` folder and run `/reload` in-game--this should load your frontend. You can then re-add the other datapacks back into your world's `datapacks/` folder and run `/reload` once again. 

### SCDev

[SCDev](https://github.com/sixslimemc/scdev) is a chat-based frontend with no dependencies made by the author of SlimeCore; it is designed to be accessible for all users.

The section [Using SCDev](./using_scdev.md) covers interaction with SlimeCore specifically with SCDev.

## `slimecore:-/help`

You can run `/function slimecore:-/help` to recieve a clickable link to these docs as a chat message.

## Quick Start

Install the [SCDev](https://github.com/sixslimemc/scdev) datapack and skip to [Using SCDev](./using_scdev.md).

---

**Next:** [Key Concepts](./key_concepts.md)