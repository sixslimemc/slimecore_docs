# Quick Convert

A walkthrough of quickly converting your existing datapack to a SlimeCore-loaded one. These instructions should work for *most* datapacks, but they do not cover all nuances and they do not necessarily represent strict rules. If in doubt, reference the [Full Guide](./full_guide.md).

Make a backup of your datapack before making any changes.

- [1. Make a Pack ID](#1-make-a-pack-id)
- [2. Migrate Function Tags](#2-migrate-function-tags)
- [3. Create a Manifest](#3-create-a-manifest)
- [4. Rename Datapack File](#4-rename-datapack-file)
- [5. Recommended Refactoring](#5-recommended-refactoring)
- [6. Implement Uninstallation](#6-implement-uninstallation)
- [7. Implement Disable](#7-implement-disable)
- [8. Implement Safe Mode](#8-implement-safe-mode)
- [9. Verify](#9-verify)
- [10. Publish](#10-publish)
- [Going Forward](#going-forward)

## 1. Make a Pack ID

Your datapack must only have one namespace where it defines **new** resources in. The name of this namespace is referred to as your datapack's **pack ID**.

*If a resource (registry file) would not exist without your datapack (e.g. block tag `#foo:my_blocks`), than it is considered new. If your datapack modifies/overwrites an existing resource (e.g. appending to block tag `#minecraft:infiniburn_overworld`), than it is **not** considered new.*

If your existing datapack defines new resources in more than one namespace, you have the following options:
- Move all newly defined resources into a single namespace--recommended if resources are tightly coupled.
- Split your existing datapack into multiple datapacks--recommended if your namespaces function independently or have a one-way dependence flow.

*If your pack ID just so happens to be `slimecore`, you must change it to something else.*

## 2. Migrate Function Tags

> If your existing datapack uses the [Lantern Load](https://GitHub.com/LanternMC/load) paradigm, read [this section](#migrate-from-lantern-load) instead.

Move the contents of your datapack's `#minecraft:load` function tag to `#<pack ID>:load`.

Move the contents of your datapack's `#minecraft:tick` function tag to `#<pack ID>:entrypoint/main`.

For each function specified in your pack's new `#<pack ID>:entrypoint/main`, make it schedule itself every tick (add the line `schedule function <this function> 1t`).

*Your datapack must no longer write to `#minecraft:load` or `#minecraft:tick`.*

If your datapack would start any `/schedule` loops (or any non-initailization work) within the scope of `#<pack ID>:load`, this behavior should be moved to execute in the scope of `#<pack ID>:entrypoint/main`. `#<pack ID>:load` should be used exclusively for initialization work; it will be called before `#<pack ID>/entrypoint/main`.

### Migrate From Lantern Load

> Skip this section if your existing datapack does not use the [Lantern Load](https://GitHub.com/LanternMC/load) paradigm.

If it exists, move the contents of your datapack's `#load:pre_load` function tag to `#<pack ID>:preload_entrypoint/pre_load`.

Move the contents of your datapack's `#load:load` function tag to `#<pack ID>:load`.

Move any ticking/non-initialization behavior initiated by `#<pack ID>:load` to `#<pack ID>:entrypoint/main`. `#<pack ID>:load` should exclusively do initialization work and should not start any `/schedule` loops; it will be called before `#<pack ID>:entrypoint/main`.

If it exists, move the contents of your datapack's `#load:post_load` function tag to `#<pack ID>:entrypoint/post_load`

*If your datapack has dependencies and you are using Lantern load to check/manage them, it may be worth reading the [Full Guide](./full_guide.md) to effectively leverage the SlimeCore loading system.*

## 3. Create a Manifest

Create a file for the `#slimecore:manifest` function tag in your datapack (`<datapack>/data/slimecore/tags/function/manifest.json`).

Create a new function and add it to your datapack's `#slimecore:manifest` tag. This function is referred to as your **manifest function**. In your datapack, `#slimecore:manifest` should only contain your manifest function and should not have `replace` specified.

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

Choose a 3-64 character-length string containing only lowercase letters, numbers, and `_` that uniquely represents you as a datapack author. Generally, this should closely match your name on your primary authoring platform (GitHub, Modrinth, etc.) or your in-game username.

#### `version`

Your datapack's version ([SemVer](https://semver.org) adhering). Can be left as-is (`1.0.0`) in most cases of converting a datapack.

#### `entrypoints`

Uncomment the entrypoints that your datapack defines (`#<pack ID>:entrypoint/<id>` function tags).

#### `preload_entrypoints`

Uncomment the preload entrypoints that your datapack defines (`#<pack ID>:preload_entrypoint/<id>` function tags). *You should only have these if you followed [Migrate From Lantern Load](#migrate-from-lantern-load).*

#### `display.name`

Choose a display name for your datapack. Can be any string.

#### `display.summary`

Choose a 1-2 sentence summary/description of your datapack. It is standard to make it match `pack.description` from your datapack's `pack.mcmeta` file.

#### `display.author_name`

Choose a display name for you as a datapack author. Can be any string.

#### `display.links.author`

A browser URL to your website as a datapack author (personal website, GitHub, Modrinth, etc.).

May be omitted, but is recommended.

#### `display.links.info`

A browser URL to a central information page about your datapack (wiki, GitHub repo, Modrinth page, etc.). 

May be omitted, but is recommended.

#### `display.links.versions`

A browser URL to a page with your datapacks versions/releases (GitHub releases, Modrinth versions, etc.). 

May be omitted, but is recommended.

#### `url`

A direct download URL to your datapack.

Generally should be left alone during development but should be [properly set](./full_guide.md#url) before releasing your datapack to the public. *Must contain a valid URL.*

#### All Other Values

All other values can be left as-is. For more information on them, see the [manifest section](./full_guide.md#the-manifest) of the full guide.

## 4. Rename Datapack File

SlimeCore requires datapack 
Rename your datapack file/folder (file/folder in `<world>/datapacks/` folder) to one of the following formats (values must match those specified in your datapack's manifest function):
- `<author ID>.<pack ID>.zip` (e.g. `bar.foo.zip`)
- `<author ID>.<pack ID>` (e.g. `bar.foo`)
- `<pack ID>` (e.g. `foo`)

These datapack names are intended for active development, however, if/when your datapack is realeased for public download, it's name when downloaded should match one of:
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>.zip` (e.g. `bar.foo.1.2.3.zip`)
- `<author ID>.<pack ID>.<major version>.<minor version>.<patch version>` (e.g. `bar.foo.1.2.3`)

Your datapack must match one of these standard name formats in order for SlimeCore to recognize it. See [this section](../admin_guide/key_concepts.md#datapack-paths) for more information.

## Checkpoint

If all of the above steps have been completed correctly, your datapack should technically load via SlimeCore and function as it did pre-conversion, however, it still has some work to be done to be considered "SlimeCore-loaded". It would be reasonable to create another backup of your datapack at this point.

*If your datapack is still in early development or you do not plan to release it to the public, it may suffice to stop here. However, even so, it may be worthwhile to continue reading to see what you may need to do in the future.*

## 5. Recommended Refactoring

It is recommended to look over [Beneficial Practices](./beneficial_practices.md) and implement what you think is reasonable for your datapack. [Namespacing](./beneficial_practices.md#namespacing) and [Public and Private Resources](./beneficial_practices.md#public-and-private-resources) are strongly advised.

## 6. Implement Uninstallation

Create the function tag `#<pack ID>:uninstall` (`<datapack>/data/<pack ID>/tags/function/uninstall.json`).

Implement it according to [this section](./full_guide.md#uninstall-tag).

## 7. Implement Disable

Create the function tag `#<pack ID>:disable` (`<datapack>/data/<pack ID>/tags/function/disable.json`).

Implement it according to [this section](./full_guide.md#disable-tag).

## 8. Implement Safe Mode

Create the function tag `#<pack ID>:safe_mode` (`<datapack>/data/<pack ID>/tags/function/safe_mode.json`).

Implement it according to [this section](./full_guide.md#safe-mode-tag).

## 9. Verify

Verify that your datapack works correctly and can properly be disabled, re-enabled, uninstalled, and fresh-installed.

## 10. Publish

If all the above steps have been completed correctly, you can now consider you datapack SlimeCore-loaded.

If you'd like to publish your datapack so others can use it, see the page on [Publishing](./publishing.md). Otherwise, you're done!

## Going Forward

If you plan on creating more SlimeCore-loaded datapacks, it advised that you read the [Full Guide](./full_guide.md) *before* starting development to fully leverage and understand SlimeCore's features.

---