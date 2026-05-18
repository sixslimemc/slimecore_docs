# Dev Guide

This guide covers everything a datapack developer needs to know about creating and working with SlimeCore-loaded datapacks. \
It assumes intermediate knowledge of commands and datapack structure/development.

## SlimeCore Briefly

*From [Description](../description.md):*

> SlimeCore is a datapack that serves as a loader of other datapacks. It serves a similar purpose to [Lantern Load](https://github.com/LanternMC/load), but goes multiple steps further.
>
> SlimeCore allows datapacks to specify:
> - Version
> - Dependencies
- Entrypoints (e.g. ticking functions) and their order relative to other datapacks'
- Abstract interfaces that must be implemented by other datapacks
- Pack and author metadata
- Download URL(s)

Datapacks specify this information via a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then if validation succeeds, executes a compatible load/calling order.