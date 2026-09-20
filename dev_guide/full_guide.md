# Full Guide

If converting an existing datapack, make a backup before making any changes.

- [Setup](#setup)
- [Load Tag](#load-tag)
- [Disable Tag](#disable-tag)
- [Uninstall Tag](#uninstall-tag)
- [Safe Mode Tag](#safe-mode-tag)
- [Dependencies](#dependencies)
- [Entrypoints](#entrypoints)
- [Abstract Interfaces](#abstract-interfaces)
- [The Manifest](#the-manifest)
- [Standard Datapack Naming](#standard-datapack-naming)
- [ID Naming](#id-naming)

## Setup

### Pack ID

Designate a namespace (`<datapack>/data/<namespace>`) as your datapack's **pack ID**. This namespace **MUST** be the only namespace that your datapack defines ***new*** files in.

All other namespaces included in your datapack are considered *secondary* namespaces (e.g. `minecraft`, `slimecore`, pack IDs of your datapack's dependencies). Your datapack must not define any new files within secondary namespaces, but may intentionally *overwrite/modify* files in them (e.g. appending to a tag).

For pack ID naming requirements, see [this section](#pack-ids).

### Function Tag Structure

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

*If you are converting a datapack, move the contents of `#minecraft:load` to `#<pack id>:load`, and `#minecraft:tick` to `#<pack id>:entrypoint/main` (create a new tag, see [Entrypoints](#entrypoints) for info).*

## Load Tag

The `#<pack id>:load` function tag is called upon every world reload and conceptually replaces `#minecraft:load`. The order that each datapack's load tag is called matches the actual in-game datapack loading order.

When this tag is called, your datapack should initialize itself. Importantly, it should *only* do work related to initialization (declaring scoreboards, initializing data, etc.); it should not do anything else such as start `/schedule` loops or other independent work--that type of work is what [entrypoints](#entrypoints) are for.

## Disable Tag

The `#<pack id>:disable` function tag is called just before your datapack is disabled.

When this tag is called, your datapack should attempt to cleanly stop operation with the consideration that it may be re-enabled again in the future, ideally "continuing" where it left off.

*There is no "enable" tag; when a datapack is re-enabled, its [load tag](#load-tag) is called like normal.*

## Uninstall Tag

The `#<pack id>:uninstall` function tag defines your datapack's uninstallation process.

When this tag is called, your datapack should attempt to cleanly remove itself from the world with the assumption that it will never be enabled again, ideally leaving no trace that it was ever installed.

It is a baseline expectation that "pure data" elements of your datapack (scoreboards, NBT storage, entity tags, etc.) are removed entirely. The handling of in-world elements (entities, blocks, items, etc.) is to your discretion.

If your datapack is uninstalled while enabled, the [disable tag](#disable-tag) will be called just before the uninstall tag. If the datapack is uninstalled while disabled, it will be ephemerally re-enabled, without execution of its [load tag](#load-tag), to call its uninstall tag. In practice, this means that execution of the uninstall tag will always follow execution of the disable tag without execution of the [load tag](#load-tag) in-between.

## Safe Mode Tag

The `#<pack ID>:safe_mode` function tag is called instead of the [load tag](#load-tag) on world reload if SlimeCore detects that your datapack (or any of its dependencies) may be loaded incorrectly *(See [Safe Mode](../admin_guide/troubleshooting.md#safe-mode))*.

When this tag is called, your datapack should attempt to minimize all calls/references to *any* resource until safe mode is disabled. This most commonly includes stopping `/schedule` loops and safeguarding trigger functions from advancements/enchantments/APIs. Informally, you should assume that every reference to a resource has a chance to cause unexpected behavior during safe mode.

When safe mode is over, the load tag will be called like normal and your datapack should return to its fully functional state.

Note that the safe mode tag may be called before the load tag is ever called; this indicates that the user just installed your datapack and safe mode triggered immediately.

*It is understood that implementing support for safe mode may be a significant development burden, especially for larger datapacks. While supporting safe mode as best as possible is advised, it is not strictly required; you have the option to simply not support safe mode in your datapack. If your datapack does not meaningfully support safe mode, it should indicate such in its documentation as well as when its safe mode tag is called.*

## Entrypoints

Entrypoints are function tags matching the format `#<pack id>/entrypoint/<entrypoint id>` and are called on world reload after *all* datapacks' [load tag](#load-tag) is called. They should be used to run/start independent, non-initialization work. A datapack can define any number of entrypoints. 

While any arbitrary work can be done with entrypoints, their primary purpose is to replace `#minecraft:tick`; this can be done via schedule loops (functions that `/schedule` themselves). Defining a single entrypoint is likely sufficient for most datapacks and is recommended when just starting, but do note the benefits of defining multiple entrypoints, highlighted in [this section](./beneficial_practices.md#entrypoint-separation).

A key advantage of entrypoints is that they can be explicitly ordered against [dependencies'](#dependencies) entrypoints. For instance, if datapack X defines entrypoint `foo`, and depends on datapack Y, which defines entrypoint `bar`: datapack X can specify `foo` to run explicitly before/after `bar`. If both `foo` and `bar` start schedule loops, `foo`'s body will always run before/after `bar`'s body on any given tick.

Entrypoints are declared in a datapack's [manifest](#the-manifest).

*From [ID Naming](#id-naming):*
> If your pack has exactly one entrypoint, a general substitute for `#minecraft:tick`, it is advised to make that entrypoint's ID `main`.

### Preload Entrypoints

Preload entrypoints are function tags matching the format `#<pack id>/preload_entrypoint/<preload entrypoint id>` and are similar to entrypoints, but are called before *any* datapacks' [load tag](#load-tag) is called (including their parent datapack's). They should generally be reserved for advanced/technical work that cannot be done via regular entrypoints.

Similarly to entrypoints, preload entrypoints can be explicitly ordered against dependencies' preload entrypoints.

Preload entrypoints are declared in a datapack's [manifest](#the-manifest).

*Preload entrypoints are likely only applicable to a small minority of datapacks.*

## Contracts

Contracts exist purely in manifest data, and when *declared* by one datapack, must be *satisfied* by exactly one other datapack in the same build. A datapack can declare and/or satisfy any number of contracts.

Practically speaking, contracts represent some documented "terms" that are not satisfied by default, that other datapacks can "agree" to satisfy. You declare a contract in your datapack's manifest, describe the terms of the contract in your datapack's documentation, then any other datapack author can read and implement said terms in their datapack, specifying that they satisfy your contract in their datapack's manifest. It is the responsibility of the developer(s) to actually make sure the terms of the contract are properly described/implemented, the contract itself is just a tool to represent such. 

The primary purpose of contracts is to allow datapacks to enforce external *delegation* of behavior/implementation--similar in concept to an abstract method in programming. A contract does not "care" which datapack satisfies it; only that there exists exactly one datapack in the same build that does.


Contract declarations/satisfactions are declared in a datapack's [manifest](#the-manifest).

*Contracts are likely only applicable to a small minority of datapacks.*

*For more information on contracts, see [this section](./beneficial_practices.md#defining-contracts).*

## Dependencies

Your datapack can declare that it requires and/or optionally supports other SlimeCore-loaded datapacks. These required/supported datapacks are **dependencies** of your datapack. If your datapack uses **any** resources/functionality of another SlimeCore-loaded datapack, it must be declared as a dependency.

SlimeCore ensures that all required dependencies are installed/enabled and that all installed dependencies (both required and optional) are of a compatible version (see below) before your datapack loads at all. Your datapack will always [load](#load-tag) **after** all of its dependencies.

Dependencies are declared in a datapack's [manifest](#the-manifest).

### Version Requirements

All SlimeCore-loaded datapacks have a [version](#version), `<major>.<minor>.<patch>`. When declaring a dependency, a version requirement (`<req_major>.<req_minor>`) must be specified with it.

An installed dependency fulfills the version requirement if all of these conditions are met:
- `<major>` == `<req_major>`
- *if `<major>` > 0:*
    - `<minor>` >= `<req_minor>`
- *if `<major>` == 0:*
    - `<minor>` == `<req_minor>`

*This is a standard version requirement definition under [semantic versioning](https://semver.org).*

If an installed dependency datapack does not fulfill the version requirement, the dependency is considered unfulfilled.

### Checking for Optional Dependencies

Within your datapack, you can check if optional dependencies are enabled via [build data](../admin_guide/key_concepts.md#build-data).

```mcfunction
# checks if pack bar.foo is enabled:
execute if data storage slimecore:data build.packs[{pack_id:"foo", author_id:"bar"}]
# alternatively:
execute if data storage slimecore:data build.aux.installed_map.foo{author_id:"bar"}
```

## The Manifest

A datapack's **manifest** contains all the information about how it should be recognized and loaded--the *identity* of the datapack. Datapacks define their manifest, SlimeCore figures out the rest.

In your datapack, you must append a single function to the function tag `#slimecore:manifest`. This function is referred to as the *manifest function* and must call the function `slimecore:api/manifest` with your datapack's manifest data as input.

Manifests have the following components:
- [`pack_id`](#pack_id)
- [`author_id`](#author_id)
- [`version`](#version)
- [`is_library`](#is_library)
- [`dependencies`](#dependencies-1)
- [`entrypoints`](#entrypoints-1)
- [`preload_entrypoints`](#preload_entrypoints)
- [`contract_declarations`](#contract_declarations)
- [`contracts_satisfied`](#contracts_satisfied)
- [`display`](#display)
- [`url`](#url)
- [`loader_version`](#loader_version)

Here is a minimal manifest function template:

```mcfunction

# identity:
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

The version of your datapack--should adhere to [semantic versioning](https://semver.org).

| Key | Type | Description |
| :-- | :-- | :-- |
| `major` | int | Major version. |
| `minor` | int | Minor version. |
| `patch` | int | Patch version. |

### `is_library`

**Type:** `boolean`

Whether or not your datapack is a library.

A library is a datapack that is intended to be used as a dependency and does not provide any meaningful behavior on its own.

This value is not used by SlimeCore itself but may be used externally (similar to [`display`](#display)).

### `dependencies`

**Type:** `list<struct>`

Declares your datapack's [dependencies](#dependencies)--each element represents one dependency.

Specifying a pack as a dependency allows it to be referenced via `pack_ref` in other components of your manifest.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_id` | `string` | [Pack ID](#pack_id) of the dependency. |
| `author_id` | `string` | [Author ID](#author_id) of the dependency. |
| `version` | `{major: int, minor: int}` | [Version requirement](#versioning) of the dependency. |
| `optional` | `boolean` | If `true`, SlimeCore will not require the dependency to be installed. Your datapack should function regardless of the dependency's presence if `true`. |
| `download.url` | `URL string` | [Direct download URL](#url) of any compatible version of the dependency. |
| `download.version` | `{major: int, minor: int, patch: int}` | Exact [version](#version) of the dependency that `download.url` downloads. |

### `entrypoints`

**Type:** `list<struct>`

Declares your datapack's [entrypoints](#entrypoints)--each element represents one entrypoint.

Entrypoints will always be called in the order that they are specified in this list in addition to respecting explicit `before`/`after` ordering (see below).

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

### `contract_declarations`

**Type:** `list<struct>`

Declares your datapack's [contracts](#contracts)--each element represents one contract.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `id` | `string` | The ID of the contract. *See [ID Naming](#manifest-ids).* |

### `contracts_satisfied`

**Type:** `list<struct>`

Specifies the [contracts](#contracts) that your datapack satisfies--each element represents one satisfied contract.

Each element must have the following keys:

| Key | Type | Description |
| :-- | :-- | :-- |
| `pack_ref` | `string` | The pack ID that the satisfied contract is from. |
| `id` | `string` | The ID of the satisfied contract.  |

### `display`

**Type:** `struct`

Specifies your datapack's display information and URLs.

This value is not used by SlimeCore itself but may be used externally (e.g. by frontends and such to present your datapack nicely).

| Key | Type | Description | Default Value |
| :-- | :-- | :-- | :-- |
| `name` | `string` | The display name of your datapack. | *(required)* |
| `author_name` | `string` | Your display name as a datapack author. | *(required)* |
| `summary` | `string` | 1-2 sentence-length summary/description of your datapack. Ideally, should match `pack.description` of your datapack's `pack.mcmeta` file. | *(required)* |
| `links` | `struct` | *(See below)* | *(none)* |

`name`, `author_name`, and `summary` should not contain any escape sequences such as `\n` or `\t`.

`links` is optional and can contain the following optional keys:

| Key | Type | Description | Default Value |
| :-- | :-- | :-- | :-- |
| `info` | `URL string` | website URL where users can find more information or a wiki/docs for this datapack (e.g. main GitHub repo, Modrinth page) | *(none)* |
| `releases` | `URL string` | website URL where users can find more released versions of this datapack (e.g. GitHub releases tab, Modrinth versions tab) | *(none)* |
| `author` | `URL string` | website URL that represents you as a datapack author (e.g. GitHub, Modrinth, personal site) | *(none)* |

### `url`

**Type:** `URL string`

A direct download/source URL to the exact version of this datapack as a .zip file. The downloaded .zip file should *be* the datapack (it should work correctly if put directly in a world's `datapacks/` folder; it should not have to be extracted). The name of the downloaded .zip should follow [standard datapack naming](#standard-datapack-naming).

Opening this URL in a browser should immediately download the .zip file, there should be no required user input or timers. While discouraged, this URL may contain one redirect; however redirect chains (more than one redirect) are forbidden.

*It is acknowledged that providing a valid value for `url` requires the foresight of knowing the direct download URL of your datapack before you actually release it for download. See [Publishing](./publishing.md#the-url-manifest-field).*

### `loader_version`

**Type:** `{major: int, minor: int}`

The version(s) of SlimeCore that can load this datapack, as a [version requirement](#versioning).

*You can retrieve the installed SlimeCore version in-game via `data get storage slimecore:data slimecore.version`.*

## Standard Datapack Naming

SlimeCore expects datapack names/paths to match a specific format; see [this section](../admin_guide/key_concepts.md#datapack-paths) for more information.

When releasing your datapack for download to the public, its name should match one of the fully qualified standard formats:
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

If your pack has exactly one [entrypoint](#entrypoints) that acts as a general substitute for `#minecraft:tick`, it is advised to make its ID `main`.

---

**Next:** [Beneficial Practices](./beneficial_practices.md)