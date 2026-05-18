# Full Dev Guide

This guide is from the perspective of creating a new datapack, but should be able to be easily adapted to converting a non-SlimeCore-loaded datapack into a SlimeCore-loaded datapack. If you are converting a datapack, it is highly advised to make a backup beforeso.

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

## The Manifest

A datapack's **manifest** tells SlimeCore how it should recognize and load it. You define your datapack's manifest, SlimeCore does the rest.

In your datapack, you must append a single function to the function tag `#slimecore:manifest`. This function must call the function `slimecore:api/manifest` with proper inputs (defining the manifest), and is referred to as the *manifest function*.

Here is a manifest function template:

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
