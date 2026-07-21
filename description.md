# Description

SlimeCore is a datapack loading framework that is more robust and controllable than the default paradigm--it is a datapack that loads other datapacks. It is similar in purpose to [Lantern Load](https://github.com/LanternMC/load), but is taken many steps further.

SlimeCore allows datapacks to specify:
- Version
- Dependencies
- Entrypoints (see below)
- Abstract interfaces (i.e. contracts defined by one datapack and fulfilled by another)
- Pack and author metadata
- Download URL(s)

Datapacks specify this information via a *manifest function*. Upon world reload, SlimeCore executes all manifest functions, validates and evaluates relationships between each datapack's manifest data, and then executes a load based on the evaluation.

Instead of using `#minecraft:load` and `#minecraft:tick`, datapacks define *their own* function tags for:
- **Load:** Called on reload. Dependencies' load tags are always called before their dependents'.
- **Entrypoint(s):** Called on reload after all load tags. Any number of entrypoints can be defined per-datapack and can be explicitly ordered against entrypoints from other datapacks. Primarily used to start `/schedule` loops.
- **Disable:** Called just before the datapack is about to be disabled.
- **Uninstall:** Called just before the datapack is uninstalled.
- **Preload Entrypoint(s):** Like entrypoints, but called *before* any load tags are called--useful for meta work.

Additionally, datapacks can also define a **safe mode** tag, which is automatically called in the rare cases that may cause the datapack to function or load incorrectly (such as if the user installs a datapack with an identical namespace).

SlimeCore is designed to be **atomic**. If used properly, no changes to datapack loading will ever be made unless they are verified to work. This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow a datapack to be disabled if another enabled datapack has it specified as a dependency; it will require that the dependent is disabled before the dependency.

SlimeCore is designed to be **deterministic**. If used properly, the same set of datapacks will always load in exactly the same way across reloads and worlds.

SlimeCore is designed to be **minimal and unobtrusive**. SlimeCore only implements datapack loading and has no performance overhead outside of the world-reload tick. It intentionally does not implement any "frontend" features (chat messages, dialogs, user-facing functions etc.); instead, it provides a public API such that other datapacks can implement these frontend features easily.

### Mission Statement

The primary goal of SlimeCore is to support a community-driven, decentralized datapack ecosystem that is accessible to all datapack users and developers. It should not require the use of third party programs, but also should not significantly obstruct workflows that may include them. SlimeCore is designed to be simple and robust, such that functionality stays consistent and minimal maintenance/changes are required through Minecraft updates.

## Functional Overview

### Datapack Manifests

SlimeCore-loaded datapacks define exactly one **manifest** function that they append to the function tag `#slimecore:manifest`. This replaces the standard paradigm of appending to `#minecraft:load`/`#minecraft:tick`.

The following demonstrates a full manifest function:

```mcfunction

# Identity:
# 'pack_id' must match the datapack's namespace (function tag `#<pack_id>:load` is called during loading).
# 'author_id' should uniquely identify you.
# Together 'author_id' and 'pack_id' uniquely identify a datapack.
data modify storage slimecore:in manifest.pack.pack_id set value "mypack"
data modify storage slimecore:in manifest.pack.author_id set value "myauthorid"

# Version:
data modify storage slimecore:in manifest.pack.version set value {major:1, minor:0, patch:0}

# Direct download URL:
data modify storage slimecore:in manifest.pack.url set value "https://example.com/myauthorid.mypack.1.0.0"

# Dependencies:
# Dependencies must include a direct download URL of any valid version of the dependency.
data modify storage slimecore:in manifest.pack.dependencies set value []
data modify storage slimecore:in manifest.pack.dependencies append value {pack_id:"foopack", author_id:"fooauthor", optional:false, version:{major:1, minor:2}, download:{url:"https://example.com/fooauthor.foopack.1.2.3", version:{major:1, minor:2, patch:3}}}
# Dependencies can be optional.
data modify storage slimecore:in manifest.pack.dependencies append value {pack_id:"barpack", author_id:"barauthor", optional:true, version:{major:4, minor:5}, download:{url:"https://example.com/barauthor.barpack.4.5.6", version:{major:4, minor:5, patch:6}}}

# Entrypoints:
# Entrypoints are called after all datapacks are loaded and can be used to start tick/schedule loops.
# Each entrypoint represents the function tag `#mypack:entrypoint/<id>`.
data modify storage slimecore:in manifest.pack.entrypoints append value {id:"main"}
# This entrypoint will always be called after `#foopack:entrypoint/main`:
data modify storage slimecore:in manifest.pack.entrypoints append value {id:"my_interaction", after:[{pack_ref:"foopack", id:"main"}]}

# Preload entrypoints:
# Preload entrypoints are called before *any* datapacks are loaded, including their own.
# They should really only be used for technical or meta use cases.
# Each preload entrypoint represents function tag `#mypack:preload_entrypoint/<id>`.
data modify storage slimecore:in manifest.pack.preload_entrypoints append value {id:"my_preload"}

# Abstract interface declarations:
# Each declared abstract interface must be implemented (included in `abstract_implementations`) by exactly 1 other datapack in the world.
# A build will be invalid if there are any unimplemented or overimplemented interfaces.
# Abstract interfaces are practically uncommon, but included here for demonstration.
data modify storage slimecore:in manifest.pack.abstract_declarations append value {id:"my_interface"}

# Abstract interface implementations:
data modify storage slimecore:in manifest.pack.abstract_implementations append value {pack_ref:"barpack", id:"other_interface"}

# Library flag:
# Setting this to true indicates that your pack is only meant to be used as a dependency and does not provide any meaningful functionality on it's own.
data modify storage slimecore:in manifest.pack.is_library set value false

# Display information:
# Not directly used by SlimeCore, but may be read/displayed by other datapacks.
data modify storage slimecore:in manifest.pack.display.name set value "My Demonstration Pack"
data modify storage slimecore:in manifest.pack.display.summary set value "A pack used for demonstration!"
data modify storage slimecore:in manifest.pack.display.author_name set value "My Username"
data modify storage slimecore:in manifest.pack.display.links.author set value "https://example.com/authorwebsite"
data modify storage slimecore:in manifest.pack.display.links.info set value "https://example.com/mypack/wiki"
data modify storage slimecore:in manifest.pack.display.links.versions set value "https://example.com/mypack/releases"

# Loader version:
# Specifies the compatible version range of SlimeCore that can load this pack.
data modify storage slimecore:in manifest.pack.loader_version set value {major:1, minor:0}

# All manifests end with calling `slimecore:api/manifest`.
function slimecore:api/manifest
```

Upon world reload, SlimeCore calls `#slimecore:manifest`, collecting all manifests. If any changes to the list of manifests is detected, SlimeCore (by default) initiates a **rebuild**. Then, regardless of if a rebuild was initiated or successful, SlimeCore initiates a **load**.

A **rebuild** processes all manifests, validates datapack relationships, finds a valid load and entrypoint ordering, and if successful, sets the world's **current build** to reflect them.

A **load** calls preload entrypoints, then load tags, then entrypoints, according to the order specified by the world's current build.

A rebuild can also be explicitly initiated via the `slimecore:rebuild` function. Inputs can be provided to this function to "stage" datapacks for disabling, enabling, or uninstallation. If the staged changes would result in an invalid build, no changes to the world/build are actually made. This function is the only proper way to enable, disable, and uninstall SlimeCore-loaded datapacks.

### Datapack Paths

In order to work, SlimeCore expects datapack paths (the name of files/folders in a world's `datapacks/` folder) to follow a specific format--the most qualified being `<author id>.<pack id>.<major ver>.<minor ver>.<patch ver>.zip`, but some other variations are allowed. If for whatever reason the path of a SlimeCore-loaded datapack doesn't/can't follow this format, SlimeCore provides a method to manually link a datapack to it's path via ad hoc in-game commands.

### Performance

SlimeCore may create a significant single-tick delay during rebuilding, as well as a much lesser single-tick delay during loading. These delays scale linearly with the amount of SlimeCore-loaded datapacks.

SlimeCore does not run any commands outside of rebuilding and loading and has negligible/zero performance impact outside of those processes.

## Get Started

- **[Admin Guide](./admin_guide/index.md)** - Manage SlimeCore-loaded datapacks in your world.
- **[Datapack Development Guide](./development_guide/index.md)** - Create SlimeCore-loaded datapacks.
- **[Direct Interface Guide](./interface_guide/index.md)** - Create datapacks that directly interface with Slimecore (e.g. frontends).