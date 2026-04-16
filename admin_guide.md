# SlimeCore Admin Guide

## Acquiring a Frontend
SlimeCore is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This is primarily so *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

Because of this paradigm, frontends define how you interact with SlimeCore and SlimeCore-loaded datapacks. it is primarily the responsibilty of your chosen frontend's author to document its usage. With that said, most frontends should share a base level of functionality and adhere to the concepts covered in [Key Concepts](#essential-concepts).

If you just want to get started quickly, you can install [Scdev](https://github.com/sixslimemc/scdev), a minimal frontend created by the author of SlimeCore, and skip to the [Using Scdev](#using-scdev) section.

## Key Concepts

### Builds

SlimeCore introduces the concept of the world's current **build**. The current build contains information on which datapacks to **load** (usually the world's currently installed datapacks) and in what order.

By default, SlimeCore checks for any changed or newly installed datapacks upon world reload (`/reload`), then, if any changes are detected, it will attempt to **rebuild**. When SlimeCore rebuilds, it validates that all installed/enabled packs are compatible with the world, and then overwrites the current build if successful. If SlimeCore finds that any packs are incompatible with the world (i.e. the rebuild fails), **no changes are made to the current build**.

A large single-tick lag spike may occur during rebuild. This is expected behavior.

The current build is stored as NBT storage data in `slimecore:data build`. This data should be treated as read-only.

### Loading

After a successful rebuild and/or world reload, SlimeCore will initiate a **load**. In simple terms, this just tells all datapacks that are contained within the current build that they are enabled (conceptually equivalent to triggering the `#minecraft:load` function tag).

It is important to note the difference between rebuilding and loading: Rebuilding is setting the current build, while loading is using the current build to load datapacks.

### Enabling, Disabling, and Uninstalling Datapacks (Explicit Rebuilding)
**Explicitly rebuilding** is the only proper way to enable, disable, and/or uninstall SlimeCore-loaded datapacks. In an explicit rebuild, you may specify **staged** changes to the build (enables, disables, uninstalls), and if those changes would result in a valid build, the changes are applied. If the staged changes would result in an invalid build, **no changes are made**.

Your frontend should provide instructions on how to trigger an explicit rebuild (or some similar functionality).

**Using `/datapack` to manage SlimeCore-loaded datapacks is improper** and may create unexpected behavior.

If you only want to allow SlimeCore to rebuild explicitly, and not automatically on world reload, you can set the value of `slimecore:config explicit_rebuild_only` (NBT storage) to `true`. If this setting is `true`, newly installed packs will not be effectively enabled until an explicit reload is triggered.

## Basic Troubleshooting

### General

#### Datapack Manifests

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

#### Forcing a Clean Rebuild

If it seems like SlimeCore has "lost track" of enabled/disabled datapacks, or you suspect incorrect datapack loading behavior (most likely caused by improper datapack uninstallation or usage of `/datapack`), a **clean rebuild** may fix the issue.

A clean rebuild wipes the current build data before rebuilding. This does mean, however, if a clean rebuild fails, the current build will be left empty and no datapacks will load; this will fix itself once a rebuild succeeds.

Clean rebuilds should only be initiated if you suspect something is wrong, as they essentially make SlimeCore "forget" everything and assume that the world is brand-new.

Your frontend should provide instructions on how to initiate a clean rebuild, most likely as part of explicit rebuilding.

#### Unfinished Loading

It is possible that a SlimeCore load reaches the `max_command_sequence_length` gamerule limit (especially with a very large amount of datapacks present). If this is the case, some datapacks may not load properly and odd or unexpected behavior may occur.

This can be fixed by simply increasing the `max_command_sequence_length` gamerule.

```mcfunction
gamerule max_command_sequence_length <value>
```

#### Unfinished or Very Long Rebuilding

When first installed, SlimeCore sets the following data:
```mcfunction
data modify storage slimecore:config build_time_gamerules.max_command_sequence_length set value 2147483647
data modify storage slimecore:config build_time_gamerules.max_command_forks set value 2147483647
```

These values override their respective gamerules during rebuilding and are the maximum possible values; they should generally not be changed from their defaults without good reason. Unless these values are manually set too low, SlimeCore should always finish rebuilding (however long it may take).

If rebuilding seems to be taking very long, check server/game logs during the rebuild process. Info logs similar to these should be sent at regular intervals during rebuilding:

```
XX:XX:XX.XXX net.minecraft.world.item.crafting.RecipeManager Server thread Loaded # recipes
XX:XX:XX.XXX net.minecraft.advancements.AdvancementTree Server thread Loaded # advancements
```

If these logs are being sent, then SlimeCore is likely working as intended and you may just have a large amount of datapacks. If they are not being sent, then it may indicate a SlimeCore bug.

### Rebuild Errors

Listed below are all of the possible reasons a rebuild can fail, as well as their fixes.

---

#### Unfulfilled Dependency(s)

**Cause:** \
Datapack(s) require dependency(s) (other datapack(s)) that are not present.

This can either be because the dependency(s) are not installed/enabled (most common), or that the dependency(s) are present but have an incompatable version. This error also occurs in the rare case that a datapack has the same pack ID of a required dependency, but not the same author ID (thus, is a different datapack).

**Fix:** \
Install/enable the required dependency(s) to the build. Your frontend should display download URLs for compatible versions of missing dependencies.

---

#### Unimplemented Abstract Interface(s)

**Cause:** \
Datapack(s) define abstract interface(s) that are not implemented by any other datapacks (i.e. they require some functionality to be provided externally, but none is provided).

**Fix:** \
Find and install/enable datapack(s) that implement the abstract interface(s). Finding a datapack that implements a particular abstract interface is not a strictly defined process, but it is likely that some list or "default" implementation can be found at the info URL of the datapack that defines the abstract interface(s).

---

#### Multiple Abstract Implementations

**Cause:** \
Multiple datapacks implement the same abstract interface(s) (i.e. the same functionality is provided by multiple datapacks).

This indicates that these datapacks are conceptually incompatible with eachother.

**Fix:** \
Remove datapacks from the build, such that the abstract interface(s) are implemented exactly once.

---

#### Missing Datapack Path(s)

**Cause:** \
There are datapack(s) with non-standard names without path overrides OR datapack(s) with path overrides that do not match their names.

SlimeCore expects datapacks to have standard names, matching one of the following formats:
- `<author ID>.<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<author ID>.<pack ID>`
- `<pack ID>`

*(See [Datapack Manifests](#datapack-manifests) for getting the referenced information.)*

**Fix:** \
Rename datapack(s) with non-standard names to match standard name format.

*OR*

Add or fix the path override for the datapack(s):
```mcfunction
data modify storage slimecore:config datapack_path_overrides.<pack ID> set value "file/<datapack name>"
#                             Should match the path as shown in `/datapack list` ^^^^^^^^^^^^^^^^^^^^^^
```

To remove a path override:
```mcfunction
data modify storage slimecore:config datapack_path_overrides.<pack ID>
```

---

#### Entrypoint (or Preload Entrypoint) Order Conflicts

**Cause:** \
Some set(s) of datapacks have incompatible/conflicting entrypoint order specifications.

This error should only be encountered if you are developing your own datapack(s); it may indicate that your datapack(s)' dependencies may have entrypoint ordering specifications that you are unaware of. If this error is encountered outside of datapack development, something is very wrong.

**Fix:** \
Fix the entrypoint ordering in the datapacks' manifest function (See [Datapack Development Guide](./development_guide.md)).

---

#### Dependency Cycle(s)

**Cause:** \
Some set(s) of datapacks create a dependency cycle (e.g. A depends on B, B depends on C, C depends on A).

This error should only be encountered if you are developing your own datapack(s); it may indicate an architectural codesmell. If this error is encountered outside of datapack development, something is very wrong.

**Fix:** \
Fix the dependency cycle(s) in the datapacks' manifest function (See [Datapack Development Guide](./development_guide.md)).

---

#### Invalid Datapack Manifest(s)

**Cause:** \
One or more datapacks have an invalid manifest function.

This error should only be encountered if you are developing your own datapack(s) (or are for some reason changing the manifests of downloaded datapacks--this is not advised).

**Fix:** \
Fix the issues in the manifest function(s) (See [Datapack Development Guide](./development_guide.md)).

---

#### Duplicate Pack IDs

**Cause:** \
Multiple datapacks share the same pack ID. This is rare but can occur.

**Compromise:** \
Unfortunately, there is no easy non-destructive fix for this issue. Datapacks with the same pack ID are internally incompatable with eachother. While it is no "solution", the easiest option is to remove one of the conflicting datapacks from the build or replace it with another datapack with similar functionality. If both datapacks are well-known, there is a chance that one may have a release under a different pack ID; check info/author URLs.

Otherwise, the only "fix" for this issue is to manually edit one of the datapacks to reflect a different pack ID. This would likely include (but is not limited to) mass file renaming and text replacing to be done properly. This should be a last resort and be performed with great caution.

---

## Using Scdev

This section explains common operations using the [Scdev](https://github.com/sixslimemc/scdev) frontend.

### Logs

Logs are the primary way that Scdev provides necessary information and errors. Add the `scdev.listen` tag to yourself (and/or admins) to recieve logs.

```mcfunction
tag @s add scdev.listen
```

#### Rebuild Logs

Scdev will send logs upon [rebuild](#builds) (usually when datapacks change).

If a rebuild fails, the logs will explain the reason. (See [Rebuild Errors](#rebuild-errors) for a full list of erros that can occur and their fixes.)

If rebuilding starts but never seems to finish, see [this section](#unfinished-or-very-long-rebuilding).

#### Load Logs

Scdev will send logs upon [load](#loading) (e.g. `/reload`).

The logs contains a list of all datapacks enabled (i.e. in the [current build](#builds)), as well as their entrypoints, in the order that they are loaded/called. (Entrypoint list is primarily for datapack development is not necessary administration knowledge.)

If a datapack is not on this list but you think it should be (or vice versa), a rebuild may have failed. Check the rebuild logs (above/before the load logs).

A "Loading finished." message should be sent after each load. If not, it is an indication of an [unfinished load](#unfinished-loading).

### Managing 

TODO: make Scdev better.