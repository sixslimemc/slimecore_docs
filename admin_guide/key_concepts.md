# Key Concepts

## Manifests

Every SlimeCore-loaded datapack has a **manifest**, which is NBT data specifying information about itself (pack ID, author ID, version, dependencies, etc.), as well as other information SlimeCore needs in order to load it properly. Manifests are what allow SlimeCore to recognize and load datapacks in a structured manner--in a sense, to SlimeCore, a datapack *is* it's manifest.

To retrieve a datapack's raw manifest data (given you know the pack ID), you can do any of the following commands:
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

*Alternatively, the function tag file `<datapack>/data/slimecore/tags/function/manifest.json` within a datapack contains the path to the function that the datapack's manifest data is directly defined in.*

## Rebuilding

SlimeCore will **rebuild** when any new datapacks are installed/enabled/disabled upon world reload. A world reload that triggers a rebuild will often take longer (sometimes significantly, depending on the amount of datapacks installed) than world reloads that do not trigger a rebuild.

Rebuilding can *fail*, indicating that there exist incompatibilies, errors, and/or unfulfilled requirements of the currently installed datapacks. Your frontend notify you when and why a rebuild fails. A full list of rebuild errors and how to fix them can be found [here](./troubleshooting.md#rebuild-errors).

The most important aspect of rebuilding is that SlimeCore will not apply any changes to datapack loading until a rebuild *succeeds*. This means that, in most practical circumstances, datapacks will not be loaded unless they are garunteed to be loaded correctly. When a rebuild succeeds, the world's [build data](#build-data) is updated.

## Build Data


## Managing Datapacks (Explicit Rebuilding)
**Explicitly rebuilding** is the only proper way to enable, disable, and/or uninstall SlimeCore-loaded datapacks. In an explicit rebuild, you may specify **staged** changes to the build (enables, disables, uninstalls), and if those changes would result in a valid build, the changes are applied. If the staged changes would result in an invalid build, **no changes are made**.

Your frontend should provide instructions on how to trigger an explicit rebuild (or some similar functionality).

**Using `/datapack` to manage SlimeCore-loaded datapacks is improper** and may create unexpected behavior.

If you only want to allow SlimeCore to rebuild explicitly, and not automatically on world reload, you can set the value of `slimecore:config explicit_rebuild_only` (NBT storage) to `true`. If this setting is `true`, newly installed packs will not be effectively enabled until an explicit reload is triggered.