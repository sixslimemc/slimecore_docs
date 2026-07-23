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

If it exists, move the contents of your datapack's `#load:pre_load` function tag to `#<pack ID>:preload_entrypoint/preload`.

Move the contents of your datapack's `#load:load` function tag to `#<pack ID>:load`.

Move any ticking/non-initialization behavior initiated by `#<pack ID>:load` to `#<pack ID>:entrypoint/main`. `#<pack ID>:load` should exclusively do initialization work and should not start any `/schedule` loops; it will be called before `#<pack ID>:entrypoint/main`.

If it exists, move the contents of your datapack's `#load:post_load` function tag to `#<pack ID>:entrypoint/post_load`

*If your datapack has dependencies and you are using Lantern load to check/manage them, it may be worth reading the [Full Guide](./full_guide.md) to effectively leverage the SlimeCore loading system.*