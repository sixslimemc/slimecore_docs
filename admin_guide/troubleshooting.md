# Troubleshooting

- [Getting Manifest Data](#getting-manifest-data)
- [Clean Rebuilding](#clean-rebuilding)
- [Unfinished Loading/Rebuilding](#unfinished-loadingrebuilding)
- [Very Long Rebuilding](#very-long-rebuilding)
- [Safe Mode](#safe-mode)
- [Rebuild Errors](#rebuild-errors)

## Getting Manifest Data

Each SlimeCore-loaded datapack has a **manifest** that provides useful information about itself, including but not limited to:

- Pack ID & Author ID
- Download & Info URLs
- Dependencies (and their download URLs)
- Version

A datapack's manifest is set by a special manifest function, which is identified in `<datapack>/data/slimecore/tags/function/manifest.json`--the function included in this function tag file is the datapack's manifest function.

Datapacks in the current build have their manifest data stored in a list at NBT storage `slimecore:data build.packs`, as well as a mapping with keys being pack IDs at `slimecore:data build.aux.pack_map`.

```mcfunction
# Get a particualar datapack's manifest data based on it's pack ID:
data get storage slimecore:data build.packs[{pack_id:"<pack ID>"}]
# or
data get storage slimecore:data build.aux.pack_map."<pack ID>"
```

## Clean Rebuilding

If it seems like SlimeCore has "lost track" of enabled/disabled datapacks, or you suspect incorrect datapack loading behavior (most likely caused by improper datapack uninstallation or usage of `/datapack`), a **clean rebuild** may fix the issue.

If a clean rebuild fails, no datapacks will properly load until a rebuild succeeds. While this will most likely not cause critical errors, it is advised to try and avoid clean rebuild failures.

**IMPORTANT:** \
Clean rebuilds make SlimeCore "forget" about disable datapacks. Any datapacks that are disabled before a clean rebuild must be re-enabled via `/datapack enable` for SlimeCore to "see" them again. \
*This is one of the only cases where you should use `/datapack` to manage datapacks.*

Your frontend should provide instructions on how to initiate a clean rebuild, likely as part of explicit rebuilding.

## Unfinished Loading/Rebuilding

While rare, the chances that the `max_command_sequence_length` gamerule limit is reached during a load is increased when using SlimeCore. If this happens, simply increase the value for the `max_command_sequence_length` gamerule.

```mcfunction
gamerule max_command_sequence_length <value>
```

By default, SlimeCore automatically overrides the `max_command_sequence_length` and `max_command_forks` gamerules while a *rebuild* is in progress, setting them to their maximum values. While not advised, these override values can be changed via:

```mcfunction
data modify storage slimecore:config build_time_gamerules.max_command_sequence_length set value <int>
data modify storage slimecore:config build_time_gamerules.max_command_forks set value <int>
```

## Very Long Rebuilding

It is normal and expected behavior for rebuilding to cause significant a delay, especially when a large amount of datapacks are installed. If SlimeCore is working properly, log messages with the following format should be sent to the game/server console every ~0-2s during rebuilding:

```
XX:XX:XX.XXX net.minecraft.world.item.crafting.RecipeManager Server thread Loaded # recipes
XX:XX:XX.XXX net.minecraft.advancements.AdvancementTree Server thread Loaded # advancements
```

If these logs are not being sent and rebuilding is still hanging, or the number of these logs far exceed double the number of datapacks installed, it may indicate a bug within SlimeCore.

## Safe Mode

Datapacks that share the same pack ID are *inherently incompatible*. When a reload occurs and there exist multiple datapacks that share the same pack ID, SlimeCore will enter **safe mode** until the issue is resolved. In safe mode, datapacks will not fully reload, and the datapacks with shared pack IDs may have reduced functionality. 

Unfortunately, the best option for resolving such an issue is to uninstall/remove datapacks such that none share pack IDs. If safe mode is triggered right after you put a new datapack into your world folder, you can safely remove it from the world folder--and will likely be required to--before rebuilding/reloading; safe mode prevents the initialization of new datapacks.

An alternative, much more difficult fix would be to manually edit the new datapack to reflect a different pack ID. This process is out of this guide's scope, but would include more than just changing it's manifest definition, as datapack implementation relies on pack ID for namespacing.

## Rebuild Errors

A rebuild can fail for the following reasons:
- [Unfulfilled Dependency(s)](#unfulfilled-dependencys)
- [Unimplemented Abstract Interface(s)](#unimplemented-abstract-interfaces)
- [Multiple Abstract Implementations](#multiple-abstract-implementations)
- [Entrypoint (or Preload Entrypoint) Order Conflicts](#entrypoint-or-preload-entrypoint-order-conflicts)
- [Dependency Cycle(s)](#dependency-cycles)
- [Invalid Datapack Manifest(s)](#invalid-datapack-manifests)
- [Missing Datapack Path(s)](#missing-datapack-paths)
- [Duplicate Pack IDs](#duplicate-pack-ids)

### Unfulfilled Dependency(s)

**Cause:** \
Datapack(s) require dependency(s)--other datapack(s)--that are not present.

This can either be because the dependency(s) are not installed/enabled (most common), or that the dependency(s) are present but have an incompatable version. This error also occurs in the rare case that a datapack has the same pack ID of a required dependency, but not the same author ID (thus, is a different datapack).

**Fix:** \
Install/enable the required dependency(s) to the build. Your frontend should display download URLs for compatible versions of missing dependencies.

### Unimplemented Abstract Interface(s)

**Cause:** \
Datapack(s) define abstract interface(s) that are not implemented by any other datapacks (i.e. they require some functionality to be provided externally, but none is provided).

**Fix:** \
Find and install/enable datapack(s) that implement the abstract interface(s). Finding a datapack that implements a particular abstract interface is not a strictly defined process, but it is likely that some list or "default" implementation can be found at the info URL of the datapack that defines the abstract interface(s).

### Multiple Abstract Implementations

**Cause:** \
Multiple datapacks implement the same abstract interface(s) (i.e. the same functionality is provided by multiple datapacks).

This indicates that these datapacks are conceptually incompatible with eachother.

**Fix:** \
Remove datapacks from the build, such that the abstract interface(s) are implemented exactly once.

### Entrypoint (or Preload Entrypoint) Order Conflicts

**Cause:** \
Some set(s) of datapacks have incompatible/conflicting entrypoint order specifications.

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the entrypoint ordering in the datapacks' manifest function (See [Datapack Development Guide](../dev_guide/index.md)).

### Dependency Cycle(s)

**Cause:** \
Some set(s) of datapacks create a dependency cycle (e.g. A depends on B, B depends on C, C depends on A).

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the dependency cycle(s) in the datapacks' manifest function (See [Datapack Development Guide](../dev_guide/index.md)).

### Invalid Datapack Manifest(s)

**Cause:** \
One or more datapacks have an invalid manifest function.

This error should only be encountered if you are developing your own datapack(s) (or are for some reason changing the manifests of downloaded datapacks--this is not advised).

**Fix:** \
Fix the issues in the manifest function(s) (See [Datapack Development Guide](../dev_guide/index.md)).

### Missing Datapack Path(s)

This will trigger [safe mode](#safe-mode).

**Cause:** \
There are datapack(s) with non-standard paths (without path overrides), or datapack(s) with path overrides that do not match their actual paths.

**Fix:** \
*See [Datapack Paths](./key_concepts.md#datapack-paths).*

### Duplicate Installed Pack IDs

This will trigger [safe mode](#safe-mode).

**Cause:** \
Multiple installed datapacks share the same pack ID.

**Fix:** \
Unfortunately, datapacks with identical pack IDs are inherently incompatible with eachother. The primary remedy is to remove/uninstall datapacks such that no pack ID conflicts exist. If a newly installed datapack triggers this error (i.e. the datapack is never loaded), you can likely safely remove it from your world's `datapacks/` folder directly without further process.

---

**Next:** [Using SCDev](./using_scdev.md) (only applicable if using [SCDev](https://github.com/sixslimemc/scdev))