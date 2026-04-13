# SlimeCore Description

SlimeCore is a datapack that serves as a manager and loader of other datapacks. If you are familiar with [Lantern Load](https://github.com/LanternMC/load), SlimeCore serves a similar purpose, but takes it multiple steps further. 

SlimeCore allows datapacks to specify:
- Version
- Dependencies, required and optional
- Tick (entrypoint) order relative to other datapacks
- Abstract interfaces that must be implemented by other datapacks
- Pack and author metadata
- Download URL(s)

Datapacks specify this information via a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then executes a compatible load/calling order if everything is valid.

SlimeCore completely replaces the usage of `#minecraft:load` and `#minecraft:tick`. Datapacks instead define *their own* **load**, **disable**, and **uninstall** function tags, as well as any number of **entrypoints** to be called after all datapacks are loaded. SlimeCore then calls these tags when appropriate.

A key aspect of SlimeCore is that it is designed to be **atomic**. This means that, if used properly, **no changes to datapack loading will ever be made unless they are verified to work.** This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow you to disable a datapack if another enabled datapack has it specified as a dependency; it will require that the dependent is disabled before the dependency--which is automatically enforced if both are disabled on the same rebuild (reload).

SlimeCore intentionally does not implement any "frontend" features such as chat messages, dialogs, or user-functions. Instead, it provides a public API for other datapacks to use, such that "frontends" can be created/shared like any other datapack. For instance, a datapack could use `#slimecore:hook/meta_info/rebuild/end` to display a message to admins containing the download URL(s) for any missing dependencies (dependency download URLs are provided by dependent's manifest) after a rebuild. *[Scdev](https://github.com/sixslimemc/scdev) is a very minimal utility/frontend created by SlimeCore's author, SixSlime.*


## Conceptual Implementation

## Author Mission Statement
The goal of SlimeCore to support a robust, decentralized datapack ecosystem. It aims to 