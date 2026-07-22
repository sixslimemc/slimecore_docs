# Full Dev Guide

If converting an existing datapack, make a backup before making any changes.

- [Setup](#setup)
- [Load Tag](#load-tag)
- [Disable Tag](#disable-tag)
- [Uninstall Tag](#uninstall-tag)
- [Safe Mode Tag](#safe-mode)
- [Dependencies](#dependencies)
- [Entrypoints](#entrypoints)
- [Abstract Interfaces](#abstract-interfaces)
- [The Manifest](#the-manifest)
- [Standard Datapack Naming](#standard-datapack-naming)
- [ID Naming](#id-naming)

## Setup

### Pack ID

Designate a namespace (`<datapack>/data/<namespace>`) as your datapack's **pack ID**. This namespace **MUST** be the only namespace that your datapack defines ***new*** files in.

All other namespaces included in your datapack are considered *secondary namespaces* (e.g. `minecraft`, `slimecore`, pack IDs of your datapack's dependencies). Your datapack must not define any new files within secondary namespaces, but may intentionally *overwrite/modify* files in them (e.g. appending to a tag).

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

*If you are converting a datapack, move the contents of `#minecraft:load` to `#<pack ID>:load`, and `#minecraft:tick` to `#<pack ID>:entrypoint/main` (create a new tag, see [Entrypoints](#entrypoints) for info).*

## Load Tag

The `#<pack ID>:load` function tag is called upon every world reload and conceptually replaces `#minecraft:load`.

When this tag is called, your datapack should initialize itself. Importantly, it should *only* do work related to initialization (declaring scoreboards, initializing data, etc.); it should not do anything else such as start `/schedule` loops or other independent work--this type of work is what [entrypoints](#entrypoints) are for.

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

The `#<pack ID>:safe_mode` function tag is called instead of `#<pack ID>:load` on world reload if SlimeCore detects that your datapack (or any of it's dependencies) may be loaded incorrectly *(See [Safe Mode](../admin_guide/troubleshooting.md#safe-mode) for more information)*.

When this tag is called, your datapack should attempt to minimize all calls/references to *any* resource until safe mode is over. This may includes stopping `/schedule` loops, safeguarding advancement reward functions, etc. Informally, you should assume that, while safe mode is enabled, every file in your datapack (and its dependencies) has a chance to cause undefined behavior when referenced.

When safe mode is over, `#<pack ID>:load` will be called like normal and your datapack should return to it's fully functional state. While safe mode should be accounted for, it is more important that your datapack continues to function as expected after safe mode ends.

Note that `#<pack ID>:safe_mode` may be called before `#<pack ID>:load` is ever called; this indicates that the user just installed your datapack and safe mode triggered immediately.

## Dependencies

Your datapack can declare that it requires, or optionally supports, other SlimeCore-loaded datapacks. These required/supported datapacks are **dependencies** of your datapack.

SlimeCore will ensure that all dependencies will be loaded **before** your datapack, and that all required dependencies are installed before your datapack loads at all.

If your datapack references or uses **any** resource/feature of another SlimeCore-loaded datapack, it should be declared as a dependency.

Dependencies are declared in a datapack's [manifest](#the-manifest).

### Versioning

All SlimeCore-loaded datapacks have a [SemVer](https://semver.org) adhering version (`<major>.<minor>.<patch>`). When declaring a dependency, a *version requirement* (`<req_major>.<req_minor>`) must be specified with it.

An installed dependency fulfills the version requirement if all of these conditions are met:
- `<major>` == `<req_major>`
- *if `<major>` == 0:*
    - `<minor>` == `<req_minor>`
- *if `<major>` > 0:*
    - `<minor>` >= `<req_minor>`

If an installed dependency datapack does not fulfill the version requirement, the dependency considered unfulfilled and the dependent datapack will not load.

## Entrypoints

Entrypoints are function tags matching the format `#<pack ID>/entrypoint/<entrypoint ID>` and are called on world reload after all datapacks' `#<pack ID>:load` tags are called. They should be used to run/start independent, non-initialization work. A datapack can define any number of entrypoints.

Entrypoints can and should be used to replace `#minecraft:tick` using self-scheduling functions (`schedule <self> 1t`). If your datapack does multiple conceptually independent "chunks" of work in its tick loop, it's good practice to split them up into their own entrypoints. This gives datapacks that may depend on yours more to work with (explained below).

A key advantage of entrypoints is that they can be explicitly ordered against dependencies' entrypoints. For instance, If datapack A defines entrypoint `foo`, and datapack B depends on datapack A, any of datapack B's entrypoints can be specified to explicitly run before OR after entrypoint `foo`.

Entrypoints are declared in a datapack's [manifest](#the-manifest).

### Preload Entrypoints

*Preload entrypoints are only applicable to a small minority of datapacks.*

Preload entrypoints are function tags matching the format `#<pack ID>/preload_entrypoint/<preload entrypoint ID>` and are similar to entrypoints, but are called **before any** datapacks' `#<pack ID>:load` tags are called (including their own). They should generally be reserved for meta/pre-initialization work and should generally not be used to start `/schedule` loops.

Similarly to entrypoints, preload entrypoints can be explicitly ordered against dependencies' preload entrypoints.

Preload entrypoints are declared in a datapack's [manifest](#the-manifest).

## Abstract Interfaces

*Abstract interfaces are only applicable to a small minority of datapacks.*

Abstract interfaces represent "contracts" that are declared by one datapack, and must be fulfilled/implemented by another. The terms of said "contracts" are to be documented/explained by the author of the declaring datapack, abstract interfaces only *represent* them. Concretely, for every abstract interface that a datapack *declares*, **exactly one** other datapack must specify that it *implements* it.

Practically, an abstract interface should be declared when a datapack defines an API over some behavior, but does not actually implement that behavior--*delegating* the implementation to an external datapack (that the user chooses). It is the responsibility of the author to document/explain proper implementation of the behavior.

Likewise, a datapack should specify that it implements a given abstract interface if it properly implements the behavior/contract documented by the declaring datapack. It is the responsibility of the author of the implementing datapack to ensure proper implementation.

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
- [`loader_version`](#loader_version)

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

data modify storage slimecore:in manifest.loader_version set value {major:0, minor:3}

function slimecore:api/manifest
```

### `pack_id`

**Type:** `string`

Must exactly match your datapack's [pack ID](#pack-id).

### `author_id`

**Type:** `string`

An arbitrary identifier that represents you as a datapack author. *See [ID Naming](#author-ids).*

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

A library is a datapack that is intended to be used *exclusively* as a dependency of other datapacks and does not provide any meaningful behavior on its own.

### `dependencies`

**Type:** `list<struct>`

Declares your datapack's [dependencies](#dependencies)--each element represents one dependency.

Specifying a pack as a dependency allows it to be referenced via `pack_ref` in other manifest components.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | `string` | [Pack ID](#pack_id) of the dependency. |
| `author_id` | `string` | [Author ID](#author_id) of the dependency. |
| `version` | `{major: int, minor: int}` | [Version requirement](#versioning) of the dependency. |
| `optional` | `boolean` | If `true`, your datapack should be designed to work both with and without the dependency; SlimeCore will not require it to be installed. |
| `download.url` | `URL string` | [Direct download URL](#url) of any compatible version of the dependency. |
| `download.version` | `{major: int, minor: int, patch: int}` | Exact [version](#version) of the dependency that `download.url` downloads. |

### `entrypoints`

**Type:** `list<struct>`

Declares your datapack's [entrypoints](#entrypoints)--each element represents one entrypoint.

In addition to respecting explicit `before`/`after` ordering, entrypoints will always be called in the order that they are specified in this list.

Each element must have the following keys:

| Key | Type | Description | Default Value |
| :-- | :-- | :-- | :-- |
| `id` | `string` | The ID of the entrypoint. *See [ID Naming](#manifest-ids).* | *(required)* |
| `after` | `list<{pack_ref: string, id: string}>` | List of dependencies' entrypoints that the entrypoint must be called *after*. `pack_ref` is the dependency's pack ID, `id` is the ID of the referenced entrypoint. | `[]` |
| `before` | `list<{pack_ref: string, id: string}>` | List of dependencies' entrypoints that the entrypoint must be called *before*. `pack_ref` is the dependency's pack ID, `id` is the ID of the referenced entrypoint. | `[]` |

### `preload_entrypoints`

**Type:** `list<struct>`

Declares your datapack's [preload entrypoints](#preload-entrypoints).

*Format is identical to [`entrypoints`](#entrypoints-1).*

### `abstract_declarations`

**Type:** `list<struct>`

Declares your datapack's [abstract interfaces](#abstract-interfaces)--each element represents one abstract interface.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `id` | `string` | The ID of the abstract interface. *See [ID Naming](#manifest-ids).* |

### `abstract_implementations`

**Type:** `list<struct>`

Specifies the [abstract interfaces](#abstract-interfaces) that your datapack implements--each element represents one abstract interface implementation.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_ref` | `string` | The pack ID that the implemented abstract interface is from. |
| `id` | `string` | The ID of the implemented abstract interface.  |

### `display`

**Type:** `struct`

Specifies your datapack's display information and URLs. This information is not used by SlimeCore itself but may be used by frontends and such to present your datapack nicely.

| Key | Type | Description | Default Value |
| :-- | :-- | :-- | :-- |
| `name` | `string` | The display name of your datapack. | *(required)* |
| `author_name` | `string` | Your display name as a datapack author. | *(required)* |
| `summary` | `string` | 1-2 sentence-length summary/description of your datapack. Ideally, should match `pack.description` of your datapack's `pack.mcmeta` file. | *(required)* |
| `links` | `struct` | *(See below)* | *(none)* |

`name`, `author_name`, and `summary` should not contain any escape sequences such as `/n` or `/t`.

`links` is optional and can contain the following optional keys:

| Key | Type | Description | Default Value |
| :-- | :-- | :-- | :-- |
| `info` | `URL string` | website URL where users can find more information or a wiki/docs for this datapack (e.g. main GitHub repo, Modrinth page) | *(none)* |
| `releases` | `URL string` | website URL where users can find more released versions of this datapack (e.g. GitHub releases tab, Modrinth versions tab) | *(none)* |
| `author` | `URL string` | website URL that represents you as a datapack author (e.g. GitHub, Modrinth, personal site) | *(none)* |

### `url`

**Type:** `URL string`

A direct download/source URL to the exact version of this datapack as a .zip file. The downloaded .zip file should *be* the datapack, it should not *contain* the datapack (`pack.mcmeta` should be immediately visible when opening the .zip). The name of the downloaded .zip should follow [standard datapack naming](#standard-datapack-naming).

Opening this URL in a browser should immediately download the .zip file; there should be no required user input, timers, or redirects.

For example, if user `bar` released their datapack `foo` version `1.2.3` on GitHub, their `url` would likely be `https://github.com/bar/foo/releases/download/v1.2.3/bar.foo.1.2.3.zip`.

*It is acknowledged that providing a valid value for `url` requires some amount of foresight, as you must know the direct download URL to your datapack before you release it for download.*

> TODO: provide steps on how to retrieve GitHub and Modrinth direct download links.

### `loader_version`

**Type:** `{major: int, minor: int}`

The version(s) of SlimeCore that can load this datapack, as a [version requirement](#versioning).

*You can retrieve the installed SlimeCore version in-game via `data get storage slimecore:data slimecore.version`.*

## Standard Datapack Naming

SlimeCore expects datapack names/paths to match a specific format; see [this section](../admin_guide/key_concepts.md#datapack-paths) for more information.

When releasing your datapack for download to the public, it's name should match one of the fully qualified standard formats:
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip` (e.g. `bar.foo.1.2.3.zip`)
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>` (e.g. `bar.foo.1.2.3`)

Other standard name formats are supported for development convenience and should not be part of public releases:
- `<author ID>.<pack ID>.zip` (e.g. `bar.foo.zip`)
- `<author ID>.<pack ID>` (e.g. `bar.foo`)
- `<pack ID>` (e.g. `foo`)

## ID Naming

### Pack IDs

Pack IDs **MUST**:
- be a valid datapack [namespace](https://minecraft.wiki/w/Identifier#Namespaces)
- be 1-64 characters long.
- only contain lowercase letters, numbers, `_`, and `-`.
- not be `minecraft` or `slimecore`.

Generally, pack IDs **SHOULD**:
- be 3-32 characters long.
- not start with `_` or `-`.
- use `-` as a module separator. \
(e.g. `foo-bar` and `foo-baz` are modules of group `foo`.)
- *if for a [library](#is_library) datapack:*
    - use `_` conservatively.
    - be easy-to-type and unique. \
    (e.g. `herobrinesmathlibrary` is not easy to type, `math` is too generic, `brinemath` is easy to type and reasonably unique.)
- *if for a non-[library](#is_library)/content datapack:*
    - be at least 6 characters long.
    - use `_` to represent spaces.
    - be reasonably descriptive. \
     (e.g. `hpicks` is not descriptive and may clash with other pack IDs, `herobrines_pickaxes` is descriptive and not too long.)

### Author IDs

Author IDs **MUST**:
- be 1-64 characters long.
- only contain lowercase letters, numbers, and `_`.

Generally, author IDs **SHOULD**
- match your (lowercased) name on your primary authoring platform (GitHub, Modrinth, etc.) or in-game name.
- stay consistent between your authored datapacks.

### Manifest IDs

Entrypoint, preload entrypoint, and abstract interface IDs **MUST**:
- be 1-32 characters long.
- only contain lowercase letters, numbers, and `_`.

Generally, these IDs **SHOULD**:
- be at least 3 characters long.
- use `_` to represent spaces.
- be reasonably descriptive.
- not be shared between elements of different types. \
(e.g. you should not declare a preload entrypoint and entrypoint with the same IDs)

If your pack only has a single [entrypoint](#entrypoints) that acts as a general substitute for `#minecraft:tick`, its ID **SHOULD** be `main`.