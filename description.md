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

SlimeCore intentionally does not implement any "frontend" features such as chat messages, dialogs, or user-functions. Instead, it provides a public API for other datapacks to use, such that "frontends" can be created/shared like any other datapack. For instance, a datapack could use `#slimecore:hook/meta_info/rebuild/end` to display a message to admins containing the download URL(s) for any missing dependencies (dependency download URLs are provided by dependent's manifest) after a rebuild. \
*[Scdev](https://github.com/sixslimemc/scdev) is a very minimal development utility/frontend created by the author of SlimeCore.*

## Concepts

As mentioned above, a datapack managed by SlimeCore must include a **manifest** function. Here is a demonstration manifest:

```mcfunction
#> _namespace_:_/sc/manifest
# @ MANIFEST

data remove storage slimecore:in manifest.pack

data modify storage slimecore:in manifest.pack.pack_id set value "_namespace_"
data modify storage slimecore:in manifest.pack.author_id set value "sixslime"
data modify storage slimecore:in manifest.pack.version set value {major:0, minor:1, patch:0}
data modify storage slimecore:in manifest.pack.url set value "https://github.com/sixslimemc/_namespace_/releases/download/v0.1.0/sixslime._namespace_.0.1.0.zip"

data modify storage slimecore:in manifest.pack.display.name set value "TODO: NAME"
data modify storage slimecore:in manifest.pack.display.summary set value "TODO: DESC"
data modify storage slimecore:in manifest.pack.display.author_name set value "SixSlime"

data modify storage slimecore:in manifest.pack.display.links.author set value "https://github.com/sixslimemc"
data modify storage slimecore:in manifest.pack.display.links.info set value "https://github.com/sixslimemc/_namespace_"
data modify storage slimecore:in manifest.pack.display.links.versions set value "https://github.com/sixslimemc/_namespace_/releases"

data modify storage slimecore:in manifest.pack.entrypoints set value []
data modify storage slimecore:in manifest.pack.entrypoints append value {id:"tick"}
# data modify storage slimecore:in manifest.pack.entrypoints append value {id:"ID", before:[{pack_ref:"PACK", id:"ID"}]}

data modify storage slimecore:in manifest.pack.preload_entrypoints set value []
# data modify storage slimecore:in manifest.pack.preload_entrypoints append value {id:"ID", before:[{pack_ref:"PACK", id:"ID"}]}

data modify storage slimecore:in manifest.pack.abstract_declarations set value []
# data modify storage slimecore:in manifest.pack.abstract_declarations append value "ID"

data modify storage slimecore:in manifest.pack.abstract_implementations set value []
# data modify storage slimecore:in manifest.pack.abstract_implementations append value {pack_ref:"PACK", id:"ID"}

data modify storage slimecore:in manifest.pack.dependencies set value []
# data modify storage slimecore:in manifest.pack.dependencies append value {pack_id:"DEPENDENCY", author_id:"sixslime", optional:false, version:{major:0, minor:1}, download:{url:"https://github.com/sixslimemc/DEPENDENCY/releases/download/v0.1.0/sixslime.DEPENDENCY.0.1.0.zip", version:{major:0, minor:1, patch:0}}}

data modify storage slimecore:in manifest.pack.is_library set value false

function slimecore:api/manifest
```

## Mission Statement
The goal of SlimeCore to support a community-driven decentralized datapack ecosystem. It should be accessible to all datapack users and developers without requiring the usage of third party programs--but also not significantly obstruct workflows that may include them. SlimeCore is designed to be simple and robust, such that minimal maintenance/changes are required through Minecraft updates.