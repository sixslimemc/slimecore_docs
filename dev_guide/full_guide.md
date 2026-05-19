# Full Dev Guide

This guide is from the perspective of creating a new datapack, but should be able to be easily adapted to converting a non-SlimeCore-loaded datapack into a SlimeCore-loaded datapack. If you are converting a datapack, it is highly advised to make a backup beforeso.

- [Setup](#setup)
- [Dependencies](#dependencies)
- [Loading](#loading)
- [Entrypoints](#entrypoints)
- [Disable and Uninstall](#disable-and-uninstall)
- [Safe Mode](#safe-mode)
- [The Manifest](#the-manifest)

## Setup

### Pack ID

Designate a namespace (`<datapack>/data/<namespace>`) as your datapack's **pack ID**. This namespace must be the only namespace that your datapack defines ***new*** files in.

All other namespaces included in your datapack are considered *secondary namespaces* (e.g. `minecraft`, `slimecore`, pack IDs of your datapack's dependencies). Your datapack must not define any new files within secondary namespaces, but may *overwrite/modify* files in them (e.g. appending to a tag).

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

## Dependencies

With SlimeCore, your datapack can declare that it requires, or optionally supports, other SlimeCore-loaded datapacks. These required/supported datapacks are **dependencies** of your datapack.

SlimeCore will ensure that all dependencies will be loaded **before** your datapack, and that all required dependencies are installed before your datapack loads at all.

Dependencies are declared in a datapack's [manifest](#the-manifest).

### Versioning

All SlimeCore-loaded datapacks have a [SemVer](https://semver.org) adhering version (`<major>.<minor>.<patch>`). When declaring a dependency, a *version requirement* (`<req_major>.<req_minor>`) must be specified with it.

An installed dependency fulfills the version requirement if all of these conditions are met:
- `<major>` == `<req_major>`
- if `<major>` == 0:
    -`<minor>` == `<req_minor>`
- if `<major>` > 0:
    - `<minor>` >= `<req_minor>`a

If an installed dependency datapack does not fulfill the version requirement, the dependency is not considered fulfilled and the dependent datapack will not load.

## Loading

With SlimeCore, the `#minecraft:load` function tag is conceptually replaced by `#<pack ID>:load`--`#<pack ID:load>` will be called upon every world reload and should be used to initialize your datapack.

Importantly, `#<pack ID>:load` should *only* do work related to initialization (declaring scoreboards, initializing data, etc.); it should not do anything else such as start `/schedule` loops or other independent work--this type of work is what entrypoints are for (see next section).

From [Dependencies](#dependencies):
> TODO: "dependencies will always be loaded before your datapack"

## Entrypoints

Entrypoints are function tags that are called on world reload **after all** datapacks are loaded. They should be used to run/start independent, non-initialization work. A datapack can define any number of entrypoints.

Entrypoints can and should be used to replace `#minecraft:tick` by one or more entrypoints that contain functions that schedule themselves (`schedule <self> 1t`). If your datapack does multiple conceptually independent "chunks" of work in its tick loop, it's good practice to split them up into their own entrypoints. This gives datapacks that may depend on yours more to work with (explained below).

A key advantage of entrypoints is that they can be explicitly ordered relative to dependencies' entrypoints. For instance, If datapack A defines entrypoint `foo`, and datapack B depends on datapack A, any of datapack B's entrypoints can be specified to explicitly run before OR after entrypoint `foo`.

Entrypoints match the function tag format `#<pack ID>/entrypoint/<entrypoint ID>`, and are declared in a datapack's [manifest](#the-manifest).

## Disable Tag

The `#<pack ID>:disable` function tag is called just before your datapack is disabled, but not uninstalled.

When this tag is called, your datapack attempt to cleanly stop operation with the consideration that it be re-enabled again in the future, ideally "continuing" where it left off.

*There is no special `#<pack ID>:enable` tag; when a datapack is re-enabled, it's `#<pack ID>:load` function is called like normal.*

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

## The Manifest

A datapack's **manifest** contains information about how it should be recognized and loaded--it is essentially the *identity* of the datapack. You define your datapack's manifest, SlimeCore does the rest.

In your datapack, you must append a single function to the function tag `#slimecore:manifest`. This function is referred to as the *manifest function* and must call the function `slimecore:api/manifest` with proper inputs (defining the manifest).

A minimal manifest function template:

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

Each component is explained in the sections below:

### `dependencies`
