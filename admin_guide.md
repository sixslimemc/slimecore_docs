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

- Pack ID
- Author ID
- Version
- Download URL
- Dependencies

A datapack's manifest is set by a special manifest function, which is identified in `<datapack>/data/slimecore/tags/function/manifest.json`--the function included in this function tag file is the datapack's manifest function.

If a datapack is in the world's current build, it's manifest data can be retrieved in-game by running:

```mcfunction
data get storage slimecore:data build.packs[{pack_id:"<pack ID>"}]
# alternatively:
data get storage slimecore:data build.aux.pack_map.<pack ID>
```

### Rebuild Errors

Listed below are all of the possible reasons a rebuild can fail, as well as their fixes. Your frontend should notify you when and why a rebuild fails.

#### - Unfulfilled Dependency(s)

**Cause:** \
Datapack(s) require dependency(s) (other datapack(s)) that are not present.

This can either be because the dependency(s) are not installed/enabled (most common), or that the dependency(s) are present but have an incompatable version. This error also occurs in the rare case that a datapack has the same pack ID of a required dependency, but not the same author ID (thus, is a different datapack).

**Fix:** \
Install/enable the required dependency(s) to the build. Your frontend should display download URLs for compatible versions of missing dependencies.

#### - Unimplemented Abstract Interface(s)

**Cause:** \
Datapack(s) define abstract interface(s) that are not implemented by any other datapacks (i.e. they require some functionality to be provided externally, but none is provided).

**Fix:** \
Find and install/enable datapack(s) that implement the abstract interface(s). Finding a datapack that implements a particular abstract interface is not a strictly defined process, but it is likely that some list or "default" implementation can be found at the info URL of the datapack that defines the abstract interface(s).

#### - Multiple Abstract Implementations

**Cause:** \
Multiple datapacks implement the same abstract interface(s) (i.e. the same functionality is provided by multiple datapacks).

This indicates that these datapacks are conceptually incompatible with eachother.

**Fix:** \
Remove datapacks from the build, such that the abstract interface(s) are implemented exactly once.

#### - Missing Datapack Path(s)

**Cause:** \
There are datapack(s) with non-standard names without path overrides OR have a path overrides that do not match their names.

SlimeCore expects datapacks to have standard names, matching one of the following formats:
- `<author ID>.<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<author ID>.<pack ID>`
- `<pack ID>`

#### - Entrypoint (or Preload Entrypoint) Order Conflicts

#### - Dependency Cycle(s)

**Cause:** \
Some set(s) of datapacks create a dependency cycle (e.g. A depends on B, B depends on C, C depends on A).

This error should only be encountered if you are developing your own datapack(s), and may indicate an architectural codesmell. If this error is encountered outside of datapack development, something is very wrong.

**Fix:** \
Fix the dependency cycle(s) in the datapacks' manifest function (See [Datapack Development Guide](./development_guide.md)).


#### - Invalid Datapack Manifest(s)

**Cause:** \
One or more datapacks have an invalid manifest function.

This error should only be encountered if you are developing your own datapack(s) (or are for some reason changing the manifests of downloaded datapacks--don't do that).

**Fix:** \
Fix the issues in the manifest function(s) (See [Datapack Development Guide](./development_guide.md)).

#### - Duplicate Pack IDs

**Cause:** \
Multiple datapacks share the same pack ID. This is rare but can occur.

**Compromise:** \
Unfortunately, there is no easy non-destructive fix for this issue. Datapacks with the same pack ID are internally incompatable with eachother. While it is no "solution", the easiest option is to remove one of the conflicting datapacks from the build or replace it with another datapack with similar functionality. If both datapacks are well-known, there is a chance that one may have a release under a different pack ID; check info/author URLs.

Otherwise, the only "fix" for this issue is to manually edit one of the datapacks to reflect a different pack ID. This would likely include (but is not limited to) mass file renaming and text replacing to be done properly. This should be a last resort and be performed with great caution.

### Non-Standard Datapack Names

A datapack's name is its folder or .zip file name within the `/datapacks` directory of the world it is installed in. SlimeCore only automatically recognizes datapacks with standard names. For SlimeCore to recognize datapacks with non-standard names, extra steps must be taken.

A standard name matches one of the following formats:
- `<author ID>.<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<pack ID>.<major ver>.<minor ver>.<patch ver>`
- `<author ID>.<pack ID>`
- `<pack ID>`

If you cannot tell if datapack has a standard name or not, the data that should be present in its standard name can be found by opening the function tag `<datapack>/data/slimecore/tags/function/manifest.json`, and then opening the function contained in that tag. This manifest function should contain the relevant data in an obvious manner.

If your world has a SlimeCore-loaded datapack with a non-standard name, you can either:

#### Rename the Datapack to a Standard Name (Recommended)
Using the data found in the manifest function, rename the datapack to match one of the standard naming formats. The first format mentioned above is known as a "fully qualified" name and is always recommended over others.

#### Add a Path Override
Run the following command with the appropriate substitutions:

```mcfunction
data modify storage slimecore:config <pack ID> set value "file/<datapack name>"
```

This tells SlimeCore explicitly where the datapack is located. If a path override exists for a datapack, SlimeCore will **only** recognize it if it has that exact path/name and will not check its standard names.

To remove an override, you can run:

```mcfunction
data remove storage slimecore:config <pack ID>
```

## Using Scdev

World admins should be given the `scdev.listen` tag to recieve messages containing helpful information and errors.

```mcfunction
tag @s add scdev.listen
```


