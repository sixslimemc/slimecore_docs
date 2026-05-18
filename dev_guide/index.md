# Dev Guide

This guide covers everything a datapack developer needs to know about creating and working with SlimeCore-loaded datapacks. \
It assumes intermediate knowledge of commands and datapack structure/development.

## SlimeCore Briefly

From [Description](../description.md):

> SlimeCore is a datapack that serves as a loader of other datapacks. It serves a similar purpose to [Lantern Load](https://github.com/LanternMC/load), but goes multiple steps further.
>
> SlimeCore allows datapacks to specify:
> - Version
> - Dependencies
> - Entrypoints (e.g. ticking functions) and their order relative to other datapacks'
> - Abstract interfaces that must be implemented by other datapacks
> - Pack and author metadata
> - Download URL(s)
>
> Datapacks specify this information via a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then if validation succeeds, executes a compatible load/calling order.

## Installing a Frontend

While not *strictly* required for development, it is recommended that you install a **frontend** datapack. A frontend allows you to interact with and recieve useful information from SlimeCore in-game. *See [this section](../admin_guide/index.md#installing-a-frontend) for more info on frontends.*

### SCDev

[SCDev](https://github.com/sixslimemc/scdev) is a basic chat-based frontend made by the author of SlimeCore that is designed to be useful for datapack developers.

If you do choose to use SCDev, make sure to add the tag `scdev.listener` to yourself to recieve the chat messages it sends.

---

[Quick Convert](./quick_convert.md) - If you just want to convert your datapack quickly and dirtily.

[Breakdown](./breakdown.md) - Otherwise.