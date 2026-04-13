# SlimeCore Description

SlimeCore is a datapack that serves as a manager and loader of other datapacks. If you are familiar with [Lantern Load](https://github.com/LanternMC/load), SlimeCore serves a similar purpose, but takes it multiple steps further. 

SlimeCore allows datapacks to have and specify:
- Version
- Dependencies, required and optional
- Tick (entrypoint) order relative to other datapacks
- Abstract interfaces that must be implemented by other datapacks
- Pack and author metadata
- Download URLs

Datapacks specify this information in a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then executes a compatible load/calling order if everything is valid. 

A key aspect of SlimeCore is that it is designed to be **atomic**. This means that, if used properly, **no changes to datapack loading will ever be made unless they are verified to work.** This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow you to disable a datapack if another enabled datapack has it specified as a dependency; it will require you to disable both datapacks at once.

In addition to loading, SlimeCore also generates in-game storage NBT data containing useful information about the world's currently installed datapacks, including all information specified in datapack manifests--

## Conceptual Implementation

