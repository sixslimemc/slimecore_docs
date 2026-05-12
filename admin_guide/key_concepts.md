# Key Concepts

## Builds

SlimeCore introduces the concept of the world's current **build**. The current build contains information on which datapacks to **load** (usually the world's currently installed datapacks) and in what order.

By default, SlimeCore checks for any changed or newly installed datapacks upon world reload (`/reload`), then, if any changes are detected, it will attempt to **rebuild**. When SlimeCore rebuilds, it validates that all installed/enabled packs are compatible with the world, and then overwrites the current build if successful. If SlimeCore finds that any packs are incompatible with the world (i.e. the rebuild fails), **no changes are made to the current build**.

A large single-tick lag spike may occur during rebuild. This is expected behavior.

The current build is stored as NBT storage data in `slimecore:data build`. This data should be treated as read-only.

## Loading

After a successful rebuild and/or world reload, SlimeCore will initiate a **load**. In simple terms, this just tells all datapacks that are contained within the current build that they are enabled (conceptually equivalent to triggering the `#minecraft:load` function tag).

It is important to note the difference between rebuilding and loading: Rebuilding is setting the current build, while loading is using the current build to load datapacks.

## Enabling, Disabling, and Uninstalling Datapacks (Explicit Rebuilding)
**Explicitly rebuilding** is the only proper way to enable, disable, and/or uninstall SlimeCore-loaded datapacks. In an explicit rebuild, you may specify **staged** changes to the build (enables, disables, uninstalls), and if those changes would result in a valid build, the changes are applied. If the staged changes would result in an invalid build, **no changes are made**.

Your frontend should provide instructions on how to trigger an explicit rebuild (or some similar functionality).

**Using `/datapack` to manage SlimeCore-loaded datapacks is improper** and may create unexpected behavior.

If you only want to allow SlimeCore to rebuild explicitly, and not automatically on world reload, you can set the value of `slimecore:config explicit_rebuild_only` (NBT storage) to `true`. If this setting is `true`, newly installed packs will not be effectively enabled until an explicit reload is triggered.