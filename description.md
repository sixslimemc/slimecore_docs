# SlimeCore Description

SlimeCore is a datapack that serves as a manager and loader of other datapacks. If you are familiar with [Lantern Load](https://github.com/LanternMC/load), SlimeCore serves a similar purpose, but takes it multiple steps further. 

SlimeCore allows datapacks to have and specify:
- Version
- Dependencies, required and optional
- Tick (entrypoint) order relative to other datapacks
- Abstract interfaces that must be implemented by other datapacks
- Pack and author metadata
- Download URL(s)

Datapacks specify this information in a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then executes a compatible load/calling order if everything is valid.

SlimeCore completely replaces the paradigm of using `#minecraft:load` and `#minecraft:tick`. Datapacks instead define *their own* **load**, **disable**, and **uninstall** function tags, as well as any number of **entrypoints** that are called after all datapacks are loaded. SlimeCore calls these tags when appropriate.

A key aspect of SlimeCore is that it is designed to be **atomic**. This means that, if used properly, **no changes to datapack loading will ever be made unless they are verified to work.** This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow you to disable a datapack if another enabled datapack has it specified as a dependency; it will require you to disable both datapacks at once.

SlimeCore also includes a multitude of useful function *hooks* related to loading, allowing other datapacks to subscribe/listen to events and execute behavior when they happen. For example, a datapack can subscribe to `#slimecore:hook/meta_info/rebuild/end` and display the download URL(s) for any missing dependencies (dependency download URLs are provided by dependent's manifest).

SlimeCore intentionally does not implement any "front-end" features such as chat messages, dialogs, or user-functions. It instead provides a public API (such as the hooks mentioned above) for other datapacks, such that SlimeCore "front-ends" are decentralized and customizable.

## Conceptual Implementation

## Author Mission Statement
The goal of SlimeCore to support a robust, decentralized datapack ecosystem. It aims to 