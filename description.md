# SlimeCore Description

SlimeCore is a datapack that serves as a loader of other datapacks. If you are familiar with [Lantern Load](https://github.com/LanternMC/load), SlimeCore serves a similar purpose, but takes it multiple steps further.

SlimeCore allows datapacks to specify:
- Version
- Dependencies, required and optional
- Tick (entrypoint) order relative to other datapacks
- Abstract interfaces that must be implemented by other datapacks
- Pack and author metadata
- Download URL(s)

Datapacks specify this information via a *manifest* function. Upon world reload, SlimeCore processes all datapacks' manifests, validates relationships, then executes a compatible load/calling order if everything is valid.

SlimeCore completely replaces the usage of `#minecraft:load` and `#minecraft:tick`. Datapacks instead define *their own* **load**, **disable**, and **uninstall** function tags, as well as any number of **entrypoints** to be called after all datapacks are loaded. SlimeCore then calls these tags when appropriate.

SlimeCore is designed to be **atomic**. If used properly, **no changes to datapack loading will ever be made unless they are verified to work.** This includes enabling/disabling/uninstalling datapacks, which SlimeCore also manages. For example, SlimeCore will not allow you to disable a datapack if another enabled datapack has it specified as a dependency; it will require that the dependent is disabled before the dependency--which is automatically enforced if both are disabled on the same rebuild (reload).

SlimeCore is designed to be **deterministic**. If used properly, **previous world state/datapacks should never affect the outcome of a rebuild/load**. The same set of enabled datapacks will always load exactly the same way.

SlimeCore intentionally does not implement any "frontend" features such as chat messages, dialogs, or user-functions. Instead, it provides a public API for other datapacks to use, such that "frontends" can be created/shared like any other datapack. For instance, a datapack could use `#slimecore:hook/meta_info/rebuild/end` to display a message to admins containing the download URL(s) for any missing dependencies (dependency download URLs are provided by dependent's manifest) after a rebuild. \
*[Scdev](https://github.com/sixslimemc/scdev) is a very minimal development utility/frontend created by the author of SlimeCore.*

## Mission Statement

SlimeCore's primary goal is to support a community-driven, decentralized datapack ecosystem that is accessible to all datapack users and developers. It should not require the use of third party programs, but also should not significantly obstruct workflows that may include them. SlimeCore is designed to be simple and robust, such that functionality stays consistent and minimal maintenance/changes are required through Minecraft updates.

## Demonstration

```mcfunction

# 'pack_id' must match the datapack's namespace (function tag `#<pack_id>:load` is called during loading).
data modify storage slimecore:in manifest.pack.pack_id set value "mypack"
# 'author_id' should represent you, together 'author_id' and 'pack_id' uniquely identify a datapack.
data modify storage slimecore:in manifest.pack.author_id set value "myauthorid"

# Version:
data modify storage slimecore:in manifest.pack.version set value {major:1, minor:0, patch:0}

# Direct download URL:
data modify storage slimecore:in manifest.pack.url set value "https://github.com/mygithub/mypack/releases/download/v0.1.0/myauthorid.mypack.1.0.0.zip"

# Dependencies:
# Dependencies must include a direct download URL of any valid version of the dependency.
data modify storage slimecore:in manifest.pack.dependencies set value []
data modify storage slimecore:in manifest.pack.dependencies append value {pack_id:"otherpack_a", author_id:"otherauthor", optional:false, version:{major:1, minor:2}, download:{url:"https://github.com/otherauthor/otherpack_a/releases/download/v1.2.0/otherauthor.otherpack_a.1.2.0.zip", version:{major:1, minor:2, patch:0}}}
# Dependencies can be optional.
data modify storage slimecore:in manifest.pack.dependencies append value {pack_id:"otherpack_b", author_id:"otherauthor", optional:true, version:{major:3, minor:4}, download:{url:"https://github.com/otherauthor/otherpack_b/releases/download/v3.4.1/otherauthor.otherpack_a.3.4.1.zip", version:{major:3, minor:4, patch:1}}}

# Entrypoints:
# Entrypoints are called after all datapacks are loaded.
# each entrypoint represents function tag `#mypack:entrypoint/<id>`.
data modify storage slimecore:in manifest.pack.entrypoints append value {id:"tick"}
# This entrypoint will always be called after `#otherpack_a:entrypoint/tick`:
data modify storage slimecore:in manifest.pack.entrypoints append value {id:"my_interaction", after:[{pack_ref:"otherpack_a", id:"tick"}]}

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
data modify storage slimecore:in manifest.pack.abstract_implementations append value {pack_ref:"otherpack_a", id:"other_interface"}

# If this pack provides any functionality/features on its own, it is *not* a library.
data modify storage slimecore:in manifest.pack.is_library set value false

# Display information:
# Not directly used by SlimeCore, but may be read/displayed by other datapacks.
data modify storage slimecore:in manifest.pack.display.name set value "My Demonstration Pack"
data modify storage slimecore:in manifest.pack.display.summary set value "A pack used for demonstration!"
data modify storage slimecore:in manifest.pack.display.author_name set value "My Username"
data modify storage slimecore:in manifest.pack.display.links.author set value "https://github.com/mygithub"
data modify storage slimecore:in manifest.pack.display.links.info set value "https://github.com/mygithub/mypack"
data modify storage slimecore:in manifest.pack.display.links.versions set value "https://github.com/mygithub/mypack/releases"

# All manifests end with calling `slimecore:api/manifest`.
function slimecore:api/manifest
```

The above demonstrates a manifest function would be added to `#slimecore:manifest`, similarly to how a (non-SlimeCore-loaded) datapack would add to `#minecraft:load`.

Upon every world reload, SlimeCore calls `#slimecore:manifest`, "collecting" all manifests. If any changes to the list of manifests is detected, SlimeCore initiates a **rebuild** (by default). Then, regardless of if a rebuild was initiated or successful, SlimeCore initiates a **load**.

A **rebuild** processes all collected manifests, validates datapack relationships, finds a valid load and entrypoint ordering, and if successful, sets the worlds **current build** to reflect them.

A **load** calls preload entrypoints, then load tags, then entrypoints, according to the datapacks included and order specified by the world's current build.

A rebuild can also be manually initiated via the `slimecore:rebuild` function. Inputs can be provided to this function to "stage" datapacks for disabling, enabling, or uninstallation. If the staged changes would result in an invalid build, no changes to the world are actually made. This function is the only proper way to enable, disable, and uninstall SlimeCore-loaded datapacks.

## Performance

Depending on the amount of datapacks, a significant single-tick delay may occur during rebuilding, as well as a shorter single-tick delay during loading.

Otherwise, SlimeCore does not run any ticking commands and has negligible performance impact.

## Usage Guides

**[Admin Guide](./admin_guide.md)** - Manage SlimeCore-loaded datapacks in your world.

**[Datapack Development Guide](./development_guide.md)** - Create SlimeCore-loaded datapacks.

**[Interface Guide](./interface_guide.md)** - Create datapacks that directly interface with Slimecore (e.g. frontends).