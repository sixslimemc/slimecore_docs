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

## Errors and Troubleshooting

### Rebuild Failure

Below are all of the possible reasons a rebuild can fail, as well as their fixes. Your frontend should notify you when and why a rebuild fails.

#### Unfulfilled Dependencies

#### Duplicate Pack IDs

#### Invalid Datapack Manifest(s)

#### Multiple Abstract Implementations

#### Unimplemented Abstract Interface(s)

#### Dependency Cycle(s)

#### Entrypoint (or Preload Entrypoint) Order Conflicts

#### Missing Datapack Path(s)

### Non-Standard Datapack Names

A datapack's name is its folder or .zip file name within the `/datapacks` directory of the world it is installed in. SlimeCore only automatically recognizes datapacks with standard names. For SlimeCore to recognize datapacks with non-standard names, extra steps must be taken.

A standard name matches one of the following formats:
- `<author id>.<pack id>.<major ver>.<minor ver>.<patch ver>`
- `<pack id>.<major ver>.<minor ver>.<patch ver>`
- `<author id>.<pack id>`
- `<pack id>`

If you cannot tell if datapack has a standard name or not, the data that should be present in its standard name can be found by opening the function tag `<datapack>/data/slimecore/tags/function/manifest.json`, and then opening the function contained in that tag. This manifest function should contain the relevant data in an obvious manner.

If your world has a SlimeCore-loaded datapack with a non-standard name, you can either:

#### Rename the Datapack to a Standard Name (Recommended)
Using the data found in the manifest function, rename the datapack to match one of the standard naming formats. The first format mentioned above is known as a "fully qualified" name and is always recommended over others.

#### Add a Path Override
Run the following command with the appropriate substitutions:

```mcfunction
data modify storage slimecore:config <pack id> set value "file/<datapack name>"
```

This tells SlimeCore explicitly where the datapack is located. If a path override exists for a datapack, SlimeCore will **only** recognize it if it has that exact path/name and will not check its standard names.

To remove an override, you can run:

```mcfunction
data remove storage slimecore:config <pack id>
```

## Using Scdev

World admins should be given the `scdev.listen` tag to recieve messages containing helpful information and errors.

```mcfunction
tag @s add scdev.listen
```


