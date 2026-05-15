# Key Concepts

- [Manifests](#manifests)
- [Rebuilding](#rebuilding)
- [Managing Datapacks (Explicit Rebuilding)](#managing-datapacks-explicit-rebuilding)
- [Build Data](#build-data)
- [World Data](#world-data)

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

## Managing Datapacks (Explicit Rebuilding)

To disable, (re-)enable, or uninstall datapacks, you *must* do it through **explicit rebuilding**, not the `/datapack` command. Explicit rebuilding allows you to *stage* such changes and will verify that they are valid before applying them. If any changes are invalid (ex: disabling a datapack that is a dependency of an enabled datapack), no changes will be made.

In all practical cases (aside from recovering from a [clean rebuild](./troubleshooting.md#clean-rebuilding)), **explicit rebuilding should be the only method of disabling/enabling/uninstalling datapacks**. Using `/datapack` directly for these purposes is considered a user-error and may cause unexpected behavior.

Your chosen frontend should provide a documented method on initiating an explicit rebuild.

## Build Data

**Build data** is a struct at NBT storage location `slimecore:data` `build` containing information about the *currently enabled* datapacks and how they load. \
It is  updated upon *successful rebuild* and has the following keys:
| Key | Type | Description |
| --- | --- | --- |
| `packs` | List of pack manifests | All enabled pack manifests in the order that they are loaded. |
| `order.load` | List of `{pack_ref: <pack ID>, index: int}` | All enabled packs (references) in the order that they are loaded, with each `index` key matching the list index of the respective element. |
| `order.entrypoint` | List of `{pack_ref: <pack ID>, id: <entrypoint ID> index: int}` | Similar to `order.load`, but lists all entrypoints in their calling order. |
| `order.preload_entrypoints` | List of `{pack_ref: <pack ID>, id: <preload entrypoint ID> index: int}` | Same as `order.entrypoint`, but for preload entrypoints. |
| `aux.pack_map` | `{<pack ID...>: PackManifest}` | (Auxilary) Struct where each key is a pack ID and the value is the respective pack manifest for that pack ID. |
| `aux.impl_map` | `{<pack ID...>: {<abstract ID...>: PackManifest}}` | (Auxilary) Struct where the key-path `<pack ID>.<abstract ID>` contains the pack manifest of the pack that implements the respective abstract interface. |

## World Data
**World data**  is a struct at NBT storage location `slimecore:data` `world` containing other information that is not validated by the rebuild process. \
It is updated *every reload* and has the following keys:
| Key | Type | Description |
| --- | --- | --- |
| `installed` | List of `{pack: PackManifest, disabled: boolean}` | All installed packs that SlimeCore is tracking, in arbitrary order, with `disabled` indicating disabled status. |
| `safe_mode.enabled` | `boolean` | Whether or not [safe mode](#safe-mode) is currently enabled. |
| `safe_mode.calls` | List of `{pack_ref: <pack ID>}` | Packs that had their safe-mode tag called on load if safe mode is enabled. |
| `aux.installed_map` | `{<pack ID...>: {pack: PackManifest, disabled: boolean}}` | (Auxilary) Struct where each key is a pack ID and the value is the respective pack's entry in `installed`. |
