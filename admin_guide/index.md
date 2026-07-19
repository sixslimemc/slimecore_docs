# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [Install a Frontend](#installing-a-frontend)
- [`slimecore:-/help`](#slimecore-help)
- [TLDR](#tldr)

## SlimeCore Briefly

SlimeCore is a datapack that is a loading system for **other datapacks**. It essentially facilitates datapacks "playing nice" with eachother, much more than Minecraft's default datapack loading system. Importantly, it allows datapacks to require other datapacks (dependencies) be installed/enabled before loading. Don't fret though, SlimeCore should make installing/managing dependencies easy.

*See [Description](../description.md) for a more in-depth description.*

## Install a Frontend

In all practical cases, you should install a **frontend** datapack (of your choice). SlimeCore does not include any UI on its own; a frontend is expected to provide such. Frontends are developed and documented independently of SlimeCore like any other datapack. This guide provides foundational information that is useful regardless of frontend.

### SCDev

[SCDev](https://github.com/sixslimemc/scdev) is a chat-based frontend with no dependencies made by the author of SlimeCore; it is designed to be accessible for all users.

> While SlimeCore is still in an early-adoption phase, there may not be a selection of frontends outside of SCDev. However, if you find the current selection to be insufficient, it is not too difficult to [create your own](../interface_guide/index.md).

### Frontend Not Loading

If your frontend doesn't seem to work/load, other datapacks may be silently causing rebuild errors and not allowing your frontend to load. Try temporarily removing all datapacks except your frontend (and it's dependencies, if any) from your world's `datapacks/` folder and run `/reload` in-game--this should load your frontend. You can then re-add the other datapacks back into your world's `datapacks/` folder and run `/reload` once again. 

## `slimecore:-/help`

You can run `/function slimecore:-/help` to recieve a clickable link to these docs as a chat message.

## TLDR

1. Read [Key Concepts](./key_concepts.md).
2. Temporarily remove all SlimeCore-loaded datapacks from your world.
3. Install the [SCDev](https://github.com/sixslimemc/scdev/releases) datapack.
4. Add the tag `scdev.listener` to yourself in-game.
5. Run `/reload` in-game.
6. Re-add the SlimeCore-loaded datapacks to your world.
7. Run `/reload` in-game.
8. Use [Troubleshooting](./troubleshooting.md) as a reference.

---

**Next:** [Key Concepts](./key_concepts.md)