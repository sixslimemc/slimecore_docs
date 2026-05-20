# Full Dev Guide

This guide is from the perspective of creating a new datapack, but should provide enough information to convert a non-SlimeCore-loaded datapack into a SlimeCore-loaded datapack. If you are converting a datapack, it is highly advised to make a backup beforeso.

- [Setup](#setup)
- [Load Tag](#load-tag)
- [Disable Tag](#disable-tag)
- [Uninstall Tag](#uninstall-tag)
- [Safe Mode Tag](#safe-mode)
- [Dependencies](#dependencies)
- [Entrypoints](#entrypoints)
- [Abstract Interfaces](#abstract-interfaces)
- [The Manifest](#the-manifest)
- [ID Naming](#id-naming)

## Setup

### Pack ID

Designate a namespace (`<datapack>/data/<namespace>`) as your datapack's **pack ID**. This namespace must be the only namespace that your datapack defines ***new*** files in.

All other namespaces included in your datapack are considered *secondary namespaces* (e.g. `minecraft`, `slimecore`, pack IDs of your datapack's dependencies). Your datapack must not define any new files within secondary namespaces, but may *overwrite/modify* files in them (e.g. appending to a tag).

For full pack ID naming requirements, see [this section](#pack-ids).

### Function Tags

Create the following function tags:
```
<datapack>
└── data
    ├── slimecore/tags/function
    │   └── manifest.json
    └── <pack ID>/tags/function
        ├── disable.json
        ├── load.json
        ├── safe_mode.json
        └── uninstall.json
```

Your datapack must not include `#minecraft:load` or `#minecraft:tick`.

*If you are converting a datapack, move the contents of `#minecraft:load` to `#<pack ID>:load`, and `#minecraft:tick` to `#<pack ID>:entrypoint/tick` (create a new tag).*

## Load Tag

The `#minecraft:load` function tag is conceptually replaced by `#<pack ID>:load`--`#<pack ID:load>` will be called upon every world reload and should be used to initialize your datapack.

Importantly, `#<pack ID>:load` should *only* do work related to initialization (declaring scoreboards, initializing data, etc.); it should not do anything else such as start `/schedule` loops or other independent work--this type of work is what [entrypoints](#entrypoints) are for.

## Disable Tag

The `#<pack ID>:disable` function tag is called just before your datapack is disabled, but not uninstalled.

When this tag is called, your datapack should attempt to cleanly stop operation with the consideration that it may be re-enabled again in the future, ideally "continuing" where it left off.

*There is no `#<pack ID>:enable` tag; when a datapack is re-enabled, it's `#<pack ID>:load` function is called like normal.*

## Uninstall Tag

The `#<pack ID>:uninstall` function tag defines your datapack's uninstallation process.

When this tag is called, your datapack should attempt to cleanly remove itself from the world with the assumption that it will never be enabled again, ideally leaving no trace that it was ever installed.

It is a baseline expectation that "pure data" elements of your datapack (scoreboards, NBT storage, entity tags, etc.) are removed entirely. The handling of in-world elements (entities, blocks, items, etc.) is to your discretion.

If a datapack is uninstalled while disabled, it will be temporarily re-enabled to call `#<pack ID>:uninstall`, but `#<pack ID>:load` will not be called beforehand.

## Safe Mode Tag

The `#<pack ID>:safe_mode` function tag is called instead of `#<pack ID>:load` on world reload if there is another installed datapack with the same pack ID as your datapack.

When this tag is called, your datapack should attempt to temporarily enter a state of reduced functionality where references/calls to resources under it's pack ID are minimized until `#<pack ID>:load` is called. I.e. You should minimize the chance of referencing/calling resources that have been unintentionally overwritten by the other datapack that has the same pack ID.

When safe mode is over, `#<pack ID>:load` will be called like normal and your datapack should return to it's fully functional state. While safe mode should be accounted for, it is important that your datapack continues to function as expected after safe mode ends.

Note that `#<pack ID>:safe_mode` may be called before `#<pack ID>:load` is ever called; this indicates that the user just installed your datapack, and the world already has a datapack with it's pack ID.

## Dependencies

Your datapack can declare that it requires, or optionally supports, other SlimeCore-loaded datapacks. These required/supported datapacks are **dependencies** of your datapack.

SlimeCore will ensure that all dependencies will be loaded **before** your datapack, and that all required dependencies are installed before your datapack loads at all.

If your datapack references **any** part of another SlimeCore-loaded datapack, it should be declared as a dependency.

Dependencies are declared in a datapack's [manifest](#the-manifest).

### Versioning

All SlimeCore-loaded datapacks have a [SemVer](https://semver.org) adhering version (`<major>.<minor>.<patch>`). When declaring a dependency, a *version requirement* (`<req_major>.<req_minor>`) must be specified with it.

An installed dependency fulfills the version requirement if all of these conditions are met:
- `<major>` == `<req_major>`
- *if `<major>` == 0:*
    - `<minor>` == `<req_minor>`
- *if `<major>` > 0:*
    - `<minor>` >= `<req_minor>`

If an installed dependency datapack does not fulfill the version requirement, the dependency is not considered fulfilled and the dependent datapack will not load.

## Entrypoints

Entrypoints are function tags that are called on world reload **after all** datapacks' `#<pack ID>:load` tags are called. They should be used to run/start independent, non-initialization work. A datapack can define any number of entrypoints.

Entrypoints can and should be used to replace `#minecraft:tick` by one or more entrypoints that contain functions that schedule themselves (`schedule <self> 1t`). If your datapack does multiple conceptually independent "chunks" of work in its tick loop, it's good practice to split them up into their own entrypoints. This gives datapacks that may depend on yours more to work with (explained below).

A key advantage of entrypoints is that they can be explicitly ordered relative to dependencies' entrypoints. For instance, If datapack A defines entrypoint `foo`, and datapack B depends on datapack A, any of datapack B's entrypoints can be specified to explicitly run before OR after entrypoint `foo`.

Entrypoints match the function tag format `#<pack ID>/entrypoint/<entrypoint ID>`, and are declared in a datapack's [manifest](#the-manifest).

### Preload Entrypoints

*Preload entrypoints are only applicable to a small minority of datapacks.*

Preload entrypoints are similar to entrypoints but are called **before any** datapacks' `#<pack ID>:load` tags are called (including their own). They should generally be reserved for meta/technical/pre-initialization work and shold NOT be used to start `/schedule` loops.

Similarly to entrypoints, preload entrypoints can be explicitly ordered relative to dependencies' preload entrypoints.

Preload entrypoints match the function tag format `#<pack ID>/preload_entrypoint/<preload entrypoint ID>`, and are declared in a datapack's [manifest](#the-manifest).

## Abstract Interfaces

*Abstract interfaces are only applicable to a small minority of datapacks.*

Abstract interfaces represent "contracts" that are presented by one datapack, and must be fulfilled/implemented by another. The terms of said "contracts" are to be documented/explained by the author of the presenting datapack, abstract interfaces only *represent* them. Concretely, for every abstract interface that a datapack *declares*, **exactly one** other datapack must specify that it *implements* it.

Practically, an abstract interface should be declared when a datapack defines an API over some behavior, but does not actually implement that behavior--*delegating* the implementation to an external datapack (that the user chooses). It is the responsibility of the author to document/explain proper implementation of the behavior.

Likewise, an datapack should specify that it implements an abstract interface if it does properly implement the behavior documented by the declaring datapack. It is the responsibility of the author of the implementing datapack to ensure proper implementation.

Abstract interface declarations/implementations are declared in a datapack's [manifest](#the-manifest).

## The Manifest

A datapack's **manifest** contains all the information about how it should be recognized and loaded--the *identity* of the datapack. \
Datapacks define their manifest, SlimeCore does the rest.

In your datapack, you must append a single function to the function tag `#slimecore:manifest`. This function is referred to as the *manifest function* and must call the function `slimecore:api/manifest` with proper inputs (defining the manifest).

Manifests have the following components:
- [`pack_id`](#pack_id)
- [`author_id`](#author_id)
- [`version`](#version)
- [`is_library`](#is_library)
- [`dependencies`](#dependencies-1)
- [`entrypoints`](#entrypoints-1)
- [`preload_entrypoints`](#preload_entrypoints)
- [`abstract_declarations`](#abstract_declarations)
- [`abstract_implementations`](#abstract_implementations)
- [`display`](#display)
- [`url`](#url)

*Minimal manifest function template:*

```mcfunction
data modify storage slimecore:in manifest.pack.pack_id set value "PACK_ID"
data modify storage slimecore:in manifest.pack.author_id set value "AUTHOR_ID"
data modify storage slimecore:in manifest.pack.version set value {major:1, minor:0, patch:0}
data modify storage slimecore:in manifest.pack.is_library set value false

data modify storage slimecore:in manifest.pack.dependencies set value []

data modify storage slimecore:in manifest.pack.entrypoints set value []
data modify storage slimecore:in manifest.pack.preload_entrypoints set value []

data modify storage slimecore:in manifest.pack.abstract_declarations set value []
data modify storage slimecore:in manifest.pack.abstract_implementations set value []

data modify storage slimecore:in manifest.pack.display.name set value "DISPLAY_NAME"
data modify storage slimecore:in manifest.pack.display.summary set value "DISPLAY_SUMMARY"
data modify storage slimecore:in manifest.pack.display.author_name set value "DISPLAY_AUTHOR_NAME"

# data modify storage slimecore:in manifest.pack.display.links.author set value "AUTHOR_URL"
# data modify storage slimecore:in manifest.pack.display.links.info set value "INFO_URL"
# data modify storage slimecore:in manifest.pack.display.links.versions set value "RELEASES_URL"

data modify storage slimecore:in manifest.pack.url set value "DOWNLOAD_URL"

function slimecore:api/manifest
```

### `pack_id`

**Type:** `string`

Must exactly match your datapack's [pack ID](#pack-id).

### `author_id`

**Type:** `string`

An arbitrary identifier that represents you as a datapack author. \
See [this section](#author-ids) for requirements.

### `version`

**Type:** `struct`

The numerical [SemVer](https://semver.org) version of your datapack.

| Key | Type | Description |
| :-- | :-- | :-- |
| `major` | int | Major version. |
| `minor` | int | Minor version. |
| `patch` | int | Patch version. |

### `is_library`

**Type:** `boolean`

Whether or not your datapack is a library.

A library is a datapack that is meant to be used exclusively by other datapacks and does not make a meaningful impact on the user's world on it's own. 

### `dependencies`

**Type:** `list<struct>`

Declares your datapack's [dependencies](#dependencies)--each element represents one dependency.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | `string` | [Pack ID](#pack_id) of the dependency. |
| `author_id` | `string` | [Author ID](#author_id) of the dependency. |
| `version` | `{major: int, minor: int}` | [Version requirement](#versioning) of the dependency. |
| `optional` | `boolean` | If `true`, your datapack should be designed to work both with and without the dependency; SlimeCore will not require it to be installed. |
| `download.url` | `URL string` | [Direct download URL](#url) of any compatible version of the dependency. |
| `download.version` | `{major: int, minor: int, patch: int}` | Exact [version](#version) of the dependency that `download.url` downloads. |

Specifying a pack as a dependency allows it to be referenced via `pack_ref` in other manifest components.

### `entrypoints`

**Type:** `list<struct>`

Declares your datapack's [entrypoints](#entrypoints)--each element represents one entrypoint.

In addition to respecting explicit `before`/`after` ordering, entrypoints will always be called in the order that they are specified in this list.

| Key | Type | Description | Default Value |
| :-- | :-- | :-- |
| `id` | `string` | The ID of the entrypoint. See [this section](#manifest-ids) for naming requirements. | *(required)* |
| `after` | `list<{pack_ref: string, id: string}>` | List of dependencies' entrypoints that the entrypoint must be called *after*. `pack_ref` is the dependency's pack ID, `id` is the ID of the referenced entrypoint. | `[]` |
| `before` | `list<{pack_ref: string, id: string}>` | List of dependencies' entrypoints that the entrypoint must be called *before*. `pack_ref` is the dependency's pack ID, `id` is the ID of the referenced entrypoint. | `[]` |

### `preload_entrypoints`

**Type:** `list<struct>`

Functions identically to [`entrypoints`](#entrypoints-1), but handles [preload entrypoints](#preload-entrypoints).

### `abstract_declarations`

**Type:** `list<struct>`

Declares your datapack's [abstract interfaces](#abstract-interfaces)--each element represents one abstract interface.

| Key | Type | Description | Default Value |
| :-- | :-- | :-- |
| `id` | `string` | The ID of the abstract interface. See [this section](#manifest-ids) for naming requirements. | *(required)* |

### `abstract_implementations`

**Type:** `list<struct>`

Specifies the [abstract interfaces](#abstract-interfaces) that your datapack implements--each element represents one abstract interface implementation.

| Key | Type | Description | Default Value |
| :-- | :-- | :-- |
| `pack_ref` | `string` | The pack ID that the implemented abstract interface is from. |
| `id` | `string` | The ID of the implemented abstract interface.  |

### `display`

**Type:** `struct`

Specifies your datapack's display information and URLs. This information is not used by SlimeCore itself but may be used by frontends and such to present your datapack nicely.

| Key | Type | Description |
| :-- | :-- | :-- |
| `name` | `string` | The display name of your datapack. |
| `author_name` | `string` | Your display name as a datapack author. |
| `summary` | `string` | 1-2 sentence-length summary/description of your datapack. Ideally, should match `pack.description` of your datapack's `pack.mcmeta` file. |
| `links` | `struct` | *(See below)* |

`name`, `author_name`, and `summary` should not contain any escape sequences such as `/n` or `/t`.

`links` is optional and can contain the following optional keys:
| Key | Type | Description |
| :-- | :-- | :-- |
| `info` | `URL string` | website URL where users can find more information or a wiki/docs for this datapack (e.g. main GitHub repo, Modrinth page) |
| `releases` | `URL string` | website URL where users can find more released versions of this datapack (e.g. GitHub releases tab, Modrinth versions tab)
| `author` | `URL string` | website URL that represents you as a datapack author (e.g. GitHub, Modrinth, personal site) |


### `url`

## ID Naming

### Pack IDs

Pack IDs **MUST**:
- be a valid datapack [namespace](https://minecraft.wiki/w/Identifier#Namespaces)
- be 1-64 characters long
- only contain lowercase letters, numbers, `_`, and `-`
- not be `minecraft` or `slimecore`

Generally, pack IDs **SHOULD**:
- be 3-32 characters long
- not start with `_` or `-`
- use `-` as a module separator \
(e.g. `foo-bar` and `foo-baz` are modules of group `foo`.)
- *if for a [library](#is_library) datapack:*
    - use `_` conservatively
    - be easy-to-type and unique \
    (e.g. `herobrinesmathlibrary` is not easy to type, `math` is too generic, `brinemath` is easy to type and reasonably unique.)
- *if for a non-[library](#is_library)/content datapack:*
    - be at least 6 characters long
    - use `_` to represent spaces
    - be reasonably descriptive \
     (e.g. `hpicks` is not descriptive and may clash with other pack IDs, `herobrines_pickaxes` is descriptive and not too long.)

### Author IDs

Author IDs **MUST**:
- be 1-64 characters long
- only contain lowercase letters, numbers, and `_`

Generally, author IDs **SHOULD**
- match your (lowercased) name on your primary authoring platform (GitHub, Modrinth, etc.) or in-game name
- stay consistent between your authored datapacks

### Manifest IDs

Entrypoint/preload-entrypoint IDs and abstract interface IDs **MUST**:
- be 1-32 characters long
- only contain lowercase letters, numbers, and `_`

Generally, these IDs **SHOULD**:
- be at least 3 characters long
- use `_` to represent spaces
- be reasonably descriptive
- not be shared between elements of different types \
(e.g. you should not declare a preload entrypoint and entrypoint with the same IDs)
