# Admin Guide

This guide covers everything a world admin needs to know about SlimeCore and how to use it. \
It assumes basic knowledge of commands and datapack management.

- [SlimeCore Briefly](#slimecore-briefly)
- [Less Read More Datapacks](#less-read-more-datapacks)

## SlimeCore Briefly

SlimeCore is a datapack that is a loading system for **other datapacks**. It essentially facilitates datapacks "playing nice" with each other, much more than Minecraft's default datapack loading system. Importantly, it allows datapacks to require other datapacks (dependencies) be installed/enabled before loading, but SlimeCore intentionally makes installing/managing dependencies easy.

*See [Description](../description.md) for a more in-depth description.*

## Less Read More Datapacks

If you just want to play with your SlimeCore-loaded datapacks *as soon as possible* and figure out the rest later:

1. Remove all SlimeCore-loaded datapacks from your world (temporarily).
2. Download and add the [SlimeCore](https://github.com/sixslimemc/slimecore/releases) and [SCDev](https://github.com/sixslimemc/scdev/releases) datapacks to your world.
3. Add the tag `scdev.listener` to yourself in-game.
4. Run `/reload` in-game.
5. Re-add the SlimeCore-loaded datapacks to your world.
6. Run `/reload` in-game again.
7. Read [Key Concepts](./key_concepts.md) for important information on managing datapacks.
8. Use [the SCDev docs](https://github.com/sixslimemc/scdev) and [Troubleshooting](./troubleshooting.md) for further reference.

Otherwise, continue to the next page.

---

**Next:** [Key Concepts](./key_concepts.md)