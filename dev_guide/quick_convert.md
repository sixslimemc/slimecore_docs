# Quick Convert

A walkthrough of quickly converting your existing datapack to a SlimeCore-loaded one. These instructions should work for *most* datapacks; however, they do not cover all nuances and they do not necessarily represent strict rules. If in doubt, reference the [Full Guide](./full_guide.md).

Make a backup of your datapack before making any changes.

## 1. Make a Pack ID

Your datapack must only have one namespace where it defines **new** resources in. The name of this namespace is referred to as your datapack's **pack ID**.

*If a resource (registry item/file) would not exist without your datapack (e.g. creating a block tag `#foo:my_blocks`), than it is new; if your datapack modifies/overwrites an existing resource (e.g. appending to block tag `#minecraft:infiniburn_overworld`), than it is not considered new, and may stay as-is.*

It is likely that your existing datapack already only has one namespace where it defines new resources in; if that is the case, you may skip to the next section.

If your existing datapack defines new resources in more than one namespace, you have the following options:
- Move all newly defined resources into a single namespace--recommended if resources are tightly coupled.
- Split your existing datapack into multiple datapacks--recommended if your namespaces function independently or have a one-way dependence flow.

*If your pack ID just so happens to be `slimecore`, you must change it to something else.*

## 2. Migrate Function Tags

> If your existing datapack uses the [Lantern Load](https://github.com/LanternMC/load) paradigm, read [this section](#migrate-from-lantern-load) instead.

Move the contents of your datapack's `#minecraft:load` function tag to `#<pack ID>:load`.

Move the contents of your datapack's `#minecraft:tick` function tag to `#<pack ID>:entrypoint/main`.

For each function specified in your pack's new `#<pack ID>:entrypoint/main`, make it schedule itself every tick (add the line `schedule function <this function> 1t`).

*Your datapack must no longer write to `#minecraft:load` or `#minecraft:tick`.*

If your datapack would start any `/schedule` loops (or any non-initailization work) within the scope of `#<pack ID>:load`, this behavior should be moved to execute in the scope of `#<pack ID>:entrypoint/main`. `#<pack ID>:load` should be used exclusively for initialization work; it will be called before `#<pack ID>/entrypoint/main`.

### Migrate From Lantern Load

> Skip this section if your existing datapack does not use the [Lantern Load](https://github.com/LanternMC/load) paradigm.

If it exists, move the contents of your datapack's `#load:pre_load` function tag to `#<pack ID>:preload_entrypoint/pre_load`.

Move the contents of your datapack's `#load:load` function tag to `#<pack ID>:load`.

Move any ticking/non-initialization behavior initiated by `#<pack ID>:load` to `#<pack ID>:entrypoint/main`. `#<pack ID>:load` should exclusively do initialization work and should not start any `/schedule` loops; it will be called before `#<pack ID>:entrypoint/main`.

If it exists, move the contents of your datapack's `#load:post_load` function tag to `#<pack ID>:entrypoint/post_load`

*If your datapack has dependencies and you are using Lantern load to check/manage them, it may be worth reading the [Full Guide](./full_guide.md) to effectively leverage the SlimeCore loading system.*

## 3. Create a Manifest

Create a file for the `#slimecore:manifest` (`<datapack>/data/slimecore/tags/function/manifest.json`) function tag in your datapack.

Create a new function and add it to `#slimecore:manifest`--this is referred to as your **manifest function**. `#slimecore:manifest` in your datapack should only contain your manifest function and should not have `replace` specified.

Copy and paste the following template into your manifest function:

```mcfunction

data modify storage slimecore:in manifest.pack.pack_id set value "PACK_ID"
data modify storage slimecore:in manifest.pack.author_id set value "AUTHOR_ID"
data modify storage slimecore:in manifest.pack.version set value {major:1, minor:0, patch:0}
data modify storage slimecore:in manifest.pack.is_library set value false

data modify storage slimecore:in manifest.pack.dependencies set value []

data modify storage slimecore:in manifest.pack.entrypoints set value []
# data modify storage slimecore:in manifest.pack.entrypoints append value {id:"main"}
# data modify storage slimecore:in manifest.pack.entrypoints append value {id:"post_load"}

data modify storage slimecore:in manifest.pack.preload_entrypoints set value []
# data modify storage slimecore:in manifest.pack.preload_entrypoints append value {id:"pre_load"}

data modify storage slimecore:in manifest.pack.abstract_declarations set value []
data modify storage slimecore:in manifest.pack.abstract_implementations set value []

data modify storage slimecore:in manifest.pack.display.name set value "DISPLAY_NAME"
data modify storage slimecore:in manifest.pack.display.summary set value "DISPLAY_SUMMARY"
data modify storage slimecore:in manifest.pack.display.author_name set value "DISPLAY_AUTHOR_NAME"

# data modify storage slimecore:in manifest.pack.display.links.author set value "AUTHOR_URL"
# data modify storage slimecore:in manifest.pack.display.links.info set value "INFO_URL"
# data modify storage slimecore:in manifest.pack.display.links.versions set value "RELEASES_URL"

data modify storage slimecore:in manifest.pack.url set value "https://example.com/TODO"

data modify storage slimecore:in manifest.loader_version set value {major:0, minor:3}

function slimecore:api/manifest
```

Change the values according to the following:

#### `pack_id`

Must exactly match your datapack's pack ID.

#### `author_id`

Choose a 3-64 string containing only lowercase letters, numbers, and `_` that uniquely represents you as a datapack author. Generally, this should closely match your name on your primary authoring platform (github, modrinth, etc.) or your in-game username.

#### `entrypoints`

Uncomment the entrypoints that your datapack has (`#<pack ID>:entrypoint/<id>` function tags).

#### `preload_entrypoints`

Uncomment the preload entrypoints that your datapack has (`#<pack ID>:preload_entrypoint/<id>` function tags). *You should only have these if you read [Migrate From Lantern Load](#migrate-from-lantern-load).*

#### `display.name`

The display name for your datapack. Can be any string.

#### `display.summary`

A 1-2 sentence summary/description of your datapack. It is standard to make it match `pack.description` from your datapack's `pack.mcmeta` file.

#### `display.author_name`

The display name for you as a datapack author. Can be any string.

#### `display.links.author`

A browser URL to your website as a datapack author (personal website, github, modrinth, etc.). May be omitted, but is recommended.

#### `display.links.info`

A browser URL to a central information page about your datapack (wiki, github repo, modrinth page, etc.). May be omitted, but is recommended.

#### `display.links.versions`

A browser URL to a page with your datapacks versions/releases (github releases, modrinth versions, etc.). May be omitted, but is recommended.

#### `url`

Can be left with placeholder value during development but should be properly set if/when released to the public. See [this section](./full_guide.md#url) for properly setting.

#### All Other Values

All other values should be fine left as-is. For more information on them, see the [manifest section](./full_guide.md#the-manifest) of the full guide.