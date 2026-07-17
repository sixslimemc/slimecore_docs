# Key Concepts

- [Manifests](#manifests)
- [Rebuilding](#rebuilding)
- [Managing Datapacks (Explicit Rebuilding)](#managing-datapacks-explicit-rebuilding)
- [Build Data](#build-data)
- [World Data](#world-data)
- [Uninstalling SlimeCore](#uninstalling-slimecore)

## Manifests

Every SlimeCore-loaded datapack has a **manifest**, which is NBT data specifying information about itself and how SlimeCore should recognize and load it.

Key components of a pack manifest:
- **Pack ID:** A lowercase alphanumeric name that uniquely identifies a datapack within your world. You should generally be able to recognize a datapack by it's pack ID.
- **Author ID:** A lowercase alphanumeric name that represents the author of a datapack.
- **Version:** A [SemVer](https://semver.org/) adhering version (`<major>.<minor>.<patch>`).
- **Dependencies:** The other (SlimeCore-loaded) datapacks that a datapack requires in order to load.
- **Abstract Interfaces:** If a datapack *declares* an abstract interface, it can only load if exactly one other datapack that *implements* it.
- **Display Information:** Human-readable information about the datapack as well as URLs to the datapack's author/wiki/versions.

To retrieve a datapack's raw manifest data--given that you know the pack ID--you can do any of the following commands:
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

Rebuilding can *fail*, indicating that there exist incompatibilies, errors, and/or unfulfilled requirements of the currently installed datapacks. Your frontend notify you of when and why a rebuild fails. A full list of rebuild errors and how to fix them can be found [here](./troubleshooting.md#rebuild-errors).

The most important aspect of rebuilding is that SlimeCore will not apply any changes to datapack loading until a rebuild *succeeds*. This means that, in most practical circumstances, datapacks will not be loaded unless they are garunteed to be loaded correctly. When a rebuild succeeds, the world's [build data](#build-data) is updated.

## Managing Datapacks (Explicit Rebuilding)

Under most circumstances, disabling, (re-)enabling, or uninstalling datapacks **MUST** be done via **explicit rebuilding**. Explicit rebuilding allows you to *stage* such changes and will verify that they are valid before applying them. If any changes are invalid (e.g. disabling a datapack that is a dependency of an enabled datapack), no changes will be made.

Unless you are:
- Recovering from a [clean rebuild](./troubleshooting.md#clean-rebuilding)
- Re-enabling a previously uninstalled datapack
- Performing advanced troubleshooting

Using `/datapack` to manage SlimeCore-loaded datapacks is a user error and may cause unexpected behavior.

Your frontend should provide a documented method on initiating an explicit rebuild.

## Datapack Uninstallation

**Uninstalling** a datapack refers to the following process:

1. Explicitly rebuild, where the datapack is staged for uninstall.
2. Verify that the explicit rebuild succeeded.
    - At this point, the datapack is considered "not installed" and is no longer tracked by SlimeCore (disabled and cannot be re-enabled via explicit rebuilding).
3. Remove the datapack from your world files.

This is not only to make sure that the datapack can be safely uninstalled, but also to let the datapack "clean itself up", effectively removing it's prescence from your world.

To re-enable an uninstalled datapack, you must use `/datapack enable`. This is functionally equivalent to installing it for the first time.

## Build Data

**Build data** is a struct at NBT storage location `slimecore:data` `build` containing information about the *currently enabled* datapacks and how they load. \
It is  updated upon *successful rebuild* and has the following keys:
| Key | Type | Description |
| :-- | :-- | :-- |
| `packs` | List of pack manifests | All enabled pack manifests in the order that they are loaded. |
| `order.load` | List of `{pack_ref: <pack ID>, index: int}` | All enabled packs (references) in the order that they are loaded, with each `index` key matching the list index of the respective element. |
| `order.entrypoint` | List of `{pack_ref: <pack ID>, id: <entrypoint ID> index: int}` | Similar to `order.load`, but lists all entrypoints in their calling order. |
| `order.preload_entrypoints` | List of `{pack_ref: <pack ID>, id: <preload entrypoint ID> index: int}` | Same as `order.entrypoint`, but for preload entrypoints. |
| `aux.pack_map` | `{<pack ID...>: PackManifest}` | (Auxilary) Struct where each key is a pack ID and the value is the respective pack manifest for that pack ID. |
| `aux.impl_map` | `{<pack ID...>: {<abstract ID...>: PackManifest}}` | (Auxilary) Struct where the key-path `<pack ID>.<abstract ID>` contains the pack manifest of the pack that implements the respective abstract interface. |

**Build data should be treated as read-only.**

## World Data
**World data**  is a struct at NBT storage location `slimecore:data` `world` containing other information that is not validated by the rebuild process. \
It is updated *every reload* and has the following keys:
| Key | Type | Description |
| :-- | :-- | :-- |
| `installed` | List of `{pack: PackManifest, disabled: boolean}` | All installed packs that SlimeCore is tracking, in arbitrary order, with `disabled` indicating disabled status. |
| `safe_mode` | *(See [Safe Mode](./troubleshooting.md#safe-mode))* | Only present when safe mode is enabled. |
| `aux.installed_map` | `{<pack ID...>: {pack: PackManifest, disabled: boolean}}` | (Auxilary) Struct where each key is a pack ID and the value is the respective pack's entry in `installed`. |

**World data should be treated as read-only.**

## Datapack Paths

SlimeCore expects all SlimeCore-loaded datapacks to have their datapack path (name of datapack in world's `datapacks/` folder) follow one the following formats:
- `<author id>.<pack id>.<major ver>.<minor ver>.<patch ver>.zip` (e.g. `bar.foo.1.2.3.zip`)
- `<author id>.<pack id>.<major ver>.<minor ver>.<patch ver>` (e.g. `bar.foo.1.2.3`)
- `<author id>.<pack id>.zip` (e.g. `bar.foo.zip`; *intended for packs under active development*)
- `<author id>.<pack id>` (e.g. `bar.foo; *intended for packs under active development*`)
- `<pack id>` (e.g. `foo`; *intended for packs under active development*)

## Uninstalling SlimeCore

To uninstall SlimeCore itself, run `/function slimecore:-/uninstall_slimecore {args:{}}`.

By default, this will send a you confirmation message before uninstallation starts. You can skip the confirmation message with `/function slimecore:-/uninstall_slimecore {args:{force:true}}`.

Uninstalling SlimeCore will disable all SlimeCore-loaded packs and render them non-functional until SlimeCore is installed again. If SlimeCore is re-installed, those disabled packs must be re-enabled manually with `/datapack enable`.

---

**Next:** [Troubleshooting](./troubleshooting.md)