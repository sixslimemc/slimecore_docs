# Description

- [Mission Statement](#mission-statement)
- [Summary](#summary)
- [Functional Overview](#functional-overview)
- [Get Started](#get-started)

## Mission Statement

SlimeCore is a datapack loading framework that uses a package-manifest paradigm as a replacement for `#minecraft:load` and `#minecraft:tick`--it is a datapack that loads other datapacks. Its purpose is similar to [Lantern Load](https://github.com/LanternMC/load), but provides implementation for dependency resolution, versioning, explicit entrypoint ordering, metadata, and more. SlimeCore is designed as a greater foundation for a decentralized datapack ecosystem that is accessible regardless of toolchain.

## Summary

SlimeCore allows datapacks to specify:
- Version
- Dependencies
- Entrypoints (see below)
- Contracts (i.e. specification defined by one datapack to be satisfied/implemented by another)
- Pack and author metadata
- Download URL(s)

Datapacks specify this information via a *manifest function*. Upon world reload, SlimeCore executes all datapacks' manifest functions, validates and evaluates relationships between each datapack's manifest data, and then executes a load based on the evaluation.

Instead of using `#minecraft:load` and `#minecraft:tick`, datapacks define *their own* function tags for:
- **Load:** Called on reload. Dependencies' load tags are always called before their dependents'.
- **Entrypoint(s):** Called on reload after all load tags. Any number of entrypoints can be defined per-datapack and can be explicitly ordered against entrypoints from other datapacks. Primarily used to start `/schedule` loops.
- **Disable:** Called just before the datapack is disabled.
- **Uninstall:** Called just before the datapack is uninstalled.
- **Preload Entrypoint(s):** Like entrypoints, but called *before* any load tags are called--useful for advanced work.
- **Safe Mode:** Called in the exceptional case where SlimeCore cannot ensure the datapack loaded corectly.

SlimeCore is designed for **determinism**:
- The same set of given datapacks will *always* load in the same order across reloads and worlds.
- Entrypoints (tick loops) are decoupled from loading order, allowing for explicit definition and ordering.
- Management operations are guaranteed to happen in a defined and reasonable order. *(e.g. datapacks are always disabled/uninstalled in reverse loading order.)*

SlimeCore is designed for **atomicity**:
- Instead of managing datapacks one-at-a-time via `/datapack` (potentially creating invalid world state), management operations are *staged* via SlimeCore, and then performed all-at-once or not-at-all upon world reload, depending on if they are valid.
- With proper use, SlimeCore does not allow any datapacks within a world to be in an invalid loading state.

SlimeCore is designed to be **unobtrusive**:
- No third party tools are necessary in order to use or develop with SlimeCore.
- SlimeCore does not enforce or implement anything more than what is deemed necessary for its goal.
- SlimeCore has no performance overhead outside of the world reload tick.

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

# Contract declarations:
# Each declared contract must be satisfied by exactly 1 other datapack in the world (included in other datapack's `contracts_satisfied`).
# A build will be invalid if there are any unsatisfied or oversatisfied interfaces.
# Contracts are relatively specific in use-case and are practically uncommon.
data modify storage slimecore:in manifest.pack.contract_declarations append value {id:"my_interface"}

# Contracts satisfied:
data modify storage slimecore:in manifest.pack.contracts_satisfied append value {pack_ref:"barpack", id:"other_interface"}

# Library flag:
# Setting this to true indicates that your pack is only meant to be used as a dependency and does not provide any meaningful functionality on its own.
# This flag is intended for external utility (not directly used by SlimeCore).
data modify storage slimecore:in manifest.pack.is_library set value false

# Display information:
# Not directly used by SlimeCore, but may be read/displayed by other datapacks.
data modify storage slimecore:in manifest.pack.display.name set value "My Demonstration Pack"
data modify storage slimecore:in manifest.pack.display.summary set value "A pack used for demonstration."
data modify storage slimecore:in manifest.pack.display.author_name set value "My_Username"
data modify storage slimecore:in manifest.pack.display.links.author set value "https://example.com/authorwebsite"
data modify storage slimecore:in manifest.pack.display.links.info set value "https://example.com/mypack/wiki"
data modify storage slimecore:in manifest.pack.display.links.versions set value "https://example.com/mypack/releases"

# Loader version:
# Specifies the compatible version range of SlimeCore that can load this pack.
data modify storage slimecore:in manifest.pack.loader_version set value {major:0, minor:3}

# All manifests end with calling `slimecore:api/manifest`.
function slimecore:api/manifest
```

Upon world reload, SlimeCore executes the following in-order:
1. Collect all manifests via `#slimecore:manifest` from both enabled and disabled datapacks.
2. If any changes to the list of manifests have been made, initiate a **rebuild** (by default):
    1. Evaluate manifests and create a **build** that stores information on how to load enabled datapacks.
        - *Safe mode may be triggered at this point; if it is: involved datapacks' safe mode tags are called, and SlimeCore stops here.*
    2. If the entire build is **valid**:
        1. Set the world's **current build** to match it.
        2. Call appropriate `disable` and `uninstall` function tags.
        3. Put datapacks in the correct loading order and disable all datapacks not in the build.
3. Initiate a **load**, based on the world's current build:
    1. Call **preload entrypoints**.
    2. Call **load tags**.
    3. Call **entrypoints**.

Management (disable, re-enable, uninstall) of SlimeCore-loaded datapacks is done through the `slimecore:rebuild` function, which explicitly triggers a rebuild with changes to the bulid specified via function inputs. As described, these changes will only take effect if they would result in a valid build, otherwise doing nothing. Managing SlimeCore-loaded datapacks with `/datapack` is improper.

### Datapack Paths

By default, SlimeCore expects datapack paths (name of file/folder a world's `datapacks/` folder) to follow a specific format. If for whatever reason the path of a SlimeCore-loaded datapack doesn't or can't follow this format, SlimeCore provides a method to manually link a datapack to its path via ad hoc in-game commands.

*See [this section](./admin_guide/key_concepts.md#datapack-paths) for more details.*

### Performance

- **On rebuild:** Causes significant single-tick delay that scales linearly with the number of installed SlimeCore-loaded datapacks; nearly all of this delay comes from execution of `/datapack` internally.
- **On load:** Minor single-tick overhead--insignificant compared to base reload delay.

SlimeCore does zero work outside of these processes.

## Get Started

- **[Admin Guide](./admin_guide/index.md)** - Manage SlimeCore-loaded datapacks in your world.
- **[Datapack Development Guide](./dev_guide/index.md)** - Create SlimeCore-loaded datapacks.
- **[Direct Interface Guide](./interface_guide/index.md)** - Create datapacks that directly interface with SlimeCore (e.g. frontends).