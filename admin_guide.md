# SlimeCore Admin Guide

## Acquiring a Frontend
SlimeCore is a datapack that loads and manages other datapacks, *that's it*. It intentionally does not provide any player-facing features on its own. Instead, it is designed such that *other* datapacks, **frontends**, can provide such functionality. This is primarily so *you*, as the user/admin, can customize your experience. While not strictly required, a frontend is essential for easy in-game administration. Frontends are not "special", they are installed like any other datapack.

Because of this paradigm, frontends practically define how you interact with SlimeCore and SlimeCore-loaded datapacks; it is primarily the responsibilty of your respective frontend's author to document its usage. With that being said, most frontends should share a base level of functionality and regard the concepts covered in [Essential Concepts](#essential-concepts).

If you just want to get started quickly, you can install [Scdev](https://github.com/sixslimemc/scdev), a minimal frontend/utility created by the author of SlimeCore, and skip to the [Using Scdev](#using-scdev) section.

## Essential Concepts

### Builds

SlimeCore introduces the concept of the world's **build**. The world's build contains information on which datapacks to **load** (usually the world's currently installed datapacks) and in what order.

By default, SlimeCore checks for any changed or newly installed datapacks upon world reload (`/reload`), then, if any changes are detected, it will attempt to **rebuild**. When SlimeCore rebuilds, it validates that all installed/enabled packs are compatible with the world, and then overwrites the world's build if successful. If SlimeCore finds that any packs are incompatible with the world (i.e. the rebuild fails), **no changes are made to the world's build**.

A large single-tick lag spike may occur during rebuild. This is expected behavior.

### Explicit Rebuilds (Enabling, Disabling, and Uninstalling Datapacks)
**Explicitly rebuilding** is the only proper way to enable/disable/uninstall SlimeCore-loaded datapacks. In an explicit rebuild, you may specify **staged** changes to the build (enabling/disabling/uninstalling datapacks), and if those changes would result in a valid build, the changes are applied. If the staged changes would result in an *invalid* build, **no changes are made**.

Your frontend should provide instructions how to trigger an explicit rebuild (or some similar functionality).

### Loading

After a successful rebuild and/or world reload, SlimeCore will initiate a **load**. In simple terms, this just tells all datapacks that are contained within the world's build that they are enabled (conceptually equivalent to triggering the `#minecraft:load` function tag).

It is important to note the difference between rebuilding and loading: Rebuilding is setting the world's build, while loading is using the world's build to load datapacks.

## Configuration

## Troubleshooting


## Using Scdev

World admins should be given the `scdev.listen` tag to recieve messages containing helpful information and errors.

```mcfunction
tag @s add scdev.listen
```


