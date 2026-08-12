# Troubleshooting

- [Frontend Not Loading](#frontend-not-loading)
- [Getting Manifest Data](#getting-manifest-data)
- [Clean Rebuilding](#clean-rebuilding)
- [Frontend Not Loading](#frontend-not-loading)
- [Unfinished Loading/Rebuilding](#unfinished-loadingrebuilding)
- [Very Long Rebuilding](#very-long-rebuilding)
- [Rebuild Errors](#rebuild-errors)
- [Safe Mode](#safe-mode)

## Getting Manifest Data

The following commands can be used to get a datapack's [manifest](./key_concepts.md#manifests) data, given you know its pack ID:
```mcfunction
# will work for enabled datapacks that are part of the current build:
data get storage slimecore:data build.packs[{pack_id:"<pack ID>"}]
# OR
data get storage slimecore:data build.aux.pack_map.<pack ID>

# will work for any properly installed datapack:
data get storage slimecore:data world.installed[{pack:{pack_id:"<pack ID>"}}]
# OR
data get storage slimecore:data world.aux.installed_map.<pack ID>.pack
```

If you do not know a datapack's pack ID, your frontend should provide a reasonable method of listing installed datapacks and their information.

If all else fails, the function tag file `<datapack>/data/slimecore/tags/function/manifest.json` within a datapack contains the path to the function that the datapack's manifest data is directly defined in.

## Clean Rebuilding

If it seems like SlimeCore has "lost track" of datapacks, or you suspect incorrect datapack loading behavior, a **clean rebuild** may fix the issue.

A clean rebuild clears SlimeCore's memory of datapack/world state. As a side effect of this, if a clean rebuild fails, no datapacks will properly load until a rebuild succeeds. While this will most likely not cause critical errors, it is advised to try and avoid clean rebuild failures.

**IMPORTANT:** \
Any datapacks that are disabled before a clean rebuild must be re-enabled via `/datapack enable` for SlimeCore to track them again. \
*This is one of the only cases where you should use `/datapack` to manage datapacks.*

Your frontend should provide instructions on how to initiate a clean rebuild, likely as part of explicit rebuilding.

## Frontend Not Loading

If your frontend datapack doesn't seem to work/load, other datapacks may be silently causing rebuild errors, not allowing it to load. Try temporarily removing all SlimeCore-loaded datapacks except your frontend (and its dependencies, if any) from your world's `datapacks/` folder and run `/reload` in-game--this should load your frontend. You can then re-add the other datapacks back into your world's `datapacks/` folder and run `/reload` once again. 

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

It is normal behavior for rebuilding to cause a significant delay that lasts much longer than a default Minecraft `/reload`, especially when a large amount of datapacks are installed. If SlimeCore is working properly, log messages with the following format should be sent to the game/server console every ~0-2s during rebuilding:

```
XX:XX:XX.XXX net.minecraft.world.item.crafting.RecipeManager Server thread Loaded # recipes
XX:XX:XX.XXX net.minecraft.advancements.AdvancementTree Server thread Loaded # advancements
```

*Internally, SlimeCore uses `/datapack enable` and `/datapack disable` many times during rebuilding for datapack path resolution and datapack load ordering. Each time a datapack is enabled/disabled internally, Minecraft "soft reloads", causing roughly the same delay as a default Minecraft `/reload`. These "soft reloads" account for nearly all of the delay caused by rebuilding.*

If rebuilding is hanging but these logs are not being sent, it may indicate an infinite execution loop in one or more enable datapacks.

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

This can either be because the dependency(s) are not installed/enabled (most common), or that the dependency(s) are present but have an incompatible version. This error also occurs in the rare case that a datapack has the same pack ID as a required dependency, but not the same author ID (thus, is a different datapack).

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

This indicates that these datapacks are conceptually incompatible with each other.

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
Unfortunately, datapacks that share pack IDs are incompatible with each other. The primary remedy is to remove/uninstall datapacks such that no pack ID conflicts exist. If a newly installed datapack triggers this error (i.e. the datapack is never loaded), you can likely safely remove it from your world's `datapacks/` folder directly and then reload/rebuild without further process.

## Safe Mode

Upon rebuild, if SlimeCore detects that the current datapack/world state could be invalid and cannot be automatically recovered, **Safe mode** is enabled. In safe mode, no datapacks are normally loaded (e.g. load/entrypoint tags not called, though schedule/tick loops may continue through last load), and potentially affected datapacks, as well as their dependents, have their [safe mode tag](../dev_guide/full_guide.md#safe-mode-tag) called. This will likely result in reduced datapack functionality for the duration of safe mode.

Safe mode will be disabled upon rebuild when SlimeCore no longer detects an invalid datapack/world state.

While safe mode is enabled, storage NBT `slimecore:data` `world.safe_mode` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `calls` | List of `{pack_ref: <pack ID>}` | Packs that had their safe mode tag called when safe mode was enabled. |
| `reason` | *(See below)* | Information about the reason safe mode was enabled. |

### Safe Mode Reasons

#### Missing Datapack Path(s)

If the [Missing Datapack Path(s)](#missing-datapack-paths) rebuild error occurs, there is a possibility that some datapacks are in the wrong loading order.

*Internally, for datapacks with missing paths, SlimeCore cannot provide a path to `/datapack enable`/`/datapack disable`, thus cannot put said datapacks in their correct loading order.*

If this is the reason safe mode is triggered, storage NBT `slimecore:data` `world.safe_mode.reason` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `misloaded_datapacks_missing_path` | List of `{pack: <pack manifest>, path_override: <datapack_path>?}` | Pack manifests with missing datapack paths; `path_override` is only present if the pack has a [path override](./key_concepts.md#path-overriding). |

#### Duplicate Installed Pack IDs

If the [Duplicate Installed Pack IDs](#duplicate-installed-pack-ids) rebuild error occurs, multiple packs share the same pack ID and may have conflicting/overlapping resources, possibly leading to erroneous behavior.

If this is the reason safe mode is triggered, storage NBT `slimecore:data` `world.safe_mode.reason` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `duplicate_installed_pack_ids` | List of `{pack_id: <pack ID>, packs: <pack manifest>[]}` | Pack IDs that are shared between multiple packs (specified by `packs`). |

---
