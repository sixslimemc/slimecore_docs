# Troubleshooting

- [Getting Manifest Data](#getting-manifest-data)
- [Wipe Rebuilding](#wipe-rebuilding)
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

## Wipe Rebuilding

If it seems like SlimeCore has "lost track" of datapacks, or you suspect unusual erronious behavior, a **wipe rebuild** may fix the issue.

A wipe rebuild wipes SlimeCore's memory of datapack/world state. As a side effect of this, if a wipe rebuild fails, no datapacks will properly load until a rebuild succeeds. While this will most likely not cause critical errors, it is advised to try and avoid wipe rebuild failures.

Any datapacks that are disabled just before a wipe rebuild must be re-enabled via `/datapack enable` in order for SlimeCore to track them again.

Your frontend should provide instructions on how to initiate a wipe rebuild, likely as part of [explicit rebuilding](./key_concepts.md#managing-datapacks-explicit-rebuilding).

## Frontend Not Loading

If your frontend datapack doesn't seem be working, other datapacks may be silently causing rebuild errors, not allowing it to load. Try temporarily removing all SlimeCore-loaded datapacks except your frontend (and its dependencies, if any) from your world's `datapacks/` folder and run `/reload` in-game--this should load your frontend. You can then re-add the other datapacks back into your world's `datapacks/` folder and run `/reload` once again. 

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

It is normal behavior for rebuilding to cause a significant delay that lasts much longer than a default Minecraft `/reload`, especially when a large amount of datapacks are installed. If SlimeCore is working properly, log messages similar to the following format should be sent to the game/server console every ~0-2s during rebuilding:

```
XX:XX:XX net.minecraft.advancements.AdvancementTree Worker-Main-# Loaded # advancements
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

### Unfulfilled Dependencies

**Cause:** \
Datapack(s) require dependencies (other datapacks) that are not present.

This can either be because the dependencies are not installed/enabled (most common), the dependencies are present but have an incompatible version, or installed datapack(s) has the same pack ID but a different author ID from required dependencies (thus, are different datapacks and do not fulfill the dependency).

**Fix:** \
Install and/or enable the required dependencies. Your frontend should display download URLs for compatible versions of missing dependencies.

### Unsatisfied Contracts

**Cause:** \
There exist declared contracts that are not satisfied by any datapacks (i.e. datapack(s) require some functionality to be provided externally, but none is provided).

**Fix:** \
Find and install/enable datapack(s) that satisfy the contracts. Finding a datapack that satisfies a particular contract is not a strictly defined process, but it is likely that some list or "default" satisfier can be found at the info URL of the datapack that declares the given contract.

### Oversatisfied Contracts

**Cause:** \
There exist declared contracts that are satisfied by more than one datapack (i.e. the same functionality is provided by multiple datapacks).

Datapacks that satisfy the same contract are functionally--and likely conceptually--incompatible with eachother.

**Fix:** \
Remove datapacks from the build, such that each contract has exactly one satisfier.

### Entrypoint (or Preload Entrypoint) Order Conflicts

**Cause:** \
Some set(s) of datapacks have incompatible/conflicting entrypoint order specifications.

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the entrypoint ordering in the datapacks' manifest function (See [Datapack Development Guide](../dev_guide/index.md)).

### Dependency Cycles

**Cause:** \
Some set(s) of datapacks create a dependency cycle (e.g. A depends on B, B depends on C, C depends on A).

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the dependency cycles in the datapacks' manifest function (See [Datapack Development Guide](../dev_guide/index.md)).

### Invalid References in Manifests

**Cause:** \
Datapack(s) have a manifest that references artifacts (entrypoints, contracts, etc.) that do not exist.

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the issues in the manifest function(s) (See [Datapack Development Guide](../dev_guide/index.md)).

### Invalid Manifests

**Cause:** \
Datapack(s) have an invalid manifest function.

This error should only be encountered if you are developing your own datapack(s). If this error is encountered outside of datapack development, something is wrong with one or more installed datapacks.

**Fix:** \
Fix the issues in the manifest function(s) (See [Datapack Development Guide](../dev_guide/index.md)).

### Missing Path for Disabled Datapacks

**Cause:** \
There are disabled datapack(s) that do not have a standard path and/or their path override does not match their actual path.

**Fix:** \
Rename datapack files to match standard datapack path format or set correct path overrides (See [Datapack Paths](./key_concepts.md#datapack-paths)).

### Missing Path for Enabled Datapacks (Misloaded Datapacks)

**This error will trigger [safe mode (Misloaded Datapacks Missing Path)](#misloaded-datapacks-missing-path).**

**Cause:** \
There are enabled datapack(s) that do not have a standard path and/or their path override does not match their actual path.

**Fix:** \
Rename datapack files to match standard datapack path format or set correct path overrides (See [Datapack Paths](./key_concepts.md#datapack-paths)).

### Duplicate Installed Pack IDs

**This error will trigger [safe mode (Duplicate Installed Pack IDs)](#duplicate-installed-pack-ids-1).**

**Cause:** \
Multiple installed datapacks share the same pack ID.

**Fix:** \
Unfortunately, datapacks that share pack IDs are incompatible with each other. The primary remedy is to remove/uninstall datapacks such that no pack ID conflicts exist. If a newly installed datapack triggers this error (i.e. the datapack is never loaded), you can likely safely remove it from your world's `datapacks/` folder directly and then reload/rebuild without further process.

## Safe Mode

During a rebuild, if SlimeCore detects that the current datapack/world state could be invalid and cannot be automatically recovered, **safe mode** is enabled. In safe mode, no datapacks are normally loaded (e.g. load/entrypoint tags not called, though schedule/tick loops may continue through last load), and potentially affected datapacks, as well as their dependents, have their [safe mode tag](../dev_guide/full_guide.md#safe-mode-tag) called. This will likely result in reduced datapack functionality for the duration of safe mode.

Safe mode will be disabled upon rebuild when SlimeCore no longer detects an invalid datapack/world state.

While safe mode is enabled, storage NBT `slimecore:data` `world.safe_mode` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `calls` | List of `{pack_ref: <pack ID>}` | Packs that had their safe mode tag called when safe mode was enabled. |
| `reason` | *(See below)* | Information about the reason safe mode was enabled. |

### Safe Mode Reasons

#### Misloaded Datapacks Missing Path

If the [Missing Path for Enabled Datapacks (Misloaded Datapacks) rebuild error](#missing-path-for-enabled-datapacks-misloaded-datapacks) occurs, there is a possibility that some datapacks are in the wrong loading order and cannot be automatically re-ordered by SlimeCore.

*Internally, for datapacks with missing paths, SlimeCore cannot provide a path to `/datapack enable`/`/datapack disable`, thus cannot put said datapacks in their correct loading order.*

If this is the reason safe mode is triggered, storage NBT `slimecore:data` `world.safe_mode.reason` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `misloaded_datapacks_missing_path` | List of `{pack: PackManifest, path_override: <datapack path>?}` | Pack manifests with missing datapack paths; `path_override` is only present if the pack has a [path override](./key_concepts.md#path-overriding). |

#### Duplicate Installed Pack IDs

If the [Duplicate Installed Pack IDs rebuild error](#duplicate-installed-pack-ids) occurs, multiple packs share the same pack ID and may have conflicting/overlapping resources, possibly leading to erroneous behavior.

If this is the reason safe mode is triggered, storage NBT `slimecore:data` `world.safe_mode.reason` will contain the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `duplicate_installed_pack_ids` | List of `{pack_id: <pack id>, packs: [PackManifest]}` | Pack IDs that are shared between multiple packs (`packs` share the pack ID `pack_id`). |

---
