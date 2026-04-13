# SlimeCore Description

SlimeCore is a datapack that serves as a manager and loader of other datapacks. If you are familiar with [Lantern Load](https://github.com/LanternMC/load), SlimeCore serves a similar purpose, but takes it multiple steps further. 

SlimeCore allows datapacks to declare:
- Dependencies
- Version
- Tick (entrypoint) order, relative to other datapacks
- Abstract interfaces, to be implemented by other datapacks
- Pack & author metadata

Datapacks specify this information in a special *manifest* function, and SlimeCore processes all datapacks' manifests, validates relationships, then executes a compatible load/calling order (a *build*). 

A primary paradigm of SlimeCore is that it is designed to be **atomic**. This means that, if used properly, **no changes to datapack loading will ever be made unless they are verified to work.** This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow you to disable a datapack if another enabled datapack has it declared as a dependency. You will be required to disable both datapacks at once.*

