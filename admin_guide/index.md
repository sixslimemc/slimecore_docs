# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [`slimecore:-/help`](#slimecore-help)
- [Quick Start](#quick-start)

## SlimeCore Briefly

SlimeCore is a datapack that is a loading system for **other datapacks**. It essentially facilitates datapacks "playing nice" with eachother, much more than Minecraft's default datapack loading system. Importantly, it allows datapacks to require other datapacks (dependencies) be installed/enabled before loading. Don't fret though, SlimeCore should make installing/managing dependencies easy.

*See [Description](../description.md) for a more in-depth description.*

## `slimecore:-/help`

You can run `/function slimecore:-/help` to recieve a clickable link to these docs as a chat message.

## Quick Start

If you just want to quickly start playing with your SlimeCore-loaded datapacks:
1. Remove all SlimeCore-loaded datapacks from your world (temporarily).
2. Download and add the [SlimeCore](https://github.com/sixslimemc/slimecore/releases) and [SCDev](https://github.com/sixslimemc/scdev/releases) datapacks to your world.
3. Add the tag `scdev.listener` to yourself in-game.
4. Run `/reload` in-game.
5. Re-add the SlimeCore-loaded datapacks to your world.
6. Run `/reload` in-game again.
7. Read/skim [Key Concepts](./key_concepts.md) for important information on managing datapacks.
8. Use [the SCDev docs](https://github.com/sixslimemc/scdev) and [Troubleshooting](./troubleshooting.md) for further reference.

Otherwise, just continue reading.

---

**Next:** [Key Concepts](./key_concepts.md)