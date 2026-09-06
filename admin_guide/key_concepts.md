# Key Concepts

- [Frontend Datapacks](#frontend-datapacks)
- [Manifests](#manifests)
- [Rebuilding](#rebuilding)
- [Managing Datapacks (Explicit Rebuilding)](#managing-datapacks-explicit-rebuilding)
- [Build Data](#build-data)
- [World Data](#world-data)
- [Uninstalling SlimeCore](#uninstalling-slimecore)
- [In-Game Help](#in-game-help)

## Frontend Datapacks

In all practical cases, you should install a **frontend** datapack (of your choice) alongside SlimeCore. A frontend datapack is responsible for providing an in-game interface to SlimeCore that SlimeCore alone does not provide. Frontends are developed and documented independently, like any other datapack.

*It is advised to make sure your frontend datapack and its dependencies (if any) are properly loaded before adding any other SlimeCore-loaded datapack to your world. This is to prevent cases where other SlimeCore-loaded datapacks cause rebuild errors and your frontend is not loaded to notify you of them.*

### SCDev

[SCDev](https://github.com/sixslimemc/scdev) is a chat-based frontend made by the author of SlimeCore; it has no dependencies and is designed to be accessible for all users.

> While SlimeCore is still in an early-adoption phase, there may not be a selection of frontend datapacks. If you find the current selection to be insufficient, it is not too difficult to [create your own](../interface_guide/index.md).

## Manifests

Every SlimeCore-loaded datapack has a **manifest**, which is NBT data specifying information about itself and how SlimeCore should recognize and load it.

Key components of a pack manifest:
- **Pack ID:** A lowercase alphanumeric name that uniquely identifies the datapack within your world. You should generally be able to recognize a datapack by its pack ID.
- **Author ID:** A lowercase alphanumeric name that represents the author of the datapack.
- **Version:** A [SemVer](https://semver.org/) adhering version (`<major>.<minor>.<patch>`).
- **Dependencies:** The other (SlimeCore-loaded) datapacks that the datapack requires in order to load.
- **Contracts:** If a datapack *declares* a contract, it can only load if exactly one other enabled *satisfies* it.
- **Display Information:** Human-readable information about the datapack as well as URLs to the datapack's author/wiki/versions.

*For information on obtaining manifest data of datapacks, see [this section](./troubleshooting.md#getting-manifest-data).*

## Rebuilding

Upon world reload, SlimeCore will **rebuild** if it detects any new and/or changed datapacks since last reload. A world reload that triggers a rebuild will often take *significantly* longer (depending on the number of datapacks installed) than world reloads that do not trigger a rebuild.

Rebuilding can *fail*, indicating that there exist incompatibilities, errors, and/or unfulfilled requirements of the currently installed datapacks. Your frontend should notify you of when and why a rebuild fails. A full list of rebuild errors and how to fix them can be found [here](./troubleshooting.md#rebuild-errors).

The most important aspect of rebuilding is that SlimeCore will not apply any changes to datapack loading until a rebuild *succeeds*. Practically speaking, this means that datapacks will not be loaded unless they are guaranteed to be loaded correctly. When a rebuild succeeds, the world's [build data](#build-data) is updated.

## Managing Datapacks (Explicit Rebuilding)

In nearly all cases, disabling, re-enabling, or uninstalling datapacks **MUST** be done via **explicit rebuilding**. With explicit rebuilding, you *stage* changes to be validated and performed all-at-once (opposed to directly and one-by-one with `/datapack`). If your changes would result in an invalid datapack loading state (e.g. you disabled a datapack that is a dependency of an enabled datapack), no changes will be made.

Unless you are:
- Recovering from a [clean rebuild](./troubleshooting.md#clean-rebuilding)
- Re-enabling a previously uninstalled datapack (effectively "reinstalling" it)
- Performing advanced troubleshooting

Using `/datapack` to manage SlimeCore-loaded datapacks is a user error and may cause unexpected behavior.

Your frontend should provide a documented method on initiating an explicit rebuild.

## Datapack Uninstallation

**Uninstalling** a datapack refers to the following process:

1. Explicitly rebuild, where the datapack is staged for uninstall.
2. Verify that the explicit rebuild succeeded.
    - At this point, the datapack is considered "not installed" and is no longer tracked by SlimeCore (disabled and cannot be re-enabled via explicit rebuilding).
3. Remove the datapack from your world files.

Explicitly rebuilding for uninstall not only ensures that your datapack can be safely uninstalled, but also lets the datapack "clean itself up", as to not leave any artifacts or breadcrumbs in your world.

To re-enable an uninstalled datapack, you must use `/datapack enable`. This is functionally equivalent to installing it for the first time.

## Build Data

**Build data** is a struct at NBT storage location `slimecore:data` `build` containing information about the *currently enabled* datapacks and how they load. \
It is updated upon *successful rebuild* and has the following keys:
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
**World data** is a struct at NBT storage location `slimecore:data` `world` containing other information that is not validated by the rebuild process. \
It is updated *every reload* and has the following keys:
| Key | Type | Description |
| :-- | :-- | :-- |
| `installed` | List of `{pack: PackManifest, disabled: boolean}` | All installed packs that SlimeCore is tracking, in arbitrary order, with `disabled` indicating disabled status. |
| `safe_mode` | *(See [Safe Mode](./troubleshooting.md#safe-mode))* | Only present when safe mode is enabled. |
| `aux.installed_map` | `{<pack ID...>: {pack: PackManifest, disabled: boolean}}` | (Auxilary) Struct where each key is a pack ID and the value is the respective pack's entry in `installed`. |

**World data should be treated as read-only.**

## Datapack Paths

SlimeCore expects all SlimeCore-loaded datapacks to have their datapack path (name of file/folder in world's `datapacks/` folder) be in one the following standard formats:
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip` (e.g. `bar.foo.1.2.3.zip`)
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>` (e.g. `bar.foo.1.2.3`)

The following formats are also supported, but are intended for datapacks in active development:
- `<author ID>.<pack ID>.zip` (e.g. `bar.foo.zip`)
- `<author ID>.<pack ID>` (e.g. `bar.foo`)
- `<pack ID>` (e.g. `foo`)

When you download a SlimeCore-loaded datapack, it will likely already match one of these formats. If not, you should rename it so it does before installation. See [Getting Manifest Data](./troubleshooting.md#getting-manifest-data) for getting the relevant information (`author ID`, `pack ID`, etc.).

If any installed SlimeCore-loaded datapack does not have a standard path (and is not overridden, see below), rebuilding will fail with a [Missing Datapack Path(s)](./troubleshooting.md#missing-datapack-paths) error.

### Path Overriding

If a datapack does not have a standard path and for whatever reason you cannot or do not want to rename it, you can manually link a datapack to its path in-game by modifying storage NBT `slimecore:config` `datapack_path_overrides`.

```mcfunction
# add an override:
# <path> should be identical to how you would specify the datapack with `/datapack`, i.e. "file/<datapack name>"
data modify storage slimecore:config datapack_path_overrides.<pack ID> set value <path>

# remove an override:
data remove storage slimecore:config datapack_path_overrides.<pack ID>
```

## Uninstalling SlimeCore

To uninstall SlimeCore itself, run `/function slimecore:-/uninstall_slimecore {args:{}}`.

By default, this will send you a confirmation message before uninstallation starts. You can skip the confirmation message with `/function slimecore:-/uninstall_slimecore {args:{force:true}}`.

Uninstalling SlimeCore will disable all SlimeCore-loaded packs and render them non-functional until SlimeCore is installed again. If SlimeCore is re-installed, those disabled packs must be re-enabled manually with `/datapack enable`.

## In-Game Help

You can run `/function slimecore:-/help` to receive a clickable link to the SlimeCore docs as a chat message.

---

**Next:** [Troubleshooting](./troubleshooting.md)